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