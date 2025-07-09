핵심은 **Tuist를 사용해 모듈 간의 의존성을 관리**하고, **각 Feature 모듈 내부를 클린 아키텍처와 MVVM 패턴으로 설계**하는 것입니다.

-----

### \#\# 전체 구조 요약

전체적인 그림은 다음과 같습니다.

  * **Tuist Layer (App, Feature, Domain, Core, Shared):** 프로젝트 전체를 거대한 레고 블록처럼 나누는 **외부 골격**입니다. 모듈 간의 의존성 방향을 강제하여 아키텍처를 깔끔하게 유지합니다.
  * **Clean Architecture + MVVM:** 각 `Feature` 레고 블록 **내부를 어떻게 설계할지에 대한 상세 설계도**입니다. 사용자 인터랙션부터 데이터 처리까지의 흐름을 정의합니다.

\<br\>

-----

### \#\# 1. Tuist 레이어별 역할 및 책임

Tuist에서 제안하는 각 레이어는 명확한 역할을 가집니다. 의존성은 항상 바깥 레이어에서 안쪽 레이어로 향해야 합니다. (**App → Feature → Domain/Core → Shared**)

#### 🏢 **App**

  * **역할:** 최종 사용자에게 제공되는 실제 애플리케이션 그 자체입니다.
  * **책임:**
      * `AppDelegate`, `SceneDelegate` 등 앱의 진입점(Entry Point) 및 생명주기 관리.
      * 필요한 Feature 모듈들을 조합하고 초기 화면을 설정.
      * 푸시 알림, 딥링크 처리 등 앱 전반에 걸친 설정.
  * **의존성:** `Feature` 레이어에만 의존합니다.
  * **특징:** 자체적으로 비즈니스 로직이나 UI를 거의 갖지 않는 **'조립 공장'** 역할을 합니다.

#### ✨ **Feature (µFeatures)**

  * **역할:** 사용자가 직접 상호작용하는 기능 단위입니다. `µFeature` (마이크로피처) 개념은 이 Feature를 '로그인', '홈', '프로필 수정'처럼 매우 작고 독립적인 단위로 나누는 것을 의미합니다.
  * **책임:**
      * **클린 아키텍처 + MVVM이 적용되는 주 무대**입니다.
      * 화면(View), 화면의 상태와 로직(ViewModel), 그리고 화면으로의 네비게이션을 관리합니다.
      * 필요한 비즈니스 로직을 실행하기 위해 `Domain` 레이어의 UseCase를 호출합니다.
  * **의존성:** `Domain`, `Core`, `Shared` 레이어에 의존합니다. **다른 `Feature`에 직접 의존하지 않는 것이 중요**합니다. (Feature 간 통신은 `Coordinator`나 `URL-based` 라우팅 등을 통해 `App` 레이어에서 조율합니다.)
  * **Tip:** `LoginFeature`, `HomeFeature`, `SearchFeature`, `ProfileFeature` 등으로 나눌 수 있습니다.

#### 🧠 **Domain**

  * **역할:** 앱의 핵심 비즈니스 로직을 담고 있는, 프레임워크에 독립적인 순수한 영역입니다.
  * **책임:**
      * **Entities:** 비즈니스 모델 객체 (e.g., `User`, `Product`).
      * **UseCases (Interactors):** 특정 비즈니스 로C직 수행 (e.g., `LoginUseCase`, `FetchProductListUseCase`).
      * **Repository Interfaces (Protocols):** 데이터 저장소에 대한 **규칙(명세)** 정의 (e.g., `UserRepository`). 실제 구현은 `Core` 레이어에 있습니다.
  * **의존성:** `Shared` 레이어에만 의존할 수 있으며, 가급적 아무것에도 의존하지 않는 것을 목표로 합니다.
  * **특징:** UI, 데이터베이스, 네트워크 등 외부 요소를 전혀 몰라야 합니다. 순수한 Swift 코드로만 작성됩니다.

#### ⚙️ **Core**

  * **역할:** `Domain`의 Repository Interface를 **실제로 구현**하고, 여러 Feature에서 공통으로 사용되는 핵심 기능들을 제공합니다.
  * **책임:**
      * **Repository Implementations:** `Domain`의 `UserRepository` 프로토콜을 준수하는 `DefaultUserRepository` 구현체. 이 구현체는 실제 네트워크 요청(e.g., `Alamofire`)이나 DB 접근(`CoreData`) 로직을 포함합니다.
      * **Networking:** 네트워크 통신 모듈.
      * **Data Storage:** 데이터베이스 관리 모듈.
  * **의존성:** `Domain`, `Shared` 레이어에 의존합니다.

#### 🎨 **Shared**

  * **역할:** 모든 레이어에서 공통으로 사용될 수 있는, 가장 의존성이 없는 공유 자원을 모아놓은 곳입니다.
  * **책임:**
      * **DesignSystem:** `UIColor`, `UIFont`, 공용 `UIView` 컴포넌트(e.g., `PaddingLabel`, `RoundedButton`).
      * **Extensions:** `String`, `Date` 등 Foundation 타입 확장.
      * **Utilities:** 공통 유틸리티 함수.
  * **의존성:** **아무것에도 의존하지 않습니다 (No Dependencies).**

-----

### \#\# 2. Feature 모듈 내부에 클린 아키텍처 + MVVM 적용하기

