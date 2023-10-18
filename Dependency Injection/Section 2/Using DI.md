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