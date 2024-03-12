## `self`
* 인스턴스 자체 접근 시 사용되는 참조값
* `self`는 참조 타입
    - value type에서의 self는 stack영역에 존재하는 instance를 가리키는 형태
    - reference type에서의 self는 heap 영역에 존재하는 instance를 가리키는 형태
## `Self`
- 대문자로 시작하는 것에서 알 수 있듯이, Self는 타입을 의미
- self 프로퍼티의 타입은 Self
## `Self.self`

- Self는 type그 자체를 의미: **타입을 정의할때 사용**
- Self.self는 type object를 의미: **타입을 넘길때 사용**