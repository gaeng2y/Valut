# 3.1 Effect의 구현과 활용

## 3.1.1 Action에 따른 결과: Effect

`Effect`는 `Reducer`의 액션이 반환하는 타입으로, **액션을 거친 모든 결과물**을 칭할 수 있다. 그 중에서 외부에서 어떠한 처리가 일어나서 얻게된 예상과 다른 결과물은 `Side Effect` 라고 한다.

`Effect`는 외부 시스템과 상호작용하는 작업을 나타내는데, 이를 통해 앱의 `State`가 변경된다. `State`를 직접 변경할 때의 `Action`과 달리, `Effect`는 비동기적인 작업을 수행하고 그 결과를 `Action`으로 반환하여 `State`에 반영하기 위해 사용된다.

즉, `Effect`는 특정 `Action`을 실행한 후 그 결과에 따라 **새로운 `Action`을 생성**하고, 이를 통해 `State`를 업데이트하는 역할을 담당한다. 네트워크 호출, 데이터 로딩, 외부 서비스와의 교류 등 다양한 비동기 작업이 `Effect`로 분류될 수 있다.

Effect는 TCA에서 다음과 같은 기능을 수행한다.

- **비동기 작업 관리:** 네트워크 요청, 데이터 로딩, 파일 다운로드 등 다양한 비동기 작업을 Effect를 통해 관리할 수 있다.
- **Side Effect 분리:** `Effect`는 순수 함수형 프로그래밍의 원칙에 따라 `Side Effect를 배제`한다. 이를 통해 코드에서 State 변화를 일으키는 부분과 Side Effect를 다루는 부분을 명확하게 분리함으로써 코드의 가독성과 추론력이 향상되며, 테스트와 디버깅 과정이 용이해진다.
- **취소 및 에러 핸들링:** Effect는 비동기 작업의 성공, 실패 및 중단을 관리하는 데 사용된다. 예를 들어, 네트워크 요청 중에 발생한 `오류를 적절하게 처리`하고 State를 업데이트할 수 있다.
- **순서 보장:** TCA의 `Effect`는 순차적으로 실행되며 그 순서가 보장된다. 이로써 State 변화와 관련된 Side Effect를 적절하게 처리하면서도 `예측 가능한 결과`를 얻을 수 있다.

## 3.1.2 순수함수적인 Effect

**순수 함수**는 주어진 입력에 대해 항상 동일한 출력을 반환하고, 외부 상태를 변경하지 않으며, **부수 효과(Side Effect)가 없는 함수**를 의미한다. 그렇기 때문에 순수 함수 자체로는 비동기 작업이나 Side Effect를 처리할 수 없다.

> [!tip] Effect와 Combine의 상관관계
> Combine의 개념에는 `Publisher`와 `Subscriber` 두 가지 개념이 있다. Publisher는 데이터의 종류나 형식에 대한 정보를 포함하여 데이터 흐름의 특성을 정의하고 Subscriber에게 전달한다. Subscriber는 Publisher가 발행하는 데이터 흐름을 구독하고, 이벤트나 값을 처리한다. 
> 
> 바로 여기에 Combine과 Effect 간의 기본적인 대응이 있다. Publisher와 `Effect`의 유형이, Subscriber와 Effect의 `.run` 이 같은 개념으로 서로 대응된다. 다시 말해, Combine에는 Publisher와 Subscriber가 있다면, Effect에는 Effect와 run이 있다. 
> 
> Effect와 Combine 프레임워크는 상당히 유사한 점이 있으므로 비동기 Side Effect를 Combine에서 제공하는 함수와 기능들을 활용하여 반응형, 단방향 프로그래밍 구조를 갖도록 한다.

## 3.1.3 주요 메소드
### **.none**

먼저 `.none` 이다. `Effect`는 반환해야하는데 아무런 동작도 취하고 싶지 않을 때 사용한다.

```swift
struct CounterFeature: Reducer {
    
    struct State: Equatable {/* code */}
  
    enum Action {/* code */}
    
    var body: some ReducerOf<Self> {
        Reduce { state, action in
            switch action {
            case .incrementButtonTapped:
                state.count += 1
                return .none
            case .decrementButtonTapped:
                state.count -= 1
                return .none
        }
    }
}
```

버튼을 눌렀을 때 어떠한 비동기 처리도 필요하지 않기 때문에 `.none`을 사용한 모습이다.

### **.send**

파라미터로 `Action`을 받는 메서드로 특정 액션 이후 즉시 추가적인 동기 액션이 필요할 때 사용된다. 주로 자식 컴포넌트에서 부모 컴포넌트로 데이터를 전달할 때 사용된다. 또한 액션을 전달하는 동시에 애니메이션을 지정할 수 있다.

### **.run**

**비동기 작업**을 래핑하는 메소드다. 인자로 비동기 클로저를 받아서 실행하며, 클로저 내부에서 send를 사용하여 액션을 시스템에 전달할 수 있다.

### **.cancellable(id:)과 .cancel(id:)**

`cancellable`은 `Effect`를 취소할 수 있게 만들어주는 메소드다. `id`는 해당 `Effect`를 식별하는 값이며, 그 값은 문자열과 같은 해시가 가능한 어떠한 타입이든 들어갈 수 있다.

`cancelInFlight`라는 `Bool` 타입의 옵션을 줄 수 있는데 이는 기본적으로 `false`로 설정되어있다. 이를 `true`로 설정하게 된다면 같은 `id`로 진행중인 효과를 취소한다.

이렇게 `cancellable`로 등록한 `id`를 필요한 타이밍에 `cancel`하게 된다면 이미 진행 중인 효과를 즉시 취소하고, 취소된 결과를 전달한다.

### **.merge와 .concatenate**

`.merge` 메소드는 여러개의 `Effect`를 동시에 실행시키는 기능을 한다. 이 메소드를 사용하면 여러개의 `Effect`를 인자로 받아서 그 결과를 병합한다. 순서가 보장되지 않아서 주로 관련 없는 각각의 `State`를 변화시킬 때 사용한다. `.concatenate` 메소드는 `.merge`와 같이 여러개의 `Effect`를 묶는데 사용된다는 공통점이 있지만, 순서를 부여한다는 차이점이 있다.