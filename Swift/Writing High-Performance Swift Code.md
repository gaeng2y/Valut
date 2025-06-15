# 고성능 Swift 코드 작성하기

이 문서는 고성능 Swift 코드를 작성하기 위한 다양한 팁과 기법을 정리한 문서입니다. 이 문서의 주요 독자층은 Swift 컴파일러 및 표준 라이브러리 개발자입니다.

이 문서에 소개된 일부 팁은 Swift 프로그램의 품질을 높이고 코드의 오류 가능성을 줄이며, 가독성을 향상시키는 데 도움을 줄 수 있습니다. 예를 들어 `final` 클래스나 클래스 전용 프로토콜을 명시하는 것이 그 예입니다.

그러나 이 문서의 일부 팁은 컴파일러나 언어의 특정한 임시적인 제약을 우회하기 위한 원칙에 어긋난 트릭일 수도 있습니다. 또한 많은 팁들은 프로그램의 런타임, 바이너리 크기, 코드의 명확성 등과의 트레이드오프가 존재합니다.

## 최적화 활성화하기 (Enabling Optimizations)

가장 먼저 해야 할 일은 최적화를 활성화하는 것입니다. Swift는 세 가지 최적화 수준을 제공합니다:

- `-Onone`: 일반적인 개발 환경을 위한 모드입니다. 최소한의 최적화만 수행하고, 디버깅 정보를 모두 보존합니다.
- `-Osize`: 대부분의 프로덕션 코드에 적합합니다. 이 모드에서는 컴파일러가 매우 공격적인 최적화를 수행하여 코드의 유형과 양을 크게 변경할 수 있습니다. 디버깅 정보는 생성되지만 일부 손실될 수 있습니다.
- `-O`: 이 모드는 성능을 코드 크기보다 우선시하는 최적화 모드입니다.

### Xcode에서 최적화 설정 변경 방법

1. Project Navigator에서 프로젝트 아이콘을 클릭하여 Project Editor로 이동합니다.
2. 왼쪽 패널에서 프로젝트 이름을 선택한 후, 상단의 "Build Settings" 탭으로 이동합니다.
3. "Optimization Level" 항목에서 원하는 최적화 수준을 선택합니다.

만약 UI에서 원하는 최적화 수준이 보이지 않는 경우, "Optimization Level" 드롭다운에서 `Other...`를 선택한 후 수동으로 컴파일러 플래그를 입력할 수 있습니다.

## 모듈 전체 최적화 (Whole Module Optimization, WMO)

기본적으로 Swift는 각 파일을 개별적으로 컴파일합니다. 이는 Xcode가 여러 파일을 병렬로 빠르게 컴파일할 수 있도록 도와줍니다. 하지만 각 파일을 독립적으로 컴파일하게 되면, 특정한 컴파일러 최적화가 불가능해질 수 있습니다.

Swift는 전체 프로그램을 마치 하나의 파일처럼 컴파일하고 최적화할 수 있습니다. 이 모드는 다음과 같은 컴파일러 플래그를 사용해 활성화할 수 있습니다:

```bash
-whole-module-optimization
```

이 모드로 컴파일된 프로그램은 컴파일 시간이 길어질 수 있지만, 런타임 성능은 향상될 가능성이 높습니다.

### Xcode에서 WMO 설정하기

Xcode의 빌드 설정에서 아래 항목을 활성화하면 됩니다:

- `Build Settings > Whole Module Optimization`

> 참고: 이후 이 문서에서는 'Whole Module Optimization'을 줄여서 `WMO`로 표기합니다.
## 동적 디스패치 줄이기 (Reducing Dynamic Dispatch)

Swift는 Objective-C와 마찬가지로 기본적으로 동적 디스패치를 사용합니다. 그러나 Swift는 성능이 중요한 코드에서 이러한 동적 동작을 제거하거나 줄일 수 있도록 여러 기능을 제공합니다.

### 동적 디스패치란?

클래스의 메서드와 프로퍼티 접근은 기본적으로 동적 디스패치를 통해 이루어집니다. 예를 들어 다음 코드에서 `a.aProperty`, `a.doSomething()`, `a.doSomethingElse()`는 모두 동적으로 호출됩니다.

