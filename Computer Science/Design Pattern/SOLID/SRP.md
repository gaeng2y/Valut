### SRP(Single Responsibility Principle): 단일 책임 원칙
모든 클래스는 하나의 책임만 가지며, 클래스는 그 책임을 완전히 캡슐화해야 함
### SoC(Separation of Concerns): 관심사 분리 
SoC 법칙이란 시스템 요소가 단일 목적이고 배타성을 가져야 한다는 것을 명시한다. 
어떤 요소도 다른 요소와의 책임을 공유하면 안 되고, 그 요소와 관계가 없는 책임을 포함시킬 수 없다는 것
SoC란 경계를 세우는 것으로서 달성된다. 
**SoC의 효과** 
1. 개별 컴포넌트의 복제가 없어지고, 목적이 단일화되어 전체 시스템을 유지보수하기 쉽게 만든다. 
2. 시스템 전체가 유지보수성이 올라감으로써 생겨난 부산물로 인해 더 안정적이게 된다.

[위키](https://ko.wikipedia.org/wiki/%EB%8B%A8%EC%9D%BC_%EC%B1%85%EC%9E%84_%EC%9B%90%EC%B9%99)를 살펴보면 **모든 클래스는 하나의 책임만 가지며, 클래스는 그 책임을 완전히 캡슐화해야 함**을 일컫는다고 되어있다.

<aside> 💡 클래스를 변경해야 하는 이유는 단 하나여야 한다.

</aside>

**하나의 책임**이라고 하니 관심사 분리(SoC: Separation of Concerns)가 생각이 나는데요.

관심사 분리란 컴퓨터 프로그램을 별개의 섹션으로 분리하기 위한 설계 원칙입니다.

여기서 제가 생각하는 두 원칙의 차이점은 생각하는 시점이라고 생각합니다.

SOC는 새로운 것을 시작하거나 리팩토링을 할 때 생각하는 원칙이라면

SRP는 사후에 적용되는 원칙이다. 클래스를 작성한 후 “**이게 너무 많은 일을 하나?**”라는 생각을 하게되죠.

간단한 예를 살펴본다면

```swift
class Animal {
	var name: String
	
	func getAnimalName() { }
	func saveAnimal(with animal: Animal) { }
}
```

위의 구조체에서 살펴보면 Animal 구조체는 SRP 원칙에 위배되었습니다.

어떻게 위배되었는지 살펴보면

1. Animal의 name을 가져오는 역할
2. Animal을 DB에 저장하는 역할

두 가지의 역할을 가지고있습니다.

그렇다면 위배되지 않게 변경한다면

```swift
class Animal {
	var name: String
	
	init(name: String) {}
	func getAnimalName() { }
}

class AnimalDB {
	func getAnimal(name: String) -> Animal {}
	func saveAnimal(with animal: Animal) {}
}
```

와 같이 DB에 저장하는 역할을 하는 구조체를 새로 만들어줘서 각 클래스가 단 하나의 책임을 갖도록 만들었습니다.