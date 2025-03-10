https://developer.apple.com/documentation/storekit/implementing-a-store-in-your-app-using-the-storekit-api
## 왜하는데?
- 기존 StoreKit 대비 StoreKit 2는 Swift 언어의 최신 기능을 완전히 활용하며, async/await 같은 현대적인 비동기 API를 제공
- 영수증 시스템 완전 재설계
- 트랜잭션 처리 방식 변경
- 실시간 트랜잭션 동기화
- Apple은 2024년 WWDC에서 **StoreKit 1 API를 공식적으로 더 이상 사용하지 않을 것을 권장**하며, 개발자들에게 StoreKit 2로 마이그레이션할 것을 강력히 권장
## Configuration 파일 추가

Xcode에서 File -> New -> File을 해서 검색창에 store를 검색해보면 아래와 같은 템플릿을 찾을 수 있다.

![](iOS/StoreKit2/Resources/Pasted%20image%2020250122161209.png)

Next를 눌러보면 아래와 같은 창이 나올텐데 이미지에서 체크 박스가 있는 부분을 체크하면 원하는 프로젝트 (내가 갖고있는)의 AppStore Conenct에 등록되어있는 앱 내 구입 아이템을 동기화해서 가져올 수 있다. (파일에서 바꾸면 앱스토어 커넥트도 동기화된다.)

![](iOS/StoreKit2/Resources/Pasted%20image%2020250122161320.png)

암튼 이렇게 추가해준다.
## 판매 목록 가져오기

```swift
static func products<Identifiers>(for identifiers: Identifiers) async throws -> [Product] where Identifiers : Collection, Identifiers.Element == String
```

위 메소드를 이용해 인앱 아이템 목록을 가져와야한다.

서버에서 기존 StoreKit1에서 사용하던 skuCode를 가져와서 `Product.products(for: [skuCode])`로 호출해서 가져오면 해당 [skuCode]에 해당하는 Product들을 가져올 수 있다.
## 구매하기

Product 타입에 `purchase()` 라는 메소드를 이용해서 구매 가능하다. [`purchase(options:)`](https://developer.apple.com/documentation/storekit/product/purchase(options:)) 타입을 살펴보자. Product.PurhcaseResult 타입을 리턴하는 async 메소드이다.

![](iOS/StoreKit2/Resources/Pasted%20image%2020250203153358.png)

사용 방법을 간단히 살펴보면 아래와 같이 purhcase() 메소드를 호출하고 리턴타입을 확인하고 transaction 을 종료하면 된다.

verification 으로 오는 부분을 서버를 통해 한번 더 검증해야하나? (서버와 소통이 필요한 부분)

```swift
Task {
  let result = try await item.product.purchase()
            
  switch result {
    case let .success(verification):
      let transaction = try self.checkVerified(verification)
      await transaction.finish()
    return transaction
                
    case .userCancelled:
      throw StoreError.cancelledPurchase
    
    default:
      return nil
    }
}
```
## 테스트 코드 작성
- TBD
---
## Reference
- **WWDC21 Meet StoreKit 2**: https://developer.apple.com/videos/play/wwdc2021/10114/
