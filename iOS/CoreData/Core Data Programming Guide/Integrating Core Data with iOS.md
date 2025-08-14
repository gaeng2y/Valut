### Connecting the Model to Views
iOS 앱에서 **`NSFetchedResultsController`** 는 Core Data 모델과 `UITableView` 기반의 사용자 인터페이스를 연결하는 데 사용됩니다.
##### 주요 기능 및 작동 방식
`NSFetchedResultsController`는 다음의 세 가지 핵심 기능을 제공합니다.
1.  **데이터 가져오기**: `NSFetchRequest`를 사용하여 초기 데이터를 가져옵니다. 이때, 적어도 하나의 **`NSSortDescriptor`** 를 포함하여 데이터의 정렬 순서를 지정해야 합니다.
2.  **변경 사항 모니터링**: 데이터가 변경될 때마다 `delegate`를 통해 알림을 받아 `UITableView`를 자동으로 업데이트할 수 있습니다. 이를 통해 테이블 뷰와 데이터 모델의 동기화를 쉽게 관리할 수 있습니다.
3.  **성능 최적화**: 초기화 시 **`cache name`** 을 지정하여 데이터를 캐시할 수 있습니다. 이 캐시 덕분에 대규모 데이터 세트에서도 이후의 데이터 로딩이 거의 즉각적으로 이루어져 성능이 향상되고 사용자 경험이 개선됩니다.
##### 통합 방법
`NSFetchedResultsController`는 `UITableViewController` 인스턴스에서 초기화되며, `UITableViewDataSource` 메서드와 통합하여 테이블 뷰의 데이터 검색을 간소화합니다. 또한, `NSFetchedResultsControllerDelegate` 프로토콜을 구현하여 데이터 변경 시 테이블 뷰를 자동으로 업데이트하는 로직을 추가할 수 있습니다.
### Integrating Core Data at iOS Startup
##### iOS와 macOS의 앱 시작 차이점
macOS 애플리케이션은 시작하는 데 시간이 오래 걸리면 커서 모양이 바뀌어 사용자에게 로딩 중임을 알립니다. 사용자는 앱이 실행될 때까지 기다리거나 강제 종료할 수 있습니다. 하지만 iOS에서는 앱이 일정 시간 내에 실행되지 않으면 운영체제가 강제로 종료시킵니다. 따라서 iOS 앱은 시작 절차를 최대한 빨리 완료하는 것이 매우 중요합니다.
##### 효율적인 Core Data 초기화 방법
앱이 최대한 빨리 데이터를 사용하려면 Core Data를 앱 수명 주기의 초기 단계에서 초기화해야 합니다. 하지만 Core Data 초기화는 때때로 예상보다 오래 걸릴 수 있습니다. 이로 인해 앱 시작이 지연되어 운영체제에 의해 종료되는 것을 방지하기 위해, iOS 앱의 시작 순서를 두 단계로 나누는 것이 좋습니다.

1.  **최소한의 시작**: 사용자에게 앱이 실행 중임을 알리는 최소한의 UI를 먼저 보여줍니다.
2.  **전체 UI 로딩**: Core Data 초기화가 완료된 후 애플리케이션의 전체 UI를 로드합니다.
##### 백그라운드에서 Core Data 초기화
`application:didFinishLaunchingWithOptions:` 메서드에서 Core Data를 초기화하고 다른 작업은 거의 수행하지 않는 것이 좋습니다. 특히 영구 저장소(NSPersistentStore)를 영구 저장소 코디네이터(NSPersistentStoreCoordinator)에 추가하는 작업은 시간이 얼마나 걸릴지 알 수 없으므로 **백그라운드 큐에서 비동기적으로 처리**하는 것이 중요합니다. 이 작업을 메인 큐에서 수행하면 사용자 인터페이스가 멈추어 앱이 종료될 수 있습니다. 영구 저장소가 성공적으로 추가되면 메인 큐에서 UI 관련 작업을 계속 진행할 수 있습니다.