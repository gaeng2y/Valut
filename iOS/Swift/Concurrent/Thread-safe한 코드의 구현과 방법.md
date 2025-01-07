### Thread-Safety
- 동시적 처리를 하면서 문제없이 스레드를 안전하게 사용
## T-San
- Thread Sanitizer
- 디버그 시 T-San을 켜서 Race condition이 발생하는 상황을 체크할 수 있다
### 시리얼 큐와 sync
- 비동기를 사용하면 (어떤 객체에) 여러 스레드에서 동시다발적인 접속이 일어날 가능성이 있음
- 