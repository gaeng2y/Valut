# Drink
## Store
- var numberOfGlasses: Int
- var liter: Double
- var progress: CGFloat
- var isDisableDrinkButton: Bool
- var drinkButtonTtile: String
- 
```swift
struct State {
	var numberOfGlasses = 0
	var litter: Double {
		0.25 * Double(numberOfGlasses)
	}
	var progress: CGFloat {
		0.125 * CGFloat(numberOfGlasses)
	}
}
```

### Action


### Reducer
```swift
var body: some Reducer<State, Action> {
	Reduce
}
```

### View
```swift
struct DrinkWaterView: View {
	let store: StoreOf<DrinkWaterFeature>

	var body: some View 
}
```