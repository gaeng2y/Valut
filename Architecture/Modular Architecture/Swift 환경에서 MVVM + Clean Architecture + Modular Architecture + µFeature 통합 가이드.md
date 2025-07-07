## µFeature 정확한 구조

### Feature 모듈 (µFeature 표준)
```
AuthFeature/
├── Interface/          # 공개 인터페이스 (Protocol)
├── Source/            # 실제 구현
│   ├── Presentation/  # MVVM Layer
│   │   ├── ViewModel/
│   │   └── View/
│   ├── Domain/        # Clean Architecture Domain
│   │   ├── UseCase/
│   │   └── Repository/
│   └── Data/          # Clean Architecture Data
│       ├── Repository/
│       └── DataSource/
├── Testing/           # 테스트 유틸리티 (Mock, Stub)
├── Tests/            # 실제 테스트 코드
└── Demo/             # 독립 실행 가능한 데모 앱
```

## Tuist Project.swift 구성

### Feature 모듈 프로젝트
```swift
// AuthFeature/Project.swift
import ProjectDescription

let authFeatureProject = Project(
    name: "AuthFeature",
    targets: [
        // Interface - 공개 인터페이스
        Target(
            name: "AuthFeatureInterface",
            platform: .iOS,
            product: .framework,
            bundleId: "com.app.auth.interface",
            sources: ["Interface/**"],
            dependencies: [
                .project(target: "CoreDomain", path: "../Core"),
                .project(target: "CoreCommon", path: "../Core")
            ]
        ),
        
        // Source - 실제 구현
        Target(
            name: "AuthFeature",
            platform: .iOS,
            product: .framework,
            bundleId: "com.app.auth.source",
            sources: ["Source/**"],
            dependencies: [
                .target(name: "AuthFeatureInterface"),
                .project(target: "CoreNetwork", path: "../Core"),
                .project(target: "CoreDatabase", path: "../Core"),
                .project(target: "CoreUI", path: "../Core")
            ]
        ),
        
        // Testing - 테스트 유틸리티
        Target(
            name: "AuthFeatureTesting",
            platform: .iOS,
            product: .framework,
            bundleId: "com.app.auth.testing",
            sources: ["Testing/**"],
            dependencies: [
                .target(name: "AuthFeatureInterface")
            ]
        ),
        
        // Tests - 실제 테스트
        Target(
            name: "AuthFeatureTests",
            platform: .iOS,
            product: .unitTests,
            bundleId: "com.app.auth.tests",
            sources: ["Tests/**"],
            dependencies: [
                .target(name: "AuthFeature"),
                .target(name: "AuthFeatureTesting")
            ]
        ),
        
        // Demo - 독립 실행 앱
        Target(
            name: "AuthFeatureDemo",
            platform: .iOS,
            product: .app,
            bundleId: "com.app.auth.demo",
            sources: ["Demo/**"],
            dependencies: [
                .target(name: "AuthFeature")
            ]
        )
    ]
)
```

## 각 모듈별 구현

### Interface 모듈
```swift
// AuthFeature/Interface/AuthFeatureInterface.swift
import UIKit
import Combine

public protocol AuthFeatureInterface {
    func makeAuthViewController() -> UIViewController
    func makeAuthCoordinator(navigator: AuthNavigator) -> AuthCoordinator
}

public protocol AuthCoordinator {
    func start()
    func showLogin()
    func showSignUp()
}

public protocol AuthNavigator {
    func navigateToMain()
    func showError(_ message: String)
}

public struct AuthConfiguration {
    public let baseURL: String
    public let apiKey: String
    
    public init(baseURL: String, apiKey: String) {
        self.baseURL = baseURL
        self.apiKey = apiKey
    }
}
```

### Source 모듈

#### Domain Layer
```swift
// AuthFeature/Source/Domain/UseCase/LoginUseCase.swift
import Foundation
import Combine
import CoreDomain
import AuthFeatureInterface

public class LoginUseCase {
    private let authRepository: AuthRepository
    
    public init(authRepository: AuthRepository) {
        self.authRepository = authRepository
    }
    
    public func execute(email: String, password: String) -> AnyPublisher<User, AuthError> {
        guard !email.isEmpty, !password.isEmpty else {
            return Fail(error: AuthError.invalidCredentials)
                .eraseToAnyPublisher()
        }
        
        return authRepository.login(email: email, password: password)
    }
}

// AuthFeature/Source/Domain/Repository/AuthRepository.swift
import Foundation
import Combine
import CoreDomain

public protocol AuthRepository {
    func login(email: String, password: String) -> AnyPublisher<User, AuthError>
    func logout() -> AnyPublisher<Void, AuthError>
}
```

#### Presentation Layer (MVVM)
```swift
// AuthFeature/Source/Presentation/ViewModel/LoginViewModel.swift
import Foundation
import Combine
import CoreCommon
import AuthFeatureInterface

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
                        self?.navigator.showError(error.localizedDescription)
                    }
                },
                receiveValue: { [weak self] user in
                    self?.navigator.navigateToMain()
                }
            )
            .store(in: &cancellables)
    }
}
```

