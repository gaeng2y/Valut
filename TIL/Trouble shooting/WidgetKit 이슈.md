AppGroup을 이용하여 UserDefaults를 사용해서 데이터를 공유하는데 위젯에서 제대로 데이터를 못가져옴

```swift
final class RepositoryImpl: Repository {
    func fetchDrink() -> Drink {
        let numberOfGlassesOfToday = UserDefaults.appGroup.integer(forKey: .glassesOfToday)
        return Drink(numberOfGlasses: numberOfGlassesOfToday)
    }
    
    func setDrink(with value: Int) {
        UserDefaults.appGroup.setValue(value, forKey: .glassesOfToday)
        self.reloadWidget()
    }
    
    private func reloadWidget() {
        WidgetCenter.shared.reloadTimelines(ofKind: .widgetKind)
    }
}
```

제대로 값을 가져오지 못하는 것 같음
