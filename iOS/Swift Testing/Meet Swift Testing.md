https://developer.apple.com/videos/play/wwdc2024/10179
(https://developer.apple.com/videos/play/wwdc2024/10179?time=126)
- Swift 코드를 테스트할 수 있는 새로운 도구
- 오픈 소스다

![](iOS/Swift%20Testing/Pasted%20image%2020250407103247.png)
- 테스트를 설명하고 구성
- 장애 발생 시 대처를 위한 세부 정보 제공
- 확장성

![](iOS/Swift%20Testing/Pasted%20image%2020250407103331.png)
- Swift에 맞춰줘서 동시성 및 매크로 같은 최신 기능 사용 가능
## Building blocks

@Test 매크로를 붙여주면 테스트 코드로 전환된다
```swift
import Testing

@Test func videoMetadata() {
    // ...
}
```


## Common workflows
## Swift Testing and XCTest