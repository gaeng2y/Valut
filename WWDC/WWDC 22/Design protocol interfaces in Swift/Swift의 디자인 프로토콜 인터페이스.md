ref: https://developer.apple.com/videos/play/wwdc2022/110353/
# Agenda

1. Understand type erasure
2. Hide implementation details
3. Identify type relationships
## Understand type erasure

```swift
protocol Animal {
	associatedtype CommodityType: Food
	// produce()를 추상하기 위해 associatedtype 사용
	func produce() -> CommdityType
}
struct Chicken: Animal {
	func produce() -> Egg { ... }
}
struct Cow: Animal {
	func produce() -> Milk { ... }
}
protocol Food { ... }
struct Egg: Food { ... }
struct Milk: Food { ... }
```