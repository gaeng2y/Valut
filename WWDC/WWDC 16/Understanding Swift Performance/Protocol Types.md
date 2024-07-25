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

