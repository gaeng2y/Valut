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

만약 View에서 토글을 사용하여 Store 외부에서 이 State를 조정할 수 있다면, Action의 형태를 정의할 수 있게 된다.

```swift
struct Settings: Reducer {
  struct State: Equatable { /* code */ }

  enum Action { 
    case isHapticFeedbackEnabledChanged(Bool)
    /* code */
  }

  /* code */
}
```

이제 reducer에서 해당 Action을 조작할 수 있도록 하면 reducer에서 Action에 대한 결과로 State의 값이 변경된다.

```swift
struct Settings: Reducer {
  struct State: Equatable { /* code */ }
  enum Action { /* code */ }
  
  func reduce(
    into state: inout State, action: Action
  ) -> Effect<Action> {
    switch action {
    case let .isHapticFeedbackEnabledChanged(isEnabled):
      state.isHapticFeedbackEnabled = isEnabled
      return .none

    /* code */
    }
  }
}
```

이로서 SwiftUI UI 컨트롤의 이벤트를 단방향 통신으로 TCA에 반영할 수 있는 바인딩을 완료했다.

```swift
struct SettingsView: View {
  let store: StoreOf<Settings>
  
  var body: some View {
    WithViewStore(self.store, observe: { $0 }) { viewStore in
      Form {
        Toggle(
          "Haptic feedback",
          isOn: viewStore.binding(
            get: \.isHapticFeedbackEnabled,
            send: { .isHapticFeedbackEnabledChanged($0) }
          )
        )

        /* code */
      }
    }
  }
}
```

Toggle 에서 isOn에는 바인딩 객체를 전달해야한다. `viewStore.binding(get:send:)`에서 get에 전달된 객체를 바인딩해서 `isOn`에 전달해주고, send에 전달된 Action을 **Store에 다시 피드백** 해줌으로써 해당 Action에 맞는 Reducer 로직이 실행되어 State의 값이 변경된다.

정리하면, binding의 get을 통해 SwiftUI UI 컨트롤과 바인딩 통신을 하고, `send`를 통해 Store 내부에 바인딩된 값을 전달함으로써 비즈니스 로직이 수행되도록 하는데, 이것이 가장 기본적인 구조다.

이제 추가로 Settings의 State를 작성해서 필요한 기능을 추가하면 어떤 문제점이 있는지 확인해보자.

```swift
struct Settings: Reducer {
  struct State: Equatable {
    var isHapticFeedbackEnabled = true
    var digest = Digest.daily
    var displayName = ""
    var enableNotifications = false
    var isLoading = false
    var protectMyPosts = false
    var sendEmailNotifications = false
    var sendMobileNotifications = false
  }

  /* code */
}
```

State의 각 프로퍼티는 대부분 View에서 편집할 수 있어야 하는데, View에서 프로퍼티를 편집할 때 프로퍼티를 편집하는 Action을 **열거형의 각 case로 정의**해야한다.

```swift
struct Settings: Reducer {
  struct State: Equatable { /* code */ }

  enum Action {
    case isHapticFeedbackEnabledChanged(Bool)
    case digestChanged(Digest)
    case displayNameChanged(String)
    case enableNotificationsChanged(Bool)
    case protectMyPostsChanged(Bool)
    case sendEmailNotificationsChanged(Bool)
    case sendMobileNotificationsChanged(Bool)
  }

  /* code */
}
```

이제 case로 선언된 **각 Action에 대한 비즈니스 로직**을 Reducer에 작성한다.

```swift
struct Settings: Reducer {
  struct State: Equatable { /* code */ }
  enum Action { /* code */ }

  func reduce(
    into state: inout State, action: Action
  ) -> Effect<Action> {
    switch action {
    case let .isHapticFeedbackEnabledChanged(isEnabled):
      state.isHapticFeedbackEnabled = isEnabled
      return .none

    case let digestChanged(digest):
      state.digest = digest
      return .none

    case let displayNameChanged(displayName):
      state.displayName = displayName
      return .none

    case let enableNotificationsChanged(isOn):
      state.enableNotifications = isOn
      return .none

    case let protectMyPostsChanged(isOn):
      state.protectMyPosts = isOn
      return .none

    case let sendEmailNotificationsChanged(isOn):
      state.sendEmailNotifications = isOn
      return .none

    case let sendMobileNotificationsChanged(isOn):
      state.sendMobileNotifications = isOn
      return .none
    }
  }
}
```

