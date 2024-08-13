UI를 그리는 등 UI와 관련된 작업은 `main` 스레드에서 해야한다고 했다. 심지어 UI 작업을 main 스레드에 할당하지 않으면 에러가 일어나곤 한다. UIKit 공식문서에서도 아래와 같은 내용의 문구가 있다.

> [!error]
> Use UIKit classes only from your app’s main thread or main dispatch queue, unless otherwise indicated. This restriction particularly applies to classes derived from UIResponder or that involve manipulating your app’s user interface in any way. [UIKit 공식문서](https://developer.apple.com/documentation/uikit/)

# 1. UIKit은 Thread safe하지 않다.

UIKit은 UI를 구성하는 구성 요소들의 집합이다. 즉 화면에 즉각적으로 표현이 되는 요소들이다. 

그러나 이 UIKit의 대부분의 요소들은 `Thread Unsafe`한 성질을 가지고 있다. 여러 스레드에서 접근이 가능한 것이다. 그러면 어떤 일이 발생할까?

앞에서 배웠던 것처럼 `Race Condition`이 발생하게 된다.

우리가 보는 화면의 상태를 여러 스레드에서 접근하여 제어하는 것이다. 버튼의 모양을 바꾸는 일과 버튼의 위치를 이동시키는 일, 버튼의 색을 바꾸는 일 등이 충돌하여 우리의 의도대로 동작하지 않을 수 있다.

# 2. 꼭 Main Thread여야하는 이유가 있나? 다른 직렬 큐에 넣으면 안되나?

UI작업을 꼭 메인 스레드에서 작업해야하는 이유는 메인 스레드에는 `Main RunLoop`가 동작하고 있기 때문이다. 메인 스레드에서는 RunLoop가 일정한 주기를 유지하며 계속 동작하고 있다. 이 주기에 맞추어서 사용자의 입력을 받아서 UI를 그리게 된다. 이러한 주기를 `View Drawing Cycle`이라고 한다. 

하지만 사실 모든 스레드는 각자의 RunLoop를 가질 수 있는데, 왜 꼭 Main RunLoop에 의존해야 할까? 그 이유는 모든 스레드의 RunLoop에 따라 각자가 UI를 그리게 되면 UI가 그려지는 시점이 모두 제각각이 되기 떄문이다.