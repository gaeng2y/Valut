## Dispatch와 Operation의 차이
##### Dispatch
- 간단한 작업
- 함수를 사용하는 작업
##### Operation
- 복잡한 작업
- 데이터와 기능을 캡슐화한 객체
- **취소 / 순서지정 / 일시중지 (상태추적)**
## Operation
- 어떤 단위적인 작업을 클래스화해서 기능을 UP
- 예) 데이터 다운로드, 압축 풀기, 이미지 필터 적용 등등.. (재사용성)
- 취소, 순서지정, 상태 체크(state machine), KVO, QoS, completion 제공
![](iOS/Swift/Concurrent/Resources/Pasted%20image%2020250119213754.png)
- Operation을 상속받아 `main()`에서 로직 작성
