Instruments는 Xcode Tool의 일부로 강력하고 유연한 performance 분석과 테스트할 수 있는 툴입니다.

시간에 따라 앱과 프로세스 등 프로파일링에 따라 데이터를 수집하고 분석하여 자세한 결과를 제공해줍니다.

Instruments의 기능제공은 다음과 같습니다.

- 하나 또는 그 이상의 앱 프로세스 동작 검사
- Wi-Fi 및 Bluetooth와 같은 장치별 기능 검사
- 시뮬레이터 또는 물리적 장치에서 프로파일링 수행
- 소스 코드의 문제 추적
- 앱에서 성능 분석 수행
- leaks과 같은 메모리 문제를 앱에서 찾습니다.
- 더 높은 전력 효율성을 위해 애플리케이션을 최적화하는 방법 확인
- 일반적인 시스템 수준 문제 해결 수행
- 기기 구성을 템플릿으로 저장

- **Blank**  
    사용자 정의할 수 있는 빈 문서
- **Activity Monitor**  
    시스템과 프로세스에 대한 CPU, Memory, disk, network 사용량 통계
- **Allocations**  
    virtual memory 와 heap을 추적하고  
    객체에 대한 retain/release 기록 제공(선택적)
- **Animation Hitches**  
    scrolling 및 animation hitches를 측정하고 감지  
    graphic pipeline을 시각화하고 조사합니다.
- **App Launch**  
    스레드 상태를 추적하여 애플리케이션 성능 조정
- **Core Data**  
    Core Data fileSystem을 추적합니다
- **Counters**  
    시간 또는 sampling methods 기반으로 performance monitor counter (PMC)를 수집합니다.
- **Energy Log**  
    device 주요 구성요소들의 상태를 끄고 켤 수 있을 뿐만 아니라 에너지 사용에 관한 진단 제공
- **File Activity**  
    fileSystem 및 disk I/O를 기록하고 분석합니다.
- **Game performance**  
    게임 performance 와 스무스한 프레임 성능 이해
- **Leaks**  
    allocations 및 leaked blocks에 대한 메모리 주소 기록 통계 제공
- **Logging**  
    통합 로깅 시스템의 로그 및 표지판(?) 시각화.
- **Metal System Trace**  
    애플리케이션, driver, GPU 및 display에서 추적 정보를 제공
- **Network**  
    TCP/IP and UDP/IP 분석
- **SceneKit**  
    SceneKit을 사용하는 Application
- **SwiftUI**  
    SwiftUI. 시간에 따른 property 업데이트 및 slow frame 식별
- **System Trace**  
    운영체제에서 무슨일이 일어나는지 볼 수 있음.  
    스레드 및 virtual memory를 통해 앱 성능 파악
- **Time Profiler**  
    시간 기반 sampling 수행
- **Zombies**  
    일반적인 메모리 사용량을 측정