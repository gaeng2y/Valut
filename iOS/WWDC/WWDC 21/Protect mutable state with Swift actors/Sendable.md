```swift
actor Library {
    let idNumber: Int
    var booksOnLoan: [Book] = []
    func selectRandomBook() -> Bool? { ... }
}

class Book {
    var title: String
    var authros: [Author]
}

func visit(_ account: LibraryAccount) async {
    guard var book = await account.selectRandomBook() else {
        return
    }
    book.title = "\(book.title)!!!"
}
```

위의 상황에서 Book이 구조체였다면 크게 문제가 없을 것이다. 왜냐면 Book의 복사본을 변경해도 액터에 영향을 미치지 않으며, 그 반대도 마찬가지다.

그러나 클래스라고 하면 LibararyACcount 액터는 Book 클래스의 인스턴스를 참조하게 된다. 그 자체로는 문제가 되지 않는다.

하지만 `selectRandomBook()`을 호출하면 액터 외부에 공유된 액터의 가변 상태에 대한 참조를 할 수 있게 된다. 즉, 원래 액터 내부에 캡슐화되어야 할 변경 가능한 상태에 접근할 수 있는 참조가 외부로 노출되었음을 경고하는 내용이다. 

그렇게 되면 이제 데이터 경쟁의 가능성이 만들어졌다. visit 메소드 내에서 book의 제목을 업데이트하면, 수정은 액터 내에서 접근 가능한 상태에서 발생한다. 이는 결국 데이터 경쟁이 될 수 있다.

이를 안전하게 사용할 수 있는 타입을 위한 프로토콜이 있다.

바로 **Sendable**

Sendable 타입은 동시에 안전하게 공유할 수 있다. 한 곳에서 다른 곳으로 값을 복사할 때, 두 곳 모두 서로 간섭하지 않고 자신의 값 복사본을 안전하게 수정할 수 있다면, 그 타입은 Sendable이 될 수 있다.

- 값 타입은 각 복사본이 독립적이기에 Sendable
- 액터 타입은 가변 상태에 대한 접근을 동기화하기 때문에 Sednable
- 클래스도 Sendable이 될 수 있지만, 신중하게 구현된 경우에만 가능하다. 예를 들어 모든 하위 클래스가 불변 데이터만 보유한다면 Sendable이라고 할 수 있다. 또한 클래스가 내부적으로 안전한 동시 접근을 보장하기 위해 잠금과 같은 동기화를 수행한다면, Sendable이 될 수 있다.

함수는 반드시 Sendable이 아니므로, 액터 간에 안전하게 전달할 수 있는 함수를 위한 새로운 종류의 함수 타입이 있다.

여러분의 액터 - 사실, 모든 동시성 코드 - 는 주로 Sendable 타입으로 통신해야 한다.

Sendable 타입은 코드를 데이터 경쟁으로부터 보호한다.

#### @Sendable 함수
@Sendable 함수 타입은 Sendable 프로토콜을 따른다.
- 가변 지역 변수를 캡처할 수 없다.
- 캡처하는 모든 것은 Sendable 타입이어야 한다.
- 동기식 클로저는 액터에 격리될 수 없다.