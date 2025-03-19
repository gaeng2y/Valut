StoreKit2를 이용해 개발을 하다보면 구매 기능을 개발할 때 

```swift
Task {
    switch try await product.purchase() {
    case .success(let transaction):
        
    }
}
```