## 위젯 구조 다이어그램

인텐트를 통해 Repository에 접근하여 업데이트

### Sequence Diagram

```uml
@startuml
actor User
User -> MulimeeWidgetEntryView: Interact
MulimeeWidgetEntryView -> MulimeeIntent: perform()
MulimeeIntent -> RepositoryImpl: fetchDrink()
RepositoryImpl -> UserDefaults: integer(forKey: .glassesOfToday)
UserDefaults -> RepositoryImpl: return numberOfGlassesOfToday
RepositoryImpl -> MulimeeIntent: return Drink
MulimeeIntent -> RepositoryImpl: setDrink(with: drink.numberOfGlasses + 1)
RepositoryImpl -> UserDefaults: setValue(value, forKey: .glassesOfToday)
RepositoryImpl -> MulimeeIntent: return 
MulimeeIntent -> MulimeeWidgetEntryView: return IntentResult
@enduml
```

![seq diagram](seq\ diagram.png)

### Class Diagram

```uml
@startuml
class MulimeeWidgetEntryView {
    - entry: Provider.Entry
    - drinkWater: Int
    + body: some View
}

class MulimeeIntent {
    + title: LocalizedStringResource
    + description: IntentDescription
    - repository: Repository
    + perform() async throws -> some IntentResult
}

interface Repository {
    + fetchDrink() -> Drink
    + setDrink(with value: Int)
}

class RepositoryImpl {
    + fetchDrink() -> Drink
    + setDrink(with value: Int)
}

class Drink {
    + numberOfGlasses: Int
}

class UserDefaults {
    + appGroup: UserDefaults
    + integer(forKey: String) -> Int
    + setValue(_: Any?, forKey: String)
}

MulimeeWidgetEntryView --> MulimeeIntent
MulimeeWidgetEntryView --> Provider.Entry
MulimeeIntent --> Repository
RepositoryImpl --|> Repository
RepositoryImpl --> Drink
UserDefaults --> Drink
@enduml
```

![class diagram](class\ diagram.png)