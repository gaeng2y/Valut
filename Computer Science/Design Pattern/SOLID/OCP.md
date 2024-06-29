
OCP: 개방 폐쇄 원칙(Open-Closed Principle)

클래스는 확장에 대해서는 개방적이고, 수정에 대해서는 폐쇄적이어야 한다.

다형성과 확장을 가능케 하는 객체지향의 장점을 극대화하는 설계 원칙으로 객체를 추상화함으로써, 확장엔 열려있고, 변경엔 닫혀있는 유연한 구조를 만드는 것이다.

코드를 살펴보면서 이해해보자.

```swift
class Animal {
  var name: String

  init(name: String) {}
  func getAnimalName() { }
}

class MakeSound {
  func makeSound(with animal: Animal) {
    if animal.name == "Cat" {
      print("야옹")
    } else if animal.name == "Dog" {
      print("멍멍")
    } else if animal.name == "Lion" {
      print("어흥")
    }
  }
}

func main() {
  let dog = Animal(name: "Dog")
  let cat = Animal(name: "Cat")
  let lion = Animal(name: "Lion")

  let makeSound = MakeSound()
  makeSound.makeSound(with: dog)
  makeSound.makeSound(with: cat)
  makeSound.makeSound(with: lion)
}
```

위와 같은 코드에서 양이 추가된다면 `makeSound(with:)` 메소드에서 분기문을 추가적으로 구성해줘야한다.

이런 식으로 코드를 구성하면, 동물이 추가될 때마다 코드를 변경해줘야하는 번거로움이 생긴다.

이는 설계가 잘못되었기 때문에 일어나는 현상이다.

OCP 설계 원칙에 따라 적절한 추상화 클래스(프로토콜)을 구성하고 이를 채택하여 구현하는 관계로 구성하면 변경에는 닫히고 확장에는 열려있는 클래스가 된다.

1. 먼저 변경(확장)될 것과 변하지 않을 것을 엄격히 구분한다.
2. 이 두 모듈이 만나는 지점에 추상화를 정의한다.
3. 구현체에 의존하기보다 정의한 추상화에 의존하도록 코드를 작성 한다.