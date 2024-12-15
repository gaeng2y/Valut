# Tuist란?

Tuist는 iOS 및 macOS 애플리케이션 개발을 위한 도구로, Xcode 프로젝트의 생성을 자동화하고 관리하는 데 사용된다. 개발자들은 Tuist를 사용하여 복잡한 프로젝트 구조를 간단하고 효율적으로 관리할 수 있다.

Tuist의 주요 기능 중 하나는 Swift 언어로 정의된 매니페스트 파일 (`Project.swift`)을 사용해 프로젝트 설정을 선언적으로 작성할 수 있다는 점이다. 이를 통해 여러 타겟(target), 설정(configuration), 종속성(dependency) 등을 손쉽게 정의할 수 있으며, 수동으로 Xcode에서 설정하는 번거로움을 줄일 수 있다.

또한 Tuist는 프로젝트 간의 모듈화를 쉽게 하고, 팀 협업 시 발생할 수 있는 설정 충돌 문제를 줄여준다. Xcode 프로젝트를 안정적이고 일관되게 관리할 수 있게 해주며, 프로젝트 생성, 빌드 속도 최적화, 종속성 관리 등의 다양한 기능을 제공한다.

결국, Tuist는 큰 규모의 iOS/macOS 프로젝트를 체계적으로 관리하고자 하는 개발자들에게 매우 유용한 도구다.

Tuist 설치는 관련 내용은 [여기](https://docs.tuist.dev/en/guides/quick-start/install-tuist)에서 확인해주세요

### Tuist를 이용한 프로젝트 생성
```bash
tuist init --platform ios
```

그 후 Project.swift 파일을 수정하여 작업 혹은 Workspace.swift 파일을 만들어 작업 가능.
# Manifest

Tuist의 `Manifest`는 Xcode 프로젝트를 정의하고 관리하기 위한 설정 파일입니다. Tuist는 Swift를 기반으로 하는 프로젝트 관리 도구로, 복잡한 Xcode 프로젝트와 워크스페이스 설정을 간단하게 구성하고 유지보수할 수 있도록 도와줍니다. `Manifest` 파일은 프로젝트 설정을 코드로 작성하며, 일반적으로 Swift 파일로 작성됩니다.
# Project

- 특정 프로젝트를 정의하는 파일입니다. 프로젝트의 대상(Targets), 설정(Configuration), 종속성(Dependencies) 등을 Swift 코드로 정의합니다.
- 이 파일에는 주로 다음과 같은 정보가 포함됩니다:
    - 프로젝트 이름
    - 타겟 설정 (ex: 앱 타겟, 테스트 타겟)
    - 소스 파일 위치
    - 빌드 설정
    - 종속성 (라이브러리 및 프레임워크 등)
예시
```swift
import ProjectDescription

let project = Project(
    name: "MyApp",
    targets: [
        Target(
            name: "MyApp",
            platform: .iOS,
            product: .app,
            bundleId: "com.myapp",
            infoPlist: "Config/Info.plist",
            sources: ["Sources/**"],
            resources: ["Resources/**"],
            dependencies: []
        )
    ]
)
```
# Workspace

- 여러 프로젝트를 포함한 Xcode 워크스페이스를 정의하는 파일입니다. 복수의 프로젝트를 한 곳에서 관리할 때 사용됩니다.
- 워크스페이스 파일은 주로 다음을 포함합니다:
    - 워크스페이스에 포함될 프로젝트 리스트
    - 다양한 플러그인 및 스킴 설정
예시
```swift
import ProjectDescription

let workspace = Workspace(
    name: "MyWorkspace",
    projects: [
        "Projects/App",
        "Projects/Framework",
        "Projects/Shared"
    ]
)
```

