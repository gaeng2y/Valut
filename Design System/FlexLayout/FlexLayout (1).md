안녕하세요... **FlexLayout**에 대해 살펴보면서 왜 제가 공부하게 됐는지 나열해보겠습니다...



<p align="center">
  <img src="https://github.com/layoutBox/FlexLayout/raw/master/docs_markdown/images/flexlayout-logo-text.png" alt="FlexLayout" width="210"/>
  </p>
FlexLayout은 간결하고 직관적이며 연결 가능한 구문을 제공하는 고도로 최적화된 [Yoga](https://github.com/facebook/yoga) flexbox 구현에 나이스한 Swift 인터페이스를 더한다.
#### 장점
- Flexbox 레이아웃은 **빠르고 단순**하다.
- 구문이 간결하고 체이닝을 해서 **손쉽게 사용**할 수 있다.
- 수동 레이아웃(UIStackView 등)보다 월등히 빠르다.
- 코드 구조가 flexbox 구조와 동일해 **코드 자체를 이해하기 쉽고 수정이 용이**하다.
- LTR / RTL 지원을 합니다.
#### **요구사항**
- iOS 12.0+
- Xcode 12.0+
- Swift 4.0
#### FlexLayout + PinLayout
FlexLayout은 [PinLayout](https://github.com/layoutBox/PinLayout)의 동반자이다. 그것들은 비슷한 문법과 메소드 이름을 공유한다. PinLayout은 CSS 절대 포지셔닝에서 크게 영감을 받은 레이아웃 프레임워크이며, 특히 미세한 제어와 애니메이션에 유용하다.
대부분 **두 라이브러리가 같이 사용**된다. 

### 왜 FlexLayout?
저는 SnapKit을 이용해서 주로 뷰 드로잉을 했었는데 이번에 DesignSystem을 만들면서 `DesignSystemFoundation` -> `App'sUISystem` -> `App` 이러한 의존성을 가지도록 개발을 하던 중 뷰 드로잉을 FlexLayout을 이용하면 어떨까 싶어서 한번 공부해볼 겸 선택하게 되었다.

