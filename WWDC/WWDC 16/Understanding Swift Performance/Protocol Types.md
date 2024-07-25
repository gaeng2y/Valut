이제는 클래스가 아니고 프로토콜을 선언해서 만들었다.

![protocol1](https://github.com/gaeng2y/Valut/blob/main/Combine/Combine%20Study/Resources/Protocol1.png?raw=true)

![protocol2](https://github.com/gaeng2y/Valut/blob/main/Combine/Combine%20Study/Resources/Protocol2.png?raw=true)

상속을 하지 않은 관계에서는 V-Table을 사용하지 않고 다이나믹 디스패치를 한다.

![protocol3](https://github.com/gaeng2y/Valut/blob/main/Combine/Combine%20Study/Resources/Protocol3.png?raw=true)

그렇다면 어떻게 찾아가는거임?

답은, **table기반 메커니즘인 Protocol Witness Table**이다.

해당 테이블의 엔트리는 해당 타입의 구현에 연결(link)된다.

![pwt](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Ft1.daumcdn.net%2Fcfile%2Ftistory%2F998BD8335BA8667705)

그래서 이렇게 메소드를 찾을 수 있다.

![pwt2](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Ft1.daumcdn.net%2Fcfile%2Ftistory%2F99CCC6405BA8677C36)

값을 어떻게 균일하게 저장하는지..?

![pwt](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Ft1.daumcdn.net%2Fcfile%2Ftistory%2F99865A425BA86A5F2F)

배열에 Line, Point 타입을 저장하려고하는데 배열은 똑같은 사이즈를 가져야 하는데,,, 

이럴 때는 Existential Container라는 특수한 storage layout을 사용한다.

## Existential Container

![ec](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Ft1.daumcdn.net%2Fcfile%2Ftistory%2F99F33C4C5BA86AB726)

5워드로 구성되어있는데 3워드는 Value Buffer로 예약되어있다. 구조체를 만들 때 프로퍼티들을 3워드 안에 넣을 수 있다면 넣고 못넣는다면 힙에 저장하고 포인터를 저장한다.

![small](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Ft1.daumcdn.net%2Fcfile%2Ftistory%2F9967FE3E5BA86B1904)

![large](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Ft1.daumcdn.net%2Fcfile%2Ftistory%2F99FCE0465BA86BAB3A)

Swift는 Line같이 큰 타입은 Heap에 메모리를 할당하고 해당 메모리에대한 포인터를 Existential Container에 저장한다.

Existential Container는 이 차이(difference)를 관리 할 필요가 있다.

이게 바로 table 기반 메커니즘인 Value Witness table(VWT)이다.

VWT는 value의 라이프타임을 관리하며 타입마다 VWT가 있다.

![vwt](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Ft1.daumcdn.net%2Fcfile%2Ftistory%2F99A764435BA8709B09)

Line은 Existential Container에 들어맞지 않기때문에 Heap에 메모리 할당이 필요하다.

이 allocate함수 안에서는 Heap에 메모리를 할당하고, 해당 메모리에 대한 포인터를 Existential Container의 valueBuffer내에 저장한다.

![vwt](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Ft1.daumcdn.net%2Fcfile%2Ftistory%2F9991A13F5BA870A419)

다음으로 인스턴스를 생성하면 assignment소스에서 Existential Container로 값을 **복사**해야한다.

이제 라이프타임이 끝나고 destruct 엔트리를 호출한다.

![vwt](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Ft1.daumcdn.net%2Fcfile%2Ftistory%2F9984373C5BA870BD16)

그리고 이제 해제하게될 때는 deallocate를 호출하는데 힙 영역에 공간 조차 없죠

![vwt](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Ft1.daumcdn.net%2Fcfile%2Ftistory%2F997ED73C5BA870C70A)

그래서 이제 Existential Container에는 value buffer, vwt, pwt가 3,1,1워드 씩 저장된다.

![ec](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Ft1.daumcdn.net%2Fcfile%2Ftistory%2F9993A44F5BA871F049)

![ec](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Ft1.daumcdn.net%2Fcfile%2Ftistory%2F99C95C3E5BA8725010)

## 이제 코드로 살펴보자

```swift
// Protocol Types
// The Existential Container in action
func drawACopy(local : Drawable) {
   local.draw()
}
let val : Drawable = Point()
drawACopy(val)
```

위와 같은 코드를 만들고 컴파일을 하게 되면

```swift
// Generated code
struct ExistContDrawable {
   var valueBuffer: (Int, Int, Int)
   var vwt: ValueWitnessTable
   var pwt: DrawableProtocolWitnessTable
}
```

이 만들어진다. 이제 `drawACopy(val)`을 호출하면

![ex](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Ft1.daumcdn.net%2Fcfile%2Ftistory%2F995FF6365BA875E714)

Swift는 drawACopy의 인자를 Existential container로 전달 후

![ex2](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Ft1.daumcdn.net%2Fcfile%2Ftistory%2F99E9073F5BA8771E01)

함수가 실행되면, 매개변수에 대한 로컬 변수가 만들어지고 인자가 할당된다.

![ex3](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Ft1.daumcdn.net%2Fcfile%2Ftistory%2F991821405BA877552C)

이제 vwt를 읽고 local를 초기화하고 pwt를 통해 draw 메소드를 다이나믹 디스패치를 한다.

![ex4](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Ft1.daumcdn.net%2Fcfile%2Ftistory%2F995817405BA87EDB1F)

근데 더 보면 projectBuffer가 등장한다

이게 뭐임?

![ecec](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Ft1.daumcdn.net%2Fcfile%2Ftistory%2F9927404A5BA882D534)

draw메소드는 인풋의 value로 주소가 들어올것으로 예상하고 있고 value는 밸류버퍼에 들어가는 값인지에 따라 이 주소는 existential container의 시작주소이거나 Line처럼 valueBuffer에 맞지않는 value면 Heap에 할당된 메모리의 시작주소다.

따라서. 이 value witness 함수는 타입에 따라(Point처럼인지 Line인지) 이 차이를 추상화를 한다.

다른 예제를 살펴보자

![ex22](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Ft1.daumcdn.net%2Fcfile%2Ftistory%2F99F4E2445BA89B6102)

Swift 컴파일러는 d.draw()에서 어떤 draw()를 호출해야 하는지 아는 방법은 Existential Container에 가서 pwt에 접근해서 다이나믹 디스패치를 한다.

Pair 타입을 인스턴스를 만들면 아래와 같이 만들어진다.

![pair](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Ft1.daumcdn.net%2Fcfile%2Ftistory%2F99E5433D5BA8883911)

근데 여기서 point를 line으로 바꿔보면?

![ex3](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Ft1.daumcdn.net%2Fcfile%2Ftistory%2F996E13445BA8895110)

위와 같이 힙에 두번 할당한다. 근데 이걸 카피하면 다 똑같이 복사가 된다...

![heap](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Ft1.daumcdn.net%2Fcfile%2Ftistory%2F9945F64B5BA8897904)

이거는 자원이 너무 낭비 된다는 것 때문에 해결하는 방법은 Line을 구조체에서 클래스로 바꾸는 것이다.

근데 클래스로하면 값이 한군데서 값 바꾸면 다 바뀌는데..? 그걸 해결하기 위한게 COW(copy on write)이다.

![cow](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Ft1.daumcdn.net%2Fcfile%2Ftistory%2F99CE563C5BA88E771B)

LineStorage라는 클래스를 만들고 Line이라는 구조체가 storage라는 LineStorage타입의 저장 프로퍼티를 가진다.

즉, 원래 Line안에 직접적으로 있던 저장 프로퍼티들을 LineStorage로 옮겼다는 것이다.

LineStorage는 class이니 참조 하나만 가지고 있다

Line의 값을 읽으려고 할 때마다 storage안에 있는 값들을 읽을 것이다.

하지만 우리가 막 value를 수정할 때, 먼저 레퍼런스 카운트를 확인한다. 만약 그게 1보다 크다면, 우리는 LineStorage의 복사본을 만들고 이를 변경한다.

이제 똑같이 만들고 값을 바꿔보면...?

![fasdf](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Ft1.daumcdn.net%2Fcfile%2Ftistory%2F99F8C4385BA893851C)

값의 변화가 일어나면, (예를 들어 move()를 호출한다면?)

그럼 move 메소드에서 isUniquelyReferencedNonObjc -> 이 레퍼런스가 유일한지 확인하는 거다.

레퍼런스가 유일한게 아니라면 값이 변경되면 새로운 인스턴스를 만들어서 할당하는 것이다.

## 그래서 성능은?

![small](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Ft1.daumcdn.net%2Fcfile%2Ftistory%2F9919D9335BA8964727)

![large](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Ft1.daumcdn.net%2Fcfile%2Ftistory%2F999DCD365BA8964F2E)

![](https://github.com/gaeng2y/Valut/blob/main/Combine/Combine%20Study/Resources/Protocol4.png?raw=true)

하지만, 이는 위에서 언급되었듯이 indirect storage를 사용한다면 이러한 값비싼 Heap할당을 줄일 수 있다. (대충 cow 쓰는거..)

![indirect](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Ft1.daumcdn.net%2Fcfile%2Ftistory%2F996A65335BA8968D21)

## Summary

- Dynamic polymorphism
- Indirection through Witness Tables and Existential Container
- Copying of large values causes heap allocation