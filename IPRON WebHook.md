# IPRON WebHook
  - Event List
    + Incomming
    + Ringing
    + Connected
    + Forwarded
    + Disconnected
  - Response State
    + UnRegi
    + Busy/NoAnswer

## 메시지 정의
### 공통정보
- Token : UCID+HOP
- CallId : call-id
- ANI : 원발신번호
- DNIS : 원착신번호(최초수신)
- DN : 이벤트 발생 내선번호
- scope : "Incomming"


- Method : [POST|PUT|GET]
- URL : http://...

#### Incomming
~~~json
Content-type: application/json
{
	"event": {
		"scope": "Incomming",
		"call_id": 1001,
		"ANI": "01012345678",
		"DNIS": "15881000",
		"DN": "4138",
	}
}
~~~
#### Ringing
~~~json
x-hub-signature
X-IPRON-Info: 72d3162e-cc78-11e3-81ab-4c9367dc0958
X-IPRON-Secret: 
Content-type: application/json
{
	"event": {
		"scope": "Ringing",
		"call_id": 1001,
		"timestamp":1355517523.000005,
		"token": "1234",
		"ANI": "01012345678",
		"DNIS": "15881000",
		"DN": "4138",
		"answerUrl": "http://ipron_url/ipron/ie/call/[call_id]/answer",
		"rejectUrl": "http://ipron_url/ipron/ie/call/[call_id]/reject",
		"transferUrl": "http://ipron_url/ipron/ie/call/[call_id]/transfer?to=dest"
	}
}
~~~
