ref: https://developer.apple.com/videos/play/wwdc2022/110352/

Swift에서 다형성 구현의 여러 형태가 있다.
- 오버로드를 이용한 ad-hoc 다형성
- 서브타입을 이용한 subtype 다형성
- 제네릭을 이용한 parametric 다형성

##### 오버로드를 이용한 ad-hoc 다형성
오버로딩은 실제로 일반적인 해결책이 아니므로 배제하자
##### 서브타입을 이용한 subtype 다형성
부모 타입을 만들고 부모 타입의 함수 파라미터 타입이 또 다른 부모 타입이라면 런타임 시에 타입 체크가 너무 복잡해진다. 즉, 보일러 플레이트가 많아진다.
##### 제네릭을 이용한 parametric 다형성
기능을 나타내는 인터페이스를 프로토콜을 이용

프로토콜은 추상화 도구로 적합한 유형의 기능을 설명한다.  
`작업에 대한 아이디어` 와 `구현 세부 정보` 를 분리할 수 있다.

> A `protocol` separates ideas from implementation details.

```swift
protocol Animal {
	associatedtype Feed: AnimalTeed
	func eat(_ food: Feed)
}
```

를 통해 Animal 에 대해 필요한 부분을 명세

![](WWDC/WWDC%2022/Embrace%20Swift%20generics/Resources/Pasted%20image%2020241029142011.png)

위와 같이 사용할 수 있다.

제네릭을 이용하여 

```swift
//1번  
struct Farm {  
func feed<A: Animal>(_ animal: A) { ... }  
}  
  
//2번  
struct Farm {  
func feed<A>(_ animal: A) where A: Animal { ... }  
}
```

1번과 2번 모두 같은 코드다. 2번처럼 사용하면 정교한 요구사항 및 타입간의 관계를 지정할 수 있다.

`where`이 붙으면 읽기 어렵다. 그래서 `some`을 쓰면 간소화할 수 있다.

```swift
func feed<A>(_ animal: A) where A: Animal { ... }

// replace where
func feed(_ animal: some Animal)
```

##### Opaque type delcarations

some 키워드 혹은 <"타입": type> 에 들어가는 타입은 모두 opaque 타입이다.

opaque 타입은 입출력에 모두 사용할 수 있다.

opaque 타입의 위치는 프로그램의 어떤 부분이 구체적인 유형을 결정하는지 결정한다.

```swift
func getValue<T>(Parameter) -> Result
```

위 함수 시그니처에서 명명된 유형 매개 변수는 항상 입력측에서 선언되므로 호출자는 기본 유형을 결정하고 구현은 추상 유형을 사용한다.

일반적으로 불투명 파라미터 혹은 결과 타입에 대한 값을 제공하는 프로그램의 부분이 기본 유형을 결정하고 이 값을 사용하는 프로그램의 부분은 추상 유형을 본다.
##### Inferring the underlying type for `some`

**Local variables**
기본 유형은 값에서 추론되므로 기본 유형은 항상 값과 동일한 위치에서 나온다.

```swift
let animal: some Animal = Horse()
```

로컬 변수의 경우 기본 유형은 할당 오른쪽에 있는 값에서 추론된다. 즉, opaque 타입의 로컬 변수는 항상 초기값을 가져야 한다. 또한 기본 유형이 고정되어야 하므로 타입을 바꿀 수 없다.

```swift 
func feed(_ animal: some Animal)

feed(Horse())
feed(Chicken())
```

불투명 타입이 있는 파라미터의 경우 기본 타입은 호출 사이트의 인수 값에서 추론된다.

```swift
func makeView(for farm: Farm) -> some View {
	FramView(farm)
}
```

불투명 결과 타입의 경우 기본 유형은 구현의 반환 값에서 추론된다.

opaque return type을 가진 메소드 혹은 computed property는 전역에서 호출할 수 있으므로 값의 범위는 전역적이다.

즉, 기본 return type은 모든 반환 구문에서 동일해야 한다. 그렇지 않으면 컴파일러는 기본 반환 값의 유형이 일치하지 않는다는 오류를 보고한다.

```swift
func makeView(for farm: Farm) -> some View {
	if condition {
		return FramView(farm)
	} else {
		return EmptyView()
	}
}
```

이렇게 사용하고 싶다면 SwiftUI ViewBuilder DSL은 각 분기에 대해 동일한 기본 반환 유형을 갖도록 제어 흐름 구문을 변환할 수 있다.

```swift
@ViewBuilder
func makeView(for farm: Farm) -> some View {
	if condition {
		FramView(farm)
	} else {
		EmptyView()
	}
}
```

 다시 Animal로 돌아와서 Animal 타입에 associatdtype으로 Habitat을 추가하며 
 
 buildHome이라는 메소드에서 Habitat라는 타입의 결과값을 반환해야한다면 A의 구체 타입에 따라 Habitat가 달라지기 때문에 아래와 같이 함수 시그니처를 작성해야한다.
 
```swift
func buildHome<A>(for animal: A) -> A.Habitat where A: Animal { ... }
```

```swift
struct Silo<Material> {
	private var storage: [Material]

	init(storing materials: [Material]) {
		self.storage = materials
	}
}
```

Opaque 타입을 여러 번 참조해야 하는 경우 또 다른 일반적인 위치는 제네릭 타입이다.

다른 컨텍스트에서 제네릭 타입을 참조하려면 꺾쇠 안에 구체 타입을 명시적으로 지정해야한다.

#### any some 차이
```swift
[some Animal] // 똑같은 구체타입으로 이루어진 배열만 가능
[any Animal] // Animal을 채택한 모든 타입이 가능
```

![](WWDC/WWDC%2022/Embrace%20Swift%20generics/Resources/Pasted%20image%2020241031113802.png)