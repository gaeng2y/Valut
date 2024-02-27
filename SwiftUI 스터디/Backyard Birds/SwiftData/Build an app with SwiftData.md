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
윈도우와 뷰가 컨테이너를 상속받고 같은 그룹에서 생성된 다른 윈도우도 상속받는다. 모든 뷰는 단일 컨테이너에서 쓰기와 읽기를 한다. 

일부 앱은 스토리지 스택 여러 개가 필요한데 각각의 윈도우에 모델 컨테이너 여러 개를 만든다.
SwiftUI는 뷰 수준에서 세분화 설정도 허용하는데

![[Pasted image 20240227223545.png]]

같은 윈도우에서 다른 뷰는 개별 컨테이너를 갖기 때문에 한 컨테이너를 저장해도 다른 컨테이너에 영향은 없다.

##### 애플의 previewContainer
```swift
import SwiftData

@MainActor
let previewContainer: ModelContainer = {
    do {
        let container = try ModelContainer(
            for: Card.self,
            configurations: ModelConfiguration(isStoredInMemoryOnly: true)
        )
        let modelContext = container.mainContext
        if try modelContext.fetch(FetchDescriptor<Card>()).isEmpty {
            SampleDeck.contents.forEach { container.mainContext.insert($0) }
        }
        return container
    } catch {
        fatalError("Failed to create container")
    }
}()
```
### Creating and updating
```swift
@Environment(\.modelContext) private var modelContext
```
`ModelContext`에 접근하기 위해서 스유는 새 환경변수를 제공한다.
`ModelContainer`와 마찬가지로 각 뷰의 컨텍스트는 하나지만 일반적인 앱의 컨텍스트 개수는 무제한이다.
샘플 앱에는 컨텍스트가 이미 구성되어 있다.

이 환경 변수는 모델 컨테이너를 설정할 때 자동으로 채워진다.

### [Bonus] Document-based apps
## Reference
* WWDC 2023 [Build an app with SwiftData](https://developer.apple.com/videos/play/wwdc2023/10154)
* [Sampe](https://developer.apple.com/documentation/SwiftUI/Building-a-document-based-app-using-SwiftData)