* 브릿지는 소위 **데카르트 곱 복잡도** 폭발을 방지한다.
* 예)
	* ThreadScheduler 라는 기본 클래스가 있다고 가정해보자.
	* ThreadScheduler는 선점형이거나 협력적일 수 있으며 
	* Windows, linux에서도 실행될 수 있다.
	* WindowsPTS, UnixPTS, WindowsPTS, UnixPTS를 구현하기로 결정했다면 2x2 시나리오로 끝낼 수 있다.
* 브릿지 패턴은 바로 이 전체 엔터티 폭발을 실제로 방지하는 것이다.

ThreadScheduler는 platormScheduler(UnixScheduler, WindowsScheduler)를 채택하여 
PreemptiveThreadScheduler, CooperativeThreadScheduler를 만들 수 있다.

이것이 브리지의 일반적인 구현이다.

브릿지는 구현에서 인터페이스 또는 계층 구조를 분리하는 메커니즘이다.
