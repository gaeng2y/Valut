#### 1) 메인큐
- 우리가 흔히 알고 있는 UI를 담당하는 큐(직렬)
#### 2) 글로벌큐와 QoS의 이해
- [**DispatchQoS**](https://developer.apple.com/documentation/dispatch/dispatchqos)에서 우선순위를 정할 수 있다.
- QoS가 높을수록 더 여러개의 쓰레드를 배치
- 큐의 QoS를 지정해놓고 async 메소드에서 QoS를 변경 시 변경된 QoS로 적용
#### 3) 프라이빗(커스텀) 큐
- 커스텀 큐는 직렬과 동시 중 정할 수 있다.