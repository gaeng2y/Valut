![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/2e543167-b078-4554-bd9d-48693c424cb4/ef256d07-2726-47c4-8b95-6aa50bec4a8a/Untitled.png)

위 코드와 `drawACopy(local: Drawable)` 와 무슨 차이일까

![https://github.com/HARlBO/WWDC/raw/master/WWDC2016/images/Understanding-Swift-Performance/Untitled](https://github.com/HARlBO/WWDC/raw/master/WWDC2016/images/Understanding-Swift-Performance/Untitled) 47.png

- Generic 타입은 **정적 다형성**을 지원한다. 또 다른 말로는 **Parametric Polymorphism**라고도 한다.
- 위와 같은 코드에서 `foo` 메소드가 실행되면, Swift는 제네릭 타입인 T를 Point 타입으로 바인딩한다.
- 그리고 foot 메소드 안에 있는 local 은 Point 타입을 가지게 된다. 즉, T는 Point로 대체가 된다.

결과적으로 위와 같이 타입은 매개변수에 따라서 call 체인으로 대체된다.

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/2e543167-b078-4554-bd9d-48693c424cb4/e2cb9403-1dc5-475e-95db-a7efe803d6b1/Untitled.png)

Swift는 하나의 call 컨텍스트 당 하나의 타입만 존재하기에 Existenial container(e.c)를 활용하지 않고 call site에 추가적인 인자로 vwt와 pwt를 넣어준다.

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/2e543167-b078-4554-bd9d-48693c424cb4/2056a71b-b328-4853-87dc-d58062d8be56/Untitled.png)

그리고, 함수 실행 동안에, Swift는 VWT를 활용하여 메소드들을 실행한다. **allocate**, **copy**

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/2e543167-b078-4554-bd9d-48693c424cb4/d616911d-d945-4377-a82b-24c8e885a5bc/Untitled.png)

그리고 local의 `draw()` 를 실행하면, pwt를 통해 jump한다. 근데 e.c를 활용하지 않고 local의 메모리를 어떻게 할당하는 걸까?

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/2e543167-b078-4554-bd9d-48693c424cb4/733ac484-c6f6-4ea3-8e96-9e1ac8a669ee/Untitled.png)

valueBuffer를 스택에 할당한다. valueBuffer는 3word. 앞에서 다뤘던 것처럼 커지면 힙에 저장한다. 아래와 같이…

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/2e543167-b078-4554-bd9d-48693c424cb4/d5a57ceb-59e1-4375-8ab0-0e4fef5029bc/Untitled.png)

제네릭으로 들어간 콜체인은 existential container를 사용하지 않는다면 단순히 함수가 호출될 때 파라미터의 변수의 valuebuffer가 생긴다.

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/2e543167-b078-4554-bd9d-48693c424cb4/b0ec429b-0d1b-4f92-89d4-d022084e25e0/Untitled.png)

다시 돌아와 위와 같은 코드가 있다고 가정해보고 앞서 정적 다형성은 **One Type at the call-site**라고 했다. 따라서 Swift는 **해당 타입에 맞는 새로운 버전의 함수를 만들어** 제네릭 파라미터를 대체한다.

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/2e543167-b078-4554-bd9d-48693c424cb4/f2bda76d-3169-436b-8579-d64cb1656763/Untitled.png)

위와 같이 `drawACopy<T: Drawable>(local: T)` 에 파라미터로 Point 타입이 들어간다면 `drawACopyOfAPoint(local: Point)` 로 변경된다. 또 다른 타입으로 넣는다면 다른 버전의 메소드가 만들어질 것이다…

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/2e543167-b078-4554-bd9d-48693c424cb4/2cca7abd-fe8d-41f6-822c-b1de43fd80a4/Untitled.png)

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/2e543167-b078-4554-bd9d-48693c424cb4/a580a4c1-79c6-4e09-a86b-e53e843aa3ff/Untitled.png)

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/2e543167-b078-4554-bd9d-48693c424cb4/5da0e5e9-1501-4fe9-9730-958a9e6aa91e/Untitled.png)

그렇다면 제네릭을 쓰면 코드 사이즈가 커지는거 아님? 메소드가 더 증거 하잖슴

**그러나,** static typing infomation은 ****aggresive한 컴파일러 최적화를 가능하게 하므로 코드 사이즈 증가를 막아준다.

위의 새로운 버전으로 생성된 함수 코드는 아래와 같이 인라이닝을 적용한 상태로 변경 된다.

또한 축약하여 더더욱 짧게 만들어준다.

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/2e543167-b078-4554-bd9d-48693c424cb4/53003bc2-44f2-4912-8ed1-6820cd6f20fd/Untitled.png)

결과적으로는 위와 같이 변경된다. `drawACopy` 메소드의 Point는 더이상 참조가 되어지지 않으므로, 컴파일러는 해당 코드를 삭제하고 Line의 경우도 마찬가지로 구현된다.

그렇다면, 언제 이러한 **최적화(Specialization)**이 일어나는 것일까?

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/2e543167-b078-4554-bd9d-48693c424cb4/ec41a52d-7472-4247-b4d4-a77a0c04f12b/Untitled.png)

