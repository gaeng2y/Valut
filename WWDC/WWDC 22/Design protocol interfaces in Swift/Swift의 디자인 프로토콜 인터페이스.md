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

associatedtype을 사용하여 구체 타입의 Animal을 선언하고 `produce()` 메소드를 호출하면 구체 Animal 타입에 따라 특정 타입의 Food 가 반환되는데 이러한 관계는 다이어그램으로 나타낼 수 있다.

![](WWDC/WWDC%2022/Design%20protocol%20interfaces%20in%20Swift/Resources/Pasted%20image%2020241101114530.png)

프로토콜 `Self` 타입은 `Animal` 프로토콜을 따르는 실제 구체 타입을 나타내며 `Self` 타입에는 `Food` 를 따르는 associatedtype `CommodityType` 가 있다.

구체적인 Chicken 및 Cow 타입 간의 관계와 Animal 프로토콜에 대한 associtaedtype 다이어그램도 살펴보자

![](WWDC/WWDC%2022/Design%20protocol%20interfaces%20in%20Swift/Resources/Pasted%20image%2020241101114740.png)
![](WWDC/WWDC%2022/Design%20protocol%20interfaces%20in%20Swift/Resources/Pasted%20image%2020241101114815.png)

```swift
struct Farm {
	var animals: [any Animal]

	func produceCommodities() -> [any Food] {
		animals.map { $0.produce() }
	}
}
```

Farm의 `animals` 프로퍼티는 `[any Animal]` 이다. `[any Animal]`를 이용하는 것은 구체 타입에 대한 동일한 표현을 사용하는 전략을 타입 소거(Type erasure)라고 한다. 

`produceCommodities()` 메소드를 호출하면 연관타입에 의해 Food의 구체타입이 정해져서 return 되기때문에 `produceCommodities()` 메소드의 return 타입도 `[any Food] `여야 한다.

실제 Animal 타입에 대한 정적 타입 관계를 제거함을 알고 있으므로 이러한 타입을 검사하는 이유를 더 깊이 이해할 필요가 있다.

`map()` 클로저의 $0 매개변수는 `any Animal` 타입이며 `produce()` 의 반환 타입은 associatedtype이다. 위 다이어그램처럼 실존 타입에 대해 연관 타입을 반환하는 메소드를 호출하면 컴파일러는 타입 소거를 사용하여 호출의 결과 타입을 결정한다. 

![](WWDC/WWDC%2022/Design%20protocol%20interfaces%20in%20Swift/Resources/Pasted%20image%2020241101115733.png)

타입 소거는 이러한 연관 타입을 동등한 제약 조건을 가진 일치하는 실존 타입으로 대체한다. 

구체적인 Animal 타입과 연관 CommdodityType 간의 관계를 `any Animal`과 `any Food`로 대체하여 제거했다.

`any Food` 타입은 CommodityType의 상한 타입이라고 하는데 `any Animal`에 대해 `produce()` 메소드가 호출됐기 때문에 반환 값은 타입 소거되어 `any Food` 타입의 값이 반환된다.

![](WWDC/WWDC%2022/Design%20protocol%20interfaces%20in%20Swift/Resources/Pasted%20image%2020241101120903.png)

Cow의 `produce()` 메소드는 `Milk`를 반환하며 `Milk`는 `Animal`프로토콜과 관련된 연관 타입의 상한인 `any Food`내에 저장될 수 있다.

![](WWDC/WWDC%2022/Design%20protocol%20interfaces%20in%20Swift/Resources/Pasted%20image%2020241101121027.png)

Animal 프로토콜을 따르는 모든 구체 타입에 대해 항상 안전하다. 반면 메소드나 생성자의 매개변수 목록에 연관타입이 나타나면 어떻게 되는지 보자.

![](WWDC/WWDC%2022/Design%20protocol%20interfaces%20in%20Swift/Resources/Pasted%20image%2020241101121645.png)

여기에서 Animal 프로토콜의 `eat(_:)`는 소비 위치에 연관 타입인 FeedType을 갖고 있다.

소비 위치에 연관 타입이 들어있으면 타입 소거를 수행할 수 없으며 구체 타입을 알 수 없기에 연관 타입의 상한 실존 타입은 실제 구체 타입으로 안전하게 변환되지 않는다.

![](WWDC/WWDC%2022/Design%20protocol%20interfaces%20in%20Swift/Resources/Pasted%20image%2020241101122415.png)

FeedType의 상한은 `any AnimalFeed`이지만 ??? 부분에 임의의 `any AnimalFeed`가 주어질 경우 Cow에 필요한 `Hay`인지 정적으로 보장할 수 없다는 것이다.

타입 소거는 소비 위치에서 연관 타입을 사용하는 것을 허용하지 않는다. 그 대신 불분명한 `some` 타입을 취하는 함수에 입력함으로써 실존하는 `any`타입을 언박싱해야 한다.

연관 타입의 이러한 타입 소거 동작은 