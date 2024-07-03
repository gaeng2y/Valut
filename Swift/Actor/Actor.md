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

이를 해결하기 위한 방법으로 모든 읽기/쓰기를 특정 DispatchQueue에 수동으로 디스패치하여 일감을 주는 것이다.

**이를 통해 UserManager 클래스의 메서드가 사용되는 스레드 혹은 큐에 관계없이 항상 직렬적으로 작업이 수행된다.**

```swift
class UserManager {
    private var users = [User.ID: User]()
    private let queue = DispatchQueue(label: "UserStorage.sync")

    func store(_ user: User) {
        queue.sync {
            self.users[user.id] = user
        }
    }

    func user(withID id: User.ID) -> User? {
        queue.sync {
            self.users[id]
        }
    }
}
```

이 보완된 코드는 이제 **데이터 경합으로부터는 방지**되었지만 여전히 문제가 있다.

해당 API를 사용해 사전 엑세스 코드를 발송하기에 **하나의 호출이 완료될 때까지는 현재 실행이 완전히 차단**되게 됩니다.

이는 많은 **동시적인 읽기/쓰기 기능을 수행하면 문제**가 발생합니다.

또한 하나가 완료될 때까지 다른 작업이 차단되기에 성능 저하와 메모리의 과도한 낭비가 생길 수 있습니다.

### 해결법

###### 1. 메서드를 **완전한 비동기**로 만드는 것

```swift
class UserManager {
    private var users = [User.ID: User]()
    private let queue = DispatchQueue(label: "UserStorage.sync")

    func store(_ user: User) {
        queue.async {
            self.users[user.id] = user
        }
    }

    func loadUser(withID id: User.ID, handler: @escaping (User?) -> Void) {
        queue.async {
            handler(self.users[id])
        }
    }
}
```

비동기로 호출하고 escaping 클로저를 이용하여 콜백으로 처리할 수 있다. 하지만 이러면 콜백 지옥에 빠지는 경우가 많다.

그래서 이러한 점들을 보완한 Actor를 살펴보자

---

## Actor
