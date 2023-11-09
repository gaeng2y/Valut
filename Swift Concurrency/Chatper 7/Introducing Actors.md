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