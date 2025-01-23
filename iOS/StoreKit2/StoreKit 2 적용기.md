https://developer.apple.com/documentation/storekit/implementing-a-store-in-your-app-using-the-storekit-api
## 왜하는데?
- 기존 StoreKit 대비 StoreKit 2는 Swift 언어의 최신 기능을 완전히 활용하며, async/await 같은 현대적인 비동기 API를 제공
- 영수증 시스템 완전 재설계
- 트랜잭션 처리 방식 변경
- 실시간 트랜잭션 동기화
- Apple은 2024년 WWDC에서 **StoreKit 1 API를 공식적으로 더 이상 사용하지 않을 것을 권장**하며, 개발자들에게 StoreKit 2로 마이그레이션할 것을 강력히 권장
## Configuration 파일 추가

Xcode에서 File -> New -> File을 해서 검색창에 store를 검색해보면 아래와 같은 템플릿을 찾을 수 있다.

![](iOS/StoreKit2/Pasted%20image%2020250122161209.png)

Next를 눌러보면 아래와 같은 창이 나올텐데 이미지에서 체크 박스가 있는 부분을 체크하면 원하는 프로젝트 (내가 갖고있는)의 AppStore Conenct에 등록되어있는 앱 내 구입 아이템을 동기화해서 가져올 수 있다. (파일에서 바꾸면 앱스토어 커넥트도 동기화된다.)

![](iOS/StoreKit2/Pasted%20image%2020250122161320.png)

암튼 이렇게 추가해준다.
## 판매 목록 가져오기

```swift
static func products<Identifiers>(for identifiers: Identifiers) async throws -> [Product] where Identifiers : Collection, Identifiers.Element == String
```

위 메소드를 이용해 인앱 아이템 목록을 가져와야한다.

서버에서 기존 StoreKit1에서 사용하던 skuCode를 가져와서 `Product.products(for: [skuCode])`로 호출해서 가져오면 해당 [skuCode]에 해당하는 Product들을 가져올 수 있다.

---
## Reference
- **WWDC21 Meet StoreKit 2**: https://developer.apple.com/videos/play/wwdc2021/10114/
