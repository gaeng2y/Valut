# Race Condition

```swift
import Foundation

var cards = [1, 2, 3, 4, 5, 6, 7, 8, 9]

DispatchQueue.global().async {
    for _ in 1...3 {
        let card = cards.removeFirst()
        print("야곰: \(card) 카드를 뽑았습니다!")
    }
}

DispatchQueue.global().async {
    for _ in 1...3 {
        let card = cards.removeFirst()
        print("노루: \(card) 카드를 뽑았습니다!")
    }
}

DispatchQueue.global().async {
    for _ in 1...3 {
        let card = cards.removeFirst()
        print("오동나무: \(card) 카드를 뽑았습니다!")
    }
}
```

cards라는 카드 묶음에는 1부터 9까지의 카드가 있다. 야곰, 노루, 오동나무는 이 카드를 3장씩 나누어 가지려 하는데, 순서에 상관없이 앞 번호의 카드부터 가져가기로 했다. 어떻게 되는지 살펴보자.

```
/* 출력
야곰: 1 카드를 뽑았습니다!
노루: 1 카드를 뽑았습니다!
오동나무: 1 카드를 뽑았습니다!
야곰: 2 카드를 뽑았습니다!
노루: 5 카드를 뽑았습니다!
야곰: 6 카드를 뽑았습니다!
노루: 8 카드를 뽑았습니다!
오동나무: 7 카드를 뽑았습니다!
오동나무: 9 카드를 뽑았습니다!
*/
```

실행할 때마다 출력 값이 달라질 것이다. 1 카드를 3명이 모두 가져가는 상황은 발생할 수 없는데 발생했다.

이러한 문제가 발생하는 이유는 하나의 배열에 `**여러 스레드가 동시에 접근해서**`이다. 여러 개의 스레드를 사용하지 않는 프로그래밍에서는 어떤 상황에서나 코드가 순서대로 실행이 되지만 스레드가 여러 개인 상황에서는 코드가 동시에 실행되어 `하나의 값에 동시에 접근하는 경우`가 발생할 수 있다. 이러한 상황을 **Race Condition**이라고 한다.

# Thread Safe

또한 Race Condition이 발생하는 이유는 Swift의 배열이 **Thread Safe**하지 않기 때문이다. Thread Safe 하다는 것은 여러 스레드에서 동시에 접근이 불가능한 것을 말한다. 실제로 Swift의 대부분 타입들은 Thread Unsafe하다. 