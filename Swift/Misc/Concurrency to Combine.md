```swift
 Deferred { [loader] in
           Future { promise in
               Task {
                   // concurrency 코드
               }
           }
       }
       .eraseToAnyPublisher()
```