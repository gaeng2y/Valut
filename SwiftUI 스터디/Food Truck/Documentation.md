## [Overview](https://developer.apple.com/documentation/swiftui/food_truck_building_a_swiftui_multiplatform_app#overview)
푸드트럭 앱을 사용하면 푸드트럭을 운영하는 사람이 주문을 추적하고, 가장 인기 있는 메뉴를 찾고, 목적지의 날씨를 확인할 수 있습니다.
샘플은 앱의 View들을 관리하기 위한 새로운 [NavigationSplitView](https://developer.apple.com/documentation/swiftui/navigationsplitview) , 기본 인터페이스와 보류 중인 주문을 표시하는 [Layout](https://developer.apple.com/documentation/swiftui/layout), 추세를 표시하는 [Swift Chart](https://developer.apple.com/documentation/charts), 날씨 데이터를 가져오는 [WeatherService](https://developer.apple.com/documentation/weatherkit/weatherservice)를 구현합니다.
Food Truck은 또한 잠금 화면의 ActivityKit과 홈 화면의 DynamicIsland를 통해 남은 주문 준비 시간을 표시하는 라이브 활동을 구현합니다.

> [!Note]
> 샘플코드는 WWDC22 세션: [110492: State of the Union](https://developer.apple.com/wwdc22/110492/)과 연관되어있습니다.

Food Truck 샘플 프로젝트에는 두 가지 유형의 앱 타겟이 포함되어 있습니다.
- 개인 팀 서명을 사용하여 구축할 수 있는 간단한 앱 타겟입니다. 이 앱은 시뮬레이터에서 실행되며 장치에 설치하려면 표준 Apple ID만 필요합니다. 여기에는 앱 내 구매와 사용자가 iOS 홈 화면 또는 macOS 알림 센터에 위젯을 추가할 수 있는 위젯 확장 기능이 포함되어 있습니다.
- 모든 기능을 갖춘 푸드트럭 올 앱 타겟입니다. 전체 앱은 시뮬레이터와 Apple 개발자 멤버십이 있는 장치에서 실행됩니다. 또한 암호키를 생성하고 로그인할 수도 있습니다.

