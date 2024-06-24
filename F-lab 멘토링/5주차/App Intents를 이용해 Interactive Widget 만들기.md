## App Intents
iOS 16 에서 등장한 앱 인텐트는, 쉽게 표현하면 **'앱의 기능을 시스템에 노출시키는 것'** 입니다. 내가 좋아하는 앱의 기능을 앱을 실행하지 않고도 쓸 수 있는 것이죠.

단축어, Siri, 그리고 위젯을 이용해서 사용할 수 있다.

## 2. 앱 인텐트 만들기

### 1단계: 프로젝트 만들기

1. Xcode를 열고 새로운 프로젝트를 만듭니다.
2. "App" 템플릿을 선택하고, 프로젝트 이름을 정합니다. 예: MyAppIntent

### 2단계: 인텐트 정의하기

인텐트는 우리가 하고 싶은 일을 정의하는 거예요. 예를 들어, 물 마시기 기록하기 같은 것.

#### 인텐트 파일 만들기

1. Xcode에서 새로운 파일을 추가한다.
2. "Intent Definition File"을 선택한다.
3. 이 파일에서 새로운 인텐트를 추가

### 3단계: 인텐트 처리하기

우리가 정의한 인텐트를 처리하는 코드를 작성

#### 코드 예제

```swift
import AppIntents

struct LogWaterIntent: AppIntent {
    static var title: LocalizedStringResource = "Log Water Intake"
    
    @Parameter(title: "Amount")
    var amount: Int

    func perform() async throws -> some IntentResult {
        // 물 마신 양을 기록하는 코드
        print("Recorded \(amount) ml of water")
        return .result()
    }
}
```

이 코드는 "Log Water Intake"라는 인텐트를 정의하고, 물 마신 양을 기록하는 기능을 해요.

### 4단계: 인텐트 추가하기

프로젝트의 `Info.plist` 파일에 인텐트를 추가해줘야 해요.

1. `Info.plist` 파일을 열고, `IntentsSupported` 항목을 추가합니다.
2. 방금 만든 인텐트 이름을 여기에 적어줍니다. 예: LogWaterIntent

### 5단계: 인텐트 사용하기

이제 시리나 단축어 앱을 통해서 우리가 만든 인텐트를 사용할 수 있어요. 예를 들어, "시리야, 200ml 물 마시기 기록해줘"라고 말하면 우리의 앱이 물 마신 양을 기록해줄 거예요.

이제 여러분도 앱 인텐트를 만들어서 시리와 같은 멋진 기능을 사용할 수 있게 되었어요!

## Interactive Widget

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