DIP는 다음과 같은 정의를 가지고 있다.

1. 상위 모듈은 하위 모듈에 의존해서는 안된다.
2. 추상화는 세부 사항에 의존해서는 안된다.

OCP와 LSP의 합체라고 해야되나…?

클라이언트가 상속 관계로 이루어진 모듈을 가져다 사용할 때, 하위 모듈을 **직접 인스턴스를 가져다 쓰지 말라**는 뜻이다. 하위 모듈에 변경이 있을 때마다 클라이언트나 상위 모듈의 코드를 자주 수정해야 하기 때문이다.

상위의 인터페이스 타입의 객체로 통신하라는 원칙이다

```swift
import Foundation

// hl modules should not depend on low-level; both should depend on abstractions
// abstractions should not depend on details; details should depend on abstractions

enum Relationship {
    case parent
    case child
    case sibling
}

class Person {
    var name = ""
    // dob, etc
    init(_ name: String) {
        self.name = name
    }
}

protocol RelationshipBrowser {
    func findAllChildrenOf(_ name: String) -> [Person]
}

class Relationships : RelationshipBrowser {
    var relations = [(Person, Relationship, Person)]()
    
    func addParentAndChild(_ p: Person, _ c: Person) {
        relations.append((p, Relationship.parent, c))
        relations.append((c, Relationship.child, p))
    }
    
    func findAllChildrenOf(_ name: String) -> [Person] {
        return relations
            .filter({$0.name == name && $1 == Relationship.parent && $2 != nil})
            .map({$2})
    }
}

class Research {
    init(_ relationships: Relationships) {
        // high-level: find all of job's children
        let relations = relationships.relations
        for r in relations where r.0.name == "John" && r.1 == Relationship.parent {
            print("John has a child called \\(r.2.name)")
        }
    }
    
    init(_ browser: RelationshipBrowser) {
        for p in browser.findAllChildrenOf("John") {
            print("John has a child called \\(p.name)")
        }
    }
}

func main() {
    let parent = Person("John")
    let child1 = Person("Chris")
    let child2 = Person("Matt")
    
    var relationships = Relationships()
    relationships.addParentAndChild(parent, child1)
    relationships.addParentAndChild(parent, child2)
    
    let _ = Research(relationships)
}

main()
```