# Protocol Oriented Programming, POP?

>[!info]
> **Apple**은 **2015년 9월, WWDC**에서 **Swift 2.0**을 발표하면서 **Swift**는 **프로토콜 지향 언어(Protocol-Oriented Language)**라고 발표했다.

**객체 지향 프로그래밍(Object Oriented Programming, OOP)** 패러다임에 기반을 둔 언어들은 **class**의 **상속**을 사용해 해당 타입이 가지는 **공통적인 기능**을 **모듈화**해 구현한다.

하지만 **class**와 같은 **참조 타입**의 경우 **다중 스레드 환경**에서 사용할 때 잘못된 참조로 인해 **원본 데이터**가 **변경**될 수 있는 **위험**이 있다.

실제 **Swift**의 **표준 라이브러리**에서 **타입**과 관련된 것들을 살펴보면 대부분 **구조체**로 구현되어 있고 다양한 이유로 **Apple**은 특정 상황을 제외하고 **구조체** 사용을 **권장**한다.

하지만 **struct** 혹은 **enum**의 경우 **상속이 불가능**한데, **어떻게** 다양한 **공통 기능**을 **모듈화**할 수 있을까? 이에 대한 해답은 **protocol**과 **extension**에 있다.
## What is the Protocols
> [!info]
> **Protocol**은 특정 작업이나 기능에 적합한 **메소드**, **프로퍼티** 그리고 그 외 **‘필수 조건 (requirements)’** 들의 **'밑바탕 설계(blueprint)'** 를 정의한다.
> 
> 그런 다음 **class** , **struct** 또는 **enum**은 **Protocol**의 요구 사항을 실제 구현하기 위해 **Protocol**을 **채택(*adopted*)** 할 수 있다. **Protocol** 의 요구 사항을 충족하는 모든 유형은 **'Protocol을 준수(*conforming*)한다.'** 라고 한다.
> ------------------ [Protocols](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/protocols/)

### Protocol Composition
동시에 여러 프로토콜을 준수하도록 유형을 요구하는 것이 유용할 수 있습니다. 프로토콜 구성을 사용하여 여러 프로토콜을 단일 요구 사항으로 결합할 수 있습니다. 프로토콜 구성은 구성의 모든 프로토콜의 결합된 요구 사항이 있는 임시 로컬 프로토콜을 정의한 것처럼 작동합니다. 프로토콜 구성은 새로운 프로토콜 유형을 정의하지 않습니다.

프로토콜 구성은 SomeProtocol & AnotherProtocol 형식을 갖습니다. 필요한 만큼 많은 프로토콜을 나열할 수 있으며, 앰퍼샌드(&)로 구분합니다. 프로토콜 구성은 프로토콜 목록 외에도 하나의 클래스 유형을 포함할 수 있으며, 이를 사용하여 필요한 슈퍼클래스를 지정할 수 있습니다.

```swift
protocol Named {
    var name: String { get }
}
protocol Aged {
    var age: Int { get }
}
struct Person: Named, Aged {
    var name: String
    var age: Int
}
func wishHappyBirthday(to celebrator: Named & Aged) {
    print("Happy birthday, \(celebrator.name), you're \(celebrator.age)!")
}
let birthdayPerson = Person(name: "Malcolm", age: 21)
wishHappyBirthday(to: birthdayPerson)
// Prints "Happy birthday, Malcolm, you're 21!"
```

## Protocol Extensions
프로토콜은 메서드, 초기화 프로그램, 서브스크립트 및 계산된 속성 구현을 **준수하는 유형**에 제공하도록 확장될 수 있습니다. 이를 통해 각 유형의 개별 준수 또는 글로벌 함수가 아닌 프로토콜 자체에서 동작을 정의할 수 있습니다.
### 기본 구현 제공

> [!note]
> 확장에 의해 제공되는 기본 구현이 있는 프로토콜 요구 사항은 선택적 프로토콜 요구 사항과 구별됩니다. 준수 유형은 둘 중 하나의 자체 구현을 제공할 필요는 없지만, 기본 구현이 있는 요구 사항은 선택적 체인 없이 호출될 수 있습니다.

```swift
extension PrettyTextRepresentable  {
    var prettyTextualDescription: String {
        return textualDescription
    }
}
```

### Protocol Extension using Generic
**Generic**과 **associatedtype**을 이용하면 더욱 유연한 **protocol** 구현이 가능하다.

```swift
// protocol 정의
protocol Stackable {
    // 프로토콜을 채택하는 데이터 타입이 Generic일 경우 사용
    associatedtype item
     
    var items: [item] { get set }
    
    mutating func add(item: item)
}

// protocol 초기 구현
extension Stackable {
    mutating func add(item: item) {
        items.append(item)
    }
}

// struct
struct Stack<T>: Stackable {
    var items: [T]
    
    typealias item = T
}

var intStack = Stack<Int>(items: [1, 2, 3])
intStack.add(item: 4)
intStack.items // ---> [1, 2, 3, 4]

var strStack = Stack<String>(items: ["하나", "둘", "셋"])
strStack.add(item: "넷")
strStack.items // ---> ["하나", "둘", "셋", "넷"]
```

## 결론 - Protocol Oriented Programming을 추구하는 이유

> - 상속은 class와 같은 참조 타입에서만 가능한데 참조 추적에는 많은 비용이 발생한다. 하지만 **POP를 통해 값 타입을 활용할 수 있게 되었고 상속과 같은 기능을 비교적은 비용으로 제공할 수 있게 되었다.**
> 
> - 상속의 경우 다중 상속을 지원하지 않는 수직적인 구조를 갖는다. 하지만 **POP는 수평적인 구조로 기능을 확장시킬 수 있고 이로 인해 기능의 모듈화를 더욱 명확히 할 수 있다.**