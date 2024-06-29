LSP: 리스코프 치환 원칙(Liskov substitution principle)

서브 타입은 언제나 기반 타입으로 교체할 수 있어야 한다.

```swift
import Foundation

class Shape {
    func doSomething() {}
}

class Square: Shape {
    func drawSquare() {}
}

class Circle: Shape {
    func drawCircle() {}
}

func draw(shape: Shape) {
    if let square = shape as? Square {
        square.drawSquare()
    } else if let circle = shape as? Circle {
        circle.drawCircle()
    }
}
```

위 예시는 LSP를 위반한 대표적인 예시다.

`Squre`, `Circle` 객체가 파라미터로 받은 상위 객체 `Shape` 와 전혀 다르게 동작하고 있기 때문이다.

그렇다면 고쳐보자.
```swift
import Foundation 

protocol Shape { 
	func draw() {} 
} 

class Square: Shape { 
	func draw() {} 
} 

class Circle: Shape { 
	func draw() {} 
} 

func draw(shape: Shape) { 
	shape.draw() 
}
```