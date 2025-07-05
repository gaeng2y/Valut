TCA에서 `AlertState`와 `ConfirmationDialogState`를 사용하는 기본 흐름은 크게 네 단계로 나뉩니다:

1. 상태(State)에 Presentation용 프로퍼티 정의
2. 액션(Action)에 PresentationAction 래퍼 적용
3. 리듀서(Reducer)에서 실제 Alert/Dialog 생성
4. SwiftUI 뷰(View)에 바인딩

---

### 1️⃣ State 정의

```swift
@ObservableState
struct State: Equatable {
    @Presents var alert: AlertState<Action.Alert>?                             // Alert 상태  
    @Presents var confirmationDialog: ConfirmationDialogState<Action.ConfirmationDialog>? // Confirmation Dialog 상태  
    var count = 0                                                             // 예제용 카운터  
}
```

* `@Presents` 애노테이션을 사용해, Optional한 Presentation 상태를 선언합니다.
  ([Velog][1])

---

### 2️⃣ Action 정의

```swift
enum Action {
    case alert(PresentationAction<Alert>)                           // Alert 관련 액션  
    case alertButtonTapped                                         // Alert 표시 트리거  
    case confirmationDialog(PresentationAction<ConfirmationDialog>)// Dialog 관련 액션  
    case confirmationDialogButtonTapped                            // Dialog 표시 트리거

    @CasePathable enum Alert {
        case incrementButtonTapped                                 // Alert 내부 버튼 이벤트  
    }
    @CasePathable enum ConfirmationDialog {
        case incrementButtonTapped                                 // Dialog 첫 번째 버튼  
        case decrementButtonTapped                                 // Dialog 두 번째 버튼  
    }
}
```

* `PresentationAction<…>` 래퍼를 통해 “표시”, “버튼 클릭”, “해제” 등을 표현할 수 있으며,
* `@CasePathable`로 내부 케이스와 매핑합니다.
  ([Velog][1])

---

### 3️⃣ Reducer 구현

```swift
var body: some Reducer<State, Action> {
  Reduce { state, action in
    switch action {
    // Alert 버튼 클릭 시
    case .alertButtonTapped:
      state.alert = AlertState(
        title: { TextState("Alert!") },
        actions: {
          ButtonState(role: .cancel) { TextState("Cancel") }
          ButtonState(action: .incrementButtonTapped) { TextState("Increment") }
        },
        message: { TextState("This is an alert") }
      )
      return .none

    // Confirmation Dialog 버튼 클릭 시
    case .confirmationDialogButtonTapped:
      state.confirmationDialog = ConfirmationDialogState {
        TextState("Confirmation dialog")
      } actions: {
        ButtonState(role: .cancel) { TextState("Cancel") }
        ButtonState(action: .incrementButtonTapped) { TextState("Increment") }
        ButtonState(action: .decrementButtonTapped) { TextState("Decrement") }
      } message: {
        TextState("This is a confirmation dialog.")
      }
      return .none

    // Alert 내부 버튼 이벤트 처리
    case .alert(.presented(.incrementButtonTapped)):
      state.count += 1
      state.alert = AlertState { TextState("1 증가!") }
      return .none

    // Dialog 내부 버튼 이벤트 처리
    case .confirmationDialog(.presented(.decrementButtonTapped)):
      state.count -= 1
      state.alert = AlertState { TextState("1 감소!") }
      return .none

    default:
      return .none
    }
  }
  // Presentation 상태와 액션을 하위 Reducer에 연결
  .ifLet(\.$alert, action: \.alert)
  .ifLet(\.$confirmationDialog, action: \.confirmationDialog)
}
```

* `state.alert = AlertState { … }` 또는 `ConfirmationDialogState { … }` 로 Presentation을 구성하고,
* `.ifLet(\.$…)` 로 내부 액션 흐름을 바인딩합니다.
  ([Velog][1])

---

### 4️⃣ SwiftUI View 바인딩

```swift
struct AlertAndConfirmationDialogView: View {
  @Bindable var store: StoreOf<AlertAndConfirmationDialog>

  var body: some View {
    Form {
      Text("Count: \(store.count)")
      Button("Alert") {
        store.send(.alertButtonTapped)
      }
      Button("Confirmation Dialog") {
        store.send(.confirmationDialogButtonTapped)
      }
    }
    .alert(
      $store.scope(state: \.alert, action: .alert)
    )
    .confirmationDialog(
      $store.scope(state: \.confirmationDialog, action: .confirmationDialog)
    )
    .navigationTitle("Alerts & Dialogs")
  }
}
```

* `.alert(_:)`와 `.confirmationDialog(_:)`에 TCA가 제공하는 바인딩 초기화를 사용하면,
* 내부 Presentation 상태(`State.alert` / `State.confirmationDialog`)가 바뀔 때 SwiftUI가 자동으로 모달을 띄워줍니다.
  ([Velog][1])

---

이 흐름을 따라가면, TCA의 단방향 데이터 흐름 속에서도 SwiftUI의 Alert와 Confirmation Dialog를 깔끔하게 관리할 수 있습니다.
필요에 따라 버튼 역할(role), 텍스트, 액션 등을 커스터마이징해서 사용하세요!

[1]: https://velog.io/%40marine8743/SwiftUITCA-TCA-Case-Studies-01.-AlertAndConfirmationDialog "[SwiftUI][TCA] TCA Case Studies - 01. AlertAndConfirmationDialog"
