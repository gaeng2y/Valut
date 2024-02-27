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
UI에서 SwiftData가 관리하는 모델을 보고싶으면 @Query를 사용하면 된다.
-> @Query는 SwiftData에서 모델을 쿼리하는 프로퍼티 래퍼다.
-> 근데? 이번에 매크로로 바뀜
모델에 일어나는 모든 변화를 뷰에 업데이트하기도 한다. -> @State와 같은 기능
뷰의 쿼리 프로퍼티 개수에는 제한이 없다.
쿼리가 제공하는 가벼운 구문은 분류를 비롯해 주문과 필터링, 변경 사항 애니메이팅까지 구성할 수 있다.
쿼리는 뷰의 `ModelContext`를 데이터 소스로 사용한다.
@Query에는 `ModelContext`를 어떻게 제공해야 할까?
`ModelContainer`에서 가져오면 된다.

SwiftUI는 새로운 View와 Scene Modifier를 개발해서 뷰의 ModelContainer를 편리하게 설정하고자 했다.
SwiftData를 사용하는 앱에는 하나 이상의 컨테이너가 필요한데 이렇게 하면 @Query가 사용할 컨텍스트를 포함하는 스토리지 스택이 생성된다.
뷰의 모델 컨테이너는 하나지만 앱은 뷰 계층 구조에 따라 컨테이너를 여러 개 생성할 수 있다.

앱이 `modelContainer`를 설정하지 않으면 윈도우와 뷰가 SwiftData를 통해 쿼리 모델을 저장할 수 없다. 
앱에 필요한 건 보통 단일 모델 컨테이너다. 이 경우 전체 윈도우 그룹 씬을 설정하면 된다. 
윈도우와 뷰가 컨테이너를 상속받고 
### Creating and updating
### [Bonus] Document-based apps
