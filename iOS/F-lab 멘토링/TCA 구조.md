# Drink
## Store
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
### State
- numberOfGlasses: Int
- 