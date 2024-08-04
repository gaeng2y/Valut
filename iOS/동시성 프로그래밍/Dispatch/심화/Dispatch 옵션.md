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
## 3. attributes

attributes는 DispatchQueue의 속성을 정해주는 값이다. `.concurrent`로 초기화한다면 다중 스레드 환경에서 코드를 처리하는 DispatchQueue, 이 값을 빈 배열, 즉 기본 값으로 아무 설정을 하지 않는다면 우리는 `Serial` DispatchQueue를 만드는 것이다.

이 외에도 한 가지 속성이 더 있는데요, 바로 `.initiallyInactive`이다. 일반적인 DispatchQueue들은 sync나 async 등 코드 블록을 호출하는 즉시 DispatchQueue 작업이 처리되었지만 .initiallyInactive는 그 작업을 제어해줄 수 있지요. 즉 sync나 async를 호출하더라도 작업을 큐에 담아놓을 뿐, `active()`를 호출하기 전까지는 작업을 처리하지 않는 것이다.
## 4. autoreleaseFrequency

DispatchQueue가 자동으로 객체를 해제하는 빈도의 값을 결정하는 파라미터입니다. 즉 객체를 autorealease해주는 빈도이며 기본값은 inherit이다.
## 5. target

코드 블록을 실행할 큐를 target으로 설정할 수 있다.