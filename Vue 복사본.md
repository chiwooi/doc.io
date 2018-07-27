[API디자인]
- 페이징
  Facebook API 스타일 : /record?offset=100&limit=25
- Partial Response 처리
  Facebook : /terry/friends?fields=id,name
  Google : ?fields=title,media:group(media:thumnail)
     => group(media:thumnail) 와 같이 JSON의 Sub-Object 개념을 지원
- 검색 (전역검색과 지역검색)
  /users?name=cho&region=seoul&offset=20&limit=10
  => 검색키워드와 페이징 정보가 모호하므로 아래와 같이 처리
  /user?q=name%3Dcho,region%3Dseoul&offset=20&limit=10
  + 전역검색 : /search?q=id%3Dterry
  + 특정리소스내검색 : /users?q=id%3Dterry
- HATEOS(Hypermedia as the engine of application state)를 이용한 링크 처리
{  

   "id":"terry",

   "links":[  

      {  

         "rel":"friends",

         "href":"http://xxx/users/terry/friends"

      }

   ]

}

