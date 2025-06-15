```swift
enum MyError: Error {
    case invalid

    var localizedDescription: String {
    ~~~
    }
}
```

위와 같이 localizedDescription을 만들어서 해버리면 Error 프로토콜의 requirement가 아니기 때문에 제대로 사용할 수 없다!