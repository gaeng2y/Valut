반응형 프로그래밍의 핵심 아이디어는 시간 흐름에 따른 비동기 이벤트 흐름을 모델링 하는 것이다. 이러한 측면에서, Combine 프레임워크는 시간을 다룰 수 있는 operator 들을 제공 한다. 특히, 시간이 지남에 따라 시퀀스가 어떻게 반응하고 값을 변형 하는지.

# Shifting time

Combine은 과거 관계 실수를 고치는 데 도움이 되지 않지만, 자가 복제가 가능할 때까지 기다릴 수 있도록 잠시 동안 시간을 정지시킬 수 있다.

가장 기본적인 시간 조작 연산자는 게시자가 방출한 값을 지연시켜 실제로 발생하는 것보다 나중에 볼 수 있도록 한다.

`delay(for:tolerance:scheduler:options)` operator 는 값 시퀀스 전체의 시간을 옮긴다. Upstream publisher 가 값을 emit 할 때 마다, `delay` 는 잠시 동안 유지한 후에 Scheduler 에 명시 해놓은 만큼 지연 시킨다.

추후에 조정 할 수 있는 두가지 상수를 정의 한다.

매 초마다 하나의 값을 emit 하는 publisher 를 생성하고, 1.5 초 지연 시키고, 비교를 위해 두가지 타임라인을 동시에 표시한다.

```swift
let valuesPerSecond = 1.0
let delayInSeconds = 1.5

let sourcePublisher = PassthroughSubject<Date, Never>()

let delayedPublisher = sourcePublisher.delay(for: .seconds(delayInSeconds), scheduler: DispatchQueue.main)

let subscription = Timer
    .publish(every: 1.0 / valuesPerSecond, on: .main, in: .common)
    .autoconnect()
    .subscribe(sourcePublisher)
```

> [!note]
> 특정 타이머는 Foundation `Timer` 클래스에 있는 Combine extension 이다. `RunLoop` 와 `RunLoop.Mode` 를 갖고, `DispatchQeue` 는 아니다. 또한, timer 는 connectable 한 publisher 의 클래스의 한 부분이다. 이것은 값을 emit 하는 것을 시작 하기 전에 연결 되어야 한다는 것이다. 첫번째 subscription 에서 즉시 연결하는 `autoconnect()` 를 사용하면 된다.

```swift
let sourceTimeline = TimelineView(title: "Emitted values (\(valuesPerSecond) per sec.):")
let delayedTimeline = TimelineView(title: "Delayed values (\(delayInSeconds)s delay):")

let view = VStack(spacing: 50) {
    sourceTimeline
    delayedTimeline
}

PlaygroundPage.current.liveView = UIHostingController(rootView: view.frame(width: 375, height: 600))
```

위 코드를 추가 하면, 화면에 두개의 빈 타임라인을 볼 수 있다. 각각의 publisher 에서 emit 된 값을 넣어 주어야 한다.

```swift
sourcePublisher.displayEvents(in: sourceTimeline)
delayedPublisher.displayEvents(in: delayedTimeline)
```

![](Combine/Combine%20Study/Resources/Pasted%20image%2020240806164043.png)

위에 타임라인은 timer 에 의해 emit 된 값을 보여준다. 아래는 값은 값을 지연된 타임라인을 보여 준다.

원 안의 숫자들은 실제 그들의 값이 아닌 emit 된 count 를 반영한다.
# Collection values

특정 상황에서, 지정된 간격으로 publisher 로 부터 emit 된 값을 수집 할 필요가 있을 것이다. 이것은 유용하게 쓰일 수 있는 버퍼링의 한 형태다.

예를 들어서, 짧은 기간 동안 값들의 평균을 내고 평균 값을 output 하고 싶을 때.

