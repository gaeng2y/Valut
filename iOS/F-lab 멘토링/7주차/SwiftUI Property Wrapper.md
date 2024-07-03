## @State

```swift
@frozen @propertyWrapper
struct State<Value>
```

- @State는 view 내부에 속해야 할 때에만 사용 되어야 합니다.
- @State는 view 내부에서 초기화해야 하고, 다른 객체로 부터 @State 를 받는 것을 권장하지 않습니다. 이런 이유로 @State는 접근 제어자를 **private 으로 관리하기를 권장**합니다. 외부에서 해당 property를 수정해서는 안되기 때문입니다.
- SwiftUI는 @State property’s value를 내부적으로 저장합니다. 이후 @State의 **변경사항에 대하여 view 는 fresh render** 됩니다. 저장된 value는 fresh render 하는 동안에도 유지가 됩니다.
- 데이터를 전달할 때는 Binding<T>로 @State property 를 자식 view에 전달할 수 있습니다. 이 경우 자식 뷰에서도 수정이 가능합니다.