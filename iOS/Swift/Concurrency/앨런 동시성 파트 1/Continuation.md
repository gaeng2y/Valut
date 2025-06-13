Continuation - 비동기 코드 내에서 일시 중지와 재개 지점을 관리하는 객체 (동기 코드를 비동기 코드로 연결하는 메커니즘)
- 직접 Continuation을 사용해서 1) 콜백방식의 API, 2) 델리게이트 방식의 API를 async/await 방식으로 전환하는 것 가능
- 메서드를 이용 (수동으로) 비동기 일시 중지 지점(suspension point)을 관리하는 객체 생성 가능
  1. CheckedContinuation (2개 메서드)
  2. safeContinuation (2개 메서드)
- Continuation을 생성하고 재개(resume)할때까지 일시 중지된 상태로 유지
  (함수의 실행이 일시중지될 때, 해당 함수의 현재 실행 상태(지역 변수, 실행 위치등)를 힙(Heap)에 저장하게 되는데, Continuation이 (스택에서 실행중인) 콜 스택의 현재 상태를 캡처해서 (힙에 잠시 저장해놓고) 함수(작업)이 재개될때 작업이 중단된 지점부터 다시 실행을 계속할 수 있게 만들어 주는 원리)
- resume메서드로 중단 되었던 작업을 다시 실행(재개)함
#### 기존 델리게이트 메소드를 async 메소드로 변환
**기존의 델리게이트 방식**
예시: 델리게이트 방식으로 위치 업데이트를 받는 메서드 (CoreLocation)

```swift
func IocationManger(...didUpdateLocation location: [CLLocation]) {
    guard let location = locations.last else { return }
    // 데이터 생성시점을 다시 알릴 필요 (처리할 필요)
    self.currentLocation = location
}
```

**Swift Concurrency - 비동기(async) 함수 형태로**
예시: 메서드를 만들어, 비동기적으로 생기는 값을 await 기다렸다가 받아서 활용 가능
전환 - Continuation 생성 메서드로 결과값 활용 가능

```swift
func getCurrentLocation( ) async throws {
    let location = await withCheckedContinuation { continuation in
        // 컨티뉴에이션을 외부 변수 등에 저장 활용 가능
        self.continuation = continuation
    }
    // location 활용 코드...
}

func IocationManger(...didUpdateLocation location: [CLLocation]) { 
    guard let location = locations.last else { return }
    // ...
    continuation .resume(returning: location)
    continuation = nil
}
```