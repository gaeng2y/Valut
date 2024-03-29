```swift
WindowGroup {
    ContentView(model: model, accountStore: accountStore)
}
#if os(macOS)
.defaultSize(width: 1000, height: 650)
#endif
        
#if os(macOS)
MenuBarExtra {
	ScrollView {
		VStack(spacing: 0) {
			BrandHeader(animated: false, size: .reduced)
            Text("Donut stuff!")
		}
    }
} label: {
	Label("Food Truck", systemImage: "box.truck")
}
.menuBarExtraStyle(.window)
#endif
```

macOS일 때는 기본 사이즈와 상단 메뉴바에서 사용가능한 기능을 넣어준다.

## ContentView
```swift 
@State private var selection: Panel? = Panel.truck
```
NavigationSplitView의 leftColumn에 `Sidebar`가 detailColumn엔 `DetailColumn`이 들어가게 된다.

전체적인 구조는 ContetnView에서 model, auth 관련 기능을 하위뷰에 계속 전달해준다.

### Sidebar
전체적인 구조는 간단하다

`List`를 사용하고 `NavigationLink`를 각각 넣어준다.

`Sidebar` 에서 변경된 `Panel`이 `DetailColumn`에도 변경되어 뷰가 변경된다.

### DetailColumn

`DetailColumn`은 뭔가의 동작을 하지는 않고 `selection`에 반응하는 동작만하여 해당 `Panel`에 맞는 뷰를 보여준다.


#### TruckView

```swift
WidthThresholdReader(widthThreshold: 520) { proxy in
	ScrollView(.vertical) {
		VStack(spacing: 16) {
			BrandHeader()
            
            Grid(horizontalSpacing: 12, verticalSpacing: 12) {
				if proxy.isCompact {
					orders
					weather
					donuts
					socialFeed
                } else {
	                GridRow {
			            orders
				        weather
				    }
				    GridRow {
					    donuts
						socialFeed
					}
                }
	        }
            .containerShape(RoundedRectangle(cornerRadius: 12, style: .continuous))
            .fixedSize(horizontal: false, vertical: true)
            .padding([.horizontal, .bottom], 16)
            .frame(maxWidth: 1200)
                }
            }
        }
```

기본적인 구조는 이렇다

우선 `WidthThresholdReader` 에 대해 먼저 살펴보자.

#### WidthThresholdReader

상단 주석을 살펴보니

자식 뷰가 수평으로 압축된 것처럼 동작해야 하는지 여부를 결정하는 데 유용한 뷰
뷰가 압축되었는지 여부를 결정하는 데에 필요한 요소
* Width
* Dynamic Type Size
* Horizontal size class (on iOS)

라고 되어있네요.

그 안에 구조체로 `WidthThresholdProxy` 이 선언되어있습니다. 
```swift
struct WidthThresholdProxy: Equatable {
    var width: Double
    var isCompact: Bool
}
```

Proxy를 이용해 width에 따라 isCompact인지 아닌지 체크하는 것 같음,,,

다시 **TruckView**로 넘어와서

`WidthThresholdReader` 안에 뷰를 만들어준다,,,

```swift
.navigationDestination(for: Panel.self) { panel in
	switch panel {
	case .orders:
		OrdersView(model: model)
	case .donuts:
		DonutGallery(model: model)
	case .socialFeed:
		SocialFeedView()
	case .city(let id):
		CityView(city: City.identified(by: id))
	default:
		DonutGallery(model: model)
	}
}
```
에서 이미 해당 뷰가 navigation안에 들어가있으니 navigationSelection에 따라 바뀌면서 적용되는거로 보임

**!! 위의 내용이 맞는지 확인하기**

`Card`는 `TruckView`에서 Grid의 뷰로 사용할 녀석들

### OrdersView
[Table](https://developer.apple.com/documentation/swiftui/table) 을 이용하여 구성

[List](https://developer.apple.com/documentation/swiftui/list)와는 어떻게 다른지 비교해봤는데

List는 UIKit에서 UITableView로 구성되는 부분이고 Table은 약간 엑셀 느낌..?으로 

OrderTable - OrderRow로 이어지는 화면,,,

### DonutGallery

DonutGallery 안에 Grid || Table로 보여준다.
위의 바버튼을 누르면 테이블 형태로 바꿀 수 있다.
각 또한 상단에 UIBarButtonItem 을 넣는 대신 SwiftUI에선 
```swift
#if os(iOS)
        .toolbarRole(.browser)
        #endif
        .toolbar {
            ToolbarItemGroup {
                toolbarItems
            }
        }
```
위와같이 간단하게 넣을 수 있다.

근데 `.toolbarRole`은 뭐지하고 살펴봤는데
Configures the semantic role for the content populating the toolbar. 라고 되어있다.

해당 모디파이어는 [ToolBarRole](https://developer.apple.com/documentation/swiftui/toolbarrole)의 타입을 파라미터로 받는데 해당 타입은 도구 모음을 채우는 콘텐츠의 목적이라는 뜻을 가진다.

### CityView
ParkingSpotShowcaseView 또한 TimelineView를 이용한다.(BrandHeader와 동일하게)
