그래서 제가 코드 조각을 텍스트로 구성할 수 있는 특별한 구성 요소를 가지고 싶다고 가정해 보겠습니다.

String 상속을 통해 추가 클래스를 만들 수 있겠지만, 

지금은 프로토콜 확장자로 확실히 할 수 있지만 문제는 문자열에 확장자만 있는 것 외에 추가 정보를 저장하고 싶을 수도 있다는 것입니다.

예를 들어, 코드를 작성하는 경우 들여쓰기 수준을 저장하고 늘리고 줄이고 싶습니다.

그래서 나는 상태가 필요하고 결과적으로 나는 상속할 수 없습니다. 왜냐하면 문자열은 참조 유형이 아니기 때문입니다,

가치형입니다.

그래서 저는 그것을 그대로 물려받을 수도 없고, 프로토콜 연장에 만족하지도 않을 것입니다.

데코레이터가 하는 일은 장식된 물체의 특정한 기능을 제공할 수 있다.

예를 들어, 만약 문자열에 부가 기능이 있고 내 코드 빌더가 문자열처럼 동작하기를 원한다면, 나는 부가 기능을 가지고 싶어질 수도 있습니다. 그래서 나는 다음과 같이 보일 수 있는 부가 기능을 추가할 수 있습니다  
  
문자열 보통의 덧셈 함수와 동일합니다.

그래서 여기에 추가할 문자열을 가지고 갑니다.

이제 저도 이것을 유창하게 해서 코드빌더를 돌려줌으로써 이 빌더를 유창하게 만들 수 있습니다.

그리고 이것은 제가 여러번 호출을 연결할 수 있게 해줄 겁니다. 좋은 일이죠.

그래서 여기서는 단순히 버퍼에 문자열을 추가하고 방법을 유창하게 만들기 위해 스스로 돌아옵니다.