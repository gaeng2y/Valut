## 1. Deep Copy (깊은 복사)

* **값 타입(Value Type)** 은 기본적으로 대입하거나 함수에 전달될 때 **복사(copy)** 가 일어납니다.
* "깊은 복사"라는 것은 **내부에 있는 모든 값들을 완전히 새로운 메모리에 복제**하는 것을 의미합니다.
* 따라서 원본과 복사본은 완전히 독립적이며, 한 쪽을 변경해도 다른 쪽에 영향을 주지 않습니다.

📌 예시 (간단한 값 타입 구조체)

```swift
struct Point {
    var x: Int
    var y: Int
}

var p1 = Point(x: 10, y: 20)
var p2 = p1 // 깊은 복사
p2.x = 99

print(p1.x) // 10
print(p2.x) // 99
```

→ `p1`과 `p2`는 별개 메모리에 저장된 값.

하지만 실제로는 Swift에서 **모든 값 타입이 무조건 Deep Copy를 바로 하는 건 아님** → 여기서 COW가 등장합니다.

---

## 2. Copy on Write (COW, 쓰기 시 복사)

* Swift의 성능 최적화를 위한 기법.
* `Array`, `Dictionary`, `Set`, `String` 같은 **컬렉션 타입(값 타입)** 에 적용됩니다.
* 대입할 때 **즉시 복사하지 않고**, 원본과 복사본이 **같은 메모리를 공유**하다가, **실제로 값을 수정하려고 할 때(copy on *write*)** 비로소 깊은 복사가 일어납니다.

📌 예시

```swift
var a = [1, 2, 3]
var b = a // 아직 같은 메모리 공유 (복사 안 됨)
print(Unmanaged.passUnretained(a as AnyObject).toOpaque())
print(Unmanaged.passUnretained(b as AnyObject).toOpaque())

b[0] = 99 // 이 시점에서 복사 발생 (Deep Copy)
print(a) // [1, 2, 3]
print(b) // [99, 2, 3]
```

👉 이 방식 덕분에 성능이 최적화됩니다:

* **읽기만 할 때는 복사 비용이 없다.**
* **쓰기 시에만 진짜 복사가 일어난다.**

---

## 3. 정리

| 구분     | Deep Copy              | COW (Copy on Write)            |
| ------ | ---------------------- | ------------------------------ |
| 언제 복사? | 대입 시 즉시 모든 데이터 복사      | 대입 시 공유 → 수정할 때 복사             |
| 비용     | 항상 높은 비용               | 필요할 때만 비용 발생                   |
| 적용     | 일반 값 타입 (struct, enum) | Swift 컬렉션 타입 (Array, String 등) |
| 장점     | 독립성 보장                 | 메모리 절약 + 성능 최적화                |

---

✅ Swift의 핵심 철학은 **값 타입을 기본으로 하고, COW로 성능을 최적화**하는 것입니다. 그래서 Swift에서 `Array` 같은 타입을 값 타입으로 안전하게 쓰면서도, 성능은 참조 타입처럼 빠르게 유지할 수 있는 거예요.

---

그러면 Swift에서 **Array의 COW(Copy on Write) 동작**을 직접 확인할 수 있는 예제를 보여드릴게요. 
핵심은 **수정 전까지는 같은 메모리를 공유**하다가, 수정 시점에 **실제 복사(Deep Copy)** 가 일어난다는 걸 확인하는 거예요.

---

## 📌 COW 동작 확인 예제

```swift
import Foundation

// 메모리 주소를 출력하는 헬퍼 함수
func address(of array: [Int]) -> String {
    return array.withUnsafeBufferPointer { buffer in
        return String(describing: buffer.baseAddress)
    }
}

var a = [1, 2, 3]
var b = a // 여기서는 복사 X (메모리 공유)

print("초기 상태")
print("a 주소:", address(of: a))
print("b 주소:", address(of: b))

// b를 수정하기 전까지는 a와 b가 같은 메모리 주소를 공유
b[0] = 99 // 이 순간 복사 발생 (Deep Copy)

print("\n수정 후")
print("a:", a)
print("b:", b)
print("a 주소:", address(of: a))
print("b 주소:", address(of: b))
```

---

## 📌 실행 결과 예시

(주소 값은 환경마다 다르지만, 흐름은 같음)

```
초기 상태
a 주소: Optional(0x00006000011a8200)
b 주소: Optional(0x00006000011a8200)   // 같은 주소 (공유)

수정 후
a: [1, 2, 3]
b: [99, 2, 3]
a 주소: Optional(0x00006000011a8200)
b 주소: Optional(0x00006000011a8400)   // 다른 주소 (Deep Copy 발생)
```

---

✅ 요약

* `a`를 `b`에 대입할 때는 실제 복사하지 않고 같은 메모리를 공유함.
* `b`를 **수정하는 순간** Swift가 자동으로 Deep Copy를 수행해 독립된 배열을 만듦.
* 이게 바로 Swift 컬렉션 타입에서의 **COW 최적화**.

---
