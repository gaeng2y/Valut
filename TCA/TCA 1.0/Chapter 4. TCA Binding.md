# 4.1 SwiftUI Binding VS TCA Binding
SwiftUI는 `@State`, `@ObservedObject` 등의 프로퍼티 래퍼를 통해 양방향 바인딩을 구현하고, `State` 관리를 간편하게 할 수 있는 탁월한 기능을 제공한다. 추가로, `@Binding`은 `@State` 프로퍼티 래퍼와 함께 `View` 간의 상호작용을 위해 사용되며, 이는 양방향 통신을 가능하게 하는 프로퍼티 래퍼다.

SwiftUI의 바인딩은 State 변화를 UI에 즉시 반영해주며, **코드 작성이 간결**한다는 장점이 있다. 다만, State 관리가 복잡해질수록 State 변화에 따른 **Side Effects를 관리**하는 데 어려움이 있다. 이러한 이유로, 앱의 전체 비즈니스 로직을 담당하고 내부 상태 변화를 관리하는 TCA에서는 `@Binding` 프로퍼티 래퍼로 상태 관리를 하기에 적합하지 않다.

TCA의 Binding은 이러한 문제를 해결하기 위해 고안한 고유의 **상태 관리 바인딩 도구들을 제공**한다. TCA는 단방향 데이터 흐름 원칙을 따르며, State 변화는 **Action을 전송하고 Reducer가 이를 수신하여 로직을 처리**한다. 이로써 State 변화에 따른 **Side Effects를 일관되게 관리**할 수 있고, State 변화 로직을 명확하게 파악할 수 있어 복잡한 State 관리가 가능합니다. 그러나 간단한 State 변화에 비해 코드가 복잡하며, 각 State 변화에 대해 Action을 정의하고 Reducer에서 이를 처리하는 로직을 작성해야한다.

> [!summary] SwiftUI의 Binding VS TCA Binding
> 
|  | SwiftUI                                        |  TCA                                                    |
| ------- | ------------------------------------------ | ----------------------------------------------------- |
| 장점      | - State의 변경이 UI에 즉시 반영 - 코드 작성이 간단         | - State 변경 로직 관리 용이 - 복잡한 State 관리, Side Effect 처리 용이 |
| 단점      | - State 변경에 따른 복잡한 Side Effect 처리에 용이하지 않다 | - 어려운 구현 난이도                                          |
|         |                                            |                                                       |
# 4.2 TCA Binding

TCA의 가장 기본적인 바인딩은 `Binding(get:send:)` 형태다. 두 개의 클로저를 매개변수로 전달받는다.

- **get**: State를 바인딩의 값으로 변환하도록 하는 클로저
- **send**: 바인딩의 값을 다시 Store에 피드백 하는 Action으로 변환하는 클로저

예시를 살펴보자. Reducer에는 사용자가 햅틱 피드백을 활성화 했는지 추적하는 도메인 기능이 있다고 가정하자. State에는 햅틱 피드백에 대한 Bool 프로퍼티를 정의할 수 있다.

```swift
struct Settings: Reducer {
  struct State: Equatable {
    var isHapticFeedbackEnabled = true
    /* code */
  }

  /* code */
}
```

