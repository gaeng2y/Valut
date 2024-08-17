# 2.1 TCA

TCA는 현재 상태가 어떤지 파악하고 이를 관리하기 쉽게 하기 위하여 고안된 단방향 아키텍처다.

TCA는 상태 변화에 대한 일관적인 가이드라인을 컴파일 단계에서부터 확보한다. 어떤 개발자든 TCA를 적용한다면 상태 변화를 일으키는 Action 정의, Action에 대한 View에서의 .send() 를 거쳐야 하는 것이다.

![diagram](https://axiomatic-fuschia-666.notion.site/image/https%3A%2F%2Fprod-files-secure.s3.us-west-2.amazonaws.com%2F73bceb19-6fe9-4dbd-9a6d-7e020bec97b4%2Fbbfc5f13-7fda-4841-853c-481af7fa4fd3%2FUntitled.png?table=block&id=d92bce4a-1245-445d-9830-203b08660f11&spaceId=73bceb19-6fe9-4dbd-9a6d-7e020bec97b4&width=1060&userId=&cache=v2)

위 그림과 같이 TCA는 원래의 목적을 달성하기 위해 Single Source Of Truth(단일 진실 공급원)를 따르는 철저한 단방향 구조를 선택했다.

위 그림과 같이 TCA는 원래의 목적을 달성하기 위해 Single Source Of Truth(단일 진실 공급원)를 따르는 철저한 단방향 구조를 선택했다. **Single Source Of Truth**는 애플리케이션의 상태나 데이터에 대한 유일한 출처를 가지는 것을 의미한다. 이로써 모든 상태 변경의 출처가 명확해지며, 그 변경이 예측, 추적 가능해지는 것이다.

TCA에서는 변동을 추적하고 의도하지 않은 변경에서 지켜내야할 상태를 `State`라고 한다. 사용자가 `View`를 통해 어떠한 작업이나 알림, 이벤트를 트리거 하면 연결된 `Action`이 `Reducer` 내에 구현된 함수로 `State`를 변화시키는 `Effect`를 반환한다. 이때, 그 작업이 크게 복잡하지 않다면 바로 상태를 변화시킬 수도 있고, 때로는 네트워크 통신과 같은 비동기적인 작업이 필요한 경우가 있을 수 있다. 후자의 경우에는 통신을 실행하는 데에 필요한 환경과 자원이 필요하고 우린 이를 “의존성(`Dependency`)을 갖게 된다”라고 표현한다. 또한 이때, 이 작업은 성공할 수도, 실패할 수도 있다. 보통 실패를 의도하는 경우는 없기에 성공한 경우를 `Effect`, 예상치 못한 경우를 `Side Effect`라고 한다. 이렇게 도출된 `Effect`에 따라 다시 `Action`을 선택해 `State`를 변경한다. 이렇게 변경된 `State`는 다시 `View`에 영향을 주게 되는 흐름으로 TCA는 작동한다.

# 2.2 앱의 상태: State

`State`는 `Reducer`의 현재 상태를 갖는 구조체다. 비즈니스 로직을 수행하거나 UI를 그릴 때 필요한 데이터에 대한 설명을 나타낼 때 사용된다.

버튼을 눌러서 어떤 숫자를 +1 해주거나 -1 해줘야 하는 간단한 `View`를 만들어야 한다고 해보자.

그 역할을 맡을 count라는 변수가 필요하다. TCA에서 이를 사용하기 위해 아래와 같이 간단하게 해당 기능을 하는 `State`를 정의할 수 있다.

```swift
struct CounterFeature: Reducer {
    // 기능을 구현하기 위한 변수를 State에 만들어줍시다.
    struct State: Equatable {
        var count = 0
    }
    
    enum Action: Equatable {/* code */}
   
    var body: some ReducerOf<Self> {/* code */}
}
```

이렇게 만든 `Reducer`를 `View`와 연결해보자.

```swift
struct CounterView: View {
    // let store: Store<CounterFeature.State, CounterFeature.Action>의 축약형이다.
    let store: StoreOf<CounterFeature>
    
    var body: some View {
        WithViewStore(self.store, observe: { $0 }) { viewStore in
						// SwiftUI에서도 작동하는 View
            Form {
                Section {
                    Button("Decrement") {
                        /* code */
                    }
                    Button("Increment") {
                        /* code */
                    }
                }
            }
        }
    }
}
```

## 2.2.1 State가 Equatable 한 Struct인 이유

위의 `State` 예제 코드에서 자연스럽게 `State` 구조체가 `Equatable`을 따를 것을 명시해줬다. 하지만 여기에서 `Action`은 `Equatable`을 빼도 문제가 없지만 `State`에 `Equatable`을 빼게 되면 에러가 발생한다.

> [!failure]
> Referencing initializer 'init(_:observe:content:file:line:)' on 'WithViewStore' requires that 'CounterFeature.State' conform to 'Equatable’

위 에러 메시지는 `WithViewStore`의 이니셜라이저에서 `State`가 `Equatable` 프로토콜을 준수하도록 요구하기 때문에 발생한다. SwiftUI는 `View`의 상태가 변경되었을 때 해당 `View`를 자동으로 업데이트한다.
-> Observation 프레임워크가 나오면서 @ObservableState 매크로가 나오면서 이제 사용하지 않아도 된다.

## 2.3 상태 변화를 일으키는 모든 동작: Action

`Action`은 디바이스와 사용자 인터랙션을 받아오기 위한 타입이다. 카운터 앱이니 당연히 숫자를 하나 올리거나 낮추는 기능이 있어야 한다. 따라서 바로 `Action`에 우리가 원하는 사용자 인터랙션 상황을 추가해보자.

```swift
struct CounterFeature: Reducer {
    struct State: Equatable {
        var count = 0
    }
    
    enum Action: Equatable {
				//숫자 증감버튼들을 탭할때의 상황
		    case incrementButtonTapped
		    case decrementButtonTapped
		}
   
    var body: some ReducerOf<Self> {/* code */}
}
```

이런 직관적인 동작뿐만이 아니라, 알림 창을 닫거나 API의 request를 받는다든지 타이머를 작동시킨다든지 하는 복잡한 행동들도 `Action`에 추가할 수 있다. 이렇게 받아온 `Action`들은 사용자가 디바이스와 상호작용할 때 일종의 신호를 받아온다. 그리고 그렇게 `Action`이 `Reducer`의 `State`를 변경하거나 외부와 통신할 수 있는 `Effect`를 반환시키도록 트리거를 하는 것이다.

## 2.3.1 Action 네이밍 컨벤션

이런 `Action`의 이름을 지을 때에는 `Reducer`가 실행할 로직이 아니라 사용자가 UI에서 수행한 작업의 이름을 그대로 따서 짓는 것을 추천한다.

# 2.4 변경을 처리한다: Reducer

