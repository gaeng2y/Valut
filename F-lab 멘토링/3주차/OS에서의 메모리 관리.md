메모리라는 용어는 특정 형식의 데이터 모음으로 정의될 수 있다. 그것은 지침을 저장하고 데이터를 처리하는 데 사용된다. 메모리는 큰 배열이나 단어나 바이트 그룹으로 구성되어 있으며, 각각 고유한 위치를 가지고 있다. 컴퓨터 시스템의 주요 목적은 프로그램을 실행하는 것이다. 이 프로그램들은 그들이 접근하는 정보와 함께 실행 중에 [메인 메모리에](https://www.geeksforgeeks.org/computer-memory/) 있어야 한다. [CPU는](https://www.geeksforgeeks.org/cpu-scheduling-in-operating-systems/) 프로그램 카운터의 값에 따라 메모리에서 명령을 가져옵니다.

어느 정도의 [멀티프로그래밍과](https://www.geeksforgeeks.org/difference-between-multitasking-multithreading-and-multiprocessing/) 메모리의 적절한 활용을 달성하기 위해, [메모리 관리는](https://www.geeksforgeeks.org/memory-management-in-operating-system/) 중요하다. 다양한 접근 방식을 반영하는 많은 메모리 관리 방법이 존재하며, 각 알고리즘의 효과는 상황에 따라 다릅니다.

## 메인 메모리가 뭐야?

메인 메모리는 현대 컴퓨터 작동의 중심이다. 메인 메모리는 수십만에서 수십억에 이르는 다양한 단어나 바이트의 배열이다. 메인 메모리는 [CPU와](https://www.geeksforgeeks.org/cpu-scheduling-in-operating-systems/) I/O 장치에서 공유하는 빠르게 사용 가능한 정보의 저장소이다. 메인 메모리는 프로세서가 효과적으로 활용할 때 프로그램과 정보가 보관되는 곳이다. [메인 메모리는](https://www.geeksforgeeks.org/computer-memory/) 프로세서와 관련이 있으므로, 프로세서 안팎으로 명령과 정보를 이동하는 것은 매우 빠릅니다. 메인 메모리는 [RAM(랜덤 액세스 메모리)](https://www.geeksforgeeks.org/random-access-memory-ram/)으로도 알려져 있다. 이 메모리는 휘발성이다. RAM은 정전이 발생하면 데이터를 잃는다.

![메인 메모리](https://media.geeksforgeeks.org/wp-content/uploads/20221116104505/1white-660x453.png)

## 메모리 관리란 무엇인가요?

멀티프로그래밍 컴퓨터에서, [운영 체제는](https://www.geeksforgeeks.org/what-is-an-operating-system/) 메모리의 일부에 상주하고, 나머지는 여러 프로세스에서 사용된다. 다른 프로세스들 사이에서 메모리를 세분화하는 작업을 메모리 관리라고 한다. 메모리 관리는 프로세스 실행 중에 메인 메모리와 디스크 간의 작업을 관리하는 운영 체제의 방법입니다. 메모리 관리의 주요 목표는 메모리의 효율적인 활용을 달성하는 것이다.

## 왜 메모리 관리가 필요한가요?

- 프로세스 실행 전후에 메모리를 할당하고 할당 해제하세요.
- 프로세스별로 사용된 메모리 공간을 추적하기 위해.
- [단편화](https://www.geeksforgeeks.org/difference-between-internal-and-external-fragmentation/) 문제를 최소화하기 위해.
- 메인 메모리의 적절한 활용을 위해.
- 프로세스를 실행하는 동안 데이터 무결성을 유지하기 위해.

이제 우리는 [논리적 주소](https://www.geeksforgeeks.org/logical-and-physical-address-in-operating-system/) 공간과 [물리적 주소 공간의](https://www.geeksforgeeks.org/logical-and-physical-address-in-operating-system/) 개념에 대해 논의하고 있습니다.

## 논리적 및 물리적 주소 공간

- ****논리적 주소 공간:**** CPU에 의해 생성된 주소는 "논리적 주소"로 알려져 있다. 그것은 [가상 주소로도](https://www.geeksforgeeks.org/memory-allocation-techniques-mapping-virtual-addresses-to-physical-addresses/) 알려져 있다. 논리적 주소 공간은 프로세스의 크기로 정의될 수 있다. 논리적인 주소는 변경될 수 있다.
- ****물리적 주소 공간:**** 메모리 유닛에서 볼 수 있는 주소(즉, 메모리의 메모리 주소 레지스터에 로드된 주소)는 일반적으로 "물리적 주소"로 알려져 있다. [실제 주소는](https://www.geeksforgeeks.org/logical-and-physical-address-in-operating-system/) 실제 주소로도 알려져 있다. 이러한 논리적 주소에 해당하는 모든 물리적 주소 집합은 물리적 주소 공간으로 알려져 있다. [물리적 주소는](https://www.geeksforgeeks.org/logical-and-physical-address-in-operating-system/) MMU에 의해 계산된다. 가상 주소에서 물리적 주소로의 런타임 매핑은 하드웨어 장치 메모리 관리 장치(MMU)에 의해 수행됩니다. 물리적 주소는 항상 일정하게 유지된다.

## 정적 및 동적 로딩

메인 메모리에 프로세스를 로드하는 것은 로더에 의해 수행된다. 로딩에는 두 가지 유형이 있습니다:

- ****정적 로딩:**** 정적 로딩은 기본적으로 전체 프로그램을 고정 주소로 로딩하는 것이다. 그것은 더 많은 메모리 공간이 필요하다.
- ****동적 로딩:**** 프로세스가 실행되려면 전체 프로그램과 프로세스의 모든 데이터가 물리적 메모리에 있어야 합니다. 그래서, 프로세스의 크기는 [물리적 메모리의](https://www.geeksforgeeks.org/logical-and-physical-address-in-operating-system/) 크기로 제한된다[.](https://www.geeksforgeeks.org/logical-and-physical-address-in-operating-system/) 적절한 메모리 활용을 위해, 동적 로딩이 사용된다. [동적 로딩에서](https://www.geeksforgeeks.org/difference-between-loading-and-linking/), 루틴은 호출될 때까지 로드되지 않는다. 모든 루틴은 [재배치 가능한](https://www.geeksforgeeks.org/memory-allocation-techniques-mapping-virtual-addresses-to-physical-addresses/) 로드 형식으로 디스크에 있다. 동적 로딩의 장점 중 하나는 사용하지 않는 [루틴이](https://www.geeksforgeeks.org/difference-between-routine-and-process/) 절대 로드되지 않는다는 것이다. 이 로딩은 효율적으로 처리하기 위해 많은 양의 코드가 필요할 때 유용합니다.

## 정적 및 동적 연결

연결 작업을 수행하려면 링커가 사용됩니다. 링커는 컴파일러에 의해 생성된 하나 이상의 개체 파일을 가져와서 단일 실행 파일로 결합하는 프로그램입니다.

- ****정적 연결:**** [정적 연결에서](https://www.geeksforgeeks.org/static-and-dynamic-linking-in-operating-systems/) 링커는 필요한 모든 프로그램 모듈을 단일 실행 프로그램으로 결합합니다. 그래서 런타임 의존성이 없다. 일부 운영 체제는 시스템 언어 라이브러리가 다른 객체 모듈처럼 취급되는 정적 연결만 지원합니다.
- ****동적 연결:**** 동적 연결의 기본 개념은 동적 로딩과 유사하다. [동적 연결에서](https://www.geeksforgeeks.org/static-and-dynamic-linking-in-operating-systems/), "스텁"은 각 적절한 라이브러리 루틴 참조에 포함되어 있습니다. 스텁은 작은 코드 조각이다. 스텁이 실행되면, 필요한 루틴이 이미 메모리에 있는지 여부를 확인합니다. 사용할 수 없다면 프로그램은 루틴을 메모리에 로드합니다.