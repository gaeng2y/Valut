값 유형 및 행위자 외부의 Swift에서 전송 가능한 유형은 Sendable 프로토콜을 따릅니다.

단순히 이 프로토콜을 준수함으로써 컴파일러는 전송 가능한 유형의 사용을 검사하여 데이터 경합을 초래할 수 있는 방식으로 사용하려고 하지 않는지 확인합니다.

### Sendable 타입 분석하기
Sendable 타입이 어떻게 행동하는지 더 잘 이해하기 위해, 우리는 그들의 단점과 특별한 고려 사항에 대해 배우기 위해 그들을 분석할 것입니다.
#### 구조체
구조체는 즉시 보낼 수 있을 것이다. 그러나 보낼 수 없는 상황도 있다.
구조체가 클래스 타입의 프로퍼티를 가지고 있는 경우를 상상해보자.

```swift
class TVShow {
    let title: String
    var rating: Int
    init(title: String, rating: Int) {
        self.title = title
        self.rating = rating
    }
}
struct TVShowLibrary {
    var shows: [TVShow] = []
}
```

구조체의 shows는 mutable 클래스이다. 우리가 그것을 본 후 몇 달 후에 TV 쇼의 등급을 바꾸고 싶다는 것은 말이 된다. 
이 구조체로 격리 경계를 넘으려고 하면(예를 들어, 격리되지 않은 컨텍스트에서 TVShowLibrary를 선언하고 Task.detached {} 내에서 캡처하는 경우), 컴파일러는 구조체임에도 불구하고 당신을 멈출 것입니다.
일반 Task {} 캡처는 영향을 받지 않습니다. 왜냐하면 이 경우 @MainActor인 상위 컨텍스트에서 actor를 상속하기 때문입니다.

그리고 출력은 항상 당신이 기대하는 대로 될 것입니다.
