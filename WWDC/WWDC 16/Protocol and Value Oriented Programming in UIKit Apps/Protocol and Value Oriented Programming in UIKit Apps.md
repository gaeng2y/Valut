Session: [youtube](https://www.youtube.com/watch?v=SCW0lb7qClw)

값 유형과 프로토콜을 사용하여 앱을 더 좋게 만들 수 있는지 알아보자.

오늘은 로컬 추론이라는 것에 집중해보자.

### 로컬 추론
바로 앞에 있는 코드를 볼 때 코드의 나머지 부분이 해당 함수와 어떻게 상호 작용하는지 생각할 필요가 없다는 것을 의미한다. 

## Overview
Value types and protocols
- Recap - Model
- Focus - View and controller
- Testing
Sample Code: https://developer.apple.com/library/archive/LucidDreams/Introduction/Intro.html
## Model
모델을 클래스로 선언했었는데 암시적 공유를 해결하기 위해 구조체로 변경
![Relationships](https://i.imgur.com/PVOijDr.jpg)

암묵적으로 그들 중 일부는 단방향 또는 양방향일 수 있고 일부는 동적이거나 정적일 수 있다.

`Dream` 타입 자체를 테스트하려면 복잡하다,,,
## View

Protocol Oriented Programming with View

루시드 드림 앱은 두 개의 섹션에서 각각의 UITableViewCell로 이루어져있다.

![[Pasted image 20240327222750.png]]
### Cell Layout
![[Pasted image 20240327222859.png]]

다른 곳에서도 재사용하기 위해 셀을 `DecoratingLayoutCell`이라는 **UITableViewCell**의 추상 하위 클래스로 작성하고, `DreamCell`로 구체적인 하위 클래스를 만들었다고 해보자.

**UIView -> UITableViewCell -> DecoratingLayoutCell -> DreamCell**

다른 셀에서 레이아웃을 재사용하는 데는 도움이 될 수 있지만 테이블 뷰 외부에서 사용하기는 어려움

그래서 앱에서 사용할 수 있는 구조를 만드는 더 나은 방법을 찾아봤는데

앱의 레이아웃은 UITableViewCell 뿐만 아니라 일반 UIView도 있고 멋진 파티클 효과를 보여주기 위해 앱에 스프라이트킷을 추가하는 SKNode에서도 레이아웃을 사용할 수 있기를 원한다.

이를 달성하기 위해 Swift에서 배운걸 사용해보자.

기존에 상속하던걸 구조체로 만들어보자...

```swift
// View Layout
struct DecoratingLayout {
	var content: UIView
	var decoration: UIView

	mutating func layout(in rect: CGRect) {
		// Perform Layout
	}
}
```

* 레이아웃 로직을 호출할 수 있는 메소드를 배치
* 레이아웃 수행 방법을 알고 있는 고립된 코드 작성

```swift
class DreamCell: UITableViewCell {
	...
    
    override func layoutSubView() {
    	var decoratingLayout = DecoratingLayout(content: content, decoration: decoration)
        decoratingLayout.layout(in: bounds)
    }
}
```

* 구조체를 사용하여 DeamCell의 레이아웃을 업데이트할 수 있다.
* 가장 좋은 점은 레이아웃 로직이 분리되었으므로 UIView 하위 클래스에서도 사용할 수 있다는 점이다.

```swift
class DreamDetailView: UIView {
	...
    
    override func layoutSubView() {
    	var decoratingLayout = DecoratingLayout(content: content, decoration: decoration)
        decoratingLayout.layout(in: bounds)
    }
}
```

이렇게 되면 유닛 테스트를 작성하는 것이 정말 쉽다. 일부 뷰를 만들어 레이아웃에 추가하면 된다.

```swift
func testLayout() {
	let child1 = UIView()
	let child2 = UIView()

	var layout = DecoratingLayout(content: child1, decoration: child2)
	layout.layout(in: .init(x: 0, y: 0, width: 120, height: 40))

	XCTAssertEqual(child1.frame, .init(x: 0, y: 5, width: 35, height: 30))
	XCTAssertEqual(child2.frame, .init(x: 35, y: 5, width: 70, height: 30))
}
```

구조체를 만들어서 사용하려했지만 `NodeDecoratingLayout`은 UIView를 사용하지 않는다. (공통 슈퍼클래스가 없다!)

![[Pasted image 20240327224352.png]]

이 레이아웃이 하는 유일한 일은 프레임 설정뿐 -> 프로토콜 요구사항으로 가지게 할 수 있다.

```swift
struct DecoratingLayout {
	var content: Layout
    var decoration: Layout
    
    mutating func layout(in rect: CGRect) {
    	// Perform layout
    }
}

protocol Layout {
	var frame: CGRect { get set }
}

extension UIView: Layout {}
extension SKNode: Layout {}
```
### Generic
`content`와 `decoration`이 다른 타입을 가질 수 있다는 문제가 있다. (ex. `content: UIVIew, decoration: SKNode`)  
-> 제네릭을 사용

```swift
struct DecoratingLayout<Child: Layout> {
	var content: Child
    var decoration: Child
    ...
}
```

> **Generic Type**
> 
> - 코드의 타입을 더 많이 제어할 수 있음
> - 컴파일러가 코드에 대한 더 많은 정보를 갖게 됨 = 최적화

비슷한 다른 레이아웃을 가지고 싶으면 어떻게 해야할까 고민을 가지게 된다.

이전에는 코드를 공유하기 위해 사용했던 도구 중 하나는 **상속**이지만 슈퍼클래스의 작업과 서브클래스가 재정의할 수 있는 작업을 모두 고려해야 함  
-> 앱 전체에 분산되어 있는 많은 양의 코드를 함께 가져와야 함  
-> 대부분 UIView와 같은 프레임워크 클래스에서 상속하고 많은 코드가 있음  
-> 상속은 로컬 추론 능력을 저하시킴
### Composition of ~~Views~~ Values
이전에 레이아웃을 만들 수 있었던 한 가지 방법은 뷰를 함께 구성하여 다음과 같은 UIView를 작성할 수 있는 것이다.

> 코드를 재사용하거나 일부 동작을 사용자 지정해야 할 때 사용

- 클래스의 인스턴스는 비용이 크다 (힙 할당, 그리기 및 이벤트 처리같은 필요한 많은 작업이 있음)
- 구조체는 매우 가벼움 + 값 타입은 복사에 대한 수정 걱정이 없어 조각을 사용하기 용이

```swift
protocol Layout {
	mutating func layout(in rect: CGRect)
}

extension UIView: Layout {...}
extension SKNode: Layout {...}

struct DecoratingLayout<Child: Layout, ...>: Layout {...}
struct CascadingLayout<Child: Layout, ...>: Layout {...}

let decoration = CascadingLayoutLayout(children: accessories)
var composedLayout = DecoratingLayout(content: content, decoration: decoration)
composedLayout.layout(in: rect)
```
### associatedtype
서브뷰들이 올바른 순서로 같은 타입들만 배치되도록 하려면

```swift
protocol Layout {
	mutating func layout(in rect: CGRect)
    associatedtype Content
    var contents: [Content] { get }
}

struct DecoratingLayout: Layout {
	...
    typealias Content = UIView
}
struct CascadingLayout: Layout {
	...
    typealias Content = SKNode
}
```

**UIView** 또는 **SKNode**로 혼합될 수 있으므로 **associatedtype**을 사용  
associatedtype은 type placeholer로 준수하는 타입이 구체적인 타입을 선택해서 사용하도록 함

```swift
struct DecoratingLayout<Child: Layout, Decoration: Layout
						where Child.Content == Decoration.Content> : Layout {
	var content: Child
    var decoration: Decoration
    
    typealias Content = Child.Content
}
```

> UIView 대신 Layout을 사용하여 단위 테스트 가능  
> Layout은 간단한 구조체에 프레임을 설정  
> -> 테스트가 UIView와 완전히 분리되어 있고, 자체 레이아웃 및 테스트 논리에만 의존함을 의미

### Techniques
* 값 타입을 이용하여 지역 추론
* 제네릭 타입은 빠르고 안전하게 다형화
* 값의 컴포지션
## Controller
앱의 실행 취소 기능  
(Dream에 대한 실행취소 기능은 구현했지만 FavoritCreature에 대한 실행 취소를 구현하지 않은 버그 상황)

FavoritCreature와 관련된 실행 취소 구조를 추가할 수 있지만 다른 코드 경로 추가로 유지보수의 어려움 발생
### Model 분리
