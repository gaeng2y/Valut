UIViewController는 이런 생명 주기를 갖고 있다.
![UIViewController LifeCycle](https://miro.medium.com/v2/resize:fit:690/1*aaUDpKdPrqlqX_Hmd2jHJA.png)

### viewDidLoad()
controller의 view가 메모리에 로드된 후에 호출하는 함수다.
![[Pasted image 20240306121614.png]]
discussion을 살펴보면 **This method is called after the view controller has loaded its view hierarchy into memory.** 라는 문구가 보인다. 말 그대로 `viewDidLoad()`는 뷰컨트롤러가 뷰 계층 구조를 메모리에 로드한 후에 호출된다고 한다.

그런데 그 다음 문장을 살펴보면 
**This method is called regardless of whether the view hierarchy was loaded from a nib file or created programmatically in the [`loadView()`](https://developer.apple.com/documentation/uikit/uiviewcontroller/1621454-loadview)method.**
라는 문장이 보이는데 살펴보면 view가 nib으로 만들어졌던지 `loadView()` 메소드를 통해 프로그래밍 방식으로 생성되었는지에 관계없이 호출된다고 한다.

이
