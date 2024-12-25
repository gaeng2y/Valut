```swift
class DispatchSemaphore : DispatchObject
```

DispatchSemaphore는 사실은 꼭 Race Condtion을 해결하기 위한 수단만은 아니다. 정확히는 공유 자원에 접근할 수 있는 스레드의 수를 제어해주는 역할을 한다. 몇 개의 스레드에 접근을 허용할 것인지 제어할 수 있기 때문에 접근을 1개의 스레드만 허용한다면 Race Condition을 방지할 수 있는 것이다.

DispatchSemaphore는 semaphore count를 카운트하는 식으로 동작한다. 예를 들어 하나의 스레드가 접근을 하면 count에 `-1`을, 접근이 끝나면 count에 `+1`을 해준다. 따라서 설정해준 count만큼만 스레드가 접근할 수 있도록 관리해주는 것이다. 

만약 허용된 스레드의 수만큼 접근된 상태라면 다른 스레드는 접근하지 못하고 줄을 서서 기다리게 된다. count가 다시 +1이 되면 기다리는 스레드가 순서대로 접근이 허용된다.

```swift
let semaphore = DispatchSemaphore(value: 1) // count = 1

DispatchQueue.global().async {
    semaphore.wait() // count -= 1

    semaphore.signal() // count += 1
}
```

wait()는 값에 접근했다고 알리는 메서드, signal()은 볼 일 다봤다는 메서드인 셈이다.

> [!info] DispatchSemaphore를 사용할 때에는 주의 사항은 반드시 `wait()와 signal()을 짝지어서` 호출해주어야 한다.

```swift
import Foundation

var cards = [1, 2, 3, 4, 5, 6, 7, 8, 9]
let semaphore = DispatchSemaphore(value: 1)

DispatchQueue.global().async {
    for _ in 1...3 {
		semaphore.wait()
        let card = cards.removeFirst()
        print("야곰: \(card) 카드를 뽑았습니다!")
        semaphore.signal()
    }
}

DispatchQueue.global().async {
    for _ in 1...3 {
	    semaphore.wait()
        let card = cards.removeFirst()
        print("노루: \(card) 카드를 뽑았습니다!")
	    semaphore.signal()
    }
}

DispatchQueue.global().async {
    for _ in 1...3 {
	    semaphore.wait()
        let card = cards.removeFirst()
        print("오동나무: \(card) 카드를 뽑았습니다!")
        semaphore.signal()
    }
}
```

# Serial Queue를 활용하여 race condition 해결

DispatchSemaphore를 활용하여 해결할 수도 있지만, SerialQueue를 활용하여 문제를 해결할 수 있다.

Race Condition이 발생하는 이유는 여러 스레드에서 `질서없이` 배열에 접근했기 때문이다. 

그러므로 우리는 DispatchSemaphore에서 했던 것 처럼 `질서`를 만들어주면 이 문제를 해결할 수 있을 것이다.

```swift
import Foundation

var cards = [1, 2, 3, 4, 5, 6, 7, 8, 9]
let pickCardsSerialQueue = DispatchQueue(label: "PickCardsQueue")

DispatchQueue.global().async {
    for _ in 1...3 {
        pickCardsSerialQueue.sync {
            let card = cards.removeFirst()
            print("야곰: \(card) 카드를 뽑았습니다!")
        }
    }
}

DispatchQueue.global().async {
    for _ in 1...3 {
        pickCardsSerialQueue.sync {
            let card = cards.removeFirst()
            print("노루: \(card) 카드를 뽑았습니다!")
        }
    }
}

DispatchQueue.global().async {
    for _ in 1...3 {
        pickCardsSerialQueue.sync {
            let card = cards.removeFirst()
            print("오동나무: \(card) 카드를 뽑았습니다!")
        }
    }
}
```