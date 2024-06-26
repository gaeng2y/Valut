스레드는 **프로세스 내의 단일 시퀀스 스트림**이다. 스레드는 프로세스의 일부 특성을 가지고 있기 때문에 경량 프로세스라고도 불린다. 각 스레드는 정확히 하나의 과정에 속한다. 멀티스레딩을 지원하는 운영 체제에서, 그 과정은 많은 스레드로 구성될 수 있다. 하지만 스레드는 CPU가 1개 이상인 경우에만 효과적일 수 있습니다. 그렇지 않으면 **두 개의 스레드가 단일 CPU에 대해 컨텍스트 스위치를 해야 합니다**.

## 운영 체제에서 스레드는 무엇인가요?

프로세스에서 스레드는 실행되는 단일 순차적 활동을 나타냅니다. 이러한 활동은 실행 스레드 또는 스레드 제어로도 알려져 있습니다. 이제 모든 운영 체제 프로세스는 스레드를 실행할 수 있습니다. 우리는 프로세스가 여러 개의 스레드를 가질 수 있다고 말할 수 있습니다.

## 왜 우리는 스레드가 필요한가요?

- 스레드는 애플리케이션 성능을 개선하는 병렬로 실행됩니다. 그러한 각 스레드는 자체 CPU 상태와 스택을 가지고 있지만, 프로세스와 환경의 주소 공간을 공유한다.
- 스레드는 공통 데이터를 공유할 수 있으므로 [프로세스 간 통신을](https://www.geeksforgeeks.org/inter-process-communication-ipc/) 사용할 필요가 없습니다. 프로세스와 마찬가지로, 스레드는 준비, 실행, 차단 등과 같은 상태를 가지고 있다.
- 프로세스와 마찬가지로 스레드에 우선 순위를 할당할 수 있으며, 우선 순위가 높은 스레드가 먼저 예약됩니다.
- 각 스레드에는 자체 [스레드 제어 블록(TCB)](https://www.geeksforgeeks.org/thread-control-block-in-operating-system/)이 있습니다. 프로세스와 마찬가지로, 스레드에 대한 컨텍스트 스위치가 발생하고, 등록 내용은 (TCB)에 저장됩니다. 스레드는 동일한 주소 공간과 리소스를 공유하기 때문에, 스레드의 다양한 활동에도 동기화가 필요합니다.

## 스레드의 구성 요소

이것들은 운영 체제의 기본 구성 요소이다.

- 스택 공간
- 등록 세트
- 프로그램 카운터

## ****운영 체제의 스레드 유형****

실은 두 가지 유형이다. 이것들은 아래에 설명되어 있다.

- 사용자 레벨 스레드
- 커널 레벨 스레드

![image1](https://media.geeksforgeeks.org/wp-content/uploads/20240226115304/Threads.png)

### 1. 사용자 레벨 스레드

사용자 레벨 스레드는 시스템 호출을 사용하여 생성되지 않은 스레드 유형입니다. 커널은 사용자 수준의 스레드를 관리하는 데 아무런 효과가 없다. 사용자 수준의 스레드는 사용자가 쉽게 구현할 수 있다. 사용자 수준의 스레드가 한 손으로 처리되는 경우, 커널 수준의 스레드가 이를 관리합니다. 사용자 수준 스레드의 장점과 단점을 살펴봅시다.

****사용자 수준 스레드의 장점****

- 사용자 레벨 스레드의 구현은 커널 레벨 스레드보다 쉽다.
- [컨텍스트 전환](https://www.geeksforgeeks.org/context-switch-in-operating-system/) 시간은 사용자 수준 스레드에서 적다.
- 사용자 레벨 스레드는 커널 레벨 스레드보다 더 효율적이다.
- 프로그램 카운터, 레지스터 세트 및 스택 공간만 있기 때문에 간단한 표현을 가지고 있다.

****사용자 수준 스레드의 단점****

- 스레드와 커널 사이에는 조정이 부족하다.
- 페이지 오류의 경우, 전체 프로세스가 차단될 수 있습니다.

### 2. 커널 레벨 스레드

[커널 레벨 스레드는](https://www.geeksforgeeks.org/kernel-level-threads-in-operating-system/) 운영 체제를 쉽게 인식할 수 있는 스레드 유형이다. 커널 레벨 스레드에는 시스템을 추적하는 자체 스레드 테이블이 있습니다. 운영 체제 커널은 스레드를 관리하는 데 도움을 준다. 커널 스레드는 어떻게든 더 긴 컨텍스트 전환 시간을 가지고 있다. 커널은 스레드 관리에 도움을 준다.

****커널 레벨 스레드의 장점****

- 그것은 모든 스레드에 대한 최신 정보를 가지고 있다.
- 주파수를 차단하는 응용 프로그램은 커널 레벨 스레드에 의해 처리되어야 한다.
- 프로세스가 처리하는 데 더 많은 시간이 필요할 때마다, 커널 레벨 스레드는 더 많은 시간을 제공한다.

****커널 레벨 스레드의 단점****

- 커널 레벨 스레드는 사용자 레벨 스레드보다 느립니다.
- 이러한 유형의 스레드의 구현은 사용자 수준의 스레드보다 조금 더 복잡하다.

자세한 내용은 [사용자 수준 스레드와 커널 수준 스레드의 차이점을](https://www.geeksforgeeks.org/difference-between-user-level-thread-and-kernel-level-thread/) 참조하십시오.