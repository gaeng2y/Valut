우리는 또한 올바르게 사용하기 어려운 낮은 수준의 대안부터 사용하기 쉽지만 유연성이 적은 더 높은 수준의 대안에 이르기까지 애플이 이 일을 위해 개발자를 위해 가지고 있는 몇 가지 옵션을 보았다.

```swift
class UserAPI {
    func fetchUserInfo(
		completionHandler: @escaping (_ userInfo: UserInfo?, _ error: Error?) -> Void ){
        let url = URL(string: "https://www.andyibanez.com/
        fairesepages.github.io/books/async-await/user_
        profile.json")!
        let session = URLSession.shared
        let dataTask = session.dataTask(with: url) { data, response, error in
            if let error = error {
                completionHandler(nil, error)
            } else if let data = data {
                do {
                    let userInfo = try JSONDecoder().
                    decode(UserInfo.self, from: data)
                    completionHandler(userInfo, nil)
                } catch {
                    completionHandler(nil, error)
				} 
			}
		}
        dataTask.resume()
    }
	//... 
}
```

이 코드는 신규 이민자들에게 위협적으로 보일 수 있지만, 노련한 개발자들은 무슨 일이 일어나고 있는지 알 수 있다. 
위의 코드는 HTTP를 통해 네트워크에서 데이터를 다운로드하기 위한 Foundation 객체인 URLSessionDataTask를 생성합니다.

## async keyword
시스템의 핵심은 비동기 및 대기 키워드이다. 시스템의 나머지 부분을 사용하기 위해 그것들을 사용하는 방법을 이해하는 것이 중요하다. 비동기적으로 실행할 수 있거나 다른 함수와 동시에 실행할 수 있는 함수는 함수의 반환 유형 전에 비동기 키워드를 가지고 있다. 이것은 컴파일러와 다른 프로그래머들에게 당신의 기능이 비동기라는 것을 알려줄 것이다.

## await keyword
비동기 함수에 대한 모든 호출은 어느 시점에서 await 키워드와 페어링되어야 합니다. await 자체는 특별한 의미를 가지고 있으며, 비동기 함수를 호출하는 구문만이 아닙니다.

코드 실행 중에 await 키워드가 발생하면, 프로그램은 현재 실행 중인 기능을 일시 중지하도록 선택할 수 있습니다. 이러한 이유로, await 키워드는 일반적으로 정지 지점으로도 알려져 있다.
이것은 시스템이 내릴 결정이며, 당신은 그것을 통제할 수 없습니다. 우리가 그것이 일시 중지될 수 있다고 말할 때, 우리는 현재 실행이 일시 중지되고 비동기 함수가 다른 곳에서 계속 실행될 것이라는 것을 의미합니다. 일시 중지되면 현재 함수에서 코드가 실행되지 않습니다.

## async get properties

Async/await는 함수 호출에 국한되지 않습니다. 읽기 전용 계산된 속성에서도 사용할 수 있습니다.