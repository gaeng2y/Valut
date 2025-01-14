Clean Architecture는 소프트웨어 설계 원칙을 따르는 아키텍처 패턴으로, 코드의 유지보수성과 확장성을 높이고, 비즈니스 로직을 외부의 의존성(예: 데이터베이스, UI 등)으로부터 독립적으로 유지하는 것을 목표로 합니다. 이 아키텍처 패턴은 2012년에 로버트 C. 마틴(일명 "Uncle Bob")이 제안했습니다.

# 궁극적 목표

1. **Independent of Frameworks**. The architecture does not depend on the existence of some library of feature laden software. This allows you to use such frameworks as tools, rather than having to cram your system into their limited constraints.  
**- 아키텍처는 소프트웨어 라이브러리 존재 여부에 의존하지 않는다.**  
  
2. **Testable**. The business rules can be tested without the UI, Database, Web Server, or any other external element.  
**- 비즈니스 규칙은 UI, 데이터베이스, 웹 서버, 기타 외부 요인없이 테스트가 가능하다.**  
  
3. **Independent of UI**. The UI can change easily, without changing the rest of the system. A Web UI could be replaced with a console UI, for example, without changing the business rules.  
**- 시스템의 나머지 부분을 변경할 필요 없이 UI를 쉽게 변경할 수 있다.**  
  
4. **Independent of Database**. You can swap out Oracle or SQL Server, for Mongo, BigTable, CouchDB, or something else. Your business rules are not bound to the database.  
**- 비즈니스 규칙은 데이터베이스에 얽매이지(바인딩되지) 않는다.**  
  
5. **Independent of any external agency**. In fact your business rules simply don’t know anything at all about the outside world.  
**- 비즈니스 로직은 외부 세계에 대해 전혀 알지 못한다.**

# Clean Architecture의 주요 개념

1. **레이어드 아키텍처**: Clean Architecture는 여러 계층(layer)으로 나뉘며, 각 계층은 특정한 역할을 담당합니다. 가장 바깥쪽의 계층은 인프라와 관련된 부분이고, 안쪽으로 갈수록 핵심 비즈니스 로직에 가까워집니다. 주요 레이어는 다음과 같습니다:
    - **Entity (엔티티)**:
        - 애플리케이션의 핵심 비즈니스 규칙을 포함하는 객체입니다.
        - 이들은 시스템 내의 개념적인 객체들을 정의하고, 비즈니스 로직의 핵심을 형성합니다.
    - **Use Case (유스케이스)**:
        - 애플리케이션의 구체적인 비즈니스 규칙을 포함합니다.
        - 유스케이스는 엔티티를 조작하며, 특정 작업을 수행하기 위한 비즈니스 로직을 담고 있습니다.
    - **Interface Adapters (인터페이스 어댑터)**:
        - 유스케이스와 엔티티를 외부 세계(예: 데이터베이스, UI)와 연결해 주는 역할을 합니다.
        - 이 레이어는 컨트롤러, 프레젠터, 게이트웨이, 뷰 모델 등을 포함할 수 있습니다.
    - **Frameworks & Drivers (프레임워크 및 드라이버)**:
        - 애플리케이션의 외부에 위치한 프레임워크와 도구들을 포함합니다.
        - 이 레이어는 데이터베이스, UI 프레임워크, 웹 서버 등과 같은 것들이 있습니다.
2. **의존성 규칙**:
    - 가장 중요한 원칙 중 하나는 **의존성 규칙(Dependency Rule)**입니다. 이 규칙에 따르면, 코드의 의존성은 항상 안쪽 레이어(가장 핵심적인 비즈니스 로직)로 향해야 합니다. 바깥쪽 레이어는 안쪽 레이어에 대해 알고 있지만, 반대는 성립하지 않습니다.
    - 즉, 엔티티는 유스케이스를 모르고, 유스케이스는 인터페이스 어댑터를 모르며, 인터페이스 어댑터는 프레임워크와 드라이버에 대해 모르도록 구성합니다.
3. **독립성**:
    - **UI 독립성**: 비즈니스 로직은 특정 UI 프레임워크에 의존하지 않습니다. UI를 바꾸더라도 비즈니스 로직은 변하지 않아야 합니다.
    - **데이터베이스 독립성**: 비즈니스 로직은 데이터베이스에 의존하지 않습니다. 데이터베이스를 바꾸더라도 비즈니스 로직은 변하지 않아야 합니다.
    - **외부 에이전시 독립성**: 비즈니스 로직은 외부 시스템이나 도구들에 의존하지 않습니다. 외부 도구나 API가 변경되더라도 비즈니스 로직은 변하지 않아야 합니다.
4. **테스트 가능성**:
    - Clean Architecture는 높은 수준의 테스트 가능성을 제공합니다. 각각의 레이어가 독립적이기 때문에, 유스케이스나 엔티티는 독립적으로 테스트할 수 있습니다.
    - Mocking이나 Stubbing을 통해 외부 의존성을 격리하여 테스트를 진행할 수 있습니다.
# Clean Architecture의 장점

1. **유지보수성**:
    - 각 레이어가 독립적이므로, 변경 사항이 최소한의 부분에서만 발생합니다. 예를 들어, UI나 데이터베이스를 변경해도 비즈니스 로직에는 영향을 미치지 않으므로, 유지보수가 용이합니다.
2. **확장성**:
    - 새로운 기능을 추가할 때, 기존의 코드에 최소한의 영향을 미치도록 설계할 수 있어 확장성이 뛰어납니다.
3. **테스트 용이성**:
    - 핵심 비즈니스 로직이 외부 의존성으로부터 격리되어 있으므로, 쉽게 유닛 테스트를 작성할 수 있습니다.
# 결론

Clean Architecture는 복잡한 애플리케이션의 구조를 깔끔하게 유지하고, 비즈니스 로직을 외부의 변화로부터 보호하며, 코드를 유지보수하기 쉽게 만드는 데 매우 유용합니다. 이 아키텍처는 대규모 애플리케이션 개발에서 특히 효과적이며, 소프트웨어의 장기적인 품질과 안정성을 높이는 데 기여합니다.