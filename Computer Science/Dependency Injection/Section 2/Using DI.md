자! 이제 Section 1에서 배운 내용을 써먹어보자.

`ProfileContentProvider.swift`를 만들자.
```swift
protocol ProfileContentProviderProtocol {
  var privacyLevel: PrivacyLevel { get }
  var canSendMessage: Bool { get }
  var canStartVideoChat: Bool { get }
  var photosView: AnyView { get }
  var feedView: AnyView { get }
  var friendsView: AnyView { get }
}
```
이 코드는 프로토콜일 뿐이지만 구현에 따라 제공할 콘텐츠 종류가 결정됩니다. 

다음으로 추가한 프로토콜 아래에 다음 클래스를 추가합니다.

```swift
final class ProfileContentProvider: ProfileContentProviderProtocol {
  let privacyLevel: PrivacyLevel
  private let user: User

  init(privacyLevel: PrivacyLevel, user: User) {
    self.privacyLevel = privacyLevel
    self.user = user
  }

  var canSendMessage: Bool {
    privacyLevel > .everyone
  }

  var canStartVideoChat: Bool {
    privacyLevel > .everyone
  }

  var photosView: AnyView {
    privacyLevel > .everyone ? 
      AnyView(PhotosView(photos: user.photos)) : 
      AnyView(EmptyView())
  }

  var feedView: AnyView {
    privacyLevel > .everyone ? 
      AnyView(HistoryFeedView(posts: user.historyFeed)) : 
      AnyView(RestrictedAccessView())
  }

  var friendsView: AnyView {
    privacyLevel > .everyone ? 
      AnyView(UsersView(title: "Friends", users: user.friends)) : 
      AnyView(EmptyView())
  }
}
```

이제 하나의 책임을 맡은 별도의 공급자가 있습니다. 개인 정보 보호 수준에 따라 사용자 프로필을 표시하는 방법을 결정합니다.

다음으로, ProfileView.swift로 전환하고 ProfileView의 body 속성 바로 위에 다음 코드를 추가하자.

```swift
private let provider: ProfileContentProviderProtocol

init(provider: ProfileContentProviderProtocol, user: User) {
  self.provider = provider
  self.user = user
}
```

이러한 변경으로 `ProfileView`는 초기화 프로그램인 생성자 주입을 통해 필요한 종속성을 수신하므로 더 이상 `PrivacyLevel` 변수에 의존하지 않습니다. `ProfileView`에서 `PrivacyLevel` 상수를 제거합니다.

이것이 접근 방식의 아름다움을 보기 시작하는 곳입니다. 이제 뷰는 프로필 콘텐츠 뒤에 있는 비즈니스 논리를 전혀 인식하지 못합니다.

**이유**

1. **관심사의 분리**: `Profile` 뷰는 UI 컴포넌트를 표시하는 것만을 담당합니다. 표시할 데이터를 가져오거나, 필터링하거나, 기타 처리하는 방법에 대해서는 알 필요가 없습니다.
    
2. **유연성**: 다양한 `ProfileContentProviderProtocol` 구현을 제공할 수 있으므로, 다양한 비즈니스 로직이나 데이터 소스를 간단히 교체하거나 추가할 수 있습니다.
    
3. **테스트 용이성**: 이 디자인은 단위 테스트를 용이하게 만듭니다. 모의 `ProfileContentProviderProtocol` 구현을 제공하여 `Profile` 뷰를 독립적으로 테스트할 수 있습니다.
    
4. **확장성**: 새로운 기능이나 데이터 유형을 추가하려면 `ProfileContentProviderProtocol`에 새로운 메서드나 속성을 추가하기만 하면 됩니다. `Profile` 뷰는 변경할 필요가 없습니다.
잠시 후에 이를 확인하게 됩니다. 먼저 모든 DI 인프라를 한곳에 수집할 수 있도록 종속성 주입 컨테이너를 설정해야 합니다.

