Swift에서 다형성의 여러 형태가 있다.
- 오버로드를 이용한 ad-hoc 다형성
- 서브타입을 이용한 subtype 다형성
- 제네릭을 이용한 parametric 다형성

##### 오버로드를 이용한 ad-hoc 다형성
오버로딩은 실제로 일반적인 해결책이 아니므로 배제하자
##### 서브타입을 이용한 subtype 다형성
부모 타입을 만들고 부모 타입의 함수 파라미터 타입이 또 다른 부모 타입이라면 런타임 시에 타입 체크가 너무 복잡해진다. 즉, 보일러 플레이트가 많아진다.
##### 제네릭을 이용한 parametric 다형성
기능을 나타내는 인터페이스를 프로토콜을 이용

> A `protocol` separates ideas from implementation details.

```
protocol Animal {
	associatedtype Feed: AnimalTeed
	func eat(_ food: Feed)
}
```

를 통해 An