### Dimensions of Performance

추상화를 구축하고 추상화 메커니즘을 선택할 때 "인스턴스가 스택이나 힙에 할당될 것인가? 이 인스턴스를 전달할 때 얼마나 많은 참조 카운팅 오버헤드를 발생시킬 것인가? 이 인스턴스에서 메서드를 호출할 때 정적으로 디스패치될 것인가 동적으로 디스패치될 것인가?"를 스스로에게 물어봐야 한다. 빠른 Swift 코드를 작성하려면 우리가 이용하지 않는 동적 처리와 런타임 비용을 피해야 할 것이다.

그리고 더 나은 성능을 위해 이러한 다양한 차원 사이에서 언제 어떻게 교환할 수 있는지 배워야 합니다.

![](Pasted%20image%2020240627081239.png)

### Allocation
Swift는 자동으로 메모리를 할당하고, 해제하는데 메모리 중 일부는 Stack에 할당된다.

다들 아시다시피, Stack은 LIFO구조로 매우 단순한 데이터 구조다.

Stack의 마지막, 즉 Top으로만 Push가 가능하며, 역시 Top에서만 Pop이 가능하다.

그러니, Stack끝에 포인터만 유지하면, Push및 Pop을 구현 할 수 있게 됩니다. 이 포인터를 Stack 포인터라고 부른다.

그래서 우리가 함수를 호출 할 때, 우리는 메모리를 일단 먼저 할당해야할텐데 그 메모리를 단순히 Stack 포인터가 가리키고 있는 곳을 단순히 줄임(decrementing)으로써 필요한 메모리를 할당 할 수 있다. 그리고 함수가 끝나면, Stack 포인터를 줄이기 전에 있던 곳으로 증가(incrementing)시킴으로써 그 메모리 할당을 해제할 수 있다.

스택은 메모리에 할당하지 않기때문에 성능에 좋다. (그래서 값타입을 애플이 유도하는 것이다.)

![](Pasted%20image%2020240627193312.png)

하지만 스택과 대조적으로 힙이라는 데이터 구조가 있는데 힙보다 좀 더 동적이지만 효율은 스택보다 좋지않다.

힙을 사용하면 스택이 할 수 없는 일을 할 수 있다. 바로 dynamic lifetime을 가진 메모리를 할당할 수 있다.

![](Pasted%20image%2020240627193502.png)

값 타입을 인스턴스로 만들면 스택에 만들어지고 해제 시 스택에서 팝된다.

![](Pasted%20image%2020240627193414.png)

하지만 참조 타입을 인스턴스로 만들면 힙 영역에 할당한 후 힙 영역의 첫 주소를 스택에 할당한다.

![](Pasted%20image%2020240627193447.png)

자 예를 살펴보자

![](Pasted%20image%2020240627193629.png)

위와 같은 말풍선을 만드는 코드가 있다.

우리가 이제 캐시를 구현하기 위해 아래와 같은 코드로 짰다고 생각해보자

![](Pasted%20image%2020240627193659.png)

그러면 String은 contents가 힙 영역에 저장되기 때문에 성능이 스택에 저장되는 거에 비해 좋지 않다.

그래서 우리는 `Attributes`라는 구조체를 만들어서 딕셔너리의 키 타입을 바꾼다.

![](Pasted%20image%2020240627193752.png)

그러면 키도 스택에 저장되니까 개꿀!

---
### Script

개발자로서 Swift는 우리가 탐구할 수 있는 광범위하고 강력한 디자인 공간을 제공합니다. Swift에는 다양한 일급 타입과 코드 재사용 및 동적 처리를 위한 다양한 메커니즘이 있습니다.

