### Designated Initializer
 클래스의 모든 프로퍼티가 초기화 될 수 있도록 해줘야한다.
### Convenience Initializer
보조 이니셜라이저로 지정 생성자를 도와주는 역할을 한다.

지정 생성자의 파라미터 중 일부를 기본값으로 설정해서, 이 편의 생성자 안에서 지정 생성자를 호출하여 초기화를 진행할 수 있다.

그러니 편의 생성자를 사용하려면 지정 생성자가 먼저 선언되어 있어야 한다.

```swift
class Person {
    var name: String
    var age: Int
    var gender: String

    init(name: String, age: Int, gender: String) {
        self.name = name
        self.age = age
        self.gender = gender
    }

    convenience init(age: Int, gender: String) {
		self.init(name: "zedd", age: age, gender: gender)
    }
}
```

