## URLProtocol이란?

![image](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FctHTxo%2FbtrB9tdUGhu%2FoqRTMSKsZwPwh5yOOjjdnK%2Fimg.png)

보통 우리가 URLSession을 사용해서 networkRequest를 처리하는 DataTask Flow는 이렇게 된다.

1. URLSessionConfiguration -> URLSession -> URLSessionDataTask 순으로 만든다.
2. Request가 들어온다.
3. Request에 맞는 response를 받는다.

[WWDC](https://developer.apple.com/videos/play/wwdc2018/417/)에 따르면, 사실, 이 과정 뒤에서 좀더 Low-level한 API가 하나 더 있다고 한다.
그게 바로 [**URLProtocol**](https://developer.apple.com/documentation/foundation/urlprotocol).
Openning Network Connection, Writing the request, Reading Back Response 등의 역할을 한다고 한다.
그리고 우리가 만든 URLProtocol의 subclass를 URLSessionConfiguration에 넣을 수 있다.
