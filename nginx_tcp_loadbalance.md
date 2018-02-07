##[nginx] TCP 로드밸런싱을 해보았다
###참고문헌
TCP Load Balancing with NGINX 1.9.0 and NGINX Plus R6

###개요
nginx는 아주 훌륭한 웹 서버입니다. nginx를 찬양하는 문구는 끝이 없지만 그 중 하나로 로드밸런싱(load balancing)을 몇 줄로 구현할 수 있다는 장점을 들 수 있을 것 같습니다. 그런데 nginx의 로드 밸런싱은 HTTP만 지원한다는 약점이 있었습니다. 그렇기 떄문에 TCP 기반의 서비스(mysql, redis, thrift 등)은 HAProxy를 사용하거나 Zookeeper 등을 사용하는 방법을 사용해야 했습니다. 하지만 위의 블로그에서 보듯, nginx 1.9.0부터 TCP 로드밸런싱을 지원하기 시작하였습니다. 이제 어떻게 TCP 로드밸런싱을 하는 지 위의 블로그를 나름 정리해서 쓰도록 하겠습니다.

####어떻게 TCP 로드 밸런싱 설정을 하는가?
걍 원문의 단락 제목을 제 멋대로 의역했습니다. TCP 로드밸런싱은 기존 http 섹션이 아닌 stream 섹션에서 관리됩니다. 그렇다보니 http 의 설정을 그대로 사용하려다가 호환이 안될 수 있습니다. 어떻게 되는 지 원문의 예제를 보겠습니다.

<pre><code>
stream {
    server {
        listen 3306;
        proxy_pass db;
    }

    upstream db {
        server db1:3306;
        server db2:3306;
        server db3:3306;
    }
}
</code></pre>

원문에서는 MySQL을 로드밸런싱 하는 것을 예제로 들었네요. 자세한 설정은 다음 stream 모듈 문서를 참고하라고 써있습니다.

<pre><code>
Stream core module
Stream proxy module
Stream SSL module
Stream upstream module
</code></pre>

그리고 원문에는 nginx 1.9.0에서는 복잡한 수준의 로드밸런싱은 서드파티 모듈을 활용하라고 합니다. 여기까지 기초였고 이제 제가 수박 겉핥기 식으로 조사한 내용들을 더 적어보고자 합니다.

####더 확인해본 것들
#####0. 읽기 전 주의 사항
제가 테스트하는 버전은 1.10.0 이며 OS는 우분투 16.04 64비트 입니다. 이 점을 참고해주시기 바랍니다.
mainline 버전 최신은 1.9.15 입니다.
stable 버전 최신은 1.10.0 입니다.
전 분명히 메인라인 패키지를 apt에 기록했는데 Stable 버전이 설치되었습니다. 제가 작성한 apt 파일입니다. 

/etc/apt/source.list.d/nginx.list
deb http://nginx.org/packages/mainline/ubuntu/ xenial nginx
deb-src http://nginx.org/packages/mainline/ubuntu/ xenial nginx


#####1. Consistent Hash를 이용한 sticky session
Sticky한 TCP 연결을 보장해주는 방법 중 하나로 Consistent Hash 기법이 있습니다. 이 알고리즘에 대한 설명은 여기서 하지 않겠습니다(저도 잘 모릅니다) 이 방법을 지원하는 지 확인해보도록 하겠습니다.

######nginx 셋팅
먼저 nginx 설정은 다음과 같습니다.

<pre><code>
stream {
        server {
                listen 9049;
                proxy_pass tester;
        }

        upstream tester {
                hash $remote_addr consistent;
                server localhost:9050;
                server localhost:9051;
        }
}
</code></pre>

######테스트 코드(thrift)
전 우연히 제 컴터 구석에 thrift 연습하던 코드가 있어서 이 것을 사용하였습니다. 혹시 모르니 공유하겠습니다.

<pre><code>
some.thrift

service Hello {
    void ping(),
    string getHello(1:string str)
}
</code></pre>

