## Prepending
### prepend
- Output 파라미터를 이용하면 똑같은 해당 스트림 전에 Output을 먼저 방출한다.
- Sequence 파라미터를 이용하면 위와 똑같이 동작하는데 스트림 순서대로 먼저 방출한다.
- publisher를 붙이면 merge랑 비슷하게 동작하되 앞에 붙임
## Appending
### append
- Output 파라미터를 이용하면 똑같은 해당 스트림 뒤에 Output을 방출한다.
- Sequence 파라미터를 이용하면 위와 똑같이 동작하는데 스트림 순서대로 기존 뒤에 방출한다.
- publisher를 붙이면 merge랑 비슷하게 동작하되 뒤에 붙임

> [!info]
> RxSwift에서 concat으로 통일
## Advanced combining
### switchToLatest
- 보류 중인 게시자 구독을 취소하는 동안 전체 게시자 구독을 즉시 전환하여 최신 구독으로 전환할 수 있다.
- 그림으로 보면 이해가 쉽다
![](https://github.com/HARlBO/Combine/raw/main/Combine/Section2_Operators/images/Chapter5_Combining_Operators/Untitled6.png)
- 유저가 네트워크 요청을 트리거 하는 버튼을 탭한다. 그 즉시, 유저가 두번째 네트워크 요청을 트리거하는 그 버튼을 또 탭한다. 보류 중인 요청은 어떻게 제거하고 가장 최신 것만 사용 할 때 `switchToLatest` 를 사용 하면 된다.
### merge
- 이 opreator 는 동일한 유형의 다른 publisher 로 부터 emission 값들을 상호 배치 한다(interleaves).
### combineLatest
- 다른 publisher 들을 조합하는 또다른 operator 이다. 다른 값 타입의 publisher 를 조합 가능하다.
- 하지만, 모든 publisher 의 emission 을 조합하는 대신, publisher 가 값을 emit 할 때 가장 최신의 값을 튜플 형태로 emit 한다.
- `combineLatest` 로 전달된 Origin publisher 와 모든 publisher 는 `combineLatest` 가 어떤 것을 emit 하기 전에, 적어도 하나의 값을 emit 해야만 한다.
### zip
- Swift standard library 메소드의 `Sequence` 타입인 것에서 온 것을 알 수 있다. 
- 이 operator 는 동일한 인덱스의 값의 쌍을 튜플로 emit 한다. publisher 가 아이템을 emit 하기를 기다리고, 모든 publisher 가 현재 인덱스의 모든 값을 emit 했을 때 튜플을 emit 한다.
- 이것은 두개의 publisher 를 zip 했을 때, 두개의 publisher 에서 emit 된 값 두개로 단일 튜플을 emit 받는 다는 것을 의미한다.
## Wrapup
- `prepend` 와 `append` operator 는 original publisher 의 전이나 이후에 하나의 publisher 에 emission 을 추가 할 수 있다.
- `switchToLatest` 는 복잡하지만 매우 유용하다. publisher 를 받아, 최신의 publisher 로 교체하고 이전 publisher 의 subscription 은 취소한다.
- `merge(with:)` 는 다수의 publisher 의 값을 interleave 할 수 있다.
- `combineLatest` 는 모든 조합된 publisher 가 적어도 하나의 값을 emit 한 후에, 모든 조합 된 publisher 가 값을 emit 할 때 마다, 가장 최신의 값을 emit 한다.
- `zip` 은 모든 publisher 가 값을 emit 한 후에, 다른 publisher 의 emission들의 튜플로 쌍을 지어준다.
- Combination operator publisher 와 그들의 emission 간의 관계를 흥미롭고 복잡한 하게 만들기 위해 혼합 시킬 수 있다.