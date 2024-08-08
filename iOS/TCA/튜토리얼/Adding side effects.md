Reduce 클로저에서 concurrency를 지원해주지 않는다.

```swift
// 🛑 'async' call in a function that does not support concurrency
// 🛑 Errors thrown from here are not handled
```

그래서 우리는 environment로 effect를 만들어야하죠

```swift
  /// Initializes a reducer with a `reduce` function.
  ///
  /// - Parameter reduce: A function that is called when ``reduce(into:action:)`` is invoked.
  @inlinable
  public init(_ reduce: @escaping (_ state: inout State, _ action: Action) -> Effect<Action>) {
    self.init(internal: reduce)
  }
```

실제로 reducer에 `Effect<Action>`을 방출한다. 기존에 살펴볼 때 Environment에서 Effect를 만들어 Action을 던져준다고 했는데 이러한 형태로 진행되는 것이다.

`Effect`를 살펴보면 Concurrency를 지원해주는 `run`이라는 메소드가 존재한다. 그래서 아래와 같이 `Effect`를 반환하면 된다.

```swift
return .run { [count = state.count] send in
	// ✅ Do async work in here, and send actions back into the system.
	let (data, _) = try await URLSession.shared
		.data(from: URL(string: "http://numbersapi.com/\(count)")!)
	let fact = String(decoding: data, as: UTF8.self)
}
```

그리고 effect에서 곧바로 state를 바꿀 수 없기에 action을 또 전달하게 하여 effect로 만들어지는 action에 대한 동작을 reduce에 정의해주면 된다.