그냥 마구잡이로 만든 거라 너무 대충이다 싶은 점은 잊어주시기 바랍니다. 위 코드를 node.js 코드로 바꿉니다.

<pre><code>
$ thrift -r --gen js:node some.thrift
</code></pre>

######테스트 코드(node.js)
<pre><code>
//client.js
var thrift = require('thrift');
var Hello = require('../gen-nodejs/Hello');
var someTypes = require('../gen-nodejs/some_types');

var connection = thrift.createConnection('localhost', 9049);

connection.on('error', function(err) {
    console.log(err);
    process.exit(1);
});

var client = thrift.createClient(Hello, connection);
var c = 0;

client.ping(function(err, resp) {
    console.log('ping()');
    ++c;
    if(c >= 2) { connection.end(); }
});

client.getHello('hello', function(err, resp) {
    console.log('hello? ' + resp);
    ++c;
    if(c >= 2) { connection.end(); }
});
//nodeServer.js
var thrift = require('thrift');
var Hello = require('../gen-nodejs/Hello');
var someTypes = require('../gen-nodejs/some_types');

var server = thrift.createServer(Hello, {
    ping : function(result) {
        console.log('ping()');
        result(null);
    },
    getHello : function(str, result) {
        console.log('hello');
        result(null, 'world!');
    }
});

server.listen(9050);
//nodeServer2.js
var thrift = require('thrift');
var Hello = require('../gen-nodejs/Hello');
var someTypes = require('../gen-nodejs/some_types');

var server = thrift.createServer(Hello, {
    ping : function(result) {
        console.log('ping()');
        result(null);
    },
    getHello : function(str, result) {
        console.log('hello');
        result(null, 'world!');
    }
});

server.listen(9050);
</code></pre>

부득이하게 Ctrl+CV를 하였습니다.

######실행
이제 실제로 실행을 해봤습니다. hash $remote_addr consistent; 이 코드가 잘 되나 보겠습니다. 잘 된다면 로컬호스트에서 실행 시 한 곳에서만 로그가 발생할 것입니다.

실제로 한 곳에서만 응답을 받는 것을 확인할 수 있었습니다. 위의 블로그를 보고 안될 줄 알았는데 된다는 것에 살짝 안심하엿습니다. 이제 응답 받던 곳을 죽이면 다른 곳에서 받게 됩니다. 아, 물론 너무 빨리 죽이고 살리고 하면 nginx 쪽에서 처리하는 시간이 있어서 그 잠깐의 시간동안은 Fail이 발생하는 것은 감안하셔야 합니다.
그렇다면 위의 코드에 주석을 치면 어떻게 될까요? 보시다시피 양 쪽에서 다 받게 됩니다. 전통적인 방법인 라운드 로빈 방식을 의미하는 것입니다. 이렇게 기존에 쓰던 upstream에서 사용되는 문법을 공유하는 것을 확인할 수 있었습니다. 안되는 부분이 있는 지 없는 지는 확인되는 대로 추가하도록 하겠습니다.

#####2. 로그 찍기

갑자기 궁금해져서 access_log와 error_log 가 찍히는 지도 확인해보기로 하였습니다. 아쉽게도 스트림 블록 안에는 access_log를 기록할 수는 없었습니다. 액세스 로그를 찍는 방법은 다른 수단을 써야 할 것 같네요. 그래도 에러 로그는 찍히니 모니터링 하실 때 유용하게 사용하실 수 있을 것 같습니다.

####마치며
이제 nginx는 완전체에 다다른 것 같습니다. TCP까지 로드밸런싱이 되니 정말 좋네요. 2015년 4월에 나온 글이라 다른 분들께서는 벌써 잘 쓰고 계실지도 모르겠습니다만 저는 이제야 글을 봤고 옮겨서 적어봅니다. 이런 뒷북성 글을 읽어주셔서 정말 감사합니다.

####기타
다음 기회에 UDP 로드밸런싱도 한 번 적어보겠습니다.


출처: http://blog.hellworld.me/11?category=653275