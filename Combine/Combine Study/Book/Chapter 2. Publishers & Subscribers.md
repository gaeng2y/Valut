## Hello Publisher

- Combine의 핵심 `Publisher` 프로토콜
- 이 프로토콜은 하나 또는 이상의 Subscriber에게 시간에 따른 일련의 값을 전달 할 수 있는 타입에 대한 요구사항을 정의 한다. ⇒ Publisher는 특정 값을 포함 할 수 있는 이벤트를 publish 하거나 emit 한다
- 기존 Apple platform에서 `NotificationCenter` 같은 종류이다.
- 실제로 `NotificationCenter` 에 broadcast notification을 publish 할 수 있는 `Publisher` 타입을 제공하는 `publisher(for:object:)` 라는 메소드가 있다.

```swift
example(of: "Publisher") {
  let myNotification = Notification.Name("MyNotification")

  let publisher = NotificationCenter.default
    .publisher(for: myNotification, object: nil)
}
```

Publisher는 두가지 이벤트를 emit 한다

1. Value (element라고도 불리는)
2. A completion event(error, completion)

Publisher는 0개 이상의 값을 emit 할 수 있지만, completion event 는 하나만 emit 할 수 있다.
Completion event는 normal comletion 이벤트 이거나 error이다.\
한번 publisher가 completion event를 emit 하면, 더이상 이벤트를 emit 하지 않고 끝난 것이다.
## Hello Subscriber

- `Subscriber` 는 Publisher 로 부터 받을 수 있는 인풋의 타입에 대한 요구사항을 정의한 프로토콜이다.
- **Publisher는 적어도 하나의 Subscriber가 존재할 때만 이벤트를 emit한다.**
### Subscribing with `sink(_: _:)`

- `sink` 는 Subscriber에게 closure를 연결하여 Publisher로부터 받은 output을 처리하는 쉬운 방법을 제공한다.
- `sink` operator는 publisher가 emit하는 가능한 많은 값을 받을 것이다. → `unlimited demand`
- `sink` operator는 실제로 두가지 클로저를 제공한다

1. 전달 된 completion event를 처리
2. 전달 받은 값을 처리

- `Just`는 각각의 Subscriber에서 output을 한번 emit하고 종료되는 Publisher이다.
- `Just`는 각각 새로운 Subscriber에게 output을 정확히 한번 보낸 뒤 종료 한다.
### Subscribing with `assign(to:on:)`

- `sink`와 내장 된 `assign(to:on:)` 은 전달 받은 값을 객체의 KVO-compliant property 를 할당 할 수 있는 Operator이다.
### Republishing with `assign(to:)`

- `@Published` property wrapper 로 표시된 또 다른 property로 부터 Publisher에서 emit 된 값을 `republish` 하는데 확용 할 수 있는 `assign` Opreator 에 ~~변동이 있습니다~~
- `assign(to:)` Operator 는 `AnyCancellable` 토큰을 리턴하지 않는다, 왜냐하면 그것은 내부적으로 lifecycle을 관리 하고 있고 `@Published` property가 deinitialize 될 때 Subscription을 취소 한다.

`assign(to:on:)` 과 비교하면 `assign(to: \.word, on:self)` 는 `AnyCancellabel`의 결과를 strong reference cycle로 저장하고 있다. `assign(to: &$word)` 는 이 문제를 방지한다.
## Hello Cancellable

- Subscriber가 완료 되고, Publisher로 부터 더이상 값을 받기를 원치 않는 경우, Subscription을 취소해 리소스를 해제하고 네트워크 콜 같은 액티비티에 응답하는 것이 좋다.
- Subscription은 `AnyCancellable` 인스턴스인 "cancellation token"을 리턴하므로, 종료 되었을 때 Subscription을 취소할 수 있다.
- `AnyCancellable` 는 그 목적을 위한 `cancle()` 메소드를 요구하는 `Cancellable` 프로토콜을 따른다.
- Subscription에 명시적으로 `cancel()` 을 호출 하지 않으면, Publisher가 완료 될 때까지 지속 되거나, 일반적인 메모리 관리로 저장된 Subscription이 deinitialize 될 때까지 지속 된다.
## Understanding what's going on