```swift
class A {
  var aProperty: [Int]
  func doSomething() { ... }
  dynamic func doSomethingElse() { ... }
}

class B: A {
  override var aProperty: [Int] {
    get { ... }
    set { ... }
  }

  override func doSomething() { ... }
}

func usingAnA(_ a: A) {
  a.doSomething()
  a.aProperty = ...
}
```

Swift에서는 기본적으로 vtable을 통해 간접 호출이 이루어집니다. 만약 `dynamic` 키워드를 명시하면 Objective-C의 메시지 전송 방식이 사용됩니다. 이들은 모두 직접 함수 호출보다 느리며, 컴파일러 최적화를 어렵게 만들고 호출 오버헤드도 있습니다.

---

### `final` 키워드 사용하기

`final` 키워드는 클래스, 메서드, 프로퍼티가 오버라이드되지 않도록 제한합니다. 이 키워드를 사용하면 컴파일러가 직접 호출을 생성할 수 있게 됩니다.

```swift
final class C {
  var array1: [Int]
  func doSomething() { ... }
}

class D {
  final var array1: [Int]
  var array2: [Int]
}

func usingC(_ c: C) {
  c.array1[i] = ...
  c.doSomething()
}

func usingD(_ d: D) {
  d.array1[i] = ...
  d.array2[i] = ...
}
```

- `C.array1`과 `C.doSomething()`은 직접 접근/호출이 가능
- `D.array2`는 여전히 동적 디스패치됨

---

### `private`, `fileprivate` 접근 제한자 사용하기

`private` 또는 `fileprivate` 키워드를 사용하면 해당 선언이 동일 파일 내에서만 접근 가능해집니다. 컴파일러는 해당 선언이 오버라이드되지 않는다고 추론할 수 있어 `final`로 간주하고 최적화를 적용할 수 있습니다.

```swift
private class E {
  func doSomething() { ... }
}

class F {
  fileprivate var myPrivateVar: Int
}

func usingE(_ e: E) {
  e.doSomething() // 직접 호출로 최적화 가능
}

func usingF(_ f: F) -> Int {
  return f.myPrivateVar // 직접 접근으로 최적화 가능
}
```

---

### `internal` + WMO 조합

WMO가 활성화된 상태에서는 모듈 내부에서만 보이는 `internal` 접근 수준의 선언에 대해서도 컴파일러가 `final`을 추론할 수 있습니다. Swift에서 기본 접근 수준이 `internal`이기 때문에 WMO만 활성화해도 추가 설정 없이 더 많은 최적화 효과를 얻을 수 있습니다.
## 컨테이너 타입을 효율적으로 사용하기 (Using Container Types Efficiently)

Swift 표준 라이브러리는 `Array`와 `Dictionary` 같은 제네릭 컨테이너 타입을 제공합니다. 이 섹션에서는 이들을 성능적으로 어떻게 활용할 수 있는지 설명합니다.

---

### 값 타입(`struct`)을 Array에 사용하기

Swift에서 타입은 크게 값 타입(예: `struct`, `enum`, `tuple`)과 참조 타입(예: `class`)으로 나뉩니다. 값 타입은 `NSArray`로 브리징될 수 없기 때문에, `Array`에서 값 타입을 사용할 경우 불필요한 브리징 오버헤드를 피할 수 있습니다.

또한 값 타입이 참조 타입을 포함하지 않는 경우, 참조 카운팅(retain/release)도 필요 없으므로 성능이 더욱 향상됩니다.

```swift
struct PhonebookEntry {
  var name: String
  var number: [Int]
}

var phonebook: [PhonebookEntry]
```

> 단, 값 타입의 크기가 매우 큰 경우 복사 비용이 높아질 수 있어 주의가 필요합니다.

---

### NSArray 브리징이 불필요할 경우 `ContiguousArray` 사용

만약 참조 타입으로 구성된 배열이지만 `NSArray`와의 브리징이 필요 없다면 `Array` 대신 `ContiguousArray`를 사용하는 것이 더 나은 선택입니다.

```swift
class C { ... }

var array: ContiguousArray<C> = [C(), C(), C()]
```

