## 전체 아키텍처 개요

Swift와 Tuist를 활용하여 세 가지 아키텍처 패턴을 통합합니다:

- **MVVM**: UI 바인딩과 프레젠테이션 로직 분리 (Combine 활용)
- **Clean Architecture**: 비즈니스 로직과 외부 의존성 분리
- **Modular Architecture**: 기능별 모듈 분리
- **µFeature**: 독립적인 마이크로 모듈 구성

## 모듈 구조 설계

### 1. Core 모듈 (공통 기능)
```
Core/
├── CoreCommon/          # 공통 유틸리티, 확장
├── CoreNetwork/         # 네트워킹 설정
├── CoreDatabase/        # Core Data 설정
├── CoreUI/             # 공통 UI 컴포넌트
├── CoreDomain/         # 공통 도메인 모델
└── CoreNavigation/     # 네비게이션 인터페이스
```

### 2. Feature 모듈 (µFeature 적용)
```
AuthFeature/
├── AuthFeature/                    # umbrella module
├── AuthFeatureInterface/           # public interface
├── AuthFeatureImplementation/      # 실제 구현
│   ├── Presentation/              # MVVM Layer
│   │   ├── ViewModel/
│   │   └── View/
│   ├── Domain/                    # Clean Architecture Domain
│   │   ├── UseCase/
│   │   └── Repository/
│   └── Data/                      # Clean Architecture Data
│       ├── Repository/
│       └── DataSource/
├── AuthFeatureTesting/            # 테스트 유틸리티
└── AuthFeatureExample/            # 독립 실행 앱
```

## Tuist Project.swift 구성

### Core 모듈 프로젝트
```swift
// Core/Project.swift
let coreProject = Project(
    name: "Core",
    targets: [
        Target(
            name: "CoreCommon",
            platform: .iOS,
            product: .framework,
            bundleId: "com.app.core.common",
            sources: ["CoreCommon/**"],
            dependencies: []
        ),
        Target(
            name: "CoreNetwork",
            platform: .iOS,
            product: .framework,
            bundleId: "com.app.core.network",
            sources: ["CoreNetwork/**"],
            dependencies: [
                .target(name: "CoreCommon")
            ]
        ),
        Target(
            name: "CoreDomain",
            platform: .iOS,
            product: .framework,
            bundleId: "com.app.core.domain",
            sources: ["CoreDomain/**"],
            dependencies: [
                .target(name: "CoreCommon")
            ]
        )
    ]
)
```

### Feature 모듈 프로젝트
```swift
// AuthFeature/Project.swift
let authFeatureProject = Project(
    name: "AuthFeature",
    targets: [
        Target(
            name: "AuthFeatureInterface",
            platform: .iOS,
            product: .framework,
            bundleId: "com.app.auth.interface",
            sources: ["AuthFeatureInterface/**"],
            dependencies: [
                .project(target: "CoreDomain", path: "../Core"),
                .project(target: "CoreCommon", path: "../Core")
            ]
        ),
        Target(
            name: "AuthFeatureImplementation",
            platform: .iOS,
            product: .framework,
            bundleId: "com.app.auth.implementation",
            sources: ["AuthFeatureImplementation/**"],
            dependencies: [
                .target(name: "AuthFeatureInterface"),
                .project(target: "CoreNetwork", path: "../Core"),
                .project(target: "CoreDatabase", path: "../Core")
            ]
        ),
        Target(
            name: "AuthFeatureExample",
            platform: .iOS,
            product: .app,
            bundleId: "com.app.auth.example",
            sources: ["AuthFeatureExample/**"],
            dependencies: [
                .target(name: "AuthFeatureImplementation")
            ]
        )
    ]
)
```

## 실제 구현 예시

### AuthFeatureInterface (공개 인터페이스)
```swift
import UIKit
import Combine

public protocol AuthFeatureInterface {
    func makeAuthViewController() -> UIViewController
    func makeAuthCoordinator() -> AuthCoordinator
}

public protocol AuthCoordinator {
    func start()
    func navigateToMain()
    func navigateToSignUp()
}

public protocol AuthNavigator {
    func navigateToMain()
    func navigateToSignUp()
}
```

### AuthFeatureImplementation (실제 구현)

#### Domain Layer
```swift
import Foundation
import Combine
import CoreDomain

// Use Case
public class LoginUseCase {
    private let authRepository: AuthRepository
    
    public init(authRepository: AuthRepository) {
        self.authRepository = authRepository
    }
    
    public func execute(email: String, password: String) -> AnyPublisher<User, AuthError> {
        return authRepository.login(email: email, password: password)
    }
}

// Repository Protocol
public protocol AuthRepository {
    func login(email: String, password: String) -> AnyPublisher<User, AuthError>
    func logout() -> AnyPublisher<Void, AuthError>
}
```

