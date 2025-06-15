# Cancel

```swift
operantion.cancel()
```

취소하는 **메소드**. 하지만 실행 중인 것을 직접적으로 취소시켜주지는 않는다.

# isAsynchronous

Operation이 Asynchronous로 동작할 것인가, Synchronous로 동작할 것인가에 대한 프로퍼티다. Operation에서는 어떻게 동작하게 될까?

- `start()` 메소드를 호출해서 실행할 경우: Operation이 동기일 때는 현재 스레드에서 작업을 처리, 비동기일 때는 새로운 스레드 만들어서 작업
- `OperationQueue`를 사용할 때: 새로운 스레드를 만들어 작업을 처리

기본값은 `false`이다. 즉 아무런 설정을 하지 않으면 동기로 동작한다. 비동기(asynchronous)로 동작하는 Operation이 필요하다면 Custom하여 Operation을 만들어주어야한다.

# Dependency

종속성은 Operation 인스턴스들 사이에 종속적인 관계를 만들어서 실행의 순서를 관리하도록 해준다. 기본적으로 Operation은 직접 실행한다면 실행한 순서대로, OperationQueue에 넣는다면 넣은 순서대로 작업을 처리한다. `addDependency(_:)` 메서드로 종속적인 관계를 만들어주면 해당 Operation은 **자신이 종속된 Operation의 작업이 끝날 때까지** 실행할 수 없게 된다.