이 모든 언어 기능은 흥미롭고 새로운 방식으로 결합될 수 있습니다. 그렇다면 이 디자인 공간을 좁히고 올바른 도구를 선택하는 방법은 무엇일까요? 먼저 Swift의 다양한 추상화 메커니즘의 모델링에 대한 영향을 고려해야 합니다. 값 의미론이 더 적합한가요, 아니면 참조 의미론이 더 적합한가요? 이 추상화가 얼마나 동적이어야 할까요? 오늘 아놀드와 저는 성능을 사용하여 디자인 공간을 좁히는 방법을 알려드리고자 합니다. 제 경험상 성능을 고려하면 더 관용적인 솔루션을 찾는 데 도움이 됩니다. 그래서 우리는 주로 성능에 초점을 맞출 것입니다. 모델링에 대해서도 조금 다루겠지만, 지난해와 올해에 강력한 Swift 프로그램 모델링 기법에 대한 훌륭한 강연이 있었습니다. 이 강연을 최대한 활용하고 싶다면, 최소한 이들 중 하나라도 시청할 것을 강력히 권장합니다.

좋습니다. 그래서 우리는 성능을 사용하여 디자인 공간을 좁히고자 합니다. Swift의 추상화 메커니즘의 성능 영향을 이해하는 가장 좋은 방법은 그 기초 구현을 이해하는 것입니다. 그래서 오늘 그 작업을 할 것입니다. 먼저 다양한 추상화 메커니즘 옵션을 평가할 때 고려해야 할 다양한 차원을 식별할 것입니다. 이 중 각 차원에 대해, 구조체와 클래스를 사용한 코드를 추적하여 관련된 오버헤드를 깊이 이해할 것입니다. 그런 다음 우리가 배운 것을 적용하여 일부 Swift 코드를 정리하고 속도를 높이는 방법을 살펴볼 것입니다.

강연의 후반부에서는 프로토콜 지향 프로그래밍의 성능을 평가할 것입니다. 프로토콜과 제네릭과 같은 고급 Swift 기능의 구현을 살펴보고 모델링과 성능에 미치는 영향을 더 잘 이해할 것입니다. 간단한 설명: Swift가 여러분을 대신하여 컴파일하고 실행하는 메모리 표현 및 생성된 코드 표현을 살펴볼 것입니다. 이것들은 필연적으로 단순화될 수밖에 없지만, 아놀드와 저는 단순성과 정확성 사이에서 매우 좋은 균형을 찾았다고 생각합니다. 그리고 이것은 여러분의 코드를 이해하는 데 있어 매우 좋은 정신 모델입니다.

좋습니다. 성능의 다양한 차원을 식별하는 것부터 시작하겠습니다.

따라서 추상화를 구축하고 추상화 메커니즘을 선택할 때 "내 인스턴스가 스택이나 힙에 할당될 것인가? 이 인스턴스를 전달할 때 얼마나 많은 참조 카운팅 오버헤드를 발생시킬 것인가? 이 인스턴스에서 메서드를 호출할 때 정적으로 디스패치될 것인가 동적으로 디스패치될 것인가?"를 스스로에게 물어봐야 합니다. 빠른 Swift 코드를 작성하려면 우리가 이용하지 않는 동적 처리와 런타임 비용을 피해야 할 것입니다.

그리고 더 나은 성능을 위해 이러한 다양한 차원 사이에서 언제 어떻게 교환할 수 있는지 배워야 합니다.

좋습니다. 각 차원을 하나씩 살펴보며 할당부터 시작하겠습니다.

Swift는 여러분을 대신하여 자동으로 메모리를 할당하고 해제합니다. 일부 메모리는 스택에 할당됩니다.

스택은 매우 간단한 데이터 구조입니다. 스택 끝에 추가하고 스택 끝에서 제거할 수 있습니다. 스택의 끝에만 추가하거나 제거할 수 있기 때문에 스택을 구현할 수 있으며, 스택 끝의 포인터만 유지하면 됩니다. 이는 우리가 함수에 호출할 때, 스택 끝의 포인터를 간단히 감소시켜 공간을 확보할 수 있음을 의미합니다. 함수 실행을 마치면 스택 포인터를 증가시켜 원래 위치로 돌아가면서 메모리를 간단히 해제할 수 있습니다. 스택이나 스택 포인터에 익숙하지 않은 경우 이 슬라이드에서 가져가길 바라는 것은 스택 할당이 얼마나 빠른지입니다. 이는 정수를 할당하는 것과 같은 비용입니다.

