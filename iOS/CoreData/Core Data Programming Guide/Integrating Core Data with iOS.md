## Connecting the Model to Views
iOS 앱에서 **`NSFetchedResultsController`** 는 Core Data 모델과 `UITableView` 기반의 사용자 인터페이스를 연결하는 데 사용됩니다.
##### 주요 기능 및 작동 방식
`NSFetchedResultsController`는 다음의 세 가지 핵심 기능을 제공합니다.
1.  **데이터 가져오기**: `NSFetchRequest`를 사용하여 초기 데이터를 가져옵니다. 이때, 적어도 하나의 **`NSSortDescriptor`** 를 포함하여 데이터의 정렬 순서를 지정해야 합니다.
2.  **변경 사항 모니터링**: 데이터가 변경될 때마다 `delegate`를 통해 알림을 받아 `UITableView`를 자동으로 업데이트할 수 있습니다. 이를 통해 테이블 뷰와 데이터 모델의 동기화를 쉽게 관리할 수 있습니다.
3.  **성능 최적화**: 초기화 시 **`cache name`** 을 지정하여 데이터를 캐시할 수 있습니다. 이 캐시 덕분에 대규모 데이터 세트에서도 이후의 데이터 로딩이 거의 즉각적으로 이루어져 성능이 향상되고 사용자 경험이 개선됩니다.
##### 통합 방법
`NSFetchedResultsController`는 `UITableViewController` 인스턴스에서 초기화되며, `UITableViewDataSource` 메서드와 통합하여 테이블 뷰의 데이터 검색을 간소화합니다. 또한, `NSFetchedResultsControllerDelegate` 프로토콜을 구현하여 데이터 변경 시 테이블 뷰를 자동으로 업데이트하는 로직을 추가할 수 있습니다.