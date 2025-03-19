StoreKit2를 이용해 개발을 하다보면 구매 기능을 개발할 때 아래와 같이 작성하고 실행해서 구매를 해보면

```swift
Task {
    switch try await product.purchase() {
    case .success(let transaction):
        ...
    }
}
```

런타임 경고가 발생할 것이다.

>"Making a purchase without listening for transaction updates risks missing successful purchases. Create a Task to iterate Transaction.updates at launch"
#### 문제 원인
이 경고는 트랜잭션 업데이트를 감시하는 리스너 없이 구매 기능을 사용할 때 발생한다. 
구매 성공 업데이트를 놓칠 위험이 있기 때문에 애플에서는 `Transaction.updates` 를 감시하는 Task를 앱 실행 시점에 생성하도록 권장하고 있다.
#### 해결 방법
앱 시작 지점에 트랜잭션 업데이트 리스너를 추가하자

```swift
var transactionListener: Task<Void, Error>?

func setupTransactionListener() {
    transactionListener = Task(priority: .background) {
        for await update in Transaction.updates {
            do {
                let transaction = try await update.payloadValue
                // 트랜잭션 처리 로직
                await transaction.finish()
            } catch {
                // 오류 처리
            }
        }
    }
}
```

이 리스너는 앱이 실행되는 동안 백그라운드에서 트랜잭션 업데이트를 감시하며, 특히 다음과 같은 상황에서 중요하다.
- 부모님 동의가 필요한 아동용 계정의 구매
- 트랜잭션이 .pending 상태에 머무르는 경우
- 앱 외부에서 발생한 구매나 구독 변경

리스너를 가진 객체가 해제될 때는 메모리 누수를 방지하기 위해 반드시 Task를 취소해야 한다.

```swift
deinit {
    transactionListener?.cancel()
}
```

## 부록: 내가 궁금해서 찾아보는 Transaction의 static 프로퍼티들
StoreKit2의 Transaction 타입에 있는 네 가지 정적 프로퍼티는 각각 다른 목적으로 사용되는 트랜잭션 시퀀스를 제공합니다.

## Transaction.updates
Transaction.updates는 앱이 실행 중일 때 발생하는 트랜잭션 업데이트를 처리하기 위한 주요 리스너다.
- 앱 실행 직후 종료되지 않은 트랜잭션을 즉시 수신한다.
- 앱이 실행 중일 때 발생하는 모든 트랜잭션 업데이트를 지속적으로 수신한다.
- Ask to Buy 트랜잭션, 구독 리딤 코드, 앱스토어에서 발생하는 구매 등의 트랜잭션을 방출한다.
- 다른 기기에서 앱 내 구매를 완료했을 때도 트랜잭션을 방출한다.

앱 실행 초기에 이 리스너를 설정하는 것이 권장되며, 그렇지 않으면 트랜잭션을 놓칠 수 있다.
## Transaction.unfinished
Transaction.unfinished는 완료되지 않은 모든 트랜잭션을 제공한다.
- 항상 완료되지 않은 모든 트랜잭션을 포함한다.
- 주로 종료되지 않은 소모성 상품에 대한 트랜잭션을 얻는 데 사용된다.
- Transaction.updates와 달리 앱 실행 시 한 번만 전달하는 것이 아니라 항상 모든 미완료 트랜잭션을 제공한다.

사용자가 로그인한 후에 트랜잭션을 처리해야 하는 경우와 같은 특정 상황에서 유용하다.
## Transaction.all
이 시퀀스는 접근하는 순간까지의 고객의 트랜잭션 기록을 반환한다. 이 시퀀스는 유한한 수의 트랜잭션을 방출한다. 만약 여러분이 이 시퀀스에 접근하는 동안 앱 스토어가 고객을 위한 추가 트랜잭션을 처리하면, 그 트랜잭션들은 트랜잭션 리스너 updates에 나타난다.
- 완료되지 않은 소모성 아이템
- 환불되거나 취소된 완료된 소모성 아이템
- 비소모성 아이템
- 모든 갱신 내역을 포함한 자동 갱신 구독
- 고객이 패밀리 쉐어링을 통해 얻은 자동 갱신 구독 및 비소모성 아이템

즉, Transaction.all은 사용자의 모든 트랜잭션 기록을 포함하는 완전한 기록을 제공하며, 소모성/비소모성 아이템, 구독, 환불된 항목, 패밀리 쉐어링을 통해 얻은 항목 등 모든 종류의 인앱 구매 기록을 포함한다.

## Transaction.currentEntitlements
Transaction.currentEntitlements는 사용자가 현재 권한을 가진 모든 제품의 트랜잭션을 제공한다.
- 비소모성 인앱 구매(Non-consumable)에 대한 트랜잭션
- 각 활성 자동 갱신 구독의 최신 트랜잭션(Product.SubscriptionInfo.RenewalState가 .subscribed, .inGracePeriod인 것)
- 각 비갱신 구독의 최신 트랜잭션(만료된 것 포함)
소모성 인앱 구매(Consumable In-App Purchases)는 currentEntitlements에 포함되지 않는다. 또한 환불되거나 더 이상 유효하지 않은 상품도 currentEntitlements에 나타나지 않는다.

## 주요 차이점
- **Transaction.updates**: 실시간 트랜잭션 업데이트를 위한 지속적인 리스너
- **Transaction.unfinished**: 완료되지 않은 모든 트랜잭션에 대한 즉시 접근
- **Transaction.all**: 모든 트랜잭션 기록에 대한 접근
- **Transaction.currentEntitlements**: 현재 사용자가 권한을 가진 제품(활성 구독, 비소모성 구매 등)에 대한 접근

각 프로퍼티는 서로 다른 사용 사례에 맞게 설계되었으며, 앱의 요구 사항에 따라 적절한 프로퍼티를 선택해야 한다.