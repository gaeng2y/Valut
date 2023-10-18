[코데코](https://www.kodeco.com/14223279-dependency-injection-tutorial-for-ios-getting-started) 강의에서 보면서 진행을 해보자.

위 링크에서의 예제 내의 `ProfileView` 의 `body`를 보면 
```swift
var body: some View {
    VStack {
      ImageView(withURL: post.pictureURL).frame(height: 200).clipped()
      HStack {
        Text(post.message)
        Spacer()
        HStack {
          Image(systemName: "hand.thumbsup")
          Text(String(post.likesCount))
        }
        HStack {
          Image(systemName: "bubble.right")
          Text(String(post.commentsCount))
        }
      }.padding()
    }
}
  ```

1. `VStack` 상단에 `ProfileHeaderView`를 추가하고 메시지 및 화상 통화 옵션을 지정하면 사용자가 친구인 경우에만 사용할 수 있습니다.
2. 사용자가 친구인 경우 친구 목록, 사진, 게시물을 표시합니다.
3. 그렇지 않으면 `RestrictedAccessView`를 보여준다.

여기서 `ProfileView`의 `privacyLevel`을 `.everyone` 으로 바꿔주고 앱을 실행하여 마치 친구 목록에 없는 사람인 것처럼 프로필이 보인다.

![[Pasted image 20231018134446.png]]
이미 기본적인 개인 정보 보호 제어 기능이 마련되어 있습니다. 

그러나 사용자가 자신의 프로필 섹션을 볼 수 있는 사람을 선택할 수 있는 방법은 없습니다. 두 가지 개인 정보 보호 수준으로는 충분하지 않습니다.

현재 `ProfileView`는 개인 정보 보호 수준에 따라 표시할 보기를 결정합니다. 이는 여러 가지 이유로 적절한 솔루션이 아닙니다.
- 테스트하기가 쉽지 않습니다. UI 테스트로 처리할 수 있지만 단위 또는 통합 테스트보다 실행 비용이 더 많이 듭니다.
- 앱 기능을 확장하거나 수정하기로 결정할 때마다 `ProfileView`에도 많은 조정이 필요합니다. `PrivacyLevel`과 긴밀하게 결합되어 있으며 필요한 것보다 더 많은 책임이 있습니다.
- 앱의 복잡성과 기능이 증가함에 따라 이 코드를 유지 관리하는 것이 더 어려워집니다.

그러나 종속성 주입을 사용하면 상황을 개선하고 새로운 기능을 원활하게 추가할 수 있습니다.

## What Are Inversion of Control and Dependency Injection?
**Inversion of Control** is a pattern that lets you invert the flow of control. To achieve this **you move all the responsibilities of a class, except its main one, outside, making them its _dependencies_**. Through abstraction you make the dependencies easily **interchangeable**.

라고 되어있습니다.

찬찬히 살펴보면

IoC는 제어흐름을 역전시킬 수 있는 패턴이다. 이를 달성하려면 클래스의 주요 책임을 제외한 모든 책임을 외부로 옮겨 종속성을 만듭니다. 추상화를 통해 종속성을 쉽게 교환할 수 있습니다.