이는 힙과 대조적입니다. 힙은 더 동적이지만 스택보다 덜 효율적입니다. 힙은 스택이 할 수 없는 동적 수명의 메모리를 할당할 수 있게 합니다.

하지만 이는 더 고급 데이터 구조가 필요합니다. 힙에 메모리를 할당하려면 적절한 크기의 미사용 블록을 찾기 위해 힙 데이터 구조를 검색해야 합니다. 사용이 끝나면 메모리를 다시 적절한 위치에 재삽입해야 합니다.

따라서 이는 스택에서 정수를 할당하는 것보다 더 많은 작업이 필요합니다. 하지만 힙 할당과 관련된 주요 비용은 아닙니다. 여러 스레드가 동시에 힙에 메모리를 할당할 수 있기 때문에 힙은 동기화 메커니즘을 사용하여 무결성을 보호해야 합니다. 이는 상당한 비용입니다. 오늘날 여러분의 프로그램에서 힙에 메모리를 할당하는 시기와 장소에 주의를 기울이지 않는다면, 조금 더 신중하게 관리함으로써 성능을 극적으로 개선할 수 있습니다.

좋습니다. 일부 코드를 추적하여 Swift가 여러분을 위해 무엇을 하고 있는지 살펴보겠습니다. 여기 x와 y 저장 프로퍼티가 있는 point 구조체가 있습니다. draw 메서드도 포함되어 있습니다. (0, 0) 위치에 point를 생성하고, point1을 point2에 할당하여 복사한 다음, point2.x에 값을 5로 할당합니다. 그런 다음 point1과 point2를 사용합니다. 이를 추적해보겠습니다. 이 함수에 들어가기 전에 point1 인스턴스와 point2 인스턴스의 공간을 스택에 할당합니다. point가 구조체이기 때문에 x와 y 프로퍼티는 스택에 인라인으로 저장됩니다. 따라서 x가 0이고 y가 0인 point를 생성할 때 이미 스택에 할당된 메모리를 초기화하는 것뿐입니다. point1을 point2에 할당할 때, point2 메모리를 초기화하는 것과 마찬가지로 point를 복사하는 것입니다. point1과 point2는 독립된 인스턴스입니다. 따라서 point2.x에 5를 할당하면 point2.x는 5가 되지만 point1.x는 여전히 0입니다. 이것은 값 의미론으로 알려져 있습니다. 그런 다음 point1을 사용하고 point2를 사용한 후 함수를 실행하는 동안 point1과 point2 메모리를 스택 포인터를 원래 위치로 간단히 증가시켜 해제할 수 있습니다.

이것을 구조체 대신 클래스를 사용한 동일한 코드와 비교해봅시다.

좋습니다. 이 함수에 들어가면 이전과 같이 스택에 메모리를 할당합니다. 하지만 이제는 point의 프로퍼티를 저장할 메모리가 아니라 point1과 point2의 참조 메모리를 할당합니다.

참조 메모리는 힙에 할당됩니다. (0, 0) 위치에 point를 생성할 때 Swift는 힙을 잠그고 적절한 크기의 미사용 블록을 찾기 위해 데이터 구조를 검색합니다. 그런 다음 그 메모리를 x가 0이고 y가 0인 상태로 초기화하고 point1 참조에 힙의 메모리 주소를 초기화합니다. 이제 point가 클래스이기 때문에, Swift는 클래스 point에 네 단어의 저장 공간을 할당합니다. 이는 구조체일 때 할당된 두 단어와 대조적입니다. 이제 point가 클래스이므로, x와 y를 저장하는 것 외에도 Swift가 관리할 두 단어를 더 할당합니다. 이는 힙 다이어그램의 파란색 상자로 표시됩니다.

