### NavigationStack (네비게이션 스택)

#### 뭐야, 그게 뭐야?
`NavigationStack`은 아이패드나 아이폰에서 앱을 사용할 때, 서로 다른 화면 사이를 왔다 갔다 할 수 있게 해주는 도구야. 예를 들어, 게임에서 캐릭터를 선택하는 화면에서 게임 시작 화면으로 넘어가는 것처럼 말이야.

#### 어떻게 생겼어?
`NavigationStack`은 여러 개의 화면을 차곡차곡 쌓아놓는 상자처럼 생겼어. 새로운 화면을 열 때마다 그 상자 위에 새로운 화면이 추가되고, 뒤로 가기 버튼을 누르면 그 상자가 하나씩 열리면서 이전 화면으로 돌아가.

#### 어떻게 사용해?
먼저, `NavigationStack`을 만들고 그 안에 다른 화면들을 넣어줘야 해.
```swift
NavigationStack {
    // 여기 안에 네비게이션할 화면들을 넣어요
}
```
이렇게 하면 `NavigationStack`이 화면들을 관리해줘.

### NavigationPath (네비게이션 경로)

#### 뭐야, 그게 뭐야?
`NavigationPath`는 우리가 지금까지 어떤 화면들을 지나왔는지 기억하는 역할을 해. 예를 들어, 숲에서 길을 잃지 않게 발자국을 남기듯이 말이야.

#### 어떻게 생겼어?
`NavigationPath`는 우리가 어떤 화면들을 지나왔는지 목록처럼 쭉 적어놓은 종이와 같아. 만약 우리가 여러 화면을 지나갔다면 그 종이에 그 화면들의 이름이 적혀 있는 거지.

#### 어떻게 사용해?
`NavigationPath`를 사용하면 어떤 화면을 지나갔는지, 지금 어떤 화면에 있는지를 쉽게 알 수 있어.
```swift
@State private var path = NavigationPath()

NavigationStack(path: $path) {
    // 여기 안에 네비게이션할 화면들을 넣어요
}
```
이렇게 하면 우리가 어떤 화면을 지나갔는지 `path`가 기억해줘.

### 같이 사용해볼까?

#### 예시
1. 처음 화면에서 게임을 시작할 수 있어.
2. 게임 시작 버튼을 누르면 캐릭터 선택 화면으로 이동해.
3. 캐릭터를 선택하면 게임이 시작돼.

```swift
@State private var path = NavigationPath()

NavigationStack(path: $path) {
    // 처음 화면
    NavigationLink("게임 시작하기", value: "캐릭터 선택 화면")

    // 캐릭터 선택 화면
    .navigationDestination(for: String.self) { value in
        if value == "캐릭터 선택 화면" {
            NavigationLink("캐릭터 선택 완료", value: "게임 화면")
        } else if value == "게임 화면" {
            // 게임 화면
            Text("게임이 시작되었습니다!")
        }
    }
}
```
이렇게 하면 버튼을 눌러서 화면을 이동할 수 있고, `NavigationPath`가 어떤 화면을 지나왔는지 기억해줘서, 우리가 원하는 화면으로 쉽게 이동할 수 있어.

### 정리
- **`NavigationStack`**: 여러 화면을 차곡차곡 쌓아놓고, 뒤로 가기 버튼을 누르면 차례차례 돌아갈 수 있게 해주는 도구.
- **`NavigationPath`**: 우리가 지나온 화면들을 기억해주는 목록.

이제 `NavigationStack`과 `NavigationPath`를 이용해서 앱에서 다양한 화면을 쉽게 탐색할 수 있어!

