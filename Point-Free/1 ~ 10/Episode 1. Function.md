```swift
func incr(_ x: Int) -> Int {
  return x + 1
}

incr(2) // 3

func square(_ x: Int) -> Int {
  return x * x
}

square(2) // 4

square(incr(2)) // 9
```

이것은 매우 간단하지만 Swift에서는 흔하지 않습니다. 최상위의 함수(free function)은 일반적으로 기피하고 메서드를 선호한다.

그렇다면 `Int`를 확장하여 `incr`과 `square` 을 메소드로 정의할 수 있다.

```swift
extension Int {
  func incr() -> Int {
    return self + 1
  }

  func square() -> Int {
    return self * self
  }
}

2.incr() // 3
2.incr().square() // 9
```

이는 왼쪽에서 오른쪽으로 자연스럽게 읽히기 떄문이다.
### Introducing |>
다른 언어에서도 free function임에도 메소드와 같이 자연스럽게 적용할 수 언어들이 있다.
이를 Swift에서도 만들어보자.

```swift
infix operator |>

func |> <A, B>(a: A, f: (A) -> B) -> B {
  return f(a)
}

2 |> incr // 3
2 |> incr |> square // Compile Error: Adjacent operators are in non-associative precedence group 'DefaultPrecedence'
```

연산자가 한 줄에 여러번 쓰이면, Swift는 어떤 연산자를 먼저 평가해야 될 지 알 수 없다. 그래서 이를 알려줘야 한다.
- 방법 1. 괄호를 친다.
```swift
(2 |> incr) |> square // 9
```
- 방법2. precedencegroup을 지정한다.
```swift
precedencegroup ForwardApplication {
  associativity: left
}

infix operator |>: ForwardApplication

2 |> incr |> square // 9
```
### Operator interlude
중첩된 함수의 가독성 문제를 해결했지만 새로운 문제가 생겼습니다. 바로 커스텀 연산자입니다. 커스텀 연산자는 그다지 일반적이지 않고 평가가 나쁘다. 

예를 들어, C++에서는 새로운 연산자를 선언할 수 없고, 선언된 연산자만 오버로딩이 가능하다.  
|> 연산자가 Swift 개발자는 모르는 연산자라 주장할 수 있지만 다른 언어에서는 친숙한 연산자이다.
### What about autocompletion?
연산자는 가독성을 제공하지만 메소드에 있는 자동 완성 기능을 지원하지 않는다. 안되는 건 아니지만 타입에 의한 범위 제한이 되지 않기 때문이다.

하지만 free function은 메소드로 불가능한 함수 합성이 가능하다. 타입으로 가져올 수도 있지만, 이건 사실상 free function이다.
### Introduce >>>
**함수 합성**은 한 함수의 출력이 다른 함수의 입력과 일치하는 두 가지 함수를 취하여 서로 연결하여 완전히 새로운 함수를 얻는 기능이다. 이를 만들어 보기 위해 >>> 연산자를 새로 정의해보자.

```swift
infix operator >>>

func >>> <A, B, C>(f: @escaping (A) -> B, g: @escaping (B) -> C) -> ((A) -> C) {
  return { a in
    g(f(a))
  }
}
```

A, B 그리고 C를 제네릭 파라미터로 받는다. 

합성을 통해서 작은 파츠들을 쉽게 조합하여 재사용성을 증대시킬 수 있다.

```swift
incr >>> square >>> String.init // (Int) -> String

String(2.incr().square())
```

스위프트에는 네임스페이스가 없기 때문에, 최상단에 함수를 정의하는 게 부담될 수 있다.
- file에 private하게 정의하거나
- struct나 enum의 static 함수로 정의하거나
- 모듈 내에서만 쓸 수 있도록 하거나
