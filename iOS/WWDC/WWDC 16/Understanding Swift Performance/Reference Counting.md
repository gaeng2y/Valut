- Swift는 힙에 할당된 메모리를 언제 할당 해제하는 것이 안전한지 어떻게 알까?
- 대답은 Swift가 힙의 모든 인스턴스에 대한 총 Reference count를 유지한다는 것이다.
- 그리고 인스턴스 자체에 유지한다.
- 참조를 추가하거나 참조를 제거하면 해당 참조 카운트가 증가하거나 감소한다.
- 그 수가 0에 도달하면 Swift는 아무도 더 이상 힙의 이 인스턴스를 가리키지 않는다는 것을 알고 해당 메모리를 할당 해제하는 것이 안전하다는 것을 알게 된다.
- 참조 카운팅에서 염두에 두어야 할 핵심 사항은 이것이 매우 빈번한 작업이며 실제로 정수를 증가 및 감소시키는 것보다 더 많은 것이 있다는 것이다.

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/2e543167-b078-4554-bd9d-48693c424cb4/b5cd7ee8-ab1e-4e66-a160-409c0e9199e1/Untitled.png)

- 첫째, 증가 및 감소를 실행하기 위한 몇 가지 수준의 간접 참조가 존재한다.
- 그러나 더 중요한 것은 힙 할당과 마찬가지로 여러 스레드가 힙 인스턴스에 참조를 동시에 추가하거나 제거할 수 있기 때문에 고려해야 할 `thread safety`가 있다는 것이다. 실제로 참조 카운트를 원자적으로 증가 및 감소 시켜야 한다.
- 그리고 참조 카운팅 작업의 빈도로 인해 이 비용이 증가될 수 있다.

```swift
// Reference Counting
// Class
class Point {
   var x, y: Double
   func draw() { ... }
}

let point1 = Point(x: 0, y: 0)
let point2 = point1
point2.x = 5
// use `point1`
// use `point2`
```

- 이제 `Point`를 Swift가 내부에서 어떻게 동작되는 지 살펴보자.

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/2e543167-b078-4554-bd9d-48693c424cb4/d1827841-017d-4d48-b37b-4cffb27f2a46/Untitled.png)

- 내부적으로 클래스에 refCount라는 프로퍼티가 만들어지고
- point1을 point2에 할당할 때 retain(point2) 메소드를 호출하여 point2가 해당 객체를 참고하고있다는 내용으로 refCount를 1 증가
- 그 뒤에는 사용이 끝나면 release 메소드를 통해 refCount를 1씩 감소하는 동작을 한다.
- 그 후에 Point의 refCount가 0이 된다면 힙 영역에서 해제된다.

### 그렇다면 구조체를 살펴보자.

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/2e543167-b078-4554-bd9d-48693c424cb4/47ead877-9532-4f68-9fd9-064a34ba7dc8/Untitled.png)

- 구조체는 단순히 값 타입이기 떄문에 스택에 쌓는다!
- 해제되면 스택에서 해제되면 끝이다.

## 레퍼런스를 포함한 구조체는?

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/2e543167-b078-4554-bd9d-48693c424cb4/48391be6-096a-4229-9da6-659c1c14df54/Untitled.png)

## 성능 지표

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/2e543167-b078-4554-bd9d-48693c424cb4/fe447f12-bfd8-41b2-86d0-ae4c48991e36/Untitled.png)

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/2e543167-b078-4554-bd9d-48693c424cb4/101f4518-861e-40c4-be8d-5ddd80d1b494/Untitled.png)

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/2e543167-b078-4554-bd9d-48693c424cb4/4fae8cf7-e002-4e03-ab2d-13ae5730c9fa/Untitled.png)

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/2e543167-b078-4554-bd9d-48693c424cb4/83971e54-7b18-4ec6-806c-549107dd7090/Untitled.png)

## 모델링 기술

이렇게 레퍼런스 카운트가 많을 경우 어떻게 바꿔야할 지 알아보자

```swift
struct Attachment {
   let fileURL: URL
   let uuid: String
   let mimeType: String
   
   init?(fileURL: URL, uuid: String, mimeType: String) {
      guard mimeType.isMimeType
      else { return nil }
      self.fileURL = fileURL
      self.uuid = uuid
      self.mimeType = mimeType
	} 
}
```

- 해당 구조체에서는 모든 프로퍼티 타입들이 참조가 발생한다.
- 두 프로퍼티를 `UUID` 타입과 열거형으로 바꿔서 적용하면!?
- fileURL만 참조한다.