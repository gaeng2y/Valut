MVI는 Model - View - Intent

![MVi.drawio](MVi.drawio.png)

- **View:** 사용자에게 제공되어 보여 지는 UI 부분으로 상태(state)를 입력 받아 화면에 출력

- **Intent:** 앱 상태를 변경 하려는 의도를 의미하며 사용자의 상호작용으로 발생한 상태를 변경 하는 동작

- **Model(State):** 앱의 상태를 의미하며 MVI에서는 하나의 상태만을 갖으며 imutable한 데이터 구조로 작성 되어야 합니다. intent를 트리거 하여 새로운 상태를 만드는 것만이 State를 바꾸는 유일한 방법입니다.

### Unidirectional Architecture
MVI에서 가장 중요한 것은 단방향 아키텍쳐라는 점이다.

**단방향 아키텍처**을 이해하기 전에 MVI 패턴의 흐름을 살펴보자.

1. View에서 User Event 발생
2. Intent로 해당 User Event 전달
3. Intent가 상황에 맞는 Model(State)을 업데이트
4. View에서는 State를 바인딩을 하고 있어 변경이 일어날 때마다 렌더링

여기서 가장 중요한 것은 Model을 View로 전달할 때 **Immutable** 하다는 것이다. 이는 View에서는 Model을 직접 변경할 수 없고 Intent를 거쳐야한다. (Single Source of Truth) 또한 Intent에서는 View에 대한 정보를 갖고 있지 않고 Intent에서 View로 Event를 전달할 수 없다.

## Example

Soruce Code: https://github.com/gaeng2y/MVI-Pattern/tree/main/MVI-example/CounterApp

#### UML diagram
![](diagram%20(1).png)