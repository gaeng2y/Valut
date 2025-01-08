- `lazy var`에서 일반적으로 사용하면 Thread-safe하지 않다
#### 해결 방법
1. serial 큐로 해결
2. Dispatch Barrier 사용
3. Semaphore도 사용 가능