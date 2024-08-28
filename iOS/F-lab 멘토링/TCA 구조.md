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

# History
## UML
```plantUML
@startuml
package "Store" {
    class Action {
        +fetchHistories
    }

    class Reducer {
        +reduce(State, Action)
    }

    class State {
        +[History] histories
    }
}

class View {
        +body
    }
    
struct HealthKitClient {
   +fetchHistories
}
    
    View --> Action : sends
    Action --> Reducer : received by
    Reducer --> State : mutates
    State --> View : observe
    Effect --> Action : returns
    HealthKitClient --> Effect : returns
	HealthKitClient --> Reducer : dependency
}

@enduml
```