point1을 point2에 할당할 때, point1이 구조체일 때와 달리 point의 내용을 복사하지 않습니다. 대신 참조를 복사합니다. 따라서 point1과 point2는 실제로 힙의 동일한 point 인스턴스를 참조합니다. 따라서 point2.x에 5를 할당하면 point1.x와 point2.x 모두 5가 됩니다. 이는 참조 의미론으로 알려져 있으며, 상태의 의도치 않은 공유를 초래할 수 있습니다. 그런 다음 point1을 사용하고 point2를 사용한 후 Swift는 힙을 잠그고 사용하지 않는 블록을 적절한 위치에 되돌려줍니다. 그런 다음 스택을 팝할 수 있습니다.

좋습니다. 방금 본 것은 무엇입니까? 클래스는 구조체보다 생성 비용이 더 많이 듭니다. 클래스는 힙 할당을 요구하기 때문입니다.

클래스는 힙에 할당되고 참조 의미론을 가지므로 클래스는 정체성과 간접 저장소와 같은 강력한 특성을 가집니다. 하지만 추상화를 위해 이러한 특성이 필요하지 않다면 구조체를 사용하는 것이 더 좋습니다.

구조체는 클래스처럼 상태의 의도

치 않은 공유에 취약하지 않습니다. 따라서 이를 적용하여 Swift 코드의 성능을 개선하는 방법을 살펴봅시다. 제가 작업 중인 메시징 애플리케이션의 예를 들어보겠습니다. 기본적으로 이는 뷰 계층에서 사용됩니다. 사용자가 텍스트 메시지를 보내면 해당 텍스트 메시지 뒤에 멋진 풍선 이미지를 그리고 싶습니다. makeBalloon 함수는 다양한 풍선 구성 공간을 지원하는 이미지를 생성합니다. 예를 들어, 이 풍선은 오른쪽 방향과 꼬리가 있는 파란색 풍선입니다. 왼쪽 방향과 거품 꼬리가 있는 회색 풍선도 지원합니다.

makeBalloon 함수는 할당 실행과 사용자 스크롤 중에 자주 호출되므로 매우 빨라야 합니다. 그래서 캐싱 레이어를 추가했습니다. 특정 구성을 위해 이 풍선 이미지를 한 번만 생성하면 다시는 생성할 필요가 없습니다. 한 번 생성하면 캐시에서 가져올 수 있습니다. 이를 위해 색상, 방향 및 꼬리를 키 문자열로 직렬화했습니다. 여기에 몇 가지 문제가 있습니다.

문자열은 이 키의 강력한 타입이 아닙니다. 이를 구성 공간을 나타내는 데 사용하고 있지만, 키에 제 강아지의 이름을 넣을 수도 있습니다. 따라서 안전성이 떨어집니다. 또한, 문자열은 힙에 문자 내용을 간접적으로 저장하므로 많은 것을 나타낼 수 있습니다. 따라서 makeBalloon 함수를 호출할 때마다 캐시 히트가 발생하더라도 힙 할당이 발생합니다.

더 나은 방법을 찾아보겠습니다. Swift에서는 색상, 방향 및 꼬리의 구성 공간을 단순히 구조체로 나타낼 수 있습니다. 이는 문자열보다 이 구성 공간을 나타내는 훨씬 안전한 방법입니다. 그리고 구조체는 Swift에서 일급 타입이므로 우리의 딕셔너리에서 키로 사용할 수 있습니다.

이제 makeBalloon 함수를 호출할 때 캐시 히트가 발생하면 할당 오버헤드가 없습니다. 속성 구조체를 생성하는 것은 힙 할당이 필요하지 않으므로 스택에 할당할 수 있습니다. 이는 훨씬 안전하며 훨씬 빠를 것입니다.