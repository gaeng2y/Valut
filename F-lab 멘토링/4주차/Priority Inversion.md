### 정의

Priority Inversion(우선순위 역전)은 낮은 우선순위를 가진 태스크가 높은 우선순위를 가진 태스크의 실행을 간접적으로 방해하는 상황을 말합니다. 이는 주로 여러 스레드가 공유 자원에 접근할 때 발생합니다. 낮은 우선순위의 스레드가 공유 자원을 점유하고 있는 동안 높은 우선순위의 스레드는 그 자원을 기다려야 하므로 실행이 지연됩니다.

### Priority Inversion의 예시

#### 시나리오
- **Low-priority Task (L)**: 우선순위가 낮은 스레드.
- **Medium-priority Task (M)**: 우선순위가 중간인 스레드.
- **High-priority Task (H)**: 우선순위가 높은 스레드.

1. Low-priority Task(L)가 공유 자원에 접근하여 이를 점유하고 작업을 진행 중입니다.
2. High-priority Task(H)가 공유 자원에 접근하려고 하지만, L이 이미 점유하고 있어서 대기 상태로 전환됩니다.
3. 이때 Medium-priority Task(M)가 실행되며, H가 대기 상태에 있기 때문에 M이 CPU를 점유합니다.
4. M이 작업을 완료한 후에야 L이 계속 실행되어 공유 자원을 놓고, H가 실행될 수 있습니다.

이로 인해 H의 실행이 지연되는 상황이 Priority Inversion입니다.
### Swift 코드 예시

아래는 Swift로 Priority Inversion이 발생할 수 있는 상황을 설명하는 예제입니다. 이 예제에서는 `DispatchQueue`와 `DispatchSemaphore`를 사용합니다.

```swift
import Foundation

// 공유 자원 보호를 위한 세마포어
let semaphore = DispatchSemaphore(value: 1)

// 낮은 우선순위의 태스크
let lowPriorityQueue = DispatchQueue.global(qos: .background)
lowPriorityQueue.async {
    semaphore.wait() // 공유 자원 점유
    print("Low-priority task started")
    // 작업을 오래 지속해서 우선순위 역전이 발생하게 함
    sleep(3)
    print("Low-priority task finished")
    semaphore.signal() // 공유 자원 해제
}

// 높은 우선순위의 태스크
let highPriorityQueue = DispatchQueue.global(qos: .userInteractive)
highPriorityQueue.async {
    // 잠시 대기하여 낮은 우선순위 태스크가 먼저 실행되도록 함
    sleep(1)
    print("High-priority task waiting for resource")
    semaphore.wait() // 공유 자원 접근 시도
    print("High-priority task started")
    print("High-priority task finished")
    semaphore.signal() // 공유 자원 해제
}

// 중간 우선순위의 태스크
let mediumPriorityQueue = DispatchQueue.global(qos: .default)
mediumPriorityQueue.async {
    // 잠시 대기하여 낮은 우선순위 태스크가 먼저 실행되도록 함
    sleep(2)
    print("Medium-priority task started")
    // 이 태스크가 낮은 우선순위 태스크보다 먼저 실행됨으로써 높은 우선순위 태스크의 실행을 더 지연시킴
    print("Medium-priority task finished")
}

// 프로그램이 끝날 때까지 대기
sleep(6)
```

이 예제에서는 Low-priority Task가 공유 자원을 점유하고 있는 동안 High-priority Task가 대기 상태로 전환되고, 그 사이에 Medium-priority Task가 실행됨으로써 Priority Inversion이 발생합니다. Medium-priority Task가 완료된 후에야 Low-priority Task가 공유 자원을 해제하고, High-priority Task가 실행될 수 있습니다.