`ContiguousArray`는 메모리 상에서 더 일관된 성능을 제공하며, Swift 전용 코드에서 사용할 때 특히 유리합니다.

---

### 객체 재할당보다 in-place 변경을 선호

Swift의 표준 컨테이너는 값 타입이지만 내부적으로 Copy-on-Write(COW)를 사용합니다. 즉, 복사가 실제로 발생하는 시점은 컨테이너가 참조되고 변경될 때입니다.

```swift
var c: [Int] = [1, 2, 3]
var d = c       // 이 시점에서는 복사되지 않음
d.append(4)     // 여기서 실제 복사가 발생함
```

> ⚠️ 함수 내부에서 파라미터를 복사하고 다시 반환하는 방식은 예상치 못한 복사를 유발할 수 있습니다:

```swift
func append_one(_ a: [Int]) -> [Int] {
  var a = a
  a.append(1)
  return a
}

var a = [1, 2, 3]
a = append_one(a) // 여기서 복사가 발생할 수 있음
```

이를 방지하기 위해 `inout` 파라미터를 사용하면 복사를 피할 수 있습니다:

```swift
func append_one_in_place(a: inout [Int]) {
  a.append(1)
}

var a = [1, 2, 3]
append_one_in_place(&a) // 복사 없이 처리됨
```
## 제네릭 최적화 (Generics)

Swift는 강력한 제네릭 타입 시스템을 제공합니다. 제네릭 함수나 타입은 다양한 타입에 대해 재사용이 가능하지만, 이로 인해 약간의 성능 오버헤드가 발생할 수 있습니다.

컴파일러는 각 제네릭 함수에 대해 공통된 코드 블록을 생성하며, 이 코드는 타입 정보를 담은 함수 포인터 테이블과 타입 값을 담은 박스를 통해 작동합니다.

```swift
class MySwiftFunc<T> { ... }

MySwiftFunc<Int> x     // Int용 코드 사용
MySwiftFunc<String> y  // String용 코드 사용
```

하지만 최적화가 활성화된 경우, 컴파일러는 호출 시점의 실제 타입을 분석하고 해당 타입에 특화된 코드를 생성합니다. 이를 **specialization(전문화)**이라 부릅니다.

### 예시

```swift
class MyStack<T> {
  func push(_ element: T) { ... }
  func pop() -> T { ... }
}

func myAlgorithm<T>(_ a: [T], length: Int) { ... }

var stackOfInts = MyStack<Int>()
stackOfInts.push(1)
stackOfInts.pop()

var arrayOfInts = [1, 2, 3]
myAlgorithm(arrayOfInts, length: arrayOfInts.count)
```

---

### 선언은 사용되는 모듈 내부에 위치시켜라

전문화(specialization)를 적용하려면, 제네릭 함수/타입의 정의가 최적화 시점에 **보여야** 합니다.  
즉, 같은 모듈 안에 제네릭 정의와 사용 코드가 있어야 하며, 모듈 외부에 있을 경우 `-whole-module-optimization`이 필요합니다.

> ✅ 표준 라이브러리는 예외입니다. 표준 라이브러리 내 정의는 모든 모듈에서 전문화가 가능합니다.

---

## 큰 값 타입의 복사 비용 (The Cost of Large Swift Values)

Swift에서 값 타입은 데이터를 고유하게 보유합니다. 이는 독립적인 상태를 보장해 주지만, 값이 클 경우 복사 비용이 커질 수 있습니다.

예를 들어 트리 구조를 값 타입으로 정의할 경우, 복사 시 많은 `malloc`, `free`, 참조 카운팅 비용이 발생할 수 있습니다.

```swift
protocol P {}
struct Node: P {
  var left, right: P?
}

struct Tree {
  var node: P?
}
```

트리를 복사하면 모든 하위 노드도 함께 복사됩니다.

---

## Copy-on-Write(COW) 사용

큰 값 타입의 복사 비용을 줄이기 위해 Swift는 `Array`와 같은 COW 구조를 제공합니다. 복사는 실제로 수정이 발생할 때만 일어납니다.

트리를 배열로 감싸면 COW의 이점을 얻을 수 있습니다:

```swift
struct Tree: P {
  var node: [P?]
  init() {
    node = [nil]
  }
}
```

### 단점

- `Array`는 `append`, `count` 등 불필요한 API를 노출함
- 인덱스 접근 시 범위 확인, Objective-C 호환을 위한 코드가 포함되어 성능 저하 가능성 있음

---

### 대안: 직접 구현한 COW 구조체

```swift
final class Ref<T> {
  var val: T
  init(_ v: T) { val = v }
}

struct Box<T> {
  private var ref: Ref<T>

  init(_ x: T) {
    ref = Ref(x)
  }

  var value: T {
    get { ref.val }
    set {
      if !isKnownUniquelyReferenced(&ref) {
        ref = Ref(newValue)
      } else {
        ref.val = newValue
      }
    }
  }
}
```

이 구조체는 COW 성능 이점을 갖는 동시에, 불필요한 API를 노출하지 않아 더 적절한 대안이 될 수 있습니다.
## Unsafe 코드 사용 (Unsafe Code)

Swift 클래스는 항상 참조 카운팅을 수행합니다. 객체를 접근할 때마다 참조 카운트가 증가하며, 이전 객체는 감소됩니다. 예를 들어 연결 리스트를 순회할 때:

```swift
elem = elem.next
```

이 코드 한 줄에도 retain/release가 발생하여 성능에 영향을 줄 수 있습니다.

---

### Unmanaged 참조 사용하기

`Unmanaged<T>` 타입을 사용하면 참조 카운팅을 생략할 수 있습니다. 단, 수명 보장을 직접 관리해야 하므로 매우 주의해야 합니다.

```swift
withExtendedLifetime(Head) {
  var ref: Unmanaged<Node> = Unmanaged.passUnretained(Head)

  while let next = ref._withUnsafeGuaranteedRef({ $0.next }) {
    ...
    ref = Unmanaged.passUnretained(next)
  }
}
```

> ⚠️ `Unmanaged<T>._withUnsafeGuaranteedRef`는 비공개 API이며, 향후 제거될 수 있으므로 외부 코드에서는 사용을 피하는 것이 좋습니다.

---

## 클래스 전용 프로토콜 (Protocols)

특정 프로토콜이 클래스에만 적용되는 경우, `AnyObject`를 상속하여 클래스 전용 프로토콜로 만들 수 있습니다. 이렇게 하면 컴파일러는 ARC 기반 최적화를 더 효과적으로 수행할 수 있습니다.

```swift
protocol Pingable: AnyObject {
  func ping() -> Int
}
```

> 클래스 전용 프로토콜을 선언하면, 구조체나 열거형이 이를 채택할 수 없어 성능 최적화에 도움이 됩니다.

---

## 클로저에서의 `let` vs `var` (The Cost of Let/Var in Escaping Closures)

클로저에서 `var`를 캡처하면 힙에 박싱(boxing)되어 성능이 저하됩니다. 반면 `let`은 복사되어 클로저 내부에 값으로 저장되므로 박싱이 발생하지 않습니다.

```swift
let f: () -> () = { ... }       // escaping closure
({ ... })()                     // non-escaping closure
x.map { ... }                   // non-escaping closure
```

### 해결 방법: `inout` 사용

실제로 클로저가 탈출하지 않는 경우, 캡처 대신 `inout` 파라미터를 사용하는 것이 성능상 이점이 있습니다.

```swift
func modify(_ x: inout Int) {
  x += 1
}
```

---

## 지원되지 않는 최적화 속성 (Unsupported Optimization Attributes)

일부 비공식 속성(attribute)은 컴파일러 최적화 지시자 역할을 합니다. 예:

```swift
@_specialize
@_transparent
```

이 속성들은 Swift Evolution을 통해 승인된 공식 기능이 아니며, 컴파일러 버전 간 변경될 수 있습니다. 따라서 일반적인 앱 코드에서는 사용을 권장하지 않으며, 내부 구현 또는 성능 실험 용도로만 활용해야 합니다.

> 이들 속성에 대한 공식 문서는 존재하지 않지만, Swift 컴파일러 소스와 최적화 패스 문서에서 예시를 확인할 수 있습니다.
