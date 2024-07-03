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

func addNewGames(to library: VideogameLibrary) {
    let pokemon1 = Videogame(title: "Pokémon Yellow", releaseYear: 1998, company: "Game Freak")
    let pokemon2 = Videogame(title: "Pokémon Gold", releaseYear: 1999, company: "Game Freak")
    let pokemon3 = Videogame(title: "Pokémon Ruby", releaseYear: 2002, company: "Game Freak")
    let games = [pokemon1, pokemon2, pokemon3]
    Task {
        await library.add(games: games)
    }
}
```

그 후 아래와 같이 두 메소드를 호출해보면

```swift
addGames(to: library)
addNewGames(to: library)
```

`addGames` 와 `addNewGames` 두 메소드 중 어떤 것이 먼저 추가될지 보장할 수 없다.

하지만 보장되는 것은 어느 메소드가 먼저 호출되더라도 먼저 호출된 메소드가 끝나기 전까지는 뒤에 호출될 메소드가 배열에 접근할 수 없다는 것이다.

또한 전체 액터 상태에 대한 액세스가 동기화된다. 만약 라이브러리에 게임을 추가하는 사람과 컬렉션을 쿼리하는 사람이 동시에 있다면, 액터는 실행이 완료될 때까지 그 호출 중 하나만 실행한다.

또 다시 예를 들어보면

```swift
Task {
    addGames(to: library)
}

Task {
    let games1998 = await library.fetchGames(by: 1998)
    print("\(games1998.count) games")
}

Task {
    addGames(to: library)
}
```

일반적으로 위의 코드가 2개의 게임을 출력하지만, 태스크가 동시에 액터에 접근하려고 시도하기 때문에
액터에 동시에 접근하려고 시도하기 때문에, 세 개의 태스크 중 하나만 해당 액터의 콘텐츠를 먼저 실행하게 된다.

코드를 실제로 실행해보면 대부분 `0 games` 라고 나오지만, `1 games`가 나올 수도 있다.

상호 배타적 접근은 한 사람만 액터에 들어가서 작업을 수행한다는 의미이며, 작업을 수행하면 어떤 액터가 호출되는지는 중요하지 않다. 액터에 대한 모든 호출은 작업을 완료할 때까지 차단한다.

이는 작업 우선순위 동작이 어떻게 동작하는 지 보여줄 수 있는 좋은 예다.

### 액터에 대한 비격리 접근

액터에 접근할 때 데이터 레이스가 발생하지 않을 것이라는 것을 알고 있는 경우가 있다. 이러한 경우에는 액터의 일부 메소드를 `nonisolated`로 표시할 수 있다.

```swift
struct Owner {
    let name: String
    let favoriteGenre: String
}

actor VideogameLibrary {
    var videogames: [Videogame] = []
    let owner: Owner
    
    init(owner: Owner) {
        self.owner = owner
    }
    
    func fetchOwnerInfo() -> String {
        return "\(owner.name) (\(owner.favoriteGenre)"
    }

	// ...
}
```

`fetchOwnerInfo` 메소드가 추가되었다. 하지만 이 예에서는 소유자 정보에 동기적으로 액세스하는 것은 위험하지 않다. 왜냐하면 불변하는 구조체이기 때문이다. 그래서 우리는 `await` 키워드를 이용할 필요가 없어졌다. 그래서 `nonisolated` 키워드를 붙여주면 Task에 감싸거나 await 키워드를 붙여서 호출할 필요가 없어진다.

### 액터와 프로토콜 준수

그냥 일반 프로토콜 채택하는 것처럼 채택하면 되고 `nonisoloated` 도 잘 붙여주면 된다.

### 액터 재진입
