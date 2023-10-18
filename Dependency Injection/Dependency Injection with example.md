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

DI 클라이언트 객체인 클래스는 구현의 종속성인 DI 서비스 객체를 인식하지 못합니다. 또한 그것들을 만드는 방법도 모릅니다. 이렇게 하면 클래스 간의 긴밀하게 결합된 관계를 제거하여 코드를 테스트하고 유지 관리할 수 있습니다.

**종속성 주입**은 제어 반전 원칙을 적용하는 데 도움이 되는 몇 가지 패턴 중 하나입니다. _생성자 주입_ , _세터 주입_ , _인터페이스 주입_ 등 여러 가지 방법으로 종속성 주입을 구현할 수 있습니다.
### 생성자 주입(Constructor Injection)
**생성자 주입** 또는 **초기화 주입**에서는 모든 클래스 종속성을 생성자 매개변수로 전달합니다. 클래스에 필요한 모든 종속성을 한 곳에서 즉시 확인할 수 있으므로 코드의 기능을 더 쉽게 이해할 수 있습니다. 예를 들어, 이 스니펫을 보세요

```swift
protocol EngineProtocol {
  func start()
  func stop()
}

protocol TransmissionProtocol {
  func changeGear(gear: Gear)
}

final class Car {
  private let engine: EngineProtocol
  private let transmission: TransmissionProtocol

  init(engine: EngineProtocol, transmission: TransmissionProtocol) {
    self.engine = engine
    self.transmission = transmission
  }
}
```

이 코드 조각에서 `EngineProtocol`및 `TransmissionProtocol`는 _서비스_ 이고 클라이언트는 `Car` 입니다 . 책임을 분할하고 추상화를 사용하므로 예상 프로토콜을 준수하는 모든 종속성을 갖춘 인스턴스를 생성할 수 있습니다.

일부 단위 테스트로 자동차를 다루기 위해 `EngineProtocol` 및 `TransmissionProtocol`의 테스트 구현을 통과할 수도 있습니다.
### Setter 
**Setter 주입** 또는 **메소드 주입**은 눈에 띄게 다릅니다. 이 예에서 볼 수 있듯이 종속성 설정자 메서드가 필요합니다.
```swift
final class Car {
  private var engine: EngineProtocol?
  private var transmission: TransmissionProtocol?

  func setEngine(engine: EngineProtocol) {
    self.engine = engine
  }

  func setTransmission(transmission: TransmissionProtocol) {
    self.transmission = transmission
  }
}
```