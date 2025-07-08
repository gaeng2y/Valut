## 전체 구조 개요

```
MyApp/
├── App/                    # 메인 애플리케이션
├── Features/               # 각 기능별 µFeature
│   ├── AuthFeature/
│   ├── UserFeature/
│   └── HomeFeature/
├── Domain/                 # 비즈니스 도메인 모델
├── Core/                   # 핵심 인프라
└── Shared/                 # 공통 유틸리티
```

## 각 레이어별 역할과 구성

### 1. App 레이어
```
App/
├── Sources/
│   ├── Application/        # AppDelegate, SceneDelegate
│   ├── DI/                # Dependency Injection Container
│   ├── Coordinator/        # 앱 전체 네비게이션
│   └── Configuration/      # 앱 설정
├── Resources/              # Assets, Info.plist
└── Project.swift
```

### 2. Domain 레이어 (순수 비즈니스 로직)
```
Domain/
├── UserDomain/
│   ├── Sources/
│   │   ├── Entity/         # User, Profile 등
│   │   ├── UseCase/        # GetUserUseCase 등
│   │   └── Repository/     # UserRepository Protocol
│   └── Project.swift
├── AuthDomain/
│   ├── Sources/
│   │   ├── Entity/         # AuthToken, LoginRequest 등
│   │   ├── UseCase/        # LoginUseCase, LogoutUseCase 등
│   │   └── Repository/     # AuthRepository Protocol
│   └── Project.swift
└── SharedDomain/           # 공통 도메인 모델
    ├── Sources/
    │   ├── Entity/         # Result, Error 등
    │   └── Protocol/       # 공통 인터페이스
    └── Project.swift
```

### 3. Core 레이어 (인프라스트럭처)
```
Core/
├── CoreNetwork/
│   ├── Sources/
│   │   ├── Service/        # APIService, NetworkClient
│   │   ├── Configuration/  # NetworkConfiguration
│   │   └── Extension/      # URLSession extensions
│   └── Project.swift
├── CoreDatabase/
│   ├── Sources/
│   │   ├── CoreData/       # Core Data Stack
│   │   ├── UserDefaults/   # UserDefaults wrapper
│   │   └── Keychain/       # Keychain wrapper
│   └── Project.swift
├── CoreUI/
│   ├── Sources/
│   │   ├── Component/      # 공통 UI 컴포넌트
│   │   ├── Style/          # Color, Font, Theme
│   │   └── Extension/      # UIView extensions
│   └── Project.swift
└── CoreNavigation/
    ├── Sources/
    │   ├── Coordinator/    # Base Coordinator
    │   └── Router/         # Routing Protocol
    └── Project.swift
```

### 4. Shared 레이어 (공통 유틸리티)
```
Shared/
├── SharedUtility/
│   ├── Sources/
│   │   ├── Extension/      # String, Date extensions
│   │   ├── Helper/         # Validation, Formatter
│   │   └── Logger/         # Logging utility
│   └── Project.swift
├── SharedTesting/
│   ├── Sources/
│   │   ├── Mock/           # 공통 Mock 객체
│   │   ├── Fixture/        # 테스트 데이터
│   │   └── Helper/         # 테스트 헬퍼
│   └── Project.swift
└── SharedDesign/
    ├── Sources/
    │   ├── Token/          # Design Token
    │   ├── Component/      # 디자인 시스템 컴포넌트
    │   └── Asset/          # 공통 에셋
    └── Project.swift
```

### 5. Feature 레이어 (µFeature 구조)
```
Features/AuthFeature/
├── Interface/
│   └── AuthFeatureInterface.swift
├── Source/
│   ├── Presentation/       # MVVM Layer
│   │   ├── ViewModel/
│   │   ├── View/
│   │   └── Coordinator/
│   └── Data/              # Data Layer (Repository 구현체)
│       ├── Repository/
│       ├── DataSource/
│       └── Mapper/
├── Testing/
│   ├── Mock/
│   └── Stub/
├── Tests/
│   ├── Presentation/
│   └── Data/
├── Demo/
│   ├── Sources/
│   └── Resources/
└── Project.swift
```

