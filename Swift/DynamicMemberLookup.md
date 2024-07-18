## DynamicMemberLookup
- class, struct, enum, protocol에 사용 가능
- subscript (dynamicMemberLookup :) 메소드를 구현해야함.

**1. 만약 dynamicMemberLookup을 "사용"하는 타입이 있다면,**

**2. 이 타입은 런타임시 확인되는 임의의 "name"에 대해** 

**3. "dot" syntax를 사용할 수 있도록 제공해준다.**

### Subscript
콜렉션, 리스트, 시퀀스 등 집합의 **특정 member elements에 간단하게 접근할 수 있는 문법**

**dynamicMemberLookup이 subscript와 유사하다**
### **1. 만약 dynamicMemberLookup을 "사용"하는 타입이 있다면,**
이를 위해 이 타입이 dynamicMemberLookup을 사용한다고 알려주어야 한다. attribute를 붙이면 된다.
```swift
@dynamicMemberLookup
struct Contact {
    
    var persons: [String: String]
    
    subscript(dynamicMember planet: String) -> String? {
        return self.persons[planet]
    }
}
```
### **2. 이 타입은 런타임시 확인되는 임의의 "name"에 대해**
### **3. "dot" syntax를 사용할 수 있도록 제공해준다.**

1번 같이 코드를 만들면
```swift
var contact = Contact(persons: [
    "지구": "Zedd",
    "달": "Marshmello"
])

let zedd = contact[dynamicMember: "지구"] // Zedd
```
이렇게 가져올 수도 있지만
```swift
var contact = Contact(persons: [ "지구": "Zedd", "달": "Marshmello" ]) let zedd = contact.지구 // Zedd
```

이렇게 가져올 수도 있다.

만약 값이 없다면 nil을 return 한다.

### 그렇다면
#### 어떻게 그래?

**[SE-0195](https://github.com/apple/swift-evolution/blob/master/proposals/0195-dynamic-member-lookup.md)** 에서도 dynamicMemberLookup을 **syntactic sugar**로 설명합니다.

컴파일러가 subscript를 평가할 때, subscript가 "런타임"에 동적으로 호출되기 때문에 가능.

(왜 "**dynamic**MemberLookup"인지 아시겠나요!!)

#### **subscript**(dynamicMember: )에 String타입 말고 다른 타입을 넣고싶어!

- **[ExpressibleByStringLiteral](https://developer.apple.com/documentation/swift/expressiblebystringliteral)** (프로토콜) 또는 keyPath만 넣을 수 있습니다.

