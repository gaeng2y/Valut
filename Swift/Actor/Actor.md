### Race condition

Actor의 장점 중 하나는 데이터 경합을 방지하는 것이다.

두 개의 개별 스레가 동시에 동일한 데이터에 액세스하거나 변경하려고 할 때 발생할 수 있는 메모리 이슈를 방지해준다.

```swift
class UserManager {
    private var users = [User.ID: User]()

    func store(_ user: User) {
        users[user.id] = user
    }

    func user(withID id: User.ID) -> User? {
        users[id]
    }
}
```

멀티 스레드 환경에서 UserManager 클래스를 인스턴스로 만들어 사용할 때 Dispatch에 대해 내부적인 변화를 수행하기에 다양한 종류의 데이터 경합인 Race Condition이 발생한다.

**결과적으로 해당 클래스는 현재 스레드로부터 안전하지 않다.**

