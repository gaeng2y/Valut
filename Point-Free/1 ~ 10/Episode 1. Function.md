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

이는 왼쪽에서 오른쪽으로 잘 읽히는 반면 자유 함수는 안쪽에서 바깥쪽으로 읽혀지며, `square`를 호출하기 전에 `incr`을 호출하는지 확인하려면 더 많은 정신 작업이 필요합니다. 이것이 아마도 Swift에서 free function이 덜 일반적인 이유일 것입니다.
## Introducing |>
free function을 가지고 있지만 함수 적용을 위해 중위 연산자를 사용하여 이러한 종류의 가독성을 유지하는 여러 언어가 있다.
여기서는 "파이프 전달" 연산자를 정의합니다. 이는 이전 기술을 기반으로 합니다. F#, Elixir 및 Elm은 모두 함수 적용을 위해 이 연산자를 사용합니다.

```swift
infix operator |>

func |> <A, B>(a: A, f: (A) -> B) -> B {
  return f(a)
}

2 |> incr // 3
2 |> incr |> square // Compile Error: Adjacent operators are in non-associative precedence group 'DefaultPrecedence'
```

연산자가 한 줄에 여러번 쓰이면, 스위프트는 어떤 연산자를 먼저 평가해야 될 지 알 수 없다. 그래서 이를 알려줘야 한다.
- 방법 1. 괄호를 친다.
```swift
(2 |> incr) |> square // 9
```
- 방법2. precedencegroup을 지정한다.