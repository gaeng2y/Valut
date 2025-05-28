세 메소드 모두 구독하는 메소드다.
## subscribe
onNext, onError, onCompleted, onDisposed 파라미터가 모두 있다.

```swift
public func subscribe(
        onNext: ((Element) -> Void)? = nil,
        onError: ((Swift.Error) -> Void)? = nil,
        onCompleted: (() -> Void)? = nil,
        onDisposed: (() -> Void)? = nil
    ) -> Disposable {
            let disposable: Disposable
            
            if let disposed = onDisposed {
                disposable = Disposables.create(with: disposed)
            }
            else {
                disposable = Disposables.create()
            }
            
            #if DEBUG
                let synchronizationTracker = SynchronizationTracker()
            #endif
            
            let callStack = Hooks.recordCallStackOnError ? Hooks.customCaptureSubscriptionCallstack() : []
            
            let observer = AnonymousObserver<Element> { event in
                
                #if DEBUG
                    synchronizationTracker.register(synchronizationErrorMessage: .default)
                    defer { synchronizationTracker.unregister() }
                #endif
                
                switch event {
                case .next(let value):
                    onNext?(value)
                case .error(let error):
                    if let onError = onError {
                        onError(error)
                    }
                    else {
                        Hooks.defaultErrorHandler(callStack, error)
                    }
                    disposable.dispose()
                case .completed:
                    onCompleted?()
                    disposable.dispose()
                }
            }
            return Disposables.create(
                self.asObservable().subscribe(observer),
                disposable
            )
    }
```

즉, subscribe를 쓰면 이벤트, 에러, 완료, 구독 해제에 대한 처리를 다 할 수 있다.

## bind
onNext만 온다.
즉, error나 completed가 발생하지 않는 스트림에서 구독할 때 사용하면 된다. (ui event 같은)

```swift
public func bind<Object: AnyObject>(
        with object: Object,
        onNext: @escaping (Object, Element) -> Void
    ) -> Disposable {
        self.subscribe(onNext: { [weak object] in
            guard let object = object else { return }
            onNext(object, $0)
        },
        onError: { error in
            rxFatalErrorInDebug("Binding error: \(error)")
        })
    }
```

bind는 메인 스케줄러에서 동작하지 않지만 complete, disposed에 대한 이벤트를 처리하지 않으며, error처리는 릴리즈에서는 로그 출력만 한다.

## drive
Driver를 구독할 때 사용한다.
bind와 비슷하지만 메인스케줄러, 그리고 share 같은 기능을 사용할 때 사용한다.

```swift
public func drive(
        onNext: ((Element) -> Void)? = nil,
        onCompleted: (() -> Void)? = nil,
        onDisposed: (() -> Void)? = nil
    ) -> Disposable {
        MainScheduler.ensureRunningOnMainThread(errorMessage: errorMessage)
        return self.asObservable().subscribe(onNext: onNext, onCompleted: onCompleted, onDisposed: onDisposed)
    }
```

drive 메소드는 `MainScheduler.ensureRunningOnMainThread(errorMessage:)` 를 통해 메인 스케줄러를 보장하며, subscribe 메소드를 통해 next, completed, disposed 이벤트를 처리할 수 있다

| 메소드         | 대상         | 처리 가능한 이벤트                       | 스레드 보장            | 주 사용처                   |
| ----------- | ---------- | -------------------------------- | ----------------- | ----------------------- |
| `subscribe` | Observable | next, error, completed, disposed | ❌ (사용자 지정)        | 전반적인 스트림 처리             |
| `bind`      | Observable | next                             | ❌ (직접 지정 필요)      | ViewModel → View 바인딩    |
| `drive`     | Driver     | next, completed, disposed        | ✅ (MainScheduler) | UI 바인딩 (안정성 + 메인스레드 보장) |
