TCA에서 기능이 구축되는 기본 단위는 Reducer 매크로와 Reducer 프로토콜이다. 해당 프로토콜에 대한 적합성은 애플리케이션 기능의 논리와 동작을 나타낸다. 여기에는 작업이 시스템으로 전송될 때 현재 상태를 다음 상태로 발전시키는 방법과 효과가 외부 세계와 통신하고 데이터를 시스템에 다시 공급하는 방법이 포함된다.

그리고 가장 중요한 점은 SwiftUI View에 대한 언급 없이 기능의 **핵심 논리와 동작을 완전히 격리**하여 구축할 수 있으므로 격리된 개발, 재사용 및 테스트가 더 쉬워진다는 것이다.

