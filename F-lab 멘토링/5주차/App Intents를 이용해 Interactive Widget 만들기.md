iOS 16 에서 등장한 앱 인텐트는, 쉽게 표현하면 **'앱의 기능을 시스템에 노출시키는 것'** 입니다. 내가 좋아하는 앱의 기능을 앱을 실행하지 않고도 쓸 수 있는 것이죠.

단축어, Siri, 그리고 위젯을 이용해서요.

인터랙티브 위젯은 App Intents 로 만들어진 앱의 기능을 **위젯 버튼**에 연결한 것 뿐입니다.
## 앱
```swift
struct DrinkWaterIntent: AppIntent {
    
    static var title: LocalizedStringResource = "Drink Water"
    static var description = IntentDescription("Glasses of Today counter")
    
    func perform() async throws -> some IntentResult {
        GlassesCounter.countUp()
        return .result()
    }
}
```

라는 `AppIntent` 프로토콜을 채택한 타입을 만든다. 그 후 `perform` 메소드에 내가 원하는 구현을 한다.

그 후 Widget에 버튼을 연결한다면
```swift
Button(intent: DrinkWaterIntent()) {
	Text("마시기")
	.font(.system(size: 10))
	.foregroundColor(.white)
}
```

이런 형태로 생성자로 intent를 할당할 수 있다.

그런 후 위젯에서 버튼을 실행한다면 perform 메소드가 호출이된다.