[코데코](https://www.kodeco.com/14223279-dependency-injection-tutorial-for-ios-getting-started) 강의에서 보면

**Inversion of Control** is a pattern that lets you invert the flow of control. To achieve this **you move all the responsibilities of a class, except its main one, outside, making them its _dependencies_**. Through abstraction you make the dependencies easily **interchangeable**.

라고 되어있습니다.

찬찬히 살펴보면

IoC는 제어흐름을 역전시킬 수 있는 패턴이다. 이를 달성하려면 클래스의 주요 책임을 제외한 모든 책임을 외부로 옮겨 종속성을 만듭니다. 추상화를 통해 종속성을 쉽게 교환할 수 있습니다.

위 링크에서의 예제를 보면
```swift
**var** body: **some** View {

    NavigationView {

      ScrollView(.vertical, showsIndicators: **true**) {

        VStack {

          ProfileHeaderView(

            user: user,

            canSendMessage: privacyLevel == .friend,

            canStartVideoChat: privacyLevel == .friend

          )

          **if** privacyLevel == .friend {

            UsersView(title: "Friends", users: user.friends)

            PhotosView(photos: user.photos)

            HistoryFeedView(posts: user.historyFeed)

          } **else** {

            RestrictedAccessView()

          }

        }

      }

      .navigationTitle("Profile")

    }

  }
```
