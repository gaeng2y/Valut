# Tech sepc

- Swift
- SwiftUI
- TCA
- Tuist
- PokemonAPI
# Clean Architecture는 어디감?

TCA 자체가 아키텍처라 클린 아키텍처랑 같이 못함... (어쩐지 자료가 없더라)

# 그러면 모듈화 어떻게 할거?

해당 [프로젝트](https://github.com/devyhan/AppStroreSearch)를 참고했다. 

![diagram](https://github.com/devyhan/AppStroreSearch/raw/main/GithubResources/Architecture.png)

- App - App의 시작점인 `@main`및 App타겟
- Feature - Search, SearchInterface등 도메인에 해당하는 타겟이 모여있는 계층. 피처 모델은 App타겟을 가지고 있어 개별 빌드가 가능
- Core - Networking, Repository와 같이 외부의 영역과 소통하는 타겟이 모여있는 계층
- UserInterface - Components처럼 UI구성에 도움을 주는 타겟이 모여있는 계층
- Share - 프로젝트의 가장 로우레벨(LowLevel)에 해당하는 계층으로 `Logger`, `Extension`등 프로젝트 유틸리티성을 가진것들이 모여있는 계층