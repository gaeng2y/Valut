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

이는 왼쪽에서 오른쪽으로 잘 읽히는 반면 자유 함수는 안쪽에서 바깥쪽으로 읽혀지며, square를 호출하기 전에 incr을 호출하는지 확인하려면 더 많은 정신 작업이 필요합니다.

