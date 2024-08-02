일을 효율적으로, 그리고 동시에 처리하기 위해서는 `스레드`를 여러 개 활용하는 것은 거의 불가피하다. 
그렇다면 우리는 어떻게 스레드를 관리해주어야할까? 애플에서는 코드로서 동기/비동기 처리만 해준다면 시스템이 알아서 스레드를 관리해주는 방식을 제공하고 있는데, 그것이 바로 `Dispatch(a.k.a Grand Central Dispatch)`이다.
# DispatchQueue
```swift
class DispatchQueue : DispatchObject
```

- Disaptch: 보내다(파견하다)
- Queue: 대기열

DispatchQueue는 **대기열에 보내다**라는 뜻이다. DispatchQueue에 작업을 넘겨주기만 하면 알아서 동작한다. DispatchQueue는 GCD를 사용하기 위한 `대기열`로, GCD 기술의 일부다. 우리는 이 대기열 들에 작업을 추가해주기만 하면 시스템은 알아서 스레드를 관리하여 작업을 처리하도록 도와줄 것이다. DispatchQueue는 `FIFO(First In, First Out)`로 작업을 처리한다.

단, DispatchQueue에 작업을 넘길 때 두 가지를 꼭 정해주어야 한다. 직렬/동시성을 사용할 것인가 (**Serial/Concurrent**) 그리고 동기/비동기로 처리할 것인가.

# Serial / Concurrent

- **Serial은 단일 스레드**에서 작업을 처리
- **Concurrent는 다중 스레드**에서 작업을 처리
![](https://user-images.githubusercontent.com/73867548/146465232-3ff833a1-3902-4fb7-980e-775acc5f755a.png)

> [!quote] main은 프로퍼티, global()은 메소드인 이유?
> 메인 스레드는 메모리에 늘 올라와 있는 Default 스레드이지만 여러 스레드에서 여러 작업을 동시에 하는 Concurrent Queue는 메인 스레드 외에 **새로운 스레드를 만들어서 사용**하게 되기에 나뉜 것으로 추론된다.

# main / global

- main: 메인 스레드(Serial Queue) (UI는 무조건 메인에서 처리해야 한다.)
- global: 메인 스레드가 아닌, 작업을 처리하기 위해 발생한 스레드 global 스레드는 main 스레드와는 달리 `global()`이 호출되면 작업을 처리하기 위해 `메모리에 올라왔다가, 작업이 끝나고 나면 메모리에서 제거`된다.

DispatchQueue로 GCD를 구현하기 위해서는 2가지를 정해주어야한다고 했다. 이렇게 Serial Queue에서 작업할 건지, Concurrent Queue에서 작업할 것인지를 정해주었다면 다음으로는 동기/비동기(`sync / async`) 처리를 정해주면 된다.
# Main Thread

**메인 스레드**는 앱의 기본이 되는 스레드다. 모든 앱은 적어도 하나 이상의 스레드가 필요한데요, 모든 작업은 스레드 위에서 처리되기 때문이다. 

이 메인 스레드는 앱의 생명주기와 같은 생명주기를 가지는, `앱이 실행되는 동안에는 늘 메모리에 올라와있는 기본 스레드`이다. 

즉 메인 스레드가 멈추는 것은 앱이 멈추는 것이며 메인 스레드가 존재하지 않으면 앱은 동작할 수 없다. 

동시성 프로그래밍에서는 여러 개의 스레드를 사용한다고 했다. 이 경우에도 메인 스레드는 늘 메모리에 올라온 상태로 존재하며 `메인 스레드에서부터 필요한 만큼의 스레드가 파생되는 것이다.` 

이때 파생되는 스레드들은 자신이 담당하는 작업이 처리되면 메모리에서 사라지게 되지요. 메인 스레드는 그림자 분신술의 실체인 셈이다.
![](https://user-images.githubusercontent.com/73867548/146414284-f9d1ac02-64a4-49a1-bc25-2fd04bd96f3c.png)
- 전역적으로 사용이 가능합니다.
- global 스레드들과는 다르게 **Run Loop**가 자동으로 설정되고 실행된다. 메인 스레드에서 동작하는 Run Loop를 Main Run Loop라고 한다.
- **UI 작업은 메인 스레드에서만** 작업할 수 있다.