가장 기본적인 `binding(get:send)`를 사용할 때, get과 send에 **직접 State와 Action을 정의**하여 전달해야하는 작업이 필요하다. 하지만 Store에 State나 Action이 추가된다면 View에는 더 많은 바인딩 코드가 작성되어야하고, **Reducer의 관리도 복잡**해지는것을 확인할 수 있다.

코드가 길어지고 복잡하면 가독성이 나빠지고 반복적인 작업이 필요하다. 따라서 TCA는 Reducer와 View에 이러한 불필요한 작업이 반복되지 않도록 하고 간단하게 **바인딩 관리할 수 있는 환경**을 제공한다.

# 4.3 다양한 TCA의 Binding tools

앞서 언급한 문제를 해결하기 위해서 TCA에서는 다양한 대안적인 Binding 도구들을 제공하고 있다. `@BindingState`, `@BindingAction`, `@BindingReducer` 등이 그러하다. 이 도구들은 프로퍼티 래퍼를 활용하여 선언이 되는데, 이를 통해 Binding이 개선되는 과정을 살펴보자.

```swift
struct Settings: Reducer {
  struct State: Equatable {
	@BindingState var isHapticFeedbackEnabled = true
    @BindingState var digest = Digest.daily
    @BindingState var displayName = ""
    @BindingState var enableNotifications = false
    var isLoading = false
    @BindingState var protectMyPosts = false
    @BindingState var sendEmailNotifications = false
    @BindingState var sendMobileNotifications = false
  }

  /* code */
}
```

State 구조체 필드에 프로퍼티 래퍼가 추가되면서 이제 해당 필드들은 SwiftUI의 UI 컨트롤에 바인딩이 가능하여 View에서 해당 필드 값을 조정할 수 있다.

`isLoading` 은 프로퍼티 래퍼를 추가하지 않으므로 `View`에서 해당 필드 값을 변경할 수 없도록 강제하는 역할을 한다.

> [!tip]
> TCA 아키텍처는 모든 필드에 `@BindingState` 사용하는 것을 권장하지 않는다. 모든 필드에 표시하면 외부에서 즉시 변경할 수 있기 때문에 기능의 캡슐화가 손상될 수 있는데, 따라서 바인딩 프로퍼티 래퍼는 SwiftUI 컴포넌트에 전달하기 위한 필드에만 사용하는 것이 좋다. `@BindingState` 뿐만 아니라 다른 프로퍼티 래퍼를 사용할 때에도 꼭 필요한 경우에만 사용해야한다.

다음으로 Action 열거형에 BindableAction 프로토콜을 채택함으로써, State의 모든 필드 Action을 하나의 case로 축소하는데 축소된 Action case는 제네릭 타입을 갖는 BindingAction을 associatedValue로 보유한다. (제네릭 타입은 reducer의 State로 결정)

```swift
 struct Settings: Reducer {
  struct State: Equatable { /* code */ }

  enum Action: BindableAction {
    case binding(BindingAction<State>)
  }

  /* code */
}
```

이제 Reducer는 `@BindingReducer`를 사용하여 State을 변경하는 비즈니스 로직을 간단하게 할 수 있는데, `@BindingReducer`는 `Binding action`이 수신된 경우, State를 업데이트 해주는 Reducer다.

```swift
struct Settings: Reducer {
  struct State: Equatable { /* code */ }
  enum Action: BindableAction { /* code */ }

  var body: some Reducer<State, Action> {
    BindingReducer()
  }
}
```

