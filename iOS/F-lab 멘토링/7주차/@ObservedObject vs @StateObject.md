
### @ObservedObject

- observable 객체를 구독하는 property wrapper
	- observable 객체가 변경되면 뷰에 업데이트 시켜주는 기능

```swift
@available(iOS 13.0, macOS 10.15, tvOS 13.0, watchOS 6.0, *)
@propertyWrapper @frozen public struct ObservedObject<ObjectType> : DynamicProperty where ObjectType : ObservableObject {
    @dynamicMemberLookup @frozen public struct Wrapper {
        public subscript<Subject>(dynamicMember keyPath: ReferenceWritableKeyPath<ObjectType, Subject>) -> Binding<Subject> { get }
    }
    public init(initialValue: ObjectType)
    public init(wrappedValue: ObjectType)
    public var wrappedValue: ObjectType
    public var projectedValue: ObservedObject<ObjectType>.Wrapper { get }
}
```
### @StateObject

- SwiftUI는 상태가 변경되면 뷰를 처음부터 다시 만들어서 그리는 동작이 있지만, @StateObject를 사용하면 뷰를 다시 만들지 않고 항상 동일한 뷰가 사용

```swift
@available(iOS 14.0, macOS 11.0, tvOS 14.0, watchOS 7.0, *)
@frozen @propertyWrapper public struct StateObject<ObjectType> : DynamicProperty where ObjectType : ObservableObject {
    @inlinable public init(wrappedValue thunk: @autoclosure @escaping () -> ObjectType)
    public var wrappedValue: ObjectType { get }
    public var projectedValue: ObservedObject<ObjectType>.Wrapper { get }
}
```

#### @ObjervedObject와 @StateObject 차이점

- 둘 다 ObservableObject를 구독하여, 이 값이 변경되면 뷰에 반영해주는 property wrapper 형태
- 상태 변경이 있을땐 @ObservedObject는 뷰를 다시 생성해서 그리지만, @StateObject는 뷰를 다시 생성하지 않고 항상 동일한 뷰가 사용 (효율)
- 기본적으로 @StateObject를 사용하되, 해당 프로퍼티를 subview에게도 주입시켜야 한다면, @ObservedObject로 선언하여 사용할것
    - subview에 @StateObject 프로퍼티를 주입하면, 해당 @StateObject의 수명 주기가 두 곳에서 관리가 되므로 의존성을 줄이기 위해 @ObservedObejct를 사용