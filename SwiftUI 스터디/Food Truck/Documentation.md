## [Overview](https://developer.apple.com/documentation/swiftui/food_truck_building_a_swiftui_multiplatform_app#overview)
푸드트럭 앱을 사용하면 푸드트럭을 운영하는 사람이 주문을 추적하고, 가장 인기 있는 메뉴를 찾고, 목적지의 날씨를 확인할 수 있습니다.
샘플은 앱의 View들을 관리하기 위한 새로운 [NavigationSplitView](https://developer.apple.com/documentation/swiftui/navigationsplitview) , 기본 인터페이스와 보류 중인 주문을 표시하는 [Layout](https://developer.apple.com/documentation/swiftui/layout), 추세를 표시하는 [Swift Chart](https://developer.apple.com/documentation/charts), 날씨 데이터를 가져오는 [WeatherService](https://developer.apple.com/documentation/weatherkit/weatherservice)를 구현합니다.
Food Truck은 또한 잠금 화면의 ActivityKit과 홈 화면의 Dynamic Island를 통해 남은 주문 준비 시간을 표시하는 라이브 활동을 구현합니다.

> [!Note]
> 샘플코드는 WWDC22 세션: [110492: State of the Union](https://developer.apple.com/wwdc22/110492/)과 연관되어있습니다.

Food Truck 샘플 프로젝트에는 두 가지 유형의 앱 타겟이 포함되어 있습니다.
- 개인 팀 서명을 사용하여 구축할 수 있는 간단한 앱 타겟입니다. 이 앱은 시뮬레이터에서 실행되며 장치에 설치하려면 표준 Apple ID만 필요합니다. 여기에는 앱 내 구매와 사용자가 iOS 홈 화면 또는 macOS 알림 센터에 위젯을 추가할 수 있는 위젯 확장 기능이 포함되어 있습니다.
- 모든 기능을 갖춘 푸드트럭 올 앱 타겟입니다. 전체 앱은 시뮬레이터와 Apple 개발자 멤버십이 있는 장치에서 실행됩니다. 또한 암호키를 생성하고 로그인할 수도 있습니다.

### ### [Construct a dynamic layout](https://developer.apple.com/documentation/swiftui/food_truck_building_a_swiftui_multiplatform_app#4143587)
TruckView의 New Orders 패널에는 가장 최근 주문 5개가 표시되고, 각 주문에는 도넛 축소판의 대각선 스택인 DonutStackView가 표시됩니다.

Layout 프로토콜을 사용하면 앱이 도넛 썸네일을 대각선 레이아웃으로 정렬하는 DiagonalDonutStackLayout을 정의할 수 있습니다.

레이아웃의 placeSubviews(in:proposal:subviews:cache:) 구현은 도넛의 위치를 ​​계산합니다.

```swift
for index in subviews.indices {
    switch (index, subviews.count) {
    case (_, 1):
        subviews[index].place(
            at: center,
            anchor: .center,
            proposal: ProposedViewSize(size)
        )
        
    case (_, 2):
        let direction = index == 0 ? -1.0 : 1.0
        let offsetX = minBound * direction * 0.15
        let offsetY = minBound * direction * 0.20
        subviews[index].place(
            at: CGPoint(x: center.x + offsetX, y: center.y + offsetY),
            anchor: .center,
            proposal: ProposedViewSize(CGSize(width: size.width * 0.7, height: size.height * 0.7))
        )
    case (1, 3):
        subviews[index].place(
            at: center,
            anchor: .center,
            proposal: ProposedViewSize(CGSize(width: size.width * 0.65, height: size.height * 0.65))
        )
        
    case (_, 3):
        let direction = index == 0 ? -1.0 : 1.0
        let offsetX = minBound * direction * 0.15
        let offsetY = minBound * direction * 0.23
        subviews[index].place(
            at: CGPoint(x: center.x + offsetX, y: center.y + offsetY),
            anchor: .center,
            proposal: ProposedViewSize(CGSize(width: size.width * 0.7, height: size.height * 0.65))
```

### [Display a chart of popular items](https://developer.apple.com/documentation/swiftui/food_truck_building_a_swiftui_multiplatform_app#4143588)
샘플에는 여러 차트가 포함되어 있습니다. 가장 인기 있는 항목은 TopFiveDonutsView에 표시됩니다. 이 차트는 BarMark를 사용하여 막대 차트를 구성하는 TopDonutSalesChart에서 구현됩니다.

```swift
Chart {
    ForEach(sortedSales) { sale in
        BarMark(
            x: .value("Donut", sale.donut.name),
            y: .value("Sales", sale.sales)
        )
        .cornerRadius(6, style: .continuous)
        .foregroundStyle(.linearGradient(colors: [Color("BarBottomColor"), .accentColor], startPoint: .bottom, endPoint: .top))
        .annotation(position: .top, alignment: .top) {
            Text(sale.sales.formatted())
                .padding(.vertical, 4)
                .padding(.horizontal, 8)
                .background(.quaternary.opacity(0.5), in: Capsule())
                .background(in: Capsule())
                .font(.caption)
        }
    }
}
```

차트의 x축에는 각 데이터 포인트에 해당하는 항목의 이름과 축소판 그림이 포함된 레이블이 표시됩니다.

```swift
.chartXAxis {
    AxisMarks { value in
        AxisValueLabel {
            let donut = donutFromAxisValue(for: value)
            VStack {
                DonutView(donut: donut)
                    .frame(height: 35)
                    
                Text(donut.name)
                    .lineLimit(2, reservesSpace: true)
                    .multilineTextAlignment(.center)
            }
            .frame(idealWidth: 80)
            .padding(.horizontal, 4)
            
        }
    }
}
```

### [Obtain a weather forecast](https://developer.apple.com/documentation/swiftui/food_truck_building_a_swiftui_multiplatform_app#4143589)
앱은 `TruckView`의 `Forecast` 패널에 예측 온도 그래프를 표시합니다. 앱은 WeatherKit 프레임워크에서 이 데이터를 얻습니다.
```swift
.task(id: city.id) {
    for parkingSpot in city.parkingSpots {
        do {
            let weather = try await WeatherService.shared.weather(for: parkingSpot.location)
            condition = weather.currentWeather.condition
            willRainSoon = weather.minuteForecast?.contains(where: { $0.precipitationChance >= 0.3 })
            cloudCover = weather.currentWeather.cloudCover
            temperature = weather.currentWeather.temperature
            symbolName = weather.currentWeather.symbolName
            
            let attribution = try await WeatherService.shared.attribution
            attributionLink = attribution.legalPageURL
            attributionLogo = colorScheme == .light ? attribution.combinedMarkLightURL : attribution.combinedMarkDarkURL
            
            if willRainSoon == false {
                spot = parkingSpot
                break
            }
        } catch {
            print("Could not gather weather information...", error.localizedDescription)
            condition = .clear
            willRainSoon = false
            cloudCover = 0.15
        }
    }
}
```

WeatherKit WWDC를 보니 눈에 띄는게 보이네요
```swift
let attribution = try await WeatherService.shared.attribution
            attributionLink = attribution.legalPageURL
            attributionLogo = colorScheme == .light ? attribution.combinedMarkLightURL : attribution.combinedMarkDarkURL
```

음 우리꺼 쓰려면 여기다 표시해~