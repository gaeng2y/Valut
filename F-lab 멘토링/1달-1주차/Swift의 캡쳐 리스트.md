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

