우선 순위가 높은 태스크가 READY상태 (실행 가능)로 바뀌었지만 더 낮은 우선순위의 태스크가 CPU를 점유하고 있어서 실행되지 못하는 상태

**I. 실시간 스케줄링의 문제점 Priority Inversion(우선순위 역전) 개요**  
    가.  Priority Inversion 의 개념 

- 우선 순위가 높은 태스크가 READY상태 (실행 가능)로 바뀌었지만 더 낮은 우선순위의 태스크가 CPU를 점유하고 있어서 실행되지 못하는 상태

    나.  Priority Inversion의 문제점

- RTOS 환경하에서 중요한 태스크의 수행이 요구된 시간에 끝내지 못함
- 중요한 태스크의 무한정 대기로 인한 전체 시스템의 이상현상 발생 가능함

    다.  Priority Inversion 의 원인

- 스케줄링과 동기화 사이의 상호작용 결과로 발생
- 스케줄링 규칙에서 실행되어야 하는 쓰레드와 동기화에서 실행되어야 하는 쓰레드가 서로 다른 경우, 결과적으로 두 쓰레드의 우선순위가 역전되어 나타남

        1) 비선점 스케쥴링에서 낮은 우선순위의 태스크가 자원을 점유  
        2) 실시간 운영체제에서의 상호배제를 위한 세마포어를 이용  
        3) 공유자원의 장기 소유  
        4) 자원점유후 릴리즈시 낮은 우선순위의 태스크가 자원을 점유하는 경우

**II. 사례 기반 Priority Inversion 설명**  
    가.  Priority Inversion Flow
    ![](Pasted%20image%2020240606234452.png)
    - task1,2,3이 각각 high, medium, low priority를 가진다고 가정 했을 때 Priority Inversion 발생 사례

  나. Priority Inversion 발생 설명

| **순서** | **우선순위역전 발생 상태**                                                                                                   |
| ------ | ------------------------------------------------------------------------------------------------------------------ |
| 1      | task3이 공유자원을 액세스 위해 binary semaphore를 가지고 수행                                                                       |
| 2      | Scheduler에 의해 task1이 수행                                                                                            |
| 3      | task1은 task3가 먼저 획득한 Semaphore를 얻으려(take)하고, task3가 그 Semaphore를 반환할 때(give)까지 Waiting 상태로 됨                       |
| 4      | Scheduler에 의해 task3 수행                                                                                             |
| 5      | Scheduler에 의해 task2가 수행<br><br>이 시점에서 task1의 priority가 task2보다 높음에도 불구하고 task2가 먼저 수행 되는 Priority Inversion 문제가 발생 |
| 6      | task2의 수행이 종료 되면, task3가 재 수행                                                                                      |
| 7      | task3가 Semaphore 반환                                                                                                |
| 8      | task1 수행                                                                                                           |