위와 같이 Point 구조체를 정의하고, 인스턴스를 생성하여 `drawACopy` 메소드의 파라미터로 넣어주면 Swift가 Specialization을 하기 위해서 함수의 call site에서 타입을 추론할 수 있어야한다.(제네릭?) 위와 같은 경우에서는 local을 확인하고, 해당 변수의 초기화 시점으로 돌아간 후, 타입을 추론한다.

그리고 Swift는 Specialization이 되는 동안에 사용된 type과 Generic 함수 그 자체 함수를 정의해야 한다. (Generic 함수와 Point와 같은 타입의 정의부를 한 파일에서 가지고 있어야한다.)

위에서는 main에 모두 있으니 Swift가 Specialization을 할 수 있다.

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/2e543167-b078-4554-bd9d-48693c424cb4/6adc7060-28c2-46b2-bad0-48e4de1f764a/Untitled.png)

그렇다면 정의와 사용을 서로 다른 파일로 옮겨보자. 이렇게 되면, 컴파일러는 서로 다른 파일은 각각 컴파일 하기에 UsePoint 파일을 컴파일할 때 Point의 정의를 알지 못한다.

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/2e543167-b078-4554-bd9d-48693c424cb4/f5e8bc10-ab9c-4a92-a402-422d2ae13ad6/Untitled.png)

그러나, **Whole Module Optimizaiton(전체 모듈 최적화)**를 활용하여 두 파일을 하나의 유닛으로 컴파일 하고, 최적화를 활용할 수 있다.

그렇다면 예를 살펴보자.

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/2e543167-b078-4554-bd9d-48693c424cb4/af7a654e-8272-444e-946e-cc22fd6e07da/Untitled.png)

위와 같은 코드를 2개의 힙 할당을 유발한다.

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/2e543167-b078-4554-bd9d-48693c424cb4/9c29c04a-a748-497f-bf7a-8a5e1e5bd273/Untitled.png)

하지만 제네릭을 활용하게 되면, 컴파일러는 같은 타입으로 파라미터 타입을 설정하도록 강제한다. 즉 두개의 파리미터 모두 Line 으로만 받게 강제된다. 그리고 해당 타입은 런타임 시점에는 변경할 수 없다.

즉, 위 그림과 같이 Pair라는 타입에 Line 타입 인스턴스 두 개가 묶여서 메모리에 올라간다.(ec를 쓰지 않으니 바로 스택에 할당한다. 이렇게 되면 스택에 올라가니 메소드도 스태틱 디스패치?

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/2e543167-b078-4554-bd9d-48693c424cb4/8c4f7423-0a4b-4c63-8417-b22182e213f3/Untitled.png)

우리는 이렇게 EC를 활용하는 unspecialized 코드와 타입 각각의 특정 버전을 만드는 제네릭 함수의 Specialized 코드를 확인해보았다.

## Performance

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/2e543167-b078-4554-bd9d-48693c424cb4/f9526af2-4143-4f45-9d07-3d6440913308/Untitled.png)

- 구조체를 가지고 있는 Generic 코드의 경우, 구조체 타입에 맞게 새로운 함수를 정의한다.
- 구조체 타입을 복사할 때 Heap 할당이 일어나지 않고, 구조체가 참조 프로퍼티를 가지고 있지 않을 경우 레퍼런스 카운팅도 없다.
- 또한 런타임을 줄여주고, 컴파일러 최적화를 해주는 Static Method Dispatch를 하게 됩니다.

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/2e543167-b078-4554-bd9d-48693c424cb4/c585b05f-03d1-4224-9047-d6e7b853fa56/Untitled.png)

- 클래스 타입의 제네릭의 경우, 힙 할당, 레퍼런스 카운팅, VTable을 이용한 동적 디스패치가 발생

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/2e543167-b078-4554-bd9d-48693c424cb4/cc24a49e-1134-413f-a82a-7931becb28f1/Untitled.png)

- 작은 값(e.c를 사용하지 않고)을 사용하는 unspecialized 코드는 valueBuffer에 들어가기에 힙할당 없으니 레퍼런스 카운팅도 없다. 그러나 call site에서 pwt를 활용하여 함수를 호출하기 때문에 동적 디스패치

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/2e543167-b078-4554-bd9d-48693c424cb4/bcd927dc-bb8b-4c1b-b31c-a87e42cb0235/Untitled.png)

- 큰 값인 경우는 valueBuffer에 들어가지 않으므로 indirect 스토리지를 활용한 heap 할당을 유발할 수 있다.
- 또한 값이 레퍼런스 값을 가지고 있다면, 하나의 제네릭 함수의 구현을 공유하는 다이나믹 디스패치를 활용합니다.

## Summary

적합한 추상화를 선택하여 동적 런타임 타입 요구 사항을 최소화해라

- **구조체 타입**: 값 의미 (value semantics)
- **클래스 타입**: 정체성(identity) 또는 객체 지향 스타일의 다형성 (OOP style polymorphism)
- **제네릭**: 정적 다형성 (static polymorphism)
- **프로토콜 타입**: 동적 다형성 (dynamic polymorphism)

큰 값을 처리하기 위해 간접 저장(indirect storage)을 사용하자.