[![/Combine/Section1_Introducing_to_Combine/images/Untitled.png](https://github.com/HARlBO/Combine/raw/main/Combine/Section1_Introducing_to_Combine/images/Untitled.png)](https://github.com/HARlBO/Combine/blob/main/Combine/Section1_Introducing_to_Combine/images/Untitled.png)

`Publihser` 프로토콜의 가장 중요한 extensions:

```swift
public protocol Publisher {

  associatedtype Output
  associatedtype Failure : Error

  func receive<S>(subscriber: S)
    where S: Subscriber,
    Self.Failure == S.Failure,
    Self.Output == S.Input
}

extension Publisher {
  public func subscribe<S>(_ subscriber: S)
    where S : Subscriber,
    Self.Failure == S.Failure,
    Self.Output == S.Input
}
```

- `subscribe(_:)` 의 구현은 Publisher에 Subscriber를 연결하기 위해 `receive(subscriber:)` 를 호출 할 것이다. 즉 Subscription을 생성한다.
- `associatedtype` 은 Subscriber가 Subscription을 생성하기 위해 반드시 맞춰야 하는 Publisher의 인터페이스이다.

`Subscriber` 프로토콜:  

```swift
public protocol Subscriber: CustomCombineIdentifierConvertible {
  
  associatedtype Input

  associatedtype Failure: Error
  
  func receive(subscription: Subscription)

  func receive(_ input: Self.Input) -> Subscribers.Demand

  func receive(completion: Subscribers.Completion<Self.Failure>)
}
```

`Publisher` 와 `Subscriber` 는 `Subscription` 프로토콜로 연결된다.

```swift
public protocol Subscription: Cancellable, CustomCombineIdentifierConvertible {
    func request(_ demand: Subscribers.Demand)
}
```

Subscriber 는 `request(_:)` 를 호출하여 더 많은 값, 무제한의 값을 받을 의향이 있음을 나타낸다.

> Note: Subscriber 가 값을 얼마나 받고 싶은지를 나타내는 것이 **backpressure meanagement** 이다. 이것 없이난, Subscriber 는 처리 할 수 있는 것보다 더 많은 값을 Publisher로부터 받아서 넘칠 수 있고, 이것은 문제가 될 수 있다.

`Subscriber` 에서, `receive(*:)` 가 `Demand`를 리턴 한다. Subscirber 가 받고자 하는 값의 최대 개수를 `subscription.requset(*:)`의 `receive(_:)`에서 처음 호출 할 때 정의하더라도, 새로운 값을 받을 때마다 최대 수를 조절 할 수 있다.

max는 양수여야 하고, 음수를 전달 할 경우 `fatalError` 가 발생한다, max 값은 늘릴 수 있지만, 줄일 수는 없다.
## Hello Future

Subscriber에게 단일 값을 전달하고 완료되는 Publisher 를 생성하기 위해 `Just` 를 사용할 수 있는 것 처럼, `Future` 도 단일 결과값과 완료를 비동기적으로 생성한다.

- `Future` 은 단일 값을 생성하고 완료 또는 실패 처리를 하는 Publisher 이다.
- 값이나 에러를 사용 가능 해졌을 때 클로저를 호출한다, 그 클로저가 Promise 이다.
- `Promise` 는 `Result` 에 `Future`에 의해 발행 된 단일 값 또는 에러를 포함 하고 있는 클로저의 type alias 이다.

몇몇 예제들로, Publisher 가 순차적이고 동기적으로 Publish 된 유한한 숫자의 값을 발행하는 Publisher들을 다루었다.

NotificationCenter 예제에서 Publisher 는 값을 무기한으로 비동기적으로 유지 할 수 있었다

1. 내제 된 notification sender 가 notification 을 emit 한다
2. 특정 notification 에 Subscriber 가 있다
## Hello Subject

- Passthrough subjects 는 요구에 따라 새로운 값을 Publish 할 수 있게 해준다.
- Publisher 처럼, emit 할 수 있는 값과 에러의 타입을 미리 선언해야 한다
- Subscriber 는 Passthrough subject 에 subscribe 하기 위해서는 인풋과 실패 타입을 따라야 한다.
- Publisher가 단일 completion 이벤트를 보개면, normal completion 또는 error 인지 상관없이, 그것은 종료가 된다.
- `PassthroughSubject` 를 통해 값을 전달 하는 것은 명령적 코드를 Combine 의 선언적 세계와 연결 해 준다.
- 하지만 가끔, 명령적 코드의 Publisher의 현재 값을 보고 싶을때, `CurrentValueSubject` 를 적적히 사용 할 수 있다.
- 각각의 Subscription 을 값으로 저장 하는 대신에, `AnyCancellable` collection 여러개의 Subscription 을 저장할 수 있다. collection 은 자동으로 deinitialed 될 때 collection 에 추가된 각각의 Subscription을 취소 할 것이다.

`CurrentValueSubject` 는 반드시 현재 값을 초기화 해야해야, 새로운 Subscriber가 즉시 값 또는 해당 Subject에 의 해 publish 된 값을 받는다.

- `PassthroughSubject` 와 다르게, current value subject는 그것의 값을 요청 할 수 있다.
- `send(_:)` 를 호출 하는 것은 새로운 값을 보내는 방법 중 하나이다.
- 새로운 값을 할당 받는 또 다른 방법인 `value` 프로퍼티가 있다.

## Type erasure

- Publihser에 대한 추가적인 세부정보에 접근하지 않고, Subscriber 가 Publisher 로 부터 받는 이벤트를 subscribe 하고 싶을 경우가 있다.
- `AnyPublisher` 은 type-erased struct 로 `Publisher` 프로토콜을 따른다.
- Type erasure 는 publihser 에 대한 세부사항을 가릴 수 있게 해준다.
- `AnyCancellable` 은 호출한 곳에서 내제된 Subscription에 접근하지 않고 subscription을 취소 할 수 있는, `Cancellable`을 채택한 type-eraed 클래스 이다.
- `eraseToAnyPublisher()` operator는 `AnyPublisher`의 인스턴스를 통해 제공된 Publisher를 감싸서, Publisher가 실제로 `PassthtoughSubject` 인것을 숨긴다.