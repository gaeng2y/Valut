https://sarunw.com/posts/xcode-previews-with-uiview/

To make your views work with Xcode previews, **you need to do three things**.

1. Import `SwiftUI` module.
2. Create a struct that conforms to the `PreviewProvider` protocol.
3. [Wrap a view into a SwiftUI view using `UIViewRepresentable`](https://sarunw.com/posts/uiview-in-swiftui/).

위의 순서대로 진행을 하면된다.

1. Xcode preview works hand in hand with `PreviewProvider`. It discovers preview providers and generates previews for us. `PreviewProvider` is a part of the `SwiftUI` module, **so we need to `import SwiftUI` to use it**.  
2. We create a struct that conforms to the `PreviewProvider`.  
3. Normally, we will return a SwiftUI view from the `previews` static property. But we want to preview a view here, so we need to wrap it using the `UIViewRepresentable` protocol.
### UIViewRepresentable
First, we need to convert a view into a SwiftUI view using `UIViewRepresentable`.

The process is straightforward. You just need to initialize your view and return it from `makeUIView()`. You can read more about this in detail in [how to convert a view to SwiftUI](https://sarunw.com/posts/uiview-in-swiftui/).

```swift
import SwiftUI
import UIKit

// 1. I create a generic struct that can accept any type that is a subclass of `UIView`. We make this struct conform to `UIViewRepresentable` since it will act as our SwiftUI view wrapper for our view.
struct PreviewContainer<T: UIView>: UIViewRepresentable {
    // 2. We define an instance property that will hold a view.
    let view: T
    
    // 3. We create an initializer that accepts a closure that returns a view. We execute it to get the resulting view.
    init(_ viewBuilder: @escaping () -> T) {
        view = viewBuilder()
    }
    
    // MARK: - UIViewRepresentable
    func makeUIView(context: Context) -> T {
        // 4. We return the resulting view from `makeUIView()`.
        return view
    }
    
    // 5. We set content hugging priority to `.defaultHigh` to mitigate the limitation mentioned in the previous section.
    func updateUIView(_ view: T, context: Context) {
        view.setContentHuggingPriority(.defaultHigh, for: .horizontal)
        view.setContentHuggingPriority(.defaultHigh, for: .vertical)
    }
}
```

### 사용법

```swift
struct SomeUIView_Preview: PreviewProvider {
    static var previews: some View {
        PreviewContainer {
            return SomeUIView()
        }
    }
}
```

