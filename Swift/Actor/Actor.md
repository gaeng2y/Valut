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

액터는 스위프트에 도입된 새로운 유형의 **참조 유형**으로 **새로운 동시성 모델**을 지원한다.

액터는 **자신의 상태를 나머지 프로그램으로부터 격리**하고, 변경 가능한 상태에 대한 동기화된 액세스를 제공한다.

변경 가능한 상태에 대한 모든 액세스는 액터를 통해 이루어진다. 액터는 참조 타입으로 클래스와 많은 기능이 유사하다. 클래스와 액터의 중요한 차이점은 **자신의 상태를 격리**한다는 점이다. 

아래와 같이 `VideogameLibrary` 라는 이름의 액터를 선언했다.

```swift
actor VideogameLibrary {
	var videogames: [Videogame] = []
	func fetchGames(by company: String) -> [Videogame] {
		let games = videogames
		.filter { $0.title.caseInsensitiveCompare(company) == .orderedSame }
		return games
	}
	
	func countGames(by company: String) -> Int {
		let games = fetchGames(by: company)
		return games.count
	}
    
	func add(games: [Videogame]) {
		self.videogames.append(contentsOf: games)
	}

    func fetchGames(by year: Int) -> [Videogame] {
		let games = videogames.filter { $0.releaseYear == year }
		return games
    }
}

struct Videogame {
	let title: String
	let releaseYear: Int
	let company: String
}
```

여기서 액터가 왜 값 유형이 아니고 참조 유형일지에 대한 의문이 생길 것이다. 그 이유는 간단하다: 액터는 **공유된 변경 가능한 상태를 캡슐화**하기 때문이다.

### 액터와 상호작용

**액터에 대한 모든 호출은 비동기적**으로 수행되어야 하므로 `awiat` 키워드가 필요하다. 이는 액터가 **자신의 상태를 동기화하고 동시 변이를 방지**하는 메커니즘이다. 왜냐하면 `await` 키워드로 액터를 호출하게되면 다른 호출자들은 일시 중단되어 접근하기 위해 자신의 차례를 기다린다.

```swift
let library = VideogameLibrary()
    
func addGames(to library: VideogameLibrary) async {
    let zelda5 = Videogame(title: "The Legend of Zelda: Ocarina of Time", releaseYear: 1998, company: "Nintendo")
    let zelda6 = Videogame(title: "The Legend of Zelda: Majora's Mask", releaseYear: 2020, company: "Nintendo")
    let tales1 = Videogame(title: "Tales of Symphonia", releaseYear: 2004, company: "Namco")
    let tales2 = Videogame(title: "Tales of the Abyss", releaseYear: 2005, company: "Namco")
    let eternalSonata = Videogame(title: "Eternal Sonata", releaseYear: 2008, company: "tri-Crescendo")
    
    let games = [zelda5, zelda6, tales1, tales2, eternalSonata]
    
    await library.add(games: games)
}
```




