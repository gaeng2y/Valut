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

레이블의 이름을 만들어주는 것이다. 이렇게 만들면 글로벌 큐가 만들어지며 큐의 레이블은 디버깅 환경에서 추적하기 위해 작성하는 String 값이다.
