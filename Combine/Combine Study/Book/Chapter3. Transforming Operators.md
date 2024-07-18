Combine에서, 게시자로부터 오는 값에 대한 작업을 수행하는 방법을 연산자라고 한다.

### **Collection values**

#### collect()
수집 연산자는 게시자의 개별 값의 흐름을 해당 값의 배열로 변환하는 편리한 방법을 제공합니다.
completion 될때까지의 값들을 모아서 배열로 만듦
![](Pasted%20image%2020240717113641.png)

> [!warning]
> 카운트나 제한을 지정할 필요가 없는 collect() 및 기타 버퍼링 연산자로 작업할 때 주의하십시오. 그들은 수신된 값을 저장하기 위해 무제한의 메모리를 사용할 것이다.

> [!info] in RxSwift -> toArray()

### **Mapping values**
#### map(_ :)
당신이 가장 먼저 배우게 될 것은 게시자로부터 방출된 값에 따라 작동한다는 점을 제외하고는 스위프트의 표준 `map`처럼 작동하는 `map`이다.
![](Pasted%20image%2020240717113647.png)
#### Map key paths
오퍼레이터의 `map` 제품군은 또한 주요 경로를 사용하여 값의 하나, 둘 또는 세 개의 속성으로 매핑할 수 있는 세 가지 버전을 포함한다. 시그니처는 이런 형태다.
**• `map<T>(_:)`**
**• `map<T0, T1>(_:_:)`**
**• `map<T0, T1, T2>(_:_:_:)`**
T는 주어진 키 경로에서 발견되는 값의 유형을 나타낸다.

> [!info] in RxSwift -> map()

#### tryMap(_ :)
map인데 안에서 에러 던질 수 있는 map completion에서 error를 잡을 수 있다.

### Flattening publishers
#### flatMap(maxPublishers:_ :)
- multiple upstream publihser 를 single downsteram publisher 로 flatten 하는데 사용
- 더 정확히는, 해당 publihser 의 emission 들을 flatten 함
- `flapMap` 에 의해 리턴되는 publisher는 종종 upstream publihser 와 다른 타입이다
- 주로 하나의 publihser에서 emit 된 요소들을 publihser를 리턴하는 메소드에 전달 하고 싶을 때, 그리고 궁극적으로 다른 publisher 로 부터 emit 된 요소들을 subscribe 하고 싶을때 사용한다
![](Pasted%20image%2020240718095100.png)

1. 다이어그램에서, `flatMap` 은 P1, P2, P3 3 개의 publihser를 받는다.
2. 각각의 publisher 들은 `value` 프로퍼티를 갖고 이것 역시 publihser이다.
3. `flatMap` 은 P1 과 P2 로 부터 받은 `value` publisher의 값을 emit 한다. 하지만 `maxPublishers` 가 2로 세팅 되어 있어 P3는 무시한다.

> [!info] RxSwift 
> - flatMap -> flatMap, 
> - flatMap(maxPublishers:) -> merge(maxConcurrent:)
### **Replacing upstream output**
#### replaceNil(with:)
![](Pasted%20image%2020240718095139.png)
- optional 값을 전달 받고 `nil`을 특정 값으로 교체 해준다.

nil 병합 연산자인 `??` 을 사용하는 것과 `replaceNil` 를 사용하는 것 사이에는 미묘하지만 중요한 차이가 있다.

- `resplaceNil` 은 아니지만 `??` 연산자는 여전히 `nil` 의 결과를 가질 수 있다.
- `replaceNil(with: "-" as String?)` 로 바꾸면 optional은 반드시 언래핑 되어야 한다는 에러가 생길 것이다.

> [!info] RxSwift -> 없다
#### replaceEmpty(with:)
- Publihser 가 값은 emit 하지 않은채로 완료 되었을 때 사용
![](Pasted%20image%2020240718095218.png)
- Publihser 가 값은 emit 하지 않은채로 완료 되었을 때, `reppaceEmpty(with:)` operator 가 값을 넣어 준 뒤 downstream 에 publish 한다.

> [!info] RxSwift -> ifEmpty(default:)
### **Incrementally transforming output**
#### scan(_ : _ :)
- upstream publisher 에서 emit 된 현재 값을 clousre 로 제공하고, 해당 클로저에 의해 리턴 된 마지막 값을 제공한다.
![](Pasted%20image%2020240718095304.png)
1. `scan` 은 시작 값 0 을 저장 하면서 시작한다.
2. Publihser로 부터 각각의 값을 전달 받으면, 이전에 저장 해놓은 값에 더해서 저장하고, 결과를 emit 한다.

- 에러를 던지는 `tryScan` operator 또한 있다. 클로저가 에러를 throw 하면, `tryScan` 은 해당 에러로 fail 된다.
## Wrap-up

- Publihser의 output 대해 작업을 수행하는 메소드를 Operator 라 부른다.
- Operator 도 역시 Publihser 이다.
- Transforming Operator 는 upstream publihser 를 downstream 사용에 적합한 ouput 으로 변환 한다.
- Marble diagram 은 Combine operator 가 어떻게 동작하는지 시각화 하는데 좋은 방법이다.
- `collect` 나 `flatMap` 같이 값을 buffer 하는 operator 를 사용할 때에는 메모리 문제에 유의해야 한다.
- Swift standard library 와 이름이 비슷 한 Combine operator는 비슷하지만 완전히 다른 방식으로 동작 한다.
- 하나의 Subscription 에서 다수의 operator를 체이닝 할 수 있다.

> [!info] RxSwift -> scan