### Using a Dependency Injection Container
`DIContainer.swift` 를 만들어보자.
```swift
protocol DIContainerProtocol {
  func register<Component>(type: Component.Type, component: Any)
  func resolve<Component>(type: Component.Type) -> Component?
}

final class DIContainer: DIContainerProtocol {
  // 1
  static let shared = DIContainer()
  
  // 2
  private init() {}

  // 3
  var components: [String: Any] = [:]

  func register<Component>(type: Component.Type, component: Any) {
    // 4
    components["\(type)"] = component
  }

  func resolve<Component>(type: Component.Type) -> Component? {
    // 5
    return components["\(type)"] as? Component
  }
}
```

1. 먼저 유형 DIContainer의 정적 속성을 만듭니다.  
2. 이니셜라이저를 비공개로 표시하므로 기본적으로 컨테이너가 싱글톤인지 확인할 수 있습니다. 이렇게 하면 의도치 않게 여러 인스턴스를 사용하거나 일부 종속성을 놓치는 것과 같은 예기치 않은 동작을 방지할 수 있습니다.  
3. 그런 다음 사전을 만들어 모든 서비스를 유지합니다.  
4. 구성 요소 유형의 문자열 표현은 사전의 키입니다.  
5. 유형을 다시 사용하여 필요한 종속성을 해결할 수 있습니다.

> [!NOTE]
> 참고: 기본적으로 DI 컨테이너는 다른 패턴과 마찬가지로 프로그래밍 문제를 해결하기 위한 접근 방식입니다. 타사 프레임워크를 포함하여 여러 가지 방법으로 구현할 수 있습니다.

다음으로, 컨테이너가 종속성을 처리하도록 하려면 `ProfileView.swift`를 열고 다음과 같이 `ProfileView`의 초기화 프로그램을 업데이트하세요.(`ProfileContentProvider.swift` 도 같이,,,)
```swift
init(
  provider: ProfileContentProviderProtocol = 
    DIContainer.shared.resolve(type: ProfileContentProviderProtocol.self)!,
  user: User = DIContainer.shared.resolve(type: User.self)!
) {
  self.provider = provider
  self.user = user
}
```

이제 `DIContainer`는 기본적으로 필요한 매개변수를 제공합니다. 그러나 테스트 목적으로 종속성을 직접 전달하거나 컨테이너에 모의 종속성을 등록할 수 있습니다.

마지막으로, 작업을 시작하기 전에 앱의 동작을 복제하기 위해 종속성의 초기 상태를 정의해야 합니다.

`SceneDelegate` 에서 profileView 초기화 코드 바로 위에 아래의 코드를 추가하자.
```swift
let container = DIContainer.shared
container.register(type: PrivacyLevel.self, component: PrivacyLevel.friend)
container.register(type: User.self, component: Mock.user())
container.register(
  type: ProfileContentProviderProtocol.self, 
  component: ProfileContentProvider()
)
```

## Extending the Functionality
때때로 사용자는 친구 목록에 있는 사람들로부터 일부 콘텐츠나 기능을 숨기고 싶어합니다. 어쩌면 그들은 가까운 친구들에게만 보여주고 싶은 파티 사진을 올릴 수도 있습니다. 아니면 가까운 친구들의 영상 통화만 받고 싶을 수도 있습니다.

이유가 무엇이든 친한 친구에게 추가 접근 권한을 부여하는 기능은 훌륭한 기능입니다.

구현을 위해 `PrivacyLevel` 에 다른 케이스를 추가하자.

다음으로, 새로운 개인 정보 보호 수준을 처리할 공급자를 업데이트합니다. `ProfileContentProvider.swift`로 이동하여 다음 속성을 업데이트하세요:
```swift
var canStartVideoChat: Bool {
  privacyLevel > .friend
}

var photosView: AnyView {
  privacyLevel > .friend ? 
    AnyView(PhotosView(photos: user.photos)) : 
    AnyView(EmptyView())
}
```

이 코드를 사용하면 가까운 친구만 사진에 액세스하고 화상 통화를 시작할 수 있습니다. 개인 정보 보호 수준을 추가하기 위해 다른 변경 사항을 적용할 필요는 없습니다.