클래스 인스턴스가 할당 해제되기 직전에 초기화 해제자가 호출됩니다. init 키워드를 사용하여 초기화 프로그램을 작성하는 방법과 유사하게 deinit 키워드를 사용하여 초기화 해제 프로그램을 작성합니다. 초기화 해제자는 클래스 유형에서만 사용할 수 있습니다.

### How Deinitialization Works

Swift는 더 이상 필요하지 않은 인스턴스를 자동으로 할당 해제하여 리소스를 확보합니다. Swift는 자동 참조 계산(ARC)을 통해 인스턴스의 메모리 관리를 처리합니다. 일반적으로 인스턴스 할당이 취소될 때 수동 정리를 수행할 필요가 없습니다. 그러나 자체 리소스를 사용하여 작업하는 경우 몇 가지 추가 정리를 직접 수행해야 할 수도 있습니다. 예를 들어 사용자 정의 클래스를 생성하여 파일을 열고 파일에 일부 데이터를 쓰는 경우 클래스 인스턴스 할당이 취소되기 전에 파일을 닫아야 할 수도 있습니다. 

클래스 정의에는 클래스당 최대 하나의 초기화 해제 프로그램이 있을 수 있습니다. 디이니셜라이저는 매개변수를 사용하지 않으며 괄호 없이 작성됩니다.

```swift
deinit {
	// perform the deinitialization
}
```

디이니셜라이저는 인스턴스 할당 해제가 발생하기 직전에 자동으로 호출된다. 디이니셜라이저를 직접 호출하는 것은 허용되지 않는다. 슈퍼클래스 해제자는 하위 클래스에 상속되며, 슈퍼 클래스 해제자는 하위 클래스 해제자 구현이 끝날 때 자동으로 호출된다. 하위 클래스가 디이니셜라이저를 제공하지 않는 경우에도 슈퍼클래스 디이니셜라이저는 항상 호출됩니다.

인스턴스는 디이니셜라이저가 호출될 때까지 할당이 해제되지 않기 때문에 디이니셜라이저는 호출된 인스턴스의 모든 속성에 액세스할 수 있으며 이러한 속성을 기반으로 해당 동작을 수정할 수 있습니다.(예: 닫아야 하는 파일의 이름 조회)
### Retain Cycle
Swift에서 Retain Cycle(순환 참조)는 두 개 이상의 객체가 서로를 강한 참조(Strong Reference)로 소유하고 있을 때 발생하는 현상입니다. 이로 인해 객체가 서로를 참조하여 메모리에서 해제되지 않게 되어 메모리 누수가 발생할 수 있습니다.
#### 강한 참조와 순환 참조

- **강한 참조(Strong Reference)**: 객체를 소유하고 해제되지 않게 유지하는 참조입니다. 기본적으로 Swift에서 모든 참조는 강한 참조입니다.
- **순환 참조(Retain Cycle)**: 두 객체가 서로를 강한 참조로 참조할 때 발생합니다. 이로 인해 두 객체 모두 메모리에서 해제되지 않는 상태가 됩니다.

```swift
class Person {
    let name: String
    var apartment: Apartment?
    
    init(name: String) {
        self.name = name
    }
    
    deinit {
        print("\(name) is being deinitialized")
    }
}

class Apartment {
    let unit: String
    var tenant: Person?
    
    init(unit: String) {
        self.unit = unit
    }
    
    deinit {
        print("Apartment \(unit) is being deinitialized")
    }
}

var john: Person? = Person(name: "John Appleseed")
var unit4A: Apartment? = Apartment(unit: "4A")

john?.apartment = unit4A
unit4A?.tenant = john

john = nil
unit4A = nil
```

#### 해결 방법: 약한 참조와 비소유 참조

Swift에서는 순환 참조 문제를 해결하기 위해 **약한 참조(Weak Reference)**와 **비소유 참조(Unowned Reference)**를 사용합니다.

#### 약한 참조(Weak Reference)

약한 참조는 참조하고 있는 객체가 더 이상 사용되지 않을 때 nil로 설정됩니다. 약한 참조는 옵셔널 타입이어야 합니다.

```swift
class Person {
    let name: String
    var apartment: Apartment?
    
    init(name: String) {
        self.name = name
    }
    
    deinit {
        print("\(name) is being deinitialized")
    }
}

class Apartment {
    let unit: String
    weak var tenant: Person?  // 약한 참조로 변경
    
    init(unit: String) {
        self.unit = unit
    }
    
    deinit {
        print("Apartment \(unit) is being deinitialized")
    }
}

var john: Person? = Person(name: "John Appleseed")
var unit4A: Apartment? = Apartment(unit: "4A")

john?.apartment = unit4A
unit4A?.tenant = john

john = nil  // Person 객체가 해제됨
unit4A = nil  // Apartment 객체가 해제됨\
```

이 예제에서는 `Apartment` 클래스의 `tenant` 프로퍼티를 약한 참조로 변경했습니다. 이제 `john`과 `unit4A`를 nil로 설정하면 순환 참조가 발생하지 않으며, `Person`과 `Apartment` 객체는 메모리에서 올바르게 해제됩니다.

#### 비소유 참조(Unowned Reference)

비소유 참조는 참조하고 있는 객체가 더 이상 사용되지 않을 때도 nil로 설정되지 않습니다. 비소유 참조는 참조하고 있는 객체가 항상 메모리에 존재할 것이라는 가정하에 사용됩니다.

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
    unowned let customer: Customer  // 비소유 참조로 변경
    
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

이 예제에서는 `CreditCard` 클래스의 `customer` 프로퍼티를 비소유 참조로 변경했습니다. 이로 인해 `john`을 nil로 설정하면 `Customer`와 `CreditCard` 객체는 메모리에서 올바르게 해제됩니다.

### 결론

Swift에서 Retain Cycle은 두 객체가 서로를 강한 참조로 참조할 때 발생하는 메모리 누수 문제입니다. 이를 해결하기 위해 약한 참조(weak)와 비소유 참조(unowned)를 사용하여 순환 참조를 방지할 수 있습니다. 상황에 맞게 약한 참조와 비소유 참조를 사용하면 메모리 관리가 용이해집니다.