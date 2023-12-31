## Adding User Preferences
좀 더 복잡한 사용 사례를 시도해 보고 싶으신가요? Provider가 사용자의 개인 정보 보호 기본 설정에 따라 결정을 내리도록 해야 한다면 어떻게 해야 합니까?

이 문제를 해결하기 위해 사용자가 프로필의 각 부분에 액세스할 수 있는 사람을 결정하고 `UserDefaults`를 사용하여 기본 설정을 저장하고 기본 설정을 업데이트할 때마다 프로필 화면을 다시 로드할 수 있는 새 화면을 추가합니다.

먼저, `PrivacyLevel.swift`를 열어 프로퍼티와 메소드를 추가하자.
```swift
var title: String {
  switch self {
  case .everyone:
    return "Everyone"
  case .friend:
    return "Friends only"
  case .closeFriend:
    return "Close friends only"
  }
}

static func from(string: String) -> PrivacyLevel? {
  switch string {
  case everyone.title:
    return everyone
  case friend.title:
    return friend
  case closeFriend.title:
    return closeFriend
  default:
    return nil
  }
}
```

`title`을 사용하여 생성하려는 새 기본 설정 화면에 개인 정보 보호 수준 옵션을 표시합니다. `from(string:)`은 저장된 `UserDefaults` 환경 설정에서 `PrivacyLevel`을 다시 생성하는 데 도움이 됩니다.

이제 프로젝트에 `PreferencesStore.swift` 를 추가해보자.
`UserDefaults`에서 사용자 기본 설정을 저장하고 읽는 역할을 담당하는 클래스입니다.

5개의 프로필 섹션 각각에 대한 속성과 기본 설정을 재설정하는 방법이 있습니다.
`PreferencesStoreProtocol`은 `ObservableObject` 프로토콜을 준수하므로 저장소에 `@Published` 속성이 변경될 때마다 내보내는 게시자가 있습니다.

### Adding the Preferences Screen
`Views` 폴더에 `UserPreferencesView.swift`를 추가하자.

![[Pasted image 20231018161012.png]]
새로운 화면은 위와 같이 생겼다.

`PreferencesStoreProtocol`을 구현하여 새 화면에서 사용자 기본 설정을 저장하도록 합니다. `UserPreferencesView` 선언을 다음과 같이 업데이트합니다.

모든 정적으로 유형이 지정된 프로그래밍 언어와 마찬가지로 유형은 컴파일 타임에 정의되고 확인됩니다.
문제는 다음과 같습니다. `Store`가 런타임에 갖게 될 정확한 유형을 모르지만 당황하지 마세요! 당신이 알고 있는 것은 `Store`가 `PreferencesStoreProtocol`을 준수한다는 것입니다. 따라서 `Store`가 이 프로토콜을 구현할 것이라고 컴파일러에 알립니다.

컴파일러는 뷰에 사용하려는 특정 유형을 알아야 합니다. 나중에 `UserPreferencesView` 인스턴스를 생성할 때 다음과 같이 꺾쇠 괄호 안에 프로토콜 대신 특정 유형을 사용해야 합니다.
```swift
UserPreferencesView<PreferencesStore>()
```

이렇게 하면 컴파일 타임에 유형을 확인할 수 있습니다. 이제, `UserPreferencesView` 에 프로퍼티와 생성자를 추가하자.

```swift
private var store: Store

init(store: Store = DIContainer.shared.resolve(type: Store.self)!) {
  self.store = store
}
```

그 후 `body` 구문을 수정하자.
```swift
var body: some View {
  NavigationView {
    VStack {
      PreferenceView(title: .photos, value: store.photosPreference) { value in
        store.photosPreference = value
      }
      PreferenceView(
        title: .friends, 
        value: store.friendsListPreference
      ) { value in
        store.friendsListPreference = value
      }
      PreferenceView(title: .feed, value: store.feedPreference) { value in
        store.feedPreference = value
      }
      PreferenceView(
        title: .videoCall, 
        value: store.videoCallsPreference
      ) { value in
        store.videoCallsPreference = value
      }
      PreferenceView(
        title: .message, 
        value: store.messagePreference
      ) { value in
        store.messagePreference = value
      }
      Spacer()
    }
  }.navigationBarTitle("Privacy preferences")
}
```

코드를 살펴보면,

- `VStack`의 각 `PreferenceView`는 개인 정보 보호 수준을 선택하는 드롭다운 메뉴가 있는 다양한 프로필 섹션을 나타냅니다.
- 스토어에서 각 기본 설정의 현재 값을 읽습니다.
- 사용자가 개인 정보 보호 옵션을 선택하면 새 값을 저장소에 저장합니다.

이제 `SceneDelegate.swift`로 가서 DIContainer에 store dependency를 등록하자.
```swift
container.register(type: PreferencesStore.self, component: PreferencesStore())
```

그리고 `ProfileContentProvider`로 이동하여 클래스 선언부와, 프로토콜 선언부를 수정해주자.
```swift
final class ProfileContentProvider<Store: PreferencesStoreProtocol>: ProfileContentProviderProtocol {
```

```swift
protocol ProfileContentProviderProtocol: ObservableObject {
```

이 코드를 사용하면 `ProfileView`가 `ProfileContentProvider`의 변경 사항을 구독하고 사용자가 새 기본 설정을 선택할 때 즉시 상태를 업데이트할 수 있습니다.

`ProfileContentProvider`에서 저장소에 대한 속성을 추가하고 초기화 프로그램을 바꿉니다.

```swift
private var store: Store
private var cancellables: Set<AnyCancellable> = []

init(
  privacyLevel: PrivacyLevel = 
    DIContainer.shared.resolve(type: PrivacyLevel.self)!,
  user: User = DIContainer.shared.resolve(type: User.self)!,
  // 1
  store: Store = DIContainer.shared.resolve(type: Store.self)!
) {
  self.privacyLevel = privacyLevel
  self.user = user
  self.store = store

  // 2
  store.objectWillChange.sink { _ in
    self.objectWillChange.send()
  }
  .store(in: &cancellables)
}
```

수행한 작업은 다음과 같습니다.
1. DI 컨테이너는 PreferencesStore의 인스턴스를 제공합니다.
2. `objectWillChange` 속성을 사용하여 `PreferencesStoreProtocol` 게시자를 구독합니다.
3. Store에서 속성이 변경되면 `ProfileContentProviderProtocol` 게시자도 내보내도록 합니다.

이제 `PrivacyLevel` 열거형의 인스턴스 대신 저장소의 속성을 사용하도록 `ProfileContentProvider`의 속성을 업데이트하세요.
### Bringing It All Together
Provider의 변경 사항을 구독하려면 `ProfileView.swift`를 열고 `ProfileView` 선언도 변경하세요.