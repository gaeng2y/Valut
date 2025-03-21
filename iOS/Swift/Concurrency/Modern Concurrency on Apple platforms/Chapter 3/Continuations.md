간단히 말해서, Continuation은 비동기 호출이 끝난 후에 일어나는 모든 것이다.

우리가 async/await를 사용할 때, Continuation은 단순히 대기 호출 아래의 모든 것입니다.

새로운 동시성 시스템을 통해 클로저 기반 코드와 심지어 위임 기반 코드의 형태로 연속을 async/await 상태로 "변환"할 수 있습니다.

CheckedThrowingContinuation과 함께, 우리는 이러한 유형의 연속을 만드는 데 사용할 수 있는 세 가지 추가 방법이 있습니다.
* withCheckedContinuation  
* withUnsafeThrowingContinuation
* withUnsafeContinuation
원래 클로저 기반 코드가 어떤 종류의 오류도 돌려주지 않을 때 withCheckedContinuation을 사용하세요.

안전하지 않은 변종 - UnsafeThrowingContinuation과 UnsafeContinuation -은 체크된 대응물과 같다.

주요 차이점은, Checked Continuations로 작업할 때, Swift는 위반이 발생했을 때 오류를 기록하여 당신을 보호하는 데 도움이 될 것입니다. 연속을 두 번 이상 호출하거나 완전히 호출하는 것을 잊어버림으로써 당신을 보호할 것입니다.

### Converting delegate-based code into async/await

대리자 기반 코드를 비동기/대기트로 변환하는 것은 이러한 유형의 API가 어떤 종류의 이벤트가 일어나고 있는지에 따라 다른 대리자 메서드를 호출할 수 있기 때문에 더 복잡한 프로세스입니다.

이것은 자연스럽게 "점피" 코드를 만들 것이다.

호출은 같은 스레드에서 발생할 수 있지만, 메소드를 호출하는 방식 때문에 예측할 수 없습니다.