## 의존성 관계

### 의존성 방향
```
App → Feature → Domain ← Core
App → Core
App → Shared
Feature → Shared
Core → Shared
Domain (독립적, 다른 모듈에 의존하지 않음)
```

### 구체적인 의존성 구성
```swift
// AuthFeature/Project.swift
Target(
    name: "AuthFeature",
    dependencies: [
        // Interface
        .target(name: "AuthFeatureInterface"),
        
        // Domain
        .project(target: "AuthDomain", path: "../../Domain/AuthDomain"),
        .project(target: "UserDomain", path: "../../Domain/UserDomain"),
        .project(target: "SharedDomain", path: "../../Domain/SharedDomain"),
        
        // Core
        .project(target: "CoreNetwork", path: "../../Core/CoreNetwork"),
        .project(target: "CoreDatabase", path: "../../Core/CoreDatabase"),
        .project(target: "CoreUI", path: "../../Core/CoreUI"),
        .project(target: "CoreNavigation", path: "../../Core/CoreNavigation"),
        
        // Shared
        .project(target: "SharedUtility", path: "../../Shared/SharedUtility"),
        .project(target: "SharedDesign", path: "../../Shared/SharedDesign")
    ]
)
```

## 실제 구현 예시

### Domain 레이어
```swift
// Domain/AuthDomain/Sources/Entity/AuthToken.swift
import Foundation

public struct AuthToken {
    public let accessToken: String
    public let refreshToken: String
    public let expiresAt: Date
    
    public init(accessToken: String, refreshToken: String, expiresAt: Date) {
        self.accessToken = accessToken
        self.refreshToken = refreshToken
        self.expiresAt = expiresAt
    }
}

// Domain/AuthDomain/Sources/UseCase/LoginUseCase.swift
import Foundation
import Combine
import SharedDomain

public protocol LoginUseCaseProtocol {
    func execute(email: String, password: String) -> AnyPublisher<AuthToken, DomainError>
}

public class LoginUseCase: LoginUseCaseProtocol {
    private let authRepository: AuthRepositoryProtocol
    
    public init(authRepository: AuthRepositoryProtocol) {
        self.authRepository = authRepository
    }
    
    public func execute(email: String, password: String) -> AnyPublisher<AuthToken, DomainError> {
        guard !email.isEmpty, !password.isEmpty else {
            return Fail(error: DomainError.invalidInput)
                .eraseToAnyPublisher()
        }
        
        return authRepository.login(email: email, password: password)
    }
}

// Domain/AuthDomain/Sources/Repository/AuthRepositoryProtocol.swift
import Foundation
import Combine
import SharedDomain

public protocol AuthRepositoryProtocol {
    func login(email: String, password: String) -> AnyPublisher<AuthToken, DomainError>
    func logout() -> AnyPublisher<Void, DomainError>
    func refreshToken() -> AnyPublisher<AuthToken, DomainError>
}
```

