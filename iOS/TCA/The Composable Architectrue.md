Composable Architecture(약칭 TCA)는 구성, 테스트, 인체공학을 염두에 두고 일관되고 이해하기 쉬운 방식으로 앱을 빌드하기 위한 라이브러리다. 

SwiftUI, UIKit 등에서 사용할 수 있으며 모든 Apple 플랫폼(iOS, macOS, visionOS, tvOS, watchOS)에서 사용할 수 있다.
## the Composable Architecture가 무엇인가?

이 라이브러리는 다양한 목적과 복잡성의 애플리케이션을 구축하는 데 사용할 수 있는 몇 가지 핵심 도구를 제공한다. 다음과 같이 앱을 구축할 때 일상적으로 직면하는 많은 문제를 해결하기 위해 따를 수 있는 설득력 있는 이야기를 제공한다.

- **State management(상태 관리)**
  값 타입을 사용하여 앱 상태를 관리하고, 한 화면의 변형을 다른 화면에서 즉시 관찰할 수 있도록 여러 화면에서 상태를 공유하는 방법이다.
- **Composition(합성)**
  큰 기능을 작은 부분으로 쪼개서, 격리된 모듈로 분리시킬 수 있고, 또한 이 모듈을 다시 합쳐서 큰 기능을 작은 컴포넌트의 집합으로 구성하는 방법을 제공한다.
- **Side effects(부작용)**
  앱의 특정 부분이 외부 세계와 제일 테스트 가능하고 이해하기 쉬운 방식으로 소통하도록 하는 방법
- **Testing(테스팅)**
  아키텍처를 포함된 기능만 테스트하는게 아니라, 많은 부분으로 구성된 기능에 대한 통합적 테스트가 가능하다.
  사이드 이펙트가 앱에 어떤 영향을 줄 지, 비즈니스 로직이 개발자가 예상한대로 작동하는지 보장할 수 있게 해주는 End to End 테스트가 가능하다.
- **Ergonomics(인체공학)**
  위에서 설명한 합성, 테스팅에서의 특징을 심플한 API 만으로 달성할 수 있다
# 작동 방식
![다이어그램](https://miro.medium.com/v2/resize:fit:640/format:webp/0*8pFhMLNtx8kYyeaJ.png)
(출처: https://medium.com/riiid-teamblog-kr/riiid%EC%9D%98-swift-composable-architecture-231a665e5f47)

View는 **State**를 구독하고

**View**의 이벤트는 Action을 보낸다.

**Action** 이라는 건, 사용자가 화면에 있는 버튼을 누른다거나, 글자를 입력해서 제출하는 등의 **‘행동’** 을 의미한다.

Action은 Reducer로 들어간다. Reducer는 어떤 Action이 들어왔을 때, State를 다음 상태로 변화시키는 방법을 가지고 있는 **함수**이다.

단, State 변경을 위해서 **‘앱 외부의 데이터’** 가 필요한 경우가 있을 수 있다. 예를 들어 API 통신을 해서 화면에 띄울 목록을 가지고 와야하는 경우다.

이 경우, 앱은 API 클라이언트에 의존성(Dependency)를 갖게 되는데, 이러한 의존성을 가지고 있는 타입을 **Environment**라고 한다.

Environment는 결과를 **Effect** 타입으로 반환한다. Environment를 통해 API 통신을 한다고 하면, 결국 Side Effect가 발생할 수 있기 때문에 Effect는 Action으로 이동한다.

**원하는 Effect가 들어온 경우** 하고 **Side Effect가 발생한 경우** 각각을 Action 으로 받아들이고, 다시 Reducer 에서 로직을 실행해주기 위해서이다.

![스토어](https://miro.medium.com/v2/resize:fit:720/format:webp/1*ypoAuV-LPiaOKBY3HqLEhA.png)

Store를 살펴보면 **Store**는 State, Reducer, Environment를 모두 감싸고 있다.

개발자가 정의한 앱의 모든 기능이 작동하는 공간으로 생각하면 된다.

[TCA 공식 문서](https://pointfreeco.github.io/swift-composable-architecture/Store/)에서는 Store 를 **Runtime(런타임)** 이라고 표현한다.
# 정리

TCA 패턴의 가장 중요한 특징은 **단방향(Unidirectional)** 이라는 것이다. 흘러가는 로직과 각 타입의 역할을 이해하게 되면, 앱을 만들 때 Side Effect 관리가 쉬워진다.

즉, 에러를 관리하기가 쉽고 유닛 테스트를 작성하는 것 또한 수월하다는 장점이 있다.