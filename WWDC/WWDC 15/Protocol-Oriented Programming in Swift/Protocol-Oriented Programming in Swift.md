## Classes are awesome
* 캡슐화
* 접근 제어
* 추상화
* 네임스페이스
* 표현 구문
* 확장성
## 클래스의 세가지 불평
#### 1. Implicit sharing
일반적인 시나리오를 살펴보겠습니다. A와 B는 일부 데이터를 공유하지만 A는 B에 영향을 주지 않고 이 데이터를 수정할 수 없습니다.
![image1](https://www.wwdcnotes.com/images/notes/wwdc15/408/problem_implicit_sharing.png)

프로그래머는 방어적으로 데이터 복사본을 만들 수 있지만 이는 비효율성을 초래합니다.

또한, 다른 스레드에서 데이터를 수정하면 경쟁 조건이 발생할 수 있으며, 방어적으로 잠금을 추가하면 비효율성과 교착 상태가 더욱 발생하게 됩니다.

이 모든 것이 더욱 복잡해집니다. 한마디로 **버그입니다**.

Cocoa 플랫폼에서 암시적 공유가 미치는 영향 중 하나:

> 변경 가능한 컬렉션을 열거하는 동안 이를 수정하는 것은 안전하지 않습니다. 일부 열거자는 현재 수정된 컬렉션의 열거를 허용할 수 있지만 이 동작은 향후 지원이 보장되지 않습니다.

**모든 Swift 컬렉션은 값 유형이므로 이는 Swift에는 적용되지 않습니다** . 우리가 반복하는 컬렉션과 수정하는 컬렉션은 서로 다릅니다.
#### 2. Inheritance all up in your business
클래스 상속은 단일체이다. 즉, 슈퍼클래스는 하나만 얻습니다.

 여러 추상화를 모델링하고 싶다면 어떻게 해야 할까요? 예를 들어, 우리 클래스가 컬렉션이 되고 직렬화되기를 원한다면 어떻게 될까요? `Collection`그리고 둘 다 `Serialized`클래스 라면 할 수 없습니다 .

또한 관련될 수 있는 모든 것이 하나로 합쳐지면서 클래스가 비대해진다.

또한 나중에 일부 확장이 아닌 클래스를 정의하는 순간 슈퍼클래스를 선택해야 합니다.

슈퍼클래스에 저장된 속성이 있다면 어떨까요? 우리는 그것들을 받아들여야 하고, 필요하지 않더라도 초기화해야 합니다. 또한 슈퍼클래스의 불변성을 깨지 않고 이 작업을 수행해야 합니다.

마지막으로, 작성자가 무엇을 재정의해야 하는지(더 중요하게는 무엇을 재정의하지 말아야 하는지) 알고 있는 것처럼 코드를 작성하는 것이 당연합니다. 예를 들어 `final`을 사용하지 않을 수도 있고 재정의 메서드를 설명하지 못할 수도 있습니다.

**이것이 Cocoa 프로그래머들이 대리자 패턴을 점점 더 많이 사용하는 이유입니다.**
#### 3. Lost type relationships
일반화된 정렬이나 이진 검색을 작성하려면 두 요소를 비교하는 방법이 필요합니다. 클래스를 사용하면 다음과 같이 끝납니다.
```swift
class Ordered {
  func precedes(other: Ordered) -> Bool
}

func binarySearch(sortedKeys: [Ordered], forKey k: Ordered) -> Int {
  var low = 0, high = sortedKeys.count

  while high > low {
    let mid = low + (high - low) / 2

    if sortedKeys[mid].precedes(k) {
      low = mid + 1
    } else {
      high = mid
    }
  }

  return low
}
```

물론 본문을 채우지 않고 클래스 함수를 정의할 수는 없습니다. `Ordered`의 임의 인스턴스에 대해 아무것도 모르는 경우 거기에 무엇을 넣을 수 있습니까? 오류가 발생하는 것 외에 우리가 할 수 있는 일은 실제로 없습니다.
```swift
class Ordered {
  func precedes(other: Ordered) -> Bool {
    fatalError("implement me!")
  }
}
```

이것은 우리가 유형 시스템과 싸우고 있다는 첫 번째 신호입니다. "`Ordered` 구현의 각 하위 클래스가 선행하는 한 괜찮을 것입니다"라고 말하면서 이 문제를 그냥 무시할 수는 없습니다.

`Ordered`의 하위 클래스를 구현하고 구현한다고 가정해 보겠습니다.
```swift
class Number: Ordered {
  var value: Double = 0

  override func precedes(other: Ordered) -> Bool {
    return value < other.value  // ERROR: 'Ordered' does not have a member named 'value'.
  }
}
```

`Ordered`는 `Label` 클래스와 같은 무엇이든 될 수 있으므로 컴파일되지 않습니다. 이제 올바른 유형을 얻으려면 다운캐스트해야 합니다.
```swift
return value < (other as! Number).value
```

`other`이 `Label`로 판명되면 어떻게 되나요? 우리 코드는 함정에 빠질 것입니다. 클래스가 자기 유형과 타인 유형 사이의 중요한 유형 관계를 표현할 수 없기 때문에 이 오류가 발생합니다.

강제적 다운타입캐스팅을 하지말라;; 지들이 만들어놓고 쓰지말래,,,
## We need a better abstraction mechanism
- Supports value types and classes
- Supports static type relationships and dynamic dispatch
- Is non-monolithic
- Supports retroactive modeling (can conform to another type's requirements in an extension)
- Doesn't impose instance data on models
- Doesn't impose initialization burdens on models
- Makes clear what to implement/override
## 정답은 프로토콜
Swift는 최초의 프로토콜 지향 프로그래밍 언어입니다. `for` 루프와 문자열 리터럴이 작동하는 방식부터 제네릭에 대한 표준 라이브러리의 강조까지 Swift의 핵심은 프로토콜 지향입니다.

프로토콜 먼저 만들자
```swift
protocol Ordered {
  func precedes(other: Ordered) -> Bool
}

struct Number: Ordered {
  var value: Double = 0

  func precedes(other: Ordered) -> Bool {
    return self.value < (other as! Number).value
  }
}
```

`Number`로 다운캐스트할 때 여전히 정적 유형의 안전 구멍이 있습니다. `Ordered`를 `Number`로 변경하면 `Number`가 `Ordered`를 따르지 않기 때문에 타입캐스팅 구문을 삭제할 수 없습니다.

```swift
func precedes(other: Number) -> Bool {
  // ERROR: protocol requires function 'precedes' with type '(Ordered) -> Bool'
  // candidate has non-matching type '(Number) -> Bool'
  return self.value < other.value
}
```

이 코드를 컴파일하려면 프로토콜에 [`Self 요구 사항`](obsidian://open?vault=Valut&file=Swift%2Fself%20vs%20Self)을 추가해야 합니다. `Self`는 프로토콜을 준수하는 동적 유형에 대한 자리 표시자입니다.

```swift
protocol Ordered {
  func precedes(other: Self) -> Bool
}
```

이제 이진 검색을 리팩토링할 수 있습니다. `Ordered`가 클래스일 때 함수 선언은 다음과 같습니다.

```swift
func binarySearch(sortedKeys: [Ordered], forKey k: Ordered) -> Int { ... }
```

`Ordered`가 클래스인 경우 `Ordered` 배열에는 모든 하위 클래스가 포함될 수 있습니다. 예를 들어, 3개가 모두 `Ordered`의 하위 클래스인 경우 [`Number`, `Label`, `Fruit`] 배열을 가질 수 있습니다.
## Replacing classes in OOP with protocols
사용자가 그리기 화면에 셰이프를 끌어서 놓은 다음 해당 셰이프와 상호 작용할 수 있는 다이어그램 앱을 모델링하겠습니다. 우리는 문서와 디스플레이 모델을 구축하고 있습니다.

첫째, 그리기 명령을 인쇄하는 기본 "Renderer"입니다.
```swift
struct Renderer {
  func move(to point: CGPoint) {
    print("Move to (\(point.x), \(point.y))")
  }

  func line(to point: CGPoint) {
    print("Line to (\(point.x), \(point.y))")
  }

  func arc(at center: CGPoint, radius: CGFloat, startAngle: CGFloat, endAngle: CGFloat) {
    print("Arc at \(center), radius: \(radius), startAngle: \(startAngle), endAngle: \(endAngle)")
  }
}
```

다음으로 모든 그리기 요소에 대한 공통 인터페이스를 제공하는 `Drawable` 프로토콜입니다.

```swift
protocol Drawable {
  func draw(using renderer: Renderer)
}
```

그런 다음 다각형과 같은 모양이 됩니다. 이는 다른 값 유형인 `CGPoint`로 구성된 값 유형입니다.

```swift
struct Polygon: Drawable {
  var corners = [CGPoint]()

  func draw(using renderer: Renderer) {
    renderer.move(to: corners.last!)

    for point in corners {
      renderer.line(to: point)
    }
  }
}
```

다음은 다른 값 유형으로 구성된 값 유형이기도 한 `Circle`입니다.

```swift
struct Circle: Drawable {
  var center: CGPoint
  var radius: CGFloat

  func draw(using renderer: Renderer) {
    renderer.arc(at: center, radius: radius, startAngle: 0.0, endAngle: twoPi)
  }
}
```

이제 원과 다각형으로 다이어그램을 만들 수 있습니다.

```swift
struct Diagram: Drawable {
  var elements = [Drawable]()

  func draw(using renderer: Renderer) {
    for element in elements {
      element.draw(using: renderer)
    }
  }
}
```
