SwiftUI 앱에서 SwiftData를 매끄럽게 통합하는 법을 알아보자.
### SwiftData models
사용하려는 클래스에 ObservableObject 프로토콜 채택 및 프로퍼티에 @Published 어트리뷰트를 없애고 타입 위에 @Model을 끼얹자.
##### @Observable
새로운 Observable 매크로와 Bindable 프로퍼티 래퍼는 전보다 더 적은 코드로도 앱의 데이터 흐름을 더 자연스럽게 만든다.
뷰가 본문에서 Observable 유형 프로퍼티를 쓰면 해당 프로퍼티가 수정될 때 자동으로 업데이트된다.
너무나 쉽게 모델의 가변 상태와 UI 요소를 바인딩할 수 있다.
-> 더 알고싶으면 [Discover Observation with SwiftUI](https://developer.apple.com/videos/play/wwdc2023/10149)이거 보셈
### Querying models to display in UI
SwiftData에서 모델을 쿼리하고 UI에 띄우기 위해 ContentView를 바꿔보자
### Creating and updating
### [Bonus] Document-based apps
