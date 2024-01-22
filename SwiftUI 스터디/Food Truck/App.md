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
@State **private** **var** selection: Panel? = Panel.truck
```
NavigationSplitView의 leftColumn에 `Sidebar`가 detailColumn엔 `DetailColumn`이 들어가게 된다.

전체적인 구조는 ContetnView에서 model, auth 관련 기능을 하위뷰에 계속 전달해준다.

### Sidebar
전체적인 구조는 간단하다

`List`를 사용하고 `NavigationLink`를 각각 넣어준다.

`Sidebar` 에서 변경된 `Panel`이 `DetailColumn`에도 변경되어 뷰가 변경된다.

`DetailColumn`은 뭔가의 동작을 하지는 않고 `selection`에 반응하는 동작만하여 해당 `Panel`에 맞는 뷰를 보여준다.