### Feature 레이어 구현
```swift
// Features/AuthFeature/Source/Data/Repository/AuthRepository.swift
import Foundation
import Combine
import AuthDomain
import UserDomain
import CoreNetwork
import CoreDatabase
import SharedDomain

class AuthRepository: AuthRepositoryProtocol {
    private let networkService: NetworkServiceProtocol
    private let tokenStorage: TokenStorageProtocol
    
    init(networkService: NetworkServiceProtocol, tokenStorage: TokenStorageProtocol) {
        self.networkService = networkService
        self.tokenStorage = tokenStorage
    }
    
    func login(email: String, password: String) -> AnyPublisher<AuthToken, DomainError> {
        let request = LoginRequest(email: email, password: password)
        
        return networkService.request(endpoint: AuthEndpoint.login, body: request)
            .decode(type: AuthTokenResponse.self, decoder: JSONDecoder())
            .map { $0.toDomain() }
            .flatMap { [weak self] token -> AnyPublisher<AuthToken, DomainError> in
                guard let self = self else {
                    return Fail(error: DomainError.unknown)
                        .eraseToAnyPublisher()
                }
                
                return self.tokenStorage.save(token)
                    .map { _ in token }
                    .eraseToAnyPublisher()
            }
            .mapError { error in
                if let networkError = error as? NetworkError {
                    return networkError.toDomainError()
                }
                return DomainError.unknown
            }
            .eraseToAnyPublisher()
    }
}

// Features/AuthFeature/Source/Presentation/ViewModel/LoginViewModel.swift
import Foundation
import Combine
import AuthDomain
import SharedUtility
import SharedDesign

class LoginViewModel: ObservableObject {
    @Published var email: String = ""
    @Published var password: String = ""
    @Published var isLoading: Bool = false
    @Published var errorMessage: String?
    
    private let loginUseCase: LoginUseCaseProtocol
    private let coordinator: AuthCoordinatorProtocol
    private var cancellables = Set<AnyCancellable>()
    
    init(loginUseCase: LoginUseCaseProtocol, coordinator: AuthCoordinatorProtocol) {
        self.loginUseCase = loginUseCase
        self.coordinator = coordinator
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
                        self?.handleError(error)
                    }
                },
                receiveValue: { [weak self] token in
                    self?.coordinator.navigateToMain()
                }
            )
            .store(in: &cancellables)
    }
    
    private func handleError(_ error: DomainError) {
        switch error {
        case .invalidInput:
            errorMessage = "이메일과 비밀번호를 입력해주세요."
        case .networkError:
            errorMessage = "네트워크 연결을 확인해주세요."
        case .unauthorized:
            errorMessage = "이메일 또는 비밀번호가 올바르지 않습니다."
        default:
            errorMessage = "알 수 없는 오류가 발생했습니다."
        }
    }
}
```

### App 레이어 DI 구성
```swift
// App/Sources/DI/AppDependencyContainer.swift
import Foundation
import AuthFeatureInterface
import UserFeatureInterface
import AuthDomain
import UserDomain
import CoreNetwork
import CoreDatabase

class AppDependencyContainer {
    
    // Core Services
    lazy var networkService: NetworkServiceProtocol = NetworkService(
        configuration: NetworkConfiguration.production
    )
    
    lazy var databaseService: DatabaseServiceProtocol = DatabaseService()
    
    // Domain Use Cases
    lazy var loginUseCase: LoginUseCaseProtocol = LoginUseCase(
        authRepository: authRepository
    )
    
    lazy var getUserUseCase: GetUserUseCaseProtocol = GetUserUseCase(
        userRepository: userRepository
    )
    
    // Repositories
    lazy var authRepository: AuthRepositoryProtocol = AuthRepository(
        networkService: networkService,
        tokenStorage: TokenStorage()
    )
    
    lazy var userRepository: UserRepositoryProtocol = UserRepository(
        networkService: networkService,
        userStorage: UserStorage(databaseService: databaseService)
    )
    
    // Features
    lazy var authFeature: AuthFeatureInterface = AuthFeature(
        loginUseCase: loginUseCase,
        coordinator: appCoordinator.authCoordinator
    )
    
    lazy var userFeature: UserFeatureInterface = UserFeature(
        getUserUseCase: getUserUseCase,
        coordinator: appCoordinator.userCoordinator
    )
    
    lazy var appCoordinator = AppCoordinator(
        authFeature: authFeature,
        userFeature: userFeature
    )
}
```

## 핵심 장점

**명확한 책임 분리**: 각 레이어가 명확한 역할을 가짐

**재사용성**: Domain과 Core 모듈은 여러 Feature에서 재사용 가능

**테스트 용이성**: 각 레이어별로 독립적인 테스트 가능

**확장성**: 새로운 Feature 추가 시 기존 코드 변경 최소화

**빌드 최적화**: 변경된 모듈만 빌드하여 빌드 시간 단축

**팀 협업**: 각 팀이 서로 다른 레이어/Feature를 담당 가능

이 구조로 대규모 iOS 앱에서도 코드 품질과 개발 효율성을 모두 확보할 수 있습니다.