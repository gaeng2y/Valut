## @State
1. SwiftUI에서 상태를 처리하는 방법
2. 뷰의 상태를 저장하는 프로퍼티로 상태 관리 주체는 해당 뷰
3. 기본적으로 private 선언이기에 다른 뷰와 값을 소통하려면 Binding을 이용
4. 값이 변경될 때마다 UI 업데이트
## @Binding
1. 뷰와 상태를 바인딩하는 방법
2. 상위 @State 변수를 전달 받아 하위 뷰에서 캐치해 변화 감지 및 연결
3. Binding은 다른 뷰가 소유한 속성을 연결하기에 소유권 및 저장 공간이 없음
## ObservableObject
1. 클래스 프로토콜로 관찰하는 어떠한 값이 변경되면 변경 사항을 알려줌
2. 뷰에서 인스턴스 변화를 감시하기 위해 뷰 모델 객체로 생성할 때 사용할 수 있음
## @Published
1. ObservableObject를 구현한 클래스 내에서 프로퍼티 선언 시 사용
2. @Published로 선언된 프로퍼티를 뷰에서 관찰할 수 있음
3. ObservableObject의 objectWillChange.send() 기능을 @Published 프로퍼티가 변경되면 자동으로 호출
## @ObservedObject
1. 뷰에서 ObservableObject 타입의 인스턴스 선언 시 사용
2. ObservableObject의 값이 업데이트되면 뷰를 업데이트
## @StateObject
1. 뷰에서 ObservableObject 타입의 인스턴스 선언 시 사용
2. 뷰마다 하나의 인스턴스를 생성하며, 뷰가 사라지기 전까지 같은 인스턴스 유지
3. @ObservedObject의 뷰 렌더링 시 인스턴스 초기화 이슈 해결을 위한 방법
4. 매번 인스턴스가 새롭게 생성되는 것처럼 외부에서 주입 받는 경우가 아닌 최초 생성 선언 시에 @StateObject를 사용하는 것이 적절함
## @Environment
1. 미리 정의되어 있는 시스템 공유 데이터
2. 사용하려는 공유 데이터의 이름을 KeyPath로 전달하여 사용
3. 시스템 공유 데이터는 가변하기에 var로 선언 필요
4. 뷰가 생성되는 시점에 값이 자동으로 초기화됨
## @EnvironmentObject
1. ObservableObject를 통해 구현된 타입의 인스턴스를 전역적으로 공유하여 사용
2. 앱 전역에 공통으로 사용할 데이터를 주입 및 사용