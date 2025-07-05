> [!note]
> 이 장의 대부분의 연산자는 try 접두사와 유사점을 가지고 있습니다. 
> 예를 들어, filter vs. tryFilter. 그들 사이의 유일한 차이점은 후자가 던지는 폐쇄를 제공한다는 것이다. 
> 클로저 내에서 발생하는 모든 오류는 발생한 **오류로 게시자를 종료**합니다. 간결함을 위해, 이 장은 사실상 동일하기 때문에 던지지 않는 변형만 다룰 것이다.
# Filtering basics
## filter
Swift Standard Library에 있는 filter와 같은데 값 방출하는거만 다름
![filter](https://github.com/gaeng2y/Valut/blob/main/Combine/Combine%20Study/Resources/Pasted%20image%2020240722152803.png?raw=true)
> [!tip] RxSwift: filter
## removeDuplicates()
앱의 수명 동안 여러 번, 무시하고 싶을 수도 있는 동일한 값을 연속으로 방출하는 게시자가 있습니다.
![removeDuplicates](https://github.com/gaeng2y/Valut/blob/main/Combine/Combine%20Study/Resources/Pasted%20image%2020240722153113.png?raw=true)
> [!tip] RxSwift: distinctUntilChanged

# Compacting and ignoring
## compactMap
Swift Standard Library에 있는거랑 똑같음
![compactMap](https://github.com/gaeng2y/Valut/blob/main/Combine/Combine%20Study/Resources/Pasted%20image%2020240722153645.png?raw=true)
> [!tip] RxSwift: compactMap

## ignoreOutput()
때때로, 당신이 알고 싶은 것은 출판사가 실제 가치를 무시하고 가치를 방출하는 것을 끝냈다는 것이다. 그러한 시나리오가 발생하면, ignoreOutput 연산자를 사용할 수 있다.
![ignoreOutput](https://github.com/gaeng2y/Valut/blob/main/Combine/Combine%20Study/Resources/Pasted%20image%2020240722153902.png?raw=true)
> [!tip] RxSwift: ignoreElements
# Finding values
## first(where:)
Swift Standard Library와 똑같음
![first](https://github.com/gaeng2y/Valut/blob/main/Combine/Combine%20Study/Resources/Pasted%20image%2020240722154142.png?raw=true)
> [!tip] RxSwift: 조건을 걸어서 가져오는 생성자는 없음

## last(where:)
Swift Standard Library와 똑같음
![last](https://github.com/gaeng2y/Valut/blob/main/Combine/Combine%20Study/Resources/Pasted%20image%2020240722154236.png?raw=true)
> [!tip] RxSwift: 조건을 걸어서 가져오는 생성자는 없음

# Dropping values
## dropFirst(_ :)
dropFirst 연산자는 카운트 매개 변수( 생략된 경우 기본값은 1)를 취하고 게시자가 방출한 첫 번째 카운트 값을 무시한다.
![dropFirst](https://github.com/gaeng2y/Valut/blob/main/Combine/Combine%20Study/Resources/Pasted%20image%2020240722155333.png?raw=true)
> [!tip] RxSwift: skip(_ :)

## drop(while:)
이것은 predicate 클로저를 취하고 predicate가 처음 충족될 때까지 게시자가 방출한 값을 무시하는 또 다른 매우 유용한 변형이다. 
![dropwhile](https://github.com/gaeng2y/Valut/blob/main/Combine/Combine%20Study/Resources/Pasted%20image%2020240722155519.png?raw=true)
> [!tip] RxSwift: skipWhile

## drop(untilOutputFrom:)
두 번째 게시자로부터 요소를 받을 때까지 업스트림 게시자의 요소를 무시한다.
![dropUntil](https://github.com/gaeng2y/Valut/blob/main/Combine/Combine%20Study/Resources/Pasted%20image%2020240722160151.png?raw=true)
> [!tip] RxSwift: skipUntil

# Limiting values
## prefix(_ :)
drop의 반대로 시작부터 파라미터의 값 개수만큼만 방출한다.
![prefix](https://github.com/gaeng2y/Valut/blob/main/Combine/Combine%20Study/Resources/Pasted%20image%2020240722160312.png?raw=true)
> [!tip] RxSwift: take

## prefix(while:)
drop의 반대로 시작부터 while 파라미터의 조건만큼만 방출한다.
![prefixWhile](https://github.com/gaeng2y/Valut/blob/main/Combine/Combine%20Study/Resources/Pasted%20image%2020240722160433.png?raw=true)
> [!tip] RxSwift: X

## prefix(untilOutputFrom:)
다른 게시자가 요소를 내보낼 때까지 요소를 다시 게시한다.
![prefixUntil](https://github.com/gaeng2y/Valut/blob/main/Combine/Combine%20Study/Resources/Pasted%20image%2020240722160541.png?raw=true)
> [!tip] RxSwift: takeUntil