DispatchQueue를 초기화하는 메서드는 아래와 같다. label을 제외한 파라미터들에는 기본값이 설정되어 있기 때문에 커스텀이 필요한 파라미터에 대해서만 초기화를 해줄 수 있다.

```swift
convenience init(label: String,
                 qos: DispatchQoS = .unspecified,
                 attributes: DispatchQueue.Attributes = [],
                 autoreleaseFrequency: DispatchQueue.AutoreleaseFrequency = .inherit,
                 target: DispatchQueue? = nil)
```

# 1. label
```swift
let queue = DispatchQueue(label: "customQueue")
```

레이블의 이름을 만들어주는 것이다. 이렇게 만들면 글로벌 큐가 만들어지며 큐의 레이블은 디버깅 환경에서 추적하기 위해 작성하는 Identifier 같은 것이다.
## 2. qos

Quality of Service로 실행 될 Task들의 **우선 순위를 정해주는 값**이다.
### 1️⃣ User-interactive

main thread에서 작업하며, 사용자 인터페이스 새로고침, 애니메이션 등 사용자와 상호작용하는 작업에 할당합니다. 작업이 빠르게 수행되지 않으면, 유저 인터페이스는 멈추게 됩니다. 반응성(responsiveness)과 성능(performance)에 중점을 둡니다.

### 2️⃣ User-initiated

문서를 열거나 버튼을 클릭해 액션을 수행하는 것처럼 빠른 결과를 요구하는 유저와의 상호작용 작업에 할당합니다. 몇 초 이내의 짧은 시간 내에 수행해야하는 작업으로 반응성과 성능에 중점을 둡니다.

### 3️⃣ Default

QoS를 할당해주지 않을 경우 기본값으로 사용되며 User Initiate와 Utility의 중간 수준의 레벨입니다.

### 4️⃣ Utility

데이터를 읽거나 다운로드하는 작업처럼 작업이 완료되는데에 어느정도 시간이 걸리거나 즉각적인 결과가 요구되지 않는 작업에 할당합니다. 반응성, 성능, 에너지 효율의 밸런스에 중점을 둡니다.

### 5️⃣ Background

index 생성, 동기화, 백업 등 사용자가 볼 수 없는, 백그라운드의 작업에 할당합니다. 에너지 효율에 중점을 둡니다.

### 6️⃣ Unspecified

Unspecified는 QoS의 정보가 없음을 나타내며, 시스템이 QoS를 추론해야 합니다.
## 3. attributes

attributes는 DispatchQueue의 속성을 정해주는 값이다. `.concurrent`로 초기화한다면 다중 스레드 환경에서 코드를 처리하는 DispatchQueue, 이 값을 빈 배열, 즉 기본 값으로 아무 설정을 하지 않는다면 우리는 `Serial` DispatchQueue를 만드는 것이다.

이 외에도 한 가지 속성이 더 있는데요, 바로 `.initiallyInactive`이다. 일반적인 DispatchQueue들은 sync나 async 등 코드 블록을 호출하는 즉시 DispatchQueue 작업이 처리되었지만 .initiallyInactive는 그 작업을 제어해줄 수 있지요. 즉 sync나 async를 호출하더라도 작업을 큐에 담아놓을 뿐, `active()`를 호출하기 전까지는 작업을 처리하지 않는 것이다.
## 4. autoreleaseFrequency

DispatchQueue가 자동으로 객체를 해제하는 빈도의 값을 결정하는 파라미터입니다. 즉 객체를 autorealease해주는 빈도이며 기본값은 inherit이다.
## 5. target

코드 블록을 실행할 큐를 target으로 설정할 수 있다.

# async 메소드 파라미터

```swift
func async(group: DispatchGroup? = nil, qos: DispatchQoS = .unspecified, flags: DispatchWorkItemFlags = [], execute work: @escaping () -> Void)
```

## 1️⃣ group

DispatchQueue의 async 코드 블록을 묶어서 관리해주는 `DispatchGroup`이다. 여러 스레드에서 비동기로 작업을 처리하다보면 여러 개의 작업을 함께 관리해주어야할 때가 있다.

## 2️⃣ qos

위에서 정리한 내용과 똑같다.

## 3️⃣ flags

DispatchWorkItemFlags 타입의 값을 받는 파라미터다. 코드 블록을 실행할 때 적용될 추가 속성을 결정한다. 기본 값으로는 아무 속성도 부여하지 않는다. flags 값은 OptionSet이므로 여러 가지의 속성을 한 번에 부여할 수도 있다.

- `assingCurrentContext`: 코드 블록을 실행하는 context(Queue 혹은 스레드)의 속성을 상속받는다. QoS와 같은 속성을 동일하게 한다는 이야기다.
- `barrier`: concurrnet queue 환경에서 barrier(장벽, 차단) 역할을 한다. barrier 속성의 코드 블록이 실행되기 전에 실행되었던 코드들은 완료까지 실행되고, barrier 속성의 코드 블록이 실행되기 전까지 다른 코드 블록은 실행되지 않는다.
- `detached`: 실행할 코드 블록에 실행 중인 context(Queue 혹은 스레드)의 속성을 적용하지 않는다.
- `enforceQoS`: 실행 중인 context의 QoS보다 실행할 코드 블록의 QoS에 더 높은 우선 순위를 부여한다.
- `inheritQoS`: enforceQoS와 반대로 실행 중인 context의 QoS에 더 높은 우선 순위를 부여한다.
- `noQoS`: QoS를 할당하지 않고 코드 블록을 실행시킨다. `assingCurrentContext`보다 우선시 되는 속성이다.