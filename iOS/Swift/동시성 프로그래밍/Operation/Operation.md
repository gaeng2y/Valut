Operation은 이렇게 `수행할 작업이 캡슐화된 객체`라고 생각해볼 수 있을 것 같다.

수행해야할 코드 블록들을 객체화하면 어떤 장점이 있을까? 객체지향 프로그래밍의 장점과 거의 비슷하다.

1. 재사용에 용이하다.
2. 타입 간의 관계를 만들어줄 수 있다.
3. 다양한 프로퍼티를 활용해볼 수 있다.
4. 스케쥴링에 용이하다.

Operation을 사용하면 동시성 프로그래밍과 관련된 모든 작업들은 `Operation이라는 객체`로서 만들어지게 된다.

![oq](https://user-images.githubusercontent.com/73867548/153408968-b42522db-d1e1-43e4-b7a2-8e101cab45d7.png)
Order Bar라는 Operation Queue에 쌓일 것이고, 먼저 들어온 순서대로 처리할 것이다.

위 그림을 코드로 작성해보자.

```swift
import Foundation

let order1 = BlockOperation {
    print("돈까스")
    print("떡볶이")
}

let order2 = BlockOperation {
    print("튀김 우동")
}

let order3 = BlockOperation {
    print("알리오 올리오")
    print("생맥주 2")
}

let order4 = BlockOperation {
    print("과일 세트")
    print("LA 갈비")
}

let order5 = BlockOperation {
    print("김치전")
    print("바닐라 아이스크림")
}

let orderBar = OperationQueue()
orderBar.maxConcurrentOperationCount = 2

orderBar.addOperation(order1)
orderBar.addOperation(order2)
orderBar.addOperation(order3)
orderBar.addOperation(order4)
orderBar.addOperation(order5)

// orderBar.addOperations([order1, order2, order3, order4, order5], waitUntilFinished: true)
```

# 정의

```swift
class Operation : NSObject
```

Operation은 추상클래스다. 그러니 항상 이를 상속받는 타입을 사용해야한다. 하위 클래스는 커스텀 클래스로 직접 만들어주는 방법이 있고, 앞서 살펴본 코드처럼 `BlockOperation`이라는 하위 클래스를 사용하는 방법이 있다. 커스텀 클래스는 다음 페이지에서 함께 살펴보도록 하자.

### Operation 만들기

```swift
class BlockOperation : Operation
```

Operation의 하위 클래스인 `BlockOperation`을 활용하여 이렇게 Operation을 만들어줄 수 있다.

```swift
let operation = BlockOperation {
    // some code
}

// BlockOperation의 메서드
operation.addExecutionBlock {
    // some code to be executed after the operation operation.
}

// Operation의 프로퍼티
operation.completionBlock = {
    // some code to be executed after the operation and executionBlocks
}
```

`DispatchWorkItem`과 비슷하게 Operation을 만들어서 넣거나, 직접 실행시킬 수 있다. `addExecutionBlock(() -> Void)`는 Operation 동작이 끝나고 난 후에 원하는 코드를 실행해준다.

### Operation 실행하기

```swift
operation.start()
```

> [!info] Operation을 실행하면 어떤 스레드에서 실행될까?
> 기본적으로 `start()` 메서드를 통해 직접 실행하면 Operation을 실행한 **현재 스레드에서** Operation을 실행하게 된다. 이 내용은 Operation이 동기냐, 비동기냐에 따라 조금 차이가 있는데, _비동기 Operation_ 의 경우에는 새로운 스레드를 만들어서 Operation을 처리한다. 그리고 Operation을 `OperationQueue`에 넣어서 실행하는 경우에는 Operation을 각각 **새로운 스레드를 만들어** 작업한다.

### Operation의 프로퍼티

```swift
var isCancelled: Bool
var isExecuting: Bool
var isFinished: Bool
var isConcurrent: Bool
var isAsynchronous: Bool
var isReady: Bool
var name: String?
```

### Operation의 상태 변화
![op](https://user-images.githubusercontent.com/73867548/153431140-fe224946-dbe3-4167-8569-0543a653308b.png)
- **`isReady`**: Operation이 실행할 준비를 마치면 isReady 상태가 된다.
- **`isExecuting`**: `Start`가 호출된 후(혹은 OperationQueue에 의해 실행된 후) isExecuting 상태가 된다.
- **`isCancel`**: Operation을 cancel하게 되면 isCancel 상태가 된다.
- **`isFinished`**: cancel하지 않고 동작을 모두 마쳤다면 isFinished 상태가 된다.