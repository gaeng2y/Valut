ref: https://medium.com/@sbenitez73/understading-clean-architecture-in-ios-74eda18fc1f4

# Overview

소프트웨어 프로젝트에서 모듈성, 유지 관리성 및 새로운 접근 방식을 도입하는 것이 중요하다. 개발자와 아키텍트로서 우리의 책임은 솔루션을 최대한 확장 가능하게 만드는 것이다. 여기서 **클린 아키텍처**가 필요하다.
# Benefits
## Separation of Concerns (SoC)

소프트웨어 솔루션을 레이어로 나누어 각 레이어가 특정 책임을 지게 한다. 예를 들어, UI와 인프라를 분리한다.
## Framework Independency

책임이 분리되면 비즈니스 규칙을 정의하는 데 집중할 수 있다. 인프라 세부 사항에 신경 쓰지 않고 SQL Server에서 MongoDB로, 또는 Alamofire에서 URLSession으로 쉽게 전환할 수 있다.
## Testability

레이어를 분리하면 잘 정의된 인터페이스(프로토콜)를 통해 자동화된 테스트를 작성할 수 있다.
# Dependencies by Layers
![cleanArchitecture](https://miro.medium.com/v2/resize:fit:720/format:webp/0*YYcFutagHH6qBj-F)
## Domain Layer
## 