#### Data Layer
```swift
// AuthFeature/Source/Data/Repository/AuthRepositoryImpl.swift
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
                    .mapError { _ in AuthError.storageError }
                    .eraseToAnyPublisher()
            }
            .eraseToAnyPublisher()
    }
}
```

### Source 모듈 - Feature 구현체
```swift
// AuthFeature/Source/AuthFeature.swift
import AuthFeatureInterface
import CoreNetwork
import CoreDatabase

public class AuthFeature: AuthFeatureInterface {
    private let configuration: AuthConfiguration
    private let coreServices: CoreServices
    
    public init(configuration: AuthConfiguration, coreServices: CoreServices) {
        self.configuration = configuration
        self.coreServices = coreServices
    }
    
    public func makeAuthViewController() -> UIViewController {
        let repository = AuthRepositoryImpl(
            authAPI: coreServices.authAPI,
            authDAO: coreServices.authDAO
        )
        let loginUseCase = LoginUseCase(authRepository: repository)
        let viewModel = LoginViewModel(
            loginUseCase: loginUseCase,
            navigator: DummyNavigator() // Demo에서는 더미 네비게이터
        )
        
        return LoginViewController(viewModel: viewModel)
    }
    
    public func makeAuthCoordinator(navigator: AuthNavigator) -> AuthCoordinator {
        return AuthCoordinatorImpl(authFeature: self, navigator: navigator)
    }
}
```

### Testing 모듈
```swift
// AuthFeature/Testing/MockAuthRepository.swift
import Foundation
import Combine
import CoreDomain
import AuthFeatureInterface

public class MockAuthRepository: AuthRepository {
    public var loginResult: AnyPublisher<User, AuthError> = Empty().eraseToAnyPublisher()
    public var logoutResult: AnyPublisher<Void, AuthError> = Empty().eraseToAnyPublisher()
    
    public init() {}
    
    public func login(email: String, password: String) -> AnyPublisher<User, AuthError> {
        return loginResult
    }
    
    public func logout() -> AnyPublisher<Void, AuthError> {
        return logoutResult
    }
}

// AuthFeature/Testing/MockAuthNavigator.swift
import AuthFeatureInterface

public class MockAuthNavigator: AuthNavigator {
    public var navigateToMainCalled = false
    public var showErrorCalled = false
    public var lastErrorMessage: String?
    
    public init() {}
    
    public func navigateToMain() {
        navigateToMainCalled = true
    }
    
    public func showError(_ message: String) {
        showErrorCalled = true
        lastErrorMessage = message
    }
}
```

### Tests 모듈
```swift
// AuthFeature/Tests/LoginUseCaseTests.swift
import XCTest
import Combine
import AuthFeature
import AuthFeatureTesting
import CoreDomain

class LoginUseCaseTests: XCTestCase {
    private var sut: LoginUseCase!
    private var mockRepository: MockAuthRepository!
    private var cancellables: Set<AnyCancellable>!
    
    override func setUp() {
        super.setUp()
        mockRepository = MockAuthRepository()
        sut = LoginUseCase(authRepository: mockRepository)
        cancellables = Set<AnyCancellable>()
    }
    
    func test_execute_withValidCredentials_shouldReturnUser() {
        // Given
        let expectedUser = User(id: "1", email: "test@test.com")
        mockRepository.loginResult = Just(expectedUser)
            .setFailureType(to: AuthError.self)
            .eraseToAnyPublisher()
        
        let expectation = XCTestExpectation(description: "Login should succeed")
        
        // When
        sut.execute(email: "test@test.com", password: "password")
            .sink(
                receiveCompletion: { _ in },
                receiveValue: { user in
                    XCTAssertEqual(user.email, expectedUser.email)
                    expectation.fulfill()
                }
            )
            .store(in: &cancellables)
        
        // Then
        wait(for: [expectation], timeout: 1.0)
    }
}
```

### Demo 모듈
```swift
// AuthFeature/Demo/DemoApp.swift
import SwiftUI
import AuthFeature
import AuthFeatureInterface

@main
struct AuthFeatureDemoApp: App {
    var body: some Scene {
        WindowGroup {
            ContentView()
        }
    }
}

struct ContentView: View {
    private let authFeature: AuthFeatureInterface
    
    init() {
        let configuration = AuthConfiguration(
            baseURL: "https://api.demo.com",
            apiKey: "demo-key"
        )
        let coreServices = DemoCoreServices()
        
        self.authFeature = AuthFeature(
            configuration: configuration,
            coreServices: coreServices
        )
    }
    
    var body: some View {
        AuthFeatureWrapper(authFeature: authFeature)
    }
}
```

## 전체 워크스페이스 구성

### Workspace.swift
```swift
// Workspace.swift
import ProjectDescription

let workspace = Workspace(
    name: "MyApp",
    projects: [
        "App",
        "Core",
        "Features/AuthFeature",
        "Features/UserFeature",
        "Features/HomeFeature"
    ]
)
```

이 구조로 각 µFeature가 완전히 독립적으로 개발, 테스트, 데모 실행이 가능하며, Clean Architecture의 계층 분리와 MVVM의 바인딩 구조를 모두 유지할 수 있습니다.