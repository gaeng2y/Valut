```swift
class DispatchGroup : DispatchObject
```

`DispatchGroup`은 비동기적으로 처리되는 작업들을 그룹으로 묶어, 그룹 단위로 작업 상태를 추적할 수 있는 기능이다. 

`DispatchGroup`을 사용하면 async들을 묶어서 그룹의 작업이 끝나는 시점을 추적하여 어떠한 동작을 수행시킬 수 있다.

이때 묶어줄 비동기 작업들이 꼭 같은 큐, 스레드에 있지 않더라도 묶어줄 수 있다.

`DispatchGroup`은 async에서만 사용할 수 있다. 왜냐하면 비동기로 처리되는 작업은 작업이 끝나는 시점을 정확하게 예측할 수 없기 때문이다.

# group에 등록하기: enter, leave

`DispatchGroup`을 사용하는 방법은 두 가지가 있다.
- async를 호출하면서 파라미터로 group을 지정
- enter, leave를 코드의 앞뒤로 호출하여 group을 지정해준다.

enter와 leave는 DispatchGroup이 `enter()부터 leave()까지 포함된다`라는 의미다.

```swift
let group = DispatchGroup()

// enter, leave를 사용하지 않는 경우
DispatchQueue.main.async(group: group) {}
DispatchQueue.global().async(group: group) {}

// enter, leave를 사용하는 경우
group.enter()
DispatchQueue.main.async {}
DispatchQueue.global().async {}
group.leave()
```

이렇게 원하는 작업들을 group으로 묶어줄 수 있다. 이제 묶어낸 그룹에 대해 `notify()` 혹은 `wait()` 으로 작업을 추적해줄 수 있다.

# notify

notify는 DispatchGroup의 업무 처리가 끝나는 시점에 원하는 동작을 수행하기 위한 메소드이다.

```swift
import Foundation

let red = DispatchWorkItem {
    for _ in 1...5 {
        print("🥵🥵🥵🥵🥵")
        sleep(1)
    }
}

let yellow = DispatchWorkItem {
    for _ in 1...5 {
        print("😀😀😀😀😀")
        sleep(1)
    }
}

let blue = DispatchWorkItem {
    for _ in 1...5 {
        print("🥶🥶🥶🥶🥶")
        sleep(2)
    }
}

let group = DispatchGroup()

DispatchQueue.global().async(group: group, execute: blue)
DispatchQueue.global().async(group: group, execute: red)

// group.enter()
// DispatchQueue.global().async(execute: blue)
// DispatchQueue.global().async(execute: red)
// group.leave()

group.notify(queue: .main) {
    print("모든 작업이 끝났습니다.")
}
```

이 코드를 실행하게 된다면, `notify` 메서드에 의해 group의 모든 작업이 끝나기를 기다렸다가 코드 블록을 실행시켜준다. 이때 notify의 파라미터 `queue`는 코드블록을 실행시킬 queue를 말한다.

# wait

wait는 DispatchGroup의 수행이 끝나기를 기다리기만 하는 메소드이다. notify와 달리 별도의 코드 블록을 실행하지 않는다. 따라서 코드 블록을 실행시킬 queue를 지정할 필요도 없다.

```swift
let group = DispatchGroup()

DispatchQueue.global().async(group: group, execute: blue)
DispatchQueue.global().async(group: group, execute: red)

group.wait()
print("모든 작업이 끝났습니다.")

// group.wait(timeout: 10)
// print("모든 작업이 끝났습니다.")
```

wait 메서드에는 `timeout` 파라미터를 설정해줄 수 있다. 

만약 timeout에 10을 전달하면 group을 딱 10초 동안만 기다리는 것이다. 

만약 **10초가 넘어갔는데도 group의 작업이 끝나지 않는다면 더 이상 기다리지 않고 다음 코드를 실행해버린다.** (asyncAfter와 마찬가지로 wallTimeout을 파라미터로 받을 수도 있다.)

DispatchGroup을 사용하면 이러한 문제를 해결할 수도 있다. 예를 들어 `global()`의 호출로 생성된 스레드에서 다시 `global()`을 호출하는 상황을 생각해보자.

```swift
import Foundation

let black = DispatchWorkItem {
    for _ in 1...3 {
        print("🖥🖥🖥🖥🖥")
        sleep(1)
    }
}

let white = DispatchWorkItem {
    for _ in 1...3 {
        print("📃📃📃📃📃")
        sleep(2)
    }

    DispatchQueue.global().async(execute: yellow)
}

let group = DispatchGroup()

DispatchQueue.global().async(group: group, execute: black)
DispatchQueue.global().async(group: group, execute: white)

group.wait()
print("모든 작업이 끝났습니다.")

/* 출력
🖥🖥🖥🖥🖥
📃📃📃📃📃
🖥🖥🖥🖥🖥
📃📃📃📃📃
🖥🖥🖥🖥🖥
📃📃📃📃📃
😀😀😀😀😀
모든 작업이 끝났습니다.
😀😀😀😀😀
😀😀😀😀😀
😀😀😀😀😀
😀😀😀😀😀
*/
```

black과 white 작업을 group으로 묶어서 `wait()`를 호출해주었지만 😀😀😀😀😀이 모두 출력되기 전에 다음 코드가 실행되었다. 비동기 작업 내부에서 파생된 비동기 작업에 대해서는 기다려주지 않은 것이다. 같은 그룹으로 묶어주었지만 왜 이런 상황이 발생했을까?

white의 작업은 😀😀😀😀😀를 또 다른 global 스레드에서 `async`로 처리하도록 DispatchQueue에 요구하고 있다. 즉 다른 스레드에 일을 넘기는 것으로 **white의 작업은 끝난 것.**