```swift
let valuesPerSecond = 1.0
let collectTimeStride = 4

let sourcePublisher = PassthroughSubject<Date, Never>()

let collectedPublisher = sourcePublisher
    .collect(.byTime(DispatchQueue.main, .seconds(collectTimeStride)))

let subscription = Timer
    .publish(every: 1.0 / valuesPerSecond, on: .main, in: .common)
    .autoconnect()
    .subscribe(sourcePublisher)

let sourceTimeline = TimelineView(title: "Emitted values:")
let collectedTimeline = TimelineView(title: "Collected values (every \(collectTimeStride)s):")

let view = VStack(spacing: 40) {
    sourceTimeline
    collectedTimeline
}

PlaygroundPage.current.liveView = UIHostingController(rootView: view.frame(width: 375, height: 600))

sourcePublisher.displayEvents(in: sourceTimeline)
collectedPublisher.displayEvents(in: collectedTimeline)
```

![](Combine/Combine%20Study/Book/Pasted%20image%2020240806165047.png)

Emitted values 타임라인에는 일정한 값이 emit 되고, 아래 Collected values 는 매 4초 마다 단일 값이 표시 된다.

각 4초 동안 받는 값이 array 이고 이 값을 보기 위해 `collectedPublisher` 에 `flatMap` operator 를 추가해 보자

```swift
let collectedPublisher = sourcePublisher
  .collect(.byTime(DispatchQueue.main, .seconds(collectTimeStride)))
  .flatMap { dates in dates.publisher }
```

`collect` 가 수집한 값의 그룹을 emit 할 때마다, `flatMap` 은 각각의 값으로 분해하고 즉시 바로 다음 값을 emit 한다. `publisher` extension 인 `Collection` 을 사용해 값의 시퀀스를 모든 시퀀스를 각각 개별 값으로 즉시 emit 하는 Pulisher 에게 보낸다.

![](Combine/Combine%20Study/Book/Pasted%20image%2020240806165257.png)

이제 매 4초마다, `collect` 가 지난 시간 간격 동안 수집 된 값의 배열을 emit 하는 것을 볼 수 있다.
# Collecting values (part 2)

`collect(_*:options:*)` operator 가 제공하는 두번째 옵션은 규칙적인 간격으로 값을 수집 할 수 있게 해주는 것이다. 또한 수집된 값의 개수에 제한을 걸 수 있다.

```swift
let valuesPerSecond = 1.0
let collectTimeStride = 4
let collectMaxCount = 2

let sourcePublisher = PassthroughSubject<Date, Never>()

let collectedPublisher = sourcePublisher
 .collect(.byTime(DispatchQueue.main, .seconds(collectTimeStride)))
 .flatMap { dates in dates.publisher }
let collectedPublisher2 = sourcePublisher
    .collect(.byTimeOrCount(DispatchQueue.main, .seconds(collectTimeStride), collectMaxCount))
    .flatMap { dates in dates.publisher }

let subscription = Timer
    .publish(every: 1.0 / valuesPerSecond, on: .main, in: .common)
    .autoconnect()
    .subscribe(sourcePublisher)

let sourceTimeline = TimelineView(title: "Emitted values:")
let collectedTimeline = TimelineView(title: "Collected values (every \(collectTimeStride)s):")
let collectedTimeline2 = TimelineView(title: "Collected values (at most \(collectMaxCount) every \(collectTimeStride)s):")

let view = VStack(spacing: 40) {
    sourceTimeline
    collectedTimeline
    collectedTimeline2
}

PlaygroundPage.current.liveView = UIHostingController(rootView: view.frame(width: 375, height: 600))

sourcePublisher.displayEvents(in: sourceTimeline)
collectedPublisher.displayEvents(in: collectedTimeline)
collectedPublisher2.displayEvents(in: collectedTimeline2)
```

![](Combine/Combine%20Study/Book/Pasted%20image%2020240806165401.png)
두번째 타임라인이 `collectMaxCount` 상수에 요구된 시간마다 두개의 값을 수집하도록 제한 하는 것을 볼 수 있다.