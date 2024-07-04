## @State

```swift
@frozen @propertyWrapper
struct State<Value>
```

- @State는 view 내부에 속해야 할 때에만 사용 되어야 합니다.
- @State는 view 내부에서 초기화해야 하고, 다른 객체로 부터 @State 를 받는 것을 권장하지 않습니다. 이런 이유로 @State는 접근 제어자를 **private 으로 관리하기를 권장**합니다. 외부에서 해당 property를 수정해서는 안되기 때문입니다.
- SwiftUI는 @State property’s value를 내부적으로 저장합니다. 이후 @State의 **변경사항에 대하여 view 는 fresh render** 됩니다. 저장된 value는 fresh render 하는 동안에도 유지가 됩니다.
- 데이터를 전달할 때는 제네릭 Binding으로 @State property 를 자식 view에 전달할 수 있습니다. 이 경우 자식 뷰에서도 수정이 가능합니다.

## @Binding

```swift
@frozen @propertyWrapper @dynamicMemberLookup
struct Binding<Value>
```

- 다른 view로부터 property들을 받을 때에 사용
- 받은 view는 외부 소스로부터 생성된 proprty 변경 사항에 대해 fresh render
- @Binding 프로퍼티를 수정하면 제공한 뷰도 같이 업데이트됨
- view가 fresh render 할 때 @Binding은 유지 되지 않습니다. 외부에서부터 전달되기 때문에 굳이 유지할 필요가 없기 때문

## @StateObject

```swift
@MainActor @frozen @propertyWrapper @preconcurrency
struct StateObject<ObjectType> where ObjectType : ObservableObject
```

- Object의 @State 버전
- @StateObject는 ObservableObject가 AnyObject라 참조 타입이어야 한다.
- 그리고 SwiftUI는 fresh render를 할 때 처음 생성된 @StateObject property를 유지

## @ObservedObject

```swift
@MainActor @propertyWrapper @preconcurrency @frozen
struct ObservedObject<ObjectType> where ObjectType : ObservableObject
```

- @State와 @Binding의 관계랑 비슷, @StateObject를 넘겨줄 때 $를 붙힐 필요가 없다.
- 사용되는 뷰에 의해 ‘생성 또는 소유’되지 않은 ObservableObject instances를 wrap하는 데에 사용

##  @Environment

- 초기화 시점에 넘겨주고 싶지 않은 경우 사용(자식 view, App Scene등에 의존성을 만들어 줄 수 있다)
- 