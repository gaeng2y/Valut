액터들은 스위프트에 도입된 새로운 유형의 참조 유형이며, 새로운 동시성 모델이 지원한다. 

액터들은 프로그램의 나머지 부분으로부터 자신의 상태를 격리하고, 가변 상태에 대한 동기화된 접근을 제공한다. 

좀 더 간단히 말하면, 여러 작업이 읽고 쓸 수 있는 간단한 변수를 가진 행위자가 있다면, 작업은 그들 중 한 명만이 한 번에 그렇게 할 수 있도록 보장한다. 주어진 자원에 대한 상호 접근을 보장한다.

가변 상태에 대한 모든 접근은 행위자를 통해 이루어진다. 행위자를 선언하기 위해서는 단순히 행위자 키워드를 사용하고 이름을 부여하는 것으로 매우 유사하다

구조체 또는 클래스를 선언하는 것. 참조 유형인 행위자는 클래스의 특징과 같은 많은 특징을 갖는다. 그들은 프로토콜을 준수할 수 있고 확장을 통해 더 많은 기능을 제공받을 수 있다. 클래스와의 주요 차이점은 행위자가 자신의 상태를 격리한다는 것이다. 행위자는 0회 이상 중단할 수 있으므로(다른 대기 호출이 그 메소드 내에 있음), 행위자를 통합할 수 있다
새로운 동시성 시스템의 나머지 부분은 쉽게 사용할 수 있습니다.
목록 6-1은 Actor를 선언하는 방법을 보여준다. 이 경우 우리는 이 장에서 예를 들어 사용할 비디오게임 구조도 만들 것이다

```swift
actor VideogameLibrary {
	var videogames: [Videogame] = []
	func fetchGames(by company: String) -> [Videogame] {
		let games = videogames.filter { 
			$0.title.caseInsensitiveCompare(company) == .orderedSame 
		}
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
```

우리는 그것의 선언이 클래스와 매우 유사해 보인다는 것을 인식할 수 있고, 그것은 참조형이기 때문에 그것의 행동이 유사할 것을 기대할 수 있다. 
왜 액터가 값 타입이 아니라 참조형인지 궁금할 수도 있다.
그 이유는 간단하다. 행위자가 공유 가변 상태를 캡슐화한다. 다수의 코드 경로에 의해 변이될 것으로 예상되는 유형이다. 구조였다면,
전화를 거는 사람들은 결국 자신의 복사본으로 끝날 것이다.

## Interacting with an Actor
액터에 대한 모든 호출은 비동기적으로 수행되어야 하므로 await 키워드가 필요합니다. 
이것은 액터들이 그들의 상태를 동기화하고 동시에 발생하는 돌연변이를 방지하기 위해 사용하는 메커니즘이다. 액터에 대한 호출은 기다릴 수 있기 때문에, 다른 호출자들은 그것에 접근할 차례를 기다리며 중단될 것이다.
내적으로는 행위자가 자신의 메소드와 속성을 비동기적으로 호출할 필요가 없다. 행위자가 고립되어 있기 때문에 내부의 어떤 호출도 작업이 완료될 때까지 작업 자체가 순차적으로 실행됩니다.

목록 6-3은 비디오 게임 라이브러리에 게임을 추가하는 방법을 보여줍니다.

```swift
let library = VideogameLibrary()

func addGames(to library: VideogameLibrary) {
	let zelda5 = Videogame(title: "The Legend of Zelda: Ocarina of Time", releaseYear: 1998, company: "Nintendo")
	let zelda6 = Videogame(title: "The Legend of Zelda: Majora's Mask", releaseYear: 2020, company: "Nintendo")
	let tales1 = Videogame(title: "Tales of Symphonia", releaseYear: 2004, company: "Namco")
	let tales2 = Videogame(title: "Tales of the Abyss", releaseYear: 2005, company: "Namco")
	let eternalSonata = Videogame(title: "Eternal Sonata", releaseYear: 2008, company: "tri-Crescendo")

    let games = [zelda5, zelda6, tales1, tales2, eternalSonata]

    library.add(games: games)
}
```

addGames 메소드는 단지 빠르게 추가하기 위한 편의 기능일 뿐입니다.
일부 비디오 게임. 그러나 이를 컴파일하려고 하면 "액터 격리된 인스턴스 메서드 'add(games:)'는 격리되지 않은 컨텍스트에서 참조할 수 없습니다."라는 오류가 발생합니다.

이 메서드는 우리가 await 키워드를 추가할 것으로 기대하고 있습니다. 사실에도 불구하고
add(games:) 메소드가 비동기로 표시되지 않았으므로 우리에게 노출된 API는 행위자에 의해 제공되는 보호로서 실제로 비동기입니다. 컴파일러는 데이터 경합의 도입을 방지하여 액터를 오용하는 데 도움을 줍니다.
이에 대한 직접적인 의미는 액터가 비동기식으로만 실행될 수 있다는 것입니다.
따라서 궁극적으로 이를 컴파일하려면 add(games:) 메소드에 대한 키워드를 기다리고 이를 Task(또는 Task.detached)로 래핑합니다.

컴파일하는 고정 메소드는 Listing 6-4에 있습니다.

```swift
func addGames(to library: VideogameLibrary) {
	let zelda5 = Videogame(title: "The Legend of Zelda: Ocarina of Time", releaseYear: 1998, company: "Nintendo")
	let zelda6 = Videogame(title: "The Legend of Zelda: Majora's Mask", releaseYear: 2020, company: "Nintendo")
	let tales1 = Videogame(title: "Tales of Symphonia", releaseYear: 2004, company: "Namco")
	let tales2 = Videogame(title: "Tales of the Abyss", releaseYear: 2005, company: "Namco")
	let eternalSonata = Videogame(title: "Eternal Sonata", releaseYear: 2008, company: "tri-Crescendo")

    let games = [zelda5, zelda6, tales1, tales2, eternalSonata]

	Task {
		await library.add(games: games)
	}
}
```

addNewGames라는 메소드를 추가해서 다른 `[VideoGame]`을 추가하는 메소드다.
그리고 
``

이제 상황이 흥미로워집니다. 보장된 바는 없습니다
addGames 또는 addNewGames의 게임이 먼저 추가되는지 확인합니다.
그러나 보장되는 것은 어느 쪽이든 먼저 실행되면 행위자는 다른 함수가 실행되기 전에 이러한 함수 중 하나의 모든 게임이 배열에 추가되도록 보장한다는 것입니다. 따라서 배열에는 addGames의 모든 게임이 추가된 순서대로 먼저 포함되고 그 뒤에 addNewGames의 게임이 추가된 순서대로 포함되거나, addNewGames의 게임이 추가된 순서대로 포함되고 그 뒤에 게임이 포함됩니다. 올바른 순서로 addGames에서도 마찬가지입니다. 게임은 인터리브 순서나 그와 유사한 순서로 추가되지 않습니다.
또한 전체 행위자 상태에 대한 액세스가 동기화됩니다. 당신이 가지고 있다면
누군가가 라이브러리에 게임을 추가하는 동시에 누군가가 컬렉션을 쿼리하는 경우 행위자는 실행이 완료될 때까지 해당 호출 중 하나만 실행합니다. 목록 6-7은 세 가지 다른 작업을 생성합니다. 두 개는 게임을 추가하고 다른 하나는 특정 연도별로 게임을 필터링하고 그 중 몇 개가 있는지 인쇄합니다.