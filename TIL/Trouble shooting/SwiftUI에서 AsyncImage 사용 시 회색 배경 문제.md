
## 문제
Pokedex 앱을 진행 중 Kingfisher를 사용했었는데 외부 프레임워크 사용하지 않고  `AsyncImage`를 이용하면 이미지를 가져오는 동안

![[Simulator Screen Recording - iPhone 15 Pro - 2024-04-12 at 01.02.43.mp4]]

이렇게 회색 배경이 생기게 되는 이슈가 있었다.

## 해결
https://stackoverflow.com/questions/71057655/swiftui-async-image-not-loading-when-frame-size-is-below-500

AsyncImage 생성자 중

```swift
public init<I, P>(url: URL?, scale: CGFloat = 1, @ViewBuilder content: @escaping (Image) -> I, @ViewBuilder placeholder: @escaping () -> P) where Content == _ConditionalContent<I, P>, I : View, P : View
```

로 구성되어 있는 것을 이용해 `Image`와 `placeholder`를 조절해줄 수 있다.
