클로저는 코드에서 사용될 수 있는 독립적인 코드 블록입니다. Swift에서 클로저는 함수와 유사하지만, 특정 기능을 캡슐화하여 변수나 상수에 저장하거나 인자로 전달할 수 있습니다.

#### 클로저의 형태

1. **전역 함수**: 이름이 있고, 값을 캡처하지 않는 클로저입니다.
2. **중첩 함수**: 이름이 있고, 상위 함수의 값을 캡처할 수 있는 클로저입니다.
3. **클로저 표현식**: 간단하고 문법적으로 간결한 클로저입니다.

Swift에서 클로저(Closures)는 주변 환경을 캡쳐하여 해당 환경의 변수와 상수를 클로저 내부에서 사용할 수 있게 합니다. 이러한 캡쳐 기능은 매우 강력하지만, 클로저가 객체를 강하게 참조하게 되어 retain cycle(순환 참조)를 발생시킬 수 있습니다. 이를 방지하기 위해 **캡쳐 리스트(Capture List)** 를 사용합니다.
### 클로저와 캡쳐

클로저는 자신이 정의된 문맥에서 변수를 캡쳐하고 저장할 수 있습니다. 클로저가 변수를 캡쳐하는 방식에 따라 강한 참조(Strong Reference) 또는 약한 참조(Weak Reference)로 캡쳐할 수 있습니다.

### 캡쳐 리스트의 사용법

캡쳐 리스트는 클로저 내부에서 사용되는 외부 변수들의 캡쳐 방식을 정의하는 데 사용됩니다. 이를 통해 클로저가 외부 변수를 강하게 참조하지 않도록 하여 retain cycle을 방지할 수 있습니다.

#### 캡쳐 리스트 문법

캡쳐 리스트는 클로저의 매개변수 목록 이전에 대괄호(`[]`) 안에 작성됩니다. 캡쳐 리스트 내에서 변수를 약하게(`weak`) 또는 비소유 참조(`unowned`)로 캡쳐할 수 있습니다.

```swift
{ [capture list] (parameters) -> returnType in
    // 클로저 본문
}
```

#### 예시: 강한 참조 순환 문제

다음은 강한 참조 순환 문제를 설명하는 예시입니다.

```swift
class HTMLElement {
    let name: String
    let text: String?
    
    lazy var asHTML: () -> String = {
        if let text = self.text {
            return "<\(self.name)>\(text)</\(self.name)>"
        } else {
            return "<\(self.name) />"
        }
    }
    
    init(name: String, text: String? = nil) {
        self.name = name
        self.text = text
    }
    
    deinit {
        print("\(name) is being deinitialized")
    }
}

var paragraph: HTMLElement? = HTMLElement(name: "p", text: "Hello, world!")
print(paragraph!.asHTML())
paragraph = nil  // 강한 참조 순환으로 인해 메모리에서 해제되지 않음
```

위 예제에서 `asHTML` 클로저는 `self`를 강한 참조로 캡쳐하여, `HTMLElement` 인스턴스가 해제되지 않습니다.

#### 예시: 캡쳐 리스트로 강한 참조 순환 해결

위의 강한 참조 순환 문제를 캡쳐 리스트를 사용하여 해결할 수 있습니다.

```swift
class HTMLElement {
    let name: String
    let text: String?
    
    lazy var asHTML: () -> String = { [weak self] in
        guard let self = self else {
            return ""
        }
        if let text = self.text {
            return "<\(self.name)>\(text)</\(self.name)>"
        } else {
            return "<\(self.name) />"
        }
    }
    
    init(name: String, text: String? = nil) {
        self.name = name
        self.text = text
    }
    
    deinit {
        print("\(name) is being deinitialized")
    }
}

var paragraph: HTMLElement? = HTMLElement(name: "p", text: "Hello, world!")
print(paragraph!.asHTML())
paragraph = nil  // 약한 참조로 인해 메모리에서 해제됨
```

이 예제에서는 캡쳐 리스트를 사용하여 `self`를 약한 참조로 캡쳐함으로써, `HTMLElement` 인스턴스가 더 이상 필요 없을 때 메모리에서 해제될 수 있도록 합니다.

### 캡쳐 리스트의 종류

캡쳐 리스트에는 두 가지 주요 종류가 있습니다:

1. **약한 참조 (weak)**: 캡쳐된 객체가 더 이상 필요 없을 때 nil로 설정됩니다. 옵셔널 타입이어야 합니다.
2. **비소유 참조 (unowned)**: 캡쳐된 객체가 항상 메모리에 존재한다고 가정합니다. 옵셔널이 아니어도 됩니다.

#### 비소유 참조 사용 예시

```swift
class Customer {
    let name: String
    var creditCard: CreditCard?
    
    init(name: String) {
        self.name = name
    }
    
    deinit {
        print("\(name) is being deinitialized")
    }
}

class CreditCard {
    let number: String
    unowned let customer: Customer
    
    init(number: String, customer: Customer) {
        self.number = number
        self.customer = customer
    }
    
    deinit {
        print("Card \(number) is being deinitialized")
    }
}

var john: Customer? = Customer(name: "John Appleseed")
john?.creditCard = CreditCard(number: "1234 5678 9012 3456", customer: john!)

john = nil  // Customer와 CreditCard 객체가 해제됨
```

여기서는 `CreditCard` 클래스의 `customer` 프로퍼티를 비소유 참조로 설정하여 강한 참조 순환을 방지합니다.

### 결론

Swift에서 클로저가 외부 변수를 강하게 참조함으로써 발생할 수 있는 retain cycle 문제를 해결하기 위해 캡쳐 리스트를 사용합니다. 캡쳐 리스트를 사용하여 약한 참조(`weak`) 또는 비소유 참조(`unowned`)로 변수를 캡쳐함으로써, 메모리 누수를 방지하고 클로저와 객체 간의 순환 참조 문제를 해결할 수 있습니다.