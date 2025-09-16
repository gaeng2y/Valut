```swift
// RevenueCatPaywallModifier
.onDisappear {
    eventTask?.cancel()
}
```

와 같은 모디파이어가 있는데 쑥티콘 페이월 작업을 하려고 보니 

modifier가 등록된 상태에서도 이벤트를 받지 못하는 근본 원인을 발견했습니다.

🔍 주원인: AsyncStream의 Single-Consumer 특성

```swift
// PaywallEventBus.swift:29
func events() -> AsyncStream<PaywallEvent> {
return stream  // ← 문제! 같은 stream 인스턴스 반환
}
```

AsyncStream은 한 번에 하나의 consumer만 지원합니다.

1. 여러 View가 동시에 구독 시도:
    - MakeSsckticonPackView
    - SsckticonPackListView
    - MediaViewer 등
2. 첫 번째 구독자만 이벤트 수신

```swift
// 첫 번째 View만 이벤트를 받음
for await event in await PaywallEventBus.shared.events() {
// 이후 구독자들은 이벤트를 받지 못함
}
```

그래서 Combine으로 변경