이제 가장 중요한 `Feature` 모듈 내부를 들여다보겠습니다. 예를 들어, `LoginFeature`를 만든다고 가정해 봅시다.

`LoginFeature` 모듈의 내부 폴더 구조는 다음과 같이 될 수 있습니다.

```
LoginFeature/
├── Presentation/
│   ├── View/
│   │   └── LoginViewController.swift
│   └── ViewModel/
│       └── LoginViewModel.swift
└── Coordinator/
    └── LoginCoordinator.swift
```

이 구조가 클린 아키텍처 및 다른 레이어와 상호작용하는 방식은 다음과 같습니다.

1.  **View (in `Presentation/View`)**

      * `LoginViewController`는 UIKit 또는 SwiftUI View입니다.
      * 사용자 입력을 받고, 화면을 그리는 역할만 하는 \*\*"Dumb View"\*\*를 지향합니다.
      * `LoginViewModel`을 소유하고, ViewModel의 상태 변화를 감지하여 UI를 업데이트합니다 (Data Binding).
      * 사용자 액션(버튼 클릭 등)이 발생하면 `ViewModel`의 함수를 호출합니다.

2.  **ViewModel (in `Presentation/ViewModel`)**

      * `LoginViewModel`은 `MVVM`의 ViewModel이자 클린 아키텍처의 **Presenter** 역할을 합니다.
      * View에 필요한 상태(e.g., `email`, `password`, `isLoading`, `errorMessage`)를 `Published` 프로퍼티 등으로 외부에 노출합니다.
      * View로부터 액션이 들어오면, **`Domain` 레이어의 `LoginUseCase`를 실행**시킵니다.
      * `LoginUseCase`의 실행 결과를 받아 View가 사용할 수 있는 상태로 가공합니다.
      * \*\*의존성 주입(DI)\*\*을 통해 `LoginUseCase`를 주입받습니다.

3.  **UseCase (in `Domain` Layer)**

      * `LoginUseCase`는 '로그인'이라는 단일 비즈니스 로직을 책임집니다.
      * `execute`와 같은 단일 메소드를 가지며, 이 메소드는 **`Core` 레이어에 구현된 Repository에 접근하기 위한 `Repository Interface`를 호출**합니다.
      * 예: `userRepository.login(email:password:)`를 호출하고 성공/실패 결과를 `ViewModel`에 반환합니다.
      * 생성자를 통해 `UserRepository` 프로토콜을 주입받습니다.

4.  **Repository (Interface in `Domain`, Implementation in `Core`)**

      * `Domain` 레이어에는 `protocol UserRepository { ... }`가 정의되어 있습니다.
      * `Core` 레이어에는 `class DefaultUserRepository: UserRepository { ... }`가 실제 네트워크 통신 로직을 포함하여 구현되어 있습니다.
      * 이러한 **의존성 역전 원칙(DIP)** 덕분에 `Domain`은 `Core`를 전혀 모르면서도 데이터 작업을 시킬 수 있습니다.

5.  **Entity (in `Domain` Layer)**

      * `User`와 같은 순수한 데이터 모델입니다. 앱의 핵심 비즈니스 데이터를 표현합니다.

### \#\# 3. Tuist `Project.swift` 설정 예시

이러한 구조를 Tuist로 설정하면 다음과 같은 의존성 관계가 `Project.swift` 파일에 명시됩니다.

**`Projects/Features/LoginFeature/Project.swift`**

```swift
import ProjectDescription

let project = Project(
    name: "LoginFeature",
    targets: [
        .target(
            name: "LoginFeature",
            destinations: .iOS,
            product: .staticFramework,
            bundleId: "com.myApp.LoginFeature",
            sources: ["Sources/**"],
            dependencies: [
                .project(target: "Domain", path: .relativeToRoot("Projects/Domain")),
                .project(target: "Shared", path: .relativeToRoot("Projects/Shared"))
            ]
        )
    ]
)
```

**`Projects/App/Project.swift`**

```swift
import ProjectDescription

let project = Project(
    name: "MyApp",
    targets: [
        .target(
            name: "MyApp",
            destinations: .iOS,
            product: .app,
            bundleId: "com.myApp",
            sources: ["Sources/**"],
            resources: ["Resources/**"],
            dependencies: [
                .project(target: "LoginFeature", path: .relativeToRoot("Projects/Features/LoginFeature")),
                .project(target: "HomeFeature", path: .relativeToRoot("Projects/Features/HomeFeature")),
                .project(target: "Core", path: .relativeToRoot("Projects/Core")) // DI Container 설정 등을 위해 App에서 Core를 알 수 있음
            ]
        )
    ]
)
```

-----

### \#\# 결론 🚀

정리하자면, **Tuist의 레이어 구조**로 모듈 간의 의존성을 명확히 하여 **빌드 속도 향상, 코드 재사용성 증대, 팀원 간의 작업 분리**라는 이점을 얻습니다. 그리고 각 **Feature 모듈 내부**는 **클린 아키텍처+MVVM**을 적용하여 **테스트 용이성, 유지보수성, 유연성**을 극대화할 수 있습니다.

이 구조는 초기 설정에 노력이 필요하지만, 프로젝트가 커질수록 그 진가를 발휘하는 매우 견고하고 현대적인 아키텍처입니다.
