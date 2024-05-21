low-level 언어들은 일반적으로 alloc-free 방식 메모리 관리 정책을 이용한다.  alloc-free 방식은 사용할 메모리를 할당하는 것이 alloc, 반대로 할당한 메모리를 다시 OS에 돌려주는 것이 free이다.

Swift 이전의 옵씨는 C에서 파생된 언어기 때문에 C의 이런 메모리 관리 방식을 그대로 사용할 수 있지만, 그보다는 진화한 메모리 관리 정책을 도입했다. 그것이 바로 레퍼런스 카운트 개념이다.

레퍼런스 카운트 방식은 Retain과 Releases라는 개념을 이용.

ARC와 더불어 특수한 상황에서 추가적으로 메모리 관리에 필요한 개념이 **AutoReleases**이다.
### AutoReleases
AutoRelease는 참조 카운트를 나중으로 미루기 위한, 그러면서도 카운트가 나중에 감소되는 것을 보장받기 위한 기법이다. 객체의 참조 카운트를 감소시킬 때 releass 대신 autorelease를 사용하면 release가 예약된다. 실제 release를 수행하면 참조 카운트가 바로 줄어들지만, autorelease 직후는 아직 release된 상황이 아니므로 참조 카운트가 감소하지 않는다. 

autoRelease된 객체는 **AutoReleasePool**에 등록된다. AutoReleasePool은 객체를 관리하는 일종의 컬렉션이다. 그리고 이 컬렉션이 해제될 때 관리하는 객체를 모두 release한다. 따라서 autorelease 된 객체의 release 시점은 아주 명료하다. 바로 AutoReleasePool은이 해제될 때다.

당연히 사용가능한 AutoReleasePool이 없다면 autoRelease 객체는 release 메시지를 받지 못하고 메모리 누수가 발생한다. 따라서 AutoReleasePool이 없는 상황에서 autoRelease 메시지를 보내면 코코아는 에러를 띄울 것이다. AppKit 혹은 UIKit 프레임워크는 각각의 이벤트 루프의 시작점에서 자동적으로 AutoReleasePool을 생성하여 작동한다.

그러나 원하면 각종 컬렉션 객체를 이용하여 별도의 pool을 간단하게 만들어 쓸 수도 있다. 다만, Foundation Framework에 AutoreleasePool이 있으므로 일반적으로 애써 만들 이유는 없다. 하지만 다음의 경우에는 별도의 AutoreleasePool을 만들어야 한다.