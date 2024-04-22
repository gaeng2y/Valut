Session: https://developer.apple.com/videos/play/wwdc2016/416/
Ref: https://zeddios.tistory.com/596#recentComments
## Dimensions of Performance
![](Pasted%20image%2020240423001321.png)

### Allocation
Swift는 자동으로 메모리를 할당하고, 해제하는데 메모리 중 일부는 Stack에 할당된다.

다들 아시다시피, Stack은 LIFO구조로 매우 단순한 데이터 구조다.

Stack의 마지막, 즉 Top으로만 Push가 가능하며, 역시 Top에서만 Pop이 가능하다.

그러니, Stack끝에 포인터만 유지하면, Push및 Pop을 구현 할 수 있게 됩니다. 이 포인터를 Stack 포인터라고 부른다.

그래서 우리가 함수를 호출 할 때, 우리는 메모리를 일단 먼저 할당해야할텐데 그 메모리를 단순히 Stack 포인터가 가리키고 있는 곳을 단순히 줄임(decrementing)으로써 필요한 메모리를 할당 할 수 있다. 그리고 함수가 끝나면, Stack 포인터를 줄이기 전에 있던 곳으로 증가(incrementing)시킴으로써 그 메모리 할당을 해제할 수 있다.

스택은 메모리에 할당하지 않기때문에 성능에 좋다. (그래서 값타입을 애플이 유도하는 것이다.)

![](Pasted%20image%2020240423001631.png)

하지만 스택과 대조적으로 힙이라는 데이터 구조가 있는데 힙보다 좀 더 동적이지만 효율은 스택보다 좋지않다.

힙을 사용하면 스택이 할 수 없는 일을 할 수 있다. 바로 dynamic lifetime을 가진 메모리를 할당할 수 있다.

![](Pasted%20image%2020240423001733.png)

힙에 메모리를 할당하려면, 실제로 힙 데이터 구조를 검색하여 사용되지 않는 적절한 크기의 블록을 찾아야한다.(일정 공간을 사용해야하기 때문에) 그리고 할당을 해제하려면 해당 메모리를 또 적절한 위치로 재삽입을 해야한다.

여기서 가장 큰 복잡함은 여러 쓰레드가 동시에 힙에 메모리를 할당할 수 있기 때문에, 힙은 lock 또는 기타 동기화 메커니즘을 사용해서 무결성을 보호해야 한다. -> 이것이 힙 할당에서 가장 큰 비용

![](Pasted%20image%2020240423002203.png)

`Point`라는 구조체를 살펴보면 프로퍼티와 메소드가 있다. point1에 Point의 인스턴스를 할당하고, point2에 point1을 복사하고, point2.x 에 5를 할당한다.

![](Pasted%20image%2020240423003852.png)

그러면 코드가 실행되기 전에 point1과 point2의 인스턴스를 스택에 할당하면 스택의 line에 저장된다.( x and y properties are stored in line on the stack)

![](Pasted%20image%2020240423004106.png)

따라서 우리가 Point의 인스턴스를 생성할 때 스택에 이미 할당한 메모리를 초기화 하는 것 뿐이다. 그리고 point2에 point1을 복사해서 할당할 때, 우리는 해당 point의 복사본을 만들고 스택상에 이미 할당한 메모리를 다시 초기화하는 것이다.

즉, point1과 point2는 독립적인 인스턴스다.

그리고 point2.x에 5를 넣는다면

![](Pasted%20image%2020240423004220.png)

우리가 point1을 사용하고, point2를 사용하고 이제 다 끝나면, 스택 포인터를 점차적으로 증가시키면 point1과 point2에 대한 메모리 할당을 쉽게 해제 할 수 있게 된다.

그렇다면 Point를 클래스로 선언하면 

![](Pasted%20image%2020240423004327.png)

그림이 조금 달라졌는데 코드를 실행하기 전에 컴파일러는 이미 스택에 point1, point2에 대한 메모리 할당이 된다. 하지만 실제로 아까처럼 