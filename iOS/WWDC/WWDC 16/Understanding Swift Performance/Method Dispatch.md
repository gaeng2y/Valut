메서드 디스패치(Method dispatch)는 메서드가 호출될 때 실행할 메서드 구현을 결정하는 메커니즘을 말합니다.

Swift는 세 가지 주요 메서드 디스패치 유형을 지원합니다:

정적 디스패치(static dispatch), 동적 디스패치(dynamic dispatch), 그리고 증명 테이블 디스패치(witness table dispatch).

### Static Dispatch

- 컴파일러가 컴파일타임에 명령어가 어디 있는지 결정이 되어 찾을 수 있으므로, 동적 디스패치와 비교할 때 훨씬 빠르다.
- 따라서, 함수가 호출 되면 , 컴파일러는 함수의 메모리 주소로 직접 점프하여 작업을 수행한다.
- 인라인과 같은 특정 컴파일러 최적화와 큰 성능 향상이 될 수 있다.
- 런타임 조회가 필요 없기 때문에 매우 효율적입니다. 정적 메서드, final 메서드, 그리고 구조체와 열거형의 메서드는 정적 디스패치를 사용합니다.
- 값 타입(value types) 과 참조 타입(reference types) 모두 지원한다.

### Dynamic Dispatch

- 앞에서 설명한 것처럼 이러한 유형의 디스패치에서는 구현이 컴파일 시간 대신 런타임에 선택되어 오버헤드가 추가된다.
- 이 유형은 오버라이드될 수 있는 클래스의 메서드에 사용됩니다.
- 동적 디스패치는 가상 테이블(vtable)이라는 메커니즘을 사용하여 런타임에 메서드 구현을 조회합니다.
- 이는 다형성을 허용하며, 실행되는 메서드가 객체의 런타임 타입에 따라 달라질 수 있습니다.
- 동적디스패치는 참조 타입(reference types)만 지원한다.
- 이러한 이유는 동적디스패치를 위해서는 상속이 필요한데 값 타입은 상속을 지원하지 않기 때문이다.

이러한 디스패치 메커니즘을 이해하면 개발자가 더 효율적이고 예측 가능한 Swift 코드를 작성하는 데 도움이 됩니다.

근데 좀 살펴보니 아래와 같이 4가지 타입으로 나뉜다고 한다.

1. Inline (Fastest)
2. Static Dispatch
3. Virtual Dispatch
4. Dynamic Dispatch (Slowest)

## Dynamic Dispatch를 왜씀?

- OOP의 특징인 유연성 (_Flexibility_) 때문이다.
- 대부분의 OOP 언어는 다형성(객체 지향 3대 원칙)이라는 특징때문에 Dynamic Dispatch를 지원한다.

## Dynamic Dispatch 타입

**Table Dispatch**

- 이 디스패치 기법은 테이블을 사용한다.
- 여기서 테이블이란 함수 포인터로 이루어진 배열을 의미하며 `Witness Table` (혹은 Virtual Table)이라고 불린다.
- 특정 메소드의 구현을 찾는(Look up) 용도로 사용
- `Witness Table`이 어떻게 작동하는가?
    - 모든 서브클래스는 각자의 테이블을 갖고있다.
    - 이 테이블은 서브클래스가 재정의한 모든 메소드에 대해 다른 함수포인터를 갖고있다.
    - 서브클래스가 새로운 메소드를 추가하면 해당 메소드에 대한 함수포인터가 배열 끝에 추가된다.
    - 컴파일러가 런타임에 이 테이블을 사용하여 어떤 메소드를 호출할지 결정한다.
- 정적 디스패치와 달리 컴파일러가 먼저 테이블로부터 메모리 주소를 읽고(read)
- 해당 위치로 이동(jump)해야 하므로 두번의 연산이 요구된다.
- 이것이 정적 디스패치보다 느린 이유다. (메시지 디스패치보다는 여전히 빠르다)

**Message Dispatch**

- 이 동적 디스패치 기법은 가장 동적이라고 할 수 있다.
- 사실 이 기법은 최적화 측면을 제외하면 매우 좋아서
- Cocoa 프레임워크에서도 KVO, 코어데이터와 같은 곳에서 자주 사용된다.
- 이 방법은 런타임에 메소드의 기능(functionality)를 바꾸는 `Method Swizzling`을 가능하게 한다.
- Swift 컴파일러는 이 기법을 Obejctive-C 런타임을 사용해 achieve한다.
- Swift 4.0 이후부터는 [@objc](https://github.com/objc) 키워드를 마크해주어야한다.

### VTable

Swift에서 `vtable`은 다형성을 지원하고 런타임 메소드 호출을 효율적으로 처리하는 메커니즘이다.

클래스 정의 시 컴파일러는 각 클래스의 가상 메소드들을 모아 `vtable`을 생성하고, 클래스 인스턴스는 이 테이블을 참조하여 메소드를 호출한다.

상속 관계에서 서브클래스는 슈퍼클래스의 메소드를 오버라이드할 수 있으며, 이때 서브클래스의 `vtable`이 슈퍼클래스의 메소드를 대체하여 동작한다.

런타임에 객체의 메소드가 호출될 때, 해당 객체의 `vtable`을 참조하여 올바른 메소드 주소로 점프하여 실행되는 방식으로 동작한다.

예제 코드를 통해 Swift의 vtable이 어떻게 동작하는지 이해할 수 있다.

```swift
class Animal {
    func makeSound() {
        print("Some generic sound")
    }
}

class Dog: Animal {
    override func makeSound() {
        print("Bark")
    }
}

class Cat: Animal {
    override func makeSound() {
        print("Meow")
    }
}

let animals: [Animal] = [Dog(), Cat(), Animal()]

for animal in animals {
    animal.makeSound()
}

```

이 예제에서 `Dog`와 `Cat` 클래스는 `Animal` 클래스의 `makeSound` 메소드를 오버라이드하여 각자의 방식으로 구현하고 있다.

배열 `animals`에 있는 각 객체는 자신의 클래스에 맞는 `makeSound` 메소드를 호출하게 되며, 이는 `vtable`을 통해 이루어지는 것이다.

Swift에서 vtable은 다형성을 구현하고 런타임에 정확한 메소드가 호출되도록 보장하는 중요한 역할을 한다.

## 메소드 디스패치 타입 테이블

![table](https://user-images.githubusercontent.com/42762236/147629573-75f15227-017c-4b73-a51a-d91e04fdf859.png)