#### Presentation Layer (MVVM)
```swift
import UIKit
import Combine
import CoreCommon

class LoginViewModel: ObservableObject {
    @Published var email: String = ""
    @Published var password: String = ""
    @Published var isLoading: Bool = false
    @Published var errorMessage: String?
    
    private let loginUseCase: LoginUseCase
    private let navigator: AuthNavigator
    private var cancellables = Set<AnyCancellable>()
    
    init(loginUseCase: LoginUseCase, navigator: AuthNavigator) {
        self.loginUseCase = loginUseCase
        self.navigator = navigator
    }
    
    func login() {
        isLoading = true
        errorMessage = nil
        
        loginUseCase.execute(email: email, password: password)
            .receive(on: DispatchQueue.main)
            .sink(
                receiveCompletion: { [weak self] completion in
                    self?.isLoading = false
                    if case .failure(let error) = completion {
                        self?.errorMessage = error.localizedDescription
                    }
                },
                receiveValue: { [weak self] user in
                    self?.navigator.navigateToMain()
                }
            )
            .store(in: &cancellables)
    }
}

class LoginViewController: UIViewController {
    private let viewModel: LoginViewModel
    private var cancellables = Set<AnyCancellable>()
    
    init(viewModel: LoginViewModel) {
        self.viewModel = viewModel
        super.init(nibName: nil, bundle: nil)
    }
    
    required init?(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
    
    override func viewDidLoad() {
        super.viewDidLoad()
        setupUI()
        bindViewModel()
    }
    
    private func bindViewModel() {
        viewModel.$isLoading
            .receive(on: DispatchQueue.main)
            .sink { [weak self] isLoading in
                // UI 업데이트
            }
            .store(in: &cancellables)
        
        viewModel.$errorMessage
            .receive(on: DispatchQueue.main)
            .sink { [weak self] errorMessage in
                // 에러 메시지 표시
            }
            .store(in: &cancellables)
    }
}
```

#### Data Layer
```swift
import Foundation
import Combine
import CoreNetwork
import CoreDatabase

class AuthRepositoryImpl: AuthRepository {
    private let authAPI: AuthAPI
    private let authDAO: AuthDAO
    
    init(authAPI: AuthAPI, authDAO: AuthDAO) {
        self.authAPI = authAPI
        self.authDAO = authDAO
    }
    
    func login(email: String, password: String) -> AnyPublisher<User, AuthError> {
        return authAPI.login(email: email, password: password)
            .flatMap { [weak self] user -> AnyPublisher<User, AuthError> in
                guard let self = self else {
                    return Fail(error: AuthError.unknown)
                        .eraseToAnyPublisher()
                }
                
                return self.authDAO.saveUser(user)
                    .map { _ in user }
                    .eraseToAnyPublisher()
            }
            .eraseToAnyPublisher()
    }
}
```

### AuthFeature 구현체
```swift
import AuthFeatureInterface
import CoreNetwork
import CoreDatabase

public class AuthFeature: AuthFeatureInterface {
    private let authAPI: AuthAPI
    private let authDAO: AuthDAO
    private let navigator: AuthNavigator
    
    public init(authAPI: AuthAPI, authDAO: AuthDAO, navigator: AuthNavigator) {
        self.authAPI = authAPI
        self.authDAO = authDAO
        self.navigator = navigator
    }
    
    public func makeAuthViewController() -> UIViewController {
        let repository = AuthRepositoryImpl(authAPI: authAPI, authDAO: authDAO)
        let loginUseCase = LoginUseCase(authRepository: repository)
        let viewModel = LoginViewModel(loginUseCase: loginUseCase, navigator: navigator)
        
        return LoginViewController(viewModel: viewModel)
    }
    
    public func makeAuthCoordinator() -> AuthCoordinator {
        return AuthCoordinatorImpl(authFeature: self)
    }
}
```

## 메인 앱에서 DI 구성

### AppDependencyContainer
```swift
import AuthFeatureInterface
import UserFeatureInterface
import CoreNetwork
import CoreDatabase

class AppDependencyContainer {
    
    // Core dependencies
    private lazy var networkModule = NetworkModule()
    private lazy var databaseModule = DatabaseModule()
    
    // Feature dependencies
    lazy var authFeature: AuthFeatureInterface = {
        let navigator = AppAuthNavigator(appCoordinator: appCoordinator)
        return AuthFeature(
            authAPI: networkModule.authAPI,
            authDAO: databaseModule.authDAO,
            navigator: navigator
        )
    }()
    
    lazy var userFeature: UserFeatureInterface = {
        return UserFeature(
            userAPI: networkModule.userAPI,
            userDAO: databaseModule.userDAO
        )
    }()
    
    private lazy var appCoordinator = AppCoordinator(
        authFeature: authFeature,
        userFeature: userFeature
    )
    
    func createRootViewController() -> UIViewController {
        return appCoordinator.start()
    }
}
```

## 핵심 장점

**SwiftUI/UIKit 호환성**: 두 UI 프레임워크 모두 지원 가능

**Combine 활용**: 반응형 프로그래밍으로 MVVM 바인딩 강화

**독립 개발**: 각 Feature가 Example 앱으로 독립 실행 가능

**빌드 최적화**: Tuist의 캐시 시스템으로 빌드 시간 단축

**테스트 용이성**: Protocol 기반으로 Mock 객체 생성이 쉬움

**팀 협업**: 각 팀이 독립적인 µFeature를 담당

이 구조로 Swift 환경에서도 대규모 앱 개발 시 코드 품질과 개발 효율성을 모두 확보할 수 있습니다.