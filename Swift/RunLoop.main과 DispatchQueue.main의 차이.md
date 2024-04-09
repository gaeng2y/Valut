Combine에서 Scheduler들은 `Scheduler` 프로토콜을 따르고 있고, RunLoop.main과 DispatchQueue.main 또한 따르고 있다.

흔하게 `receive(on:)` 에서 다운스트림을 설정할 때 사용된다.

## RunLoop.main
RunLoop은 Application의 touch와 같은 input source들을 다루는 object의 인터페이스이다.

RunLoop은 각 쓰레드의 object를 위해 RunLoop object를 생성하는 시스템에 의해 생성되고 관리된다.

메인 스레드를 나타내는 Main Run Loop도 생성한다.
## DispatchQueue.main
DispatchQueue.main은 현재 프로세스의 메인스레드와 관련이 있는 dispatch Queue이다.

DispatchQueue는 관련된 스레드에서 직렬 혹은 병렬로 작업을 실행한다.
## 둘의 차이점
가장 큰 차이점은 DispatchQueue는 즉시 실행되는 것이고 반면에 RunLoop은 busy할지도 모른다.

만약 `DispatchQueue.main`을 스케줄러로 사용할 때는, 스크롤하면서 동시에 UI가 업데이트 된다.

반면에 `RunLoop.main`은 스크롤이 마친 이후에 UI가 업데이트 된다.

즉, main run loop에 의해 scheduled된 클로저는 유저 Interation이 발생하면 즉시 실행되지 않고 끝나야 실행된다.