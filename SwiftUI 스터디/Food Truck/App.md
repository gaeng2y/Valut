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
