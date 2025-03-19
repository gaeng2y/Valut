컴퓨팅 초기 시절에는 프그램이 명렁어 시퀀스로 작성되어 제어 흐름이 여기저기로 튀는 것을 허용했기 때문에 읽기 어려웠다

오늘날에는 언어에서 구조화된 프로그래밍을 사용하여 제으 흐름을 균일하게 만들기 때문에 그러한 일은 거의 없다.

예를 들어, if-then 문은 구조화된 제어 흐름을 사용. 중첩된 코드 블록이 위에서 아래로 이동하는 동안에만 조건부로 실행되도록 지정

Swift에서는 해당 블록도 정적 범위(scoping)를 따르며, 이는 이름들이 오직 그 이름이 정의된 외부 블록에서만 보인다는 것을 의미. 또한, 이는 블록 내에서 정의된 변수들의 생명 주기가 블록을 벗어날 때 끝난다는 것을 의미.

따라서 **정적 범위를 갖춘 구조적 프로그래밍은 제어 흐름과 변수의 수명을 이해하기 쉽게 만든다.**

더 일반적으로, 구조화된 제어 흐름은 자연스럽게 순차화되고 중첩될 수 있다. 이렇게 하면 코드 전체를 위에서 아래까지 읽을 수 있다.

이것이 **구조적 프로그래밍의 기본**이다.

하지만 오늘날의 프로그램은 비동기적이고 동시성 있는 코드를 특징으로 하며, 그런 코드를 더 쉽게 작성하기 위해 구조적 프로그래밍을 사용할 수 없다.

먼저, 구조적 프로그래밍이 비동기 코드를 어떻게 더 단순하게 만드는지 살펴보자.

```swift
// 인터넷에서 여러 이미지를 가져와서 순차적으로 썸네일 크기를 조절해야 하는 코드
func fetchThumbnails(
    for ids: [String],
    completion handler: @escaping ([String: UIImage]?, Error?) -> Void
) {
    guard let id = ids.first else { return handler([:], nil) }
    let request = thumbnailURLRequest(for: id)
    let dataTask = URLSession.shared.dataTask(with: request) { data, response, error in
        // 콜백 패턴의 결과로, 이 함수는 오류 처리를 위한 구조적 제어 흐름을 사용할 수 없다.
        // 오류가 함수 내에서 발생했을 때는 해당 오류를 외부로 던져야 하며, 그렇지 않으면 구조적 제어 흐름을 통해 오류를 처리하는 것이 의미가 없다
        guard let response = response,
              let data = data
        else {
            return handler(nil, error)
        }
        // ... check response ...
        UIImage(data: data)?.prepareThumbnail(of: thumbSize) { image in
            guard let image = image else {
                return handler(nil, ThumbnailFailedError())
            }
            fetchThumbnails(for: Array(ids.dropFirst())) { thumbnails, error in
                // ... add image to thumbnails ...
            }
        }
    }
    dataTask.resume()
}
```

그러면 async/await를 사용하는 구조적인 코드를 살펴보자.

```swift
func fetchThumbnails(for ids: [String]) async throws -> [String: UIImage] {
    var thumbnails: [String: UIImage] = [:]
    for id in ids {
        let request = thumbnailURLRequest(for: id)
        let (data, response) = try await URLSession.shared.data(for: request)
        try validateResponse(response)
        guard let image = await UIImage(data: data)?.byPreparingThumbnail(ofSize: thumbSize) else {
            throw ThumbnailFailedError()
        }
        thumbnails[id] = image
    }
    return thumbnails
}
```

하지만 이 코드는 처리해야하는 썸네일의 양이 많아진다면 썸네일을 하나씩 처리하는 방식은 더 복잡하고 성능에 문제가 생긴다.

이제 여러 다운로드가 병렬로 발생할 수 있도록 동시성을 추가할 기회가 생겼다. 프로그램에 동시성을 추가하기 위해 추가적인 작업을 생성할 수 있다. 

Task는 Swift의 새로운 기능으로, async 함수와 함께 작동. Task는 비동기 코드를 실행할 수 있는 새로운 실행 컨텍스트를 제공. 각 Task는 다른 실행 컨텍스트와 동시에 실행된다. Task은 안전하고 효율적으로 병렬 실행할 수 있을 때 자동으로 병렬로 실행된다.
## Structured concurrency
**Sequential bindings**
async-let 바인딩을 사용하여 간단한 작업부터 살펴보자.

```swift
let result = URLSession.shared.data(...)
```

일반적인 let 바인딩은 **=** 를 기준으로 왼쪽에는 변수명 오른쪽에는 초기화 표현식이 있다. 

Swift가 let 바인딩에 도달하면 초기화 프로그램이 평가되어 값을 생성한다. 위의 코드 같은 상황에서는 URL을 이용하기 때문에 시간이 많이 걸릴 수 있다. 데이터가 다운로드된 후 Swift는 다음 명령문을 진행하기 전에 해당 값을 변수 이름에 바인딩한다. 

다운로드하는 시간이 오래 걸릴 수 있으므로 프로그램에서 데이터 다운로드를 시작하고 데이터가 실제로 필요할 때까지 다른 작업을 계속하도록 설정하는 것이 좋다

이를 구현하려면 기존 let 바인딩 앞에 async 키워드를 추가하면 된다.

```swift
async let result = URLSession.shared.data(...)
```

해당 코드를 만나기 전에 자식 Task를 생성한다. 이 자식 Task는 이를 생성한 Task의 하위 Task이다. 모든 작업이 프로그램의 실행 컨텍스트를 나타내기 때문에, 이 단계에서는 두 개의 화살표가 동시에 나온다.

첫 번째 화살표는 자식 작업을 위한 것이며, 이 자식 작업은 즉시 데이터를 다운로드하기 시작한다. 두 번째 화살표는 부모 작업을 위한 것으로, 부모 작업은 즉시 변수 result를 자리 표시자 값에 바인딩한다.

![](https://github.com/gaeng2y/Valut/blob/main/iOS/WWDC/WWDC%2021/Explore%20structured%20concurrency%20in%20Swift/Resources/Pasted%20image%2020250318120627.png?raw=true)

하지만 `result` 의 실제 값이 필요한 표현식에 도달하면, 부모 작업은 자식 작업의 완료를 기다리게 된다. 

그러면 자식 작업이 result에 대한 자리 표시자를 채우게 된다. 이 예제에서 URLSession 호출은 오류를 던질 수도 있다. 즉, 결과를 기다리면 오류가 발생할 수 있다. 

그래서 이를 처리하기 위해 try를 작성해야 한다. 그리고 result의 값을 다시 읽어도 그 값을 다시 계산하지는 않는다.

![](iOS/WWDC/WWDC%2021/Explore%20structured%20concurrency%20in%20Swift/Resources/Pasted%20image%2020250318120849.png)

그러면 이제 async-let을 사용한 코드로 바꿔보자

```swift
func fetchOneThumbnail(withID id: String) async throws -> UIImage {
    let imageReq = imageRequest(for: id), metadataReq = metadataRequest(for: id)
    let (data, _) = try await URLSession.shared.data(for: imageReq)
    let (metaData, _) = try await URLSession.shared.data(for: metadataReq)
    guard let size = parseSize(from: metadata),
             let image = await UIImage(data: data)?.byPreparingThumbnail(ofSize: size)
    else {
        throw ThumbnailFailedError()
    }
    return Image
}

// async-let
func fetchOneThumbnail(withID id: String) async throws -> UIImage {
    let imageReq = imageRequest(for: id), metadataReq = metadataRequest(for: id)
    async let (data, _) = URLSession.shared.data(for: imageReq)
    async let (metaData, _) = URLSession.shared.data(for: metadataReq)
    guard let size = parseSize(from: try await metadata),
             let image = try await UIImage(data: data)?.byPreparingThumbnail(ofSize: size)
    else {
        throw ThumbnailFailedError()
    }
    return Image
}
```

![](iOS/WWDC/WWDC%2021/Explore%20structured%20concurrency%20in%20Swift/Resources/Pasted%20image%2020250318150239.png)

부모 Task는 자식 Task들이 모두 끝나야 끝낼 수 있다. 이 규칙은 비정상적인 제어 흐름으로 인해 자식 작업이 await 되지 못하더라도 여전히 유효하다. 

예를 들어, 위 코드에서 metadata를 먼저 await를 하고 imageData를 await 한다. 만약, metadata 작업이 에러를 던지며 종료되면, fetchOneThumbnail 메소드는 오류를 던지며 종료돼야 한다. 하지만 두 번째 다운로드 작업을 수행하는 작업은 어떻게 될까? 비정상 종료가 발생하면 Swift는 완료되지 않은 작업을 자동으로 취소하고, 함수가 종료되기 전에 해당 작업이 끝날 때까지 기다린다.

Task를 취소한다고 해서 그 작업이 즉시 중단되는 것은 아니다. 취소는 단지 해당 Task에게 **결과가 더 이상 필요 없다**라고 알리는 것이다. 실제로 작업이 취소되면, 그 작업의 **모든 하위 작업**도 자동으로 취소된다.

![](iOS/WWDC/WWDC%2021/Explore%20structured%20concurrency%20in%20Swift/Resources/Pasted%20image%2020250318151451.png)

그래서 만약 URLSession의 구현이 이미지를 다운로드하기 위해 자체적으로 구조화된 작업들을 생성했다면, 그 작업들은 취소로 표시될 것이다.

함수 `fetchOneThumbnail` 은 자신이 직접 또는 간접적으로 생성한 모든 구조화된 작업들이 완료된 후, 오류를 던지며 종료된다. 이 보장은 구조화된 동시성에서 매우 중요한 원칙이다.

이것은 작업들의 생애 주기를 관리하는 데 도움을 주어, 메모리의 생애 주기를 자동으로 관리하는 ARC와 마찬가지로 실수로 작업이 유출되는 것을 방지한다.
### Cancellation is cooperative
![](iOS/WWDC/WWDC%2021/Explore%20structured%20concurrency%20in%20Swift/Resources/Pasted%20image%2020250319105255.png)
코드는 취소를 명시적으로 확인하고 적절한 방법으로 실행을 종료해야 한다. 현재 작업의 취소 상태는 비동기 함수이든 아니든 모든 함수에서 확인할 수 있다. 이는 특히 긴 시간 동안 실행되는 계산을 포함하는 경우, 취소를 염두에 두고 API를 구현해야 한다는 의미다.

다시 모든 썸네일을 가져오는 코드로 돌아와서

```swift
func fetchThumbnails(for ids: [String]) async throws -> [String: UIImage] {
    var thumbnails: [String: UIImage] = [:]
    for id in ids {
        try Task.checkCancellation()
        thumbnails[id] = try await fetchOneThumbnail(withID: id)
    }
    return thumbnails
}

func fetchThumbnails(for ids: [String]) async throws -> [String: UIImage] {
    var thumbnails: [String: UIImage] = [:]
    for id in ids {
        if Task.isCancelled { break }
        thumbnails[id] = try await fetchOneThumbnail(withID: id)
    }
    return thumbnails
}
```

만약 이 함수가 취소된 작업 내에서 호출되었다면, 불필요한 썸네일을 생성하여 애플리케이션을 지연시키고 싶지 않다. 따라서 각 반복(iteration)의 시작 부분에 [checkCancellation()](https://developer.apple.com/documentation/swift/task/checkcancellation()) 호출을 추가하면 된다. 이 호출은 현재 작업이 취소된 경우에만 오류를 던진다. 또한, 코드에 더 적합하다면 현재 Task의 [isCancelled](https://developer.apple.com/documentation/swift/task/iscancelled-swift.type.property)를 불리언 값으로 얻을 수도 있다.

`if Task.isCancelled { break }` 를 사용한 함수에서는 일부 요청된 썸네일만 포함된 사전(딕셔너리)을 반환하여 부분적인 결과를 반환하고 있다는 점에 유의해야한다. 이렇게 할 때는 API가 부분적인 결과가 반환될 수 있음을 명확하게 설명해야 한다. 그렇지 않으면 작업 취소가 사용자에게 치명적인 오류를 일으킬 수 있다. 왜냐하면 취소 중에도 코드가 완전한 결과를 요구하기 때문이다.
## Group tasks
모든 구조적 동시성의 좋은 속성을 포기하지 않으면서 async-let보다 더 많은 유연성을 제공한다. 앞서 보았듯이, async-let은 사용 가능한 동시성의 양이 고정되어 있을 때 잘 작동한다. 이제 제가 앞서 설명한 두 가지 함수에 대해 살펴보자.

```swift
func fetchThumbnails(for ids: [String]) async throws -> [String: UIImage] {
    var thumbnails: [String: UIImage] = [:]
    for id in ids {
        thumbnails[id] = try await fetchOneThumbnail(withID: id)
    }
    return thumbnails
}

func fetchOneThumbnail(withID id: String) async throws -> UIImage {
    // ...

    async let (data, _) = URLSession.shared.data(for: imageReq)
    async let (metadata, _) = URLSession.shared.data(for: metadataReq)

    // ...
}
```

루프에서 각 썸네일 ID에 대해 `fetchOneThumbnail`을 호출하여 처리하는데, 이 함수는 정확히 두 개의 자식 작업을 생성한다. 만약 이 함수의 본문을 이 루프에 인라인으로 작성하더라도 동시성의 양은 변하지 않는다. async-let은 변수 바인딩처럼 범위가 제한된다. 

즉, 두 개의 자식 작업은 다음 루프 반복이 시작되기 전에 완료되어야 한다. 하지만 만약 이 루프가 모든 썸네일을 동시에 가져오는 작업을 시작하도록 만들고 싶다면 어떻게 될까? 그러면 동시성의 양은 정적으로 알 수 없다. 왜냐하면 그것은 배열에 있는 ID의 수에 따라 달라지기 때문이다.

이럴 때 적절한 도구가 TaskGroup이다. 작업 그룹(task group)은 동적 동시성 양을 제공하도록 설계된 구조적 동시성의 한 형태다. `withThrowingTaskGroup` 함수를 호출하여 작업 그룹을 도입할 수 있다.

```swift
func fetchThumbnails(for ids: [String]) async throws -> [String: UIImage] {
    var thumbnails: [String: UIImage] = [:]
    try await withThrowingTaskGroup(of: Void.self) { group in
        for id in ids {
            group.async {
                // Error: Mutation of captured var 'thumbnails' in concurrently executing code
                thumbnails[id] = try await fetchOneThumbnail(withID: id)
            }
        }
    }
    return thumbnails
}
```

이 함수는 오류를 던질 수 있는 자식 작업을 생성할 수 있는 범위가 제한된 그룹 객체를 제공한다. 그룹에 추가된 작업은 그룹이 정의된 블록의 범위를 벗어나지 못한다. 전체 for-loop을 블록 안에 배치했기 때문에 이제 그룹을 사용하여 동적 수의 작업을 생성할 수 있다. 그룹 내에서 자식 작업을 생성하려면 해당 그룹의 async 메서드를 호출하면 된다.

그룹에 추가되면 자식 작업은 즉시 실행되며, 순서는 상관없다. 그룹 객체가 범위를 벗어나면, 그 안의 모든 작업이 완료될 때까지 암묵적으로 기다리게 된다. 이는 앞서 설명한 작업 트리 규칙의 결과로, 그룹 작업도 구조화되어 있기 때문이다.

이 시점에서 우리는 이미 원하는 동시성을 달성했다. 즉, `fetchOneThumbnail`을 호출할 때마다 하나의 작업이 생성되고, 그 자체로 async-let을 사용하여 두 개의 자식 작업을 생성하게 된다. 이것은 구조적 동시성의 또 다른 좋은 속성이다. async-let을 그룹 작업 내에서 사용할 수 있고, 그룹 작업을 async-let 작업 내에서 생성할 수도 있다. 이렇게 하면 트리 구조 내의 동시성 수준이 자연스럽게 결합된다.

![](iOS/WWDC/WWDC%2021/Explore%20structured%20concurrency%20in%20Swift/Resources/Pasted%20image%2020250319110938.png)

만약 이를 실행하려고 시도한다면, 컴파일러가 데이터 경합 문제를 경고해 줄 것이다. 문제는 각 자식 작업에서 단일 딕셔너리에 썸네일을 삽입하려고 한다는 점이다. 이는 프로그램에서 동시성의 양을 증가시킬 때 흔히 발생하는 실수다. 데이터 경합(data race)이 우연히 발생하게 된다.

![](iOS/WWDC/WWDC%2021/Explore%20structured%20concurrency%20in%20Swift/Resources/Pasted%20image%2020250319111212.png)

새로운 작업을 생성할 때마다, 작업이 수행하는 작업은 `@Sendable` 클로저라는 새로운 클로저 유형 내에서 실행된다. `@Sendable` 클로저의 본문은 그 범위 내에서 변경 가능한 변수들을 캡처하는 것을 제한한다. 이는 해당 변수들이 작업이 실행된 후에 수정될 수 있기 때문이다. 따라서 작업 내에서 캡처하는 값들은 안전하게 공유될 수 있어야 한다. 예를 들어, `Int`나 `String`처럼 값 타입이거나, 여러 스레드에서 접근할 수 있도록 설계된 객체들(예: 액터(actor)와 자체 동기화를 구현한 클래스들)이어야 한다.

우리 예제에서 데이터 경합을 피하려면, 각 자식 작업이 값을 반환하도록 할 수 있다. 이 디자인은 부모 작업이 결과를 처리하는 유일한 책임을 가지게 만든다.

```swift
func fetchThumbnails(for ids: [String]) async throws -> [String: UIImage] {
    var thumbnails: [String: UIImage] = [:]
    try await withThrowingTaskGroup(of: (String, UIImage).self) { group in
        for id in ids {
            group.async {
                return (id, try await fetchOneThumbnail(withID: id))
            }
        }
        // Obtain results from the child tasks, sequentially, in order of completion.
        for try await (id, thumbnail) in group {
            thumbnails[id] = thumbnail
        }
    }
    return thumbnails
}
```

이 디자인은 부모 작업이 결과를 처리하는 유일한 책임을 가지게 만든다. 이 경우, 각 자식 작업이 `String` ID와 썸네일을 위한 `UIImage`를 포함하는 튜플을 반환해야 한다고 지정했다. 그런 다음, 각 자식 작업 내에서 사전에 직접 쓰는 대신, 부모가 처리할 수 있도록 키-값 튜플을 반환하게 했다. 

부모 작업은 새로운 `for-await` 루프를 사용하여 각 자식 작업에서 결과를 순차적으로 처리할 수 있다. `for-await` 루프는 자식 작업이 완료되는 순서대로 결과를 얻는다. 이 루프는 순차적으로 실행되기 때문에, 부모 작업은 각 키-값 쌍을 안전하게 사전에 추가할 수 있다.

작업 그룹(task groups)은 구조적 동시성의 한 형태이지만, 작업 트리 규칙이 그룹 작업과 async-let 작업에 대해 구현되는 방식에는 작은 차이가 있다.

![](iOS/WWDC/WWDC%2021/Explore%20structured%20concurrency%20in%20Swift/Resources/Pasted%20image%2020250319112035.png)

이 그룹의 결과를 반복하는 중에, 오류가 발생한 자식 작업을 만났다고 가정해 보자. 그 오류가 그룹 블록에서 던져지면, 그룹 내의 모든 작업은 암묵적으로 취소되고, 이후에는 모두 기다려야 한다. 이것은 async-let과 똑같이 작동한다.

차이점은 그룹이 블록에서 정상적으로 종료되어 범위를 벗어날 때 발생한다. 이 경우, 취소는 암묵적이지 않다. 이 동작은 작업 그룹을 사용하여 fork-join 패턴을 표현하기 쉽게 만든다. 왜냐하면 작업들이 취소되지 않고 대기만 하기 때문이다. 또한, 블록을 종료하기 전에 그룹의 `cancelAll` 메서드를 사용하여 모든 작업을 수동으로 취소할 수도 있다.
## Unstructured tasks
Swift는 또한 비구조적 작업(unstructured task) API를 제공하는데, 이는 훨씬 더 많은 유연성을 제공하지만, 그만큼 더 많은 수동 관리가 필요하다. 

작업이 명확한 계층 구조에 속하지 않는 상황이 많이 존재하는데, 가장 명백한 예로, 비동기 코드가 아닌 코드에서 비동기 계산을 수행하기 위해 작업을 시작하려는 경우, 부모 작업이 전혀 없을 수 있다. 

또는 작업의 생애 주기가 하나의 범위나 하나의 함수에만 국한되지 않는 경우가 있을 수 있다. 예를 들어, 객체를 활성 상태로 만드는 메서드 호출에 반응하여 작업을 시작하고, 객체를 비활성화하는 다른 메서드 호출에 반응하여 작업을 취소하고 싶을 수 있다.

![](iOS/WWDC/WWDC%2021/Explore%20structured%20concurrency%20in%20Swift/Resources/Pasted%20image%2020250319112600.png)

가령, 우리가 컬렉션 뷰(collection view)가 있고, 아직 컬렉션 뷰 데이터 소스 API를 사용할 수 없다고 가정해 보자. 대신, 방금 작성한 fetchThumbnails 함수를 사용하여 컬렉션 뷰의 항목들이 표시될 때 네트워크에서 썸네일을 가져오고 싶다. 하지만, 델리게이트 메서드는 비동기적이지 않기 때문에, async 함수에 대한 호출을 await할 수 없다.

```swift
@MainActor
class MyDelegate: UICollectionViewDelegate {
    func collectionView(_ view: UICollectionView, willDisplay cell: UICollectionViewCell, forItemAt item: IndexPath) {
        let ids = getThumbnailIDs(for: item)
        // Error: 'await' in a function that does not support concurrency
        let thumbnails = await fetchThumbnails(for: ids)
        display(thumbnails, in: cell)
    }
}
```

이 경우, 비구조적 작업(unstructured task)을 사용하면 작업의 생애 주기를 단일 델리게이트 메서드의 범위에 묶지 않고, **메인 액터(main actor)** 에서 UI 우선 순위를 유지하면서 작업을 실행할 수 있다. Swift에서는 Task를 사용하여 비구조적 작업을 생성할 수 있으며, 이 작업은 부모 작업과 독립적인 생애 주기를 가질 수 있다.

```swift
@MainActor
class MyDelegate: UICollectionViewDelegate {
    func collectionView(_ view: UICollectionView, willDisplay cell: UICollectionViewCell, forItemAt item: IndexPath) {
        let ids = getThumbnailIDs(for: item)
        Task {
            let thumbnails = await fetchThumbnails(for: ids)
            display(thumbnails, in: cell)
        }
    }
}
```

이제 실행 시간에서 어떤 일이 일어나는지 살펴보자.

![](iOS/WWDC/WWDC%2021/Explore%20structured%20concurrency%20in%20Swift/Resources/Pasted%20image%2020250319113155.png)

우리가 작업을 생성하는 시점에, Swift는 해당 작업을 원래의 범위와 같은 액터에서 실행되도록 예약한다. 

이 경우, 그 액터는 메인 액터입니다. 그동안 제어는 즉시 호출자에게 반환된다. 썸네일 작업은 메인 스레드에서 실행되며, 이 작업은 델리게이트 메서드에서 메인 스레드를 즉시 차단하지 않고 실행될 수 있는 여유가 있을 때 수행된다. 이렇게 작업을 생성하는 방식은 구조적 코드와 비구조적 코드 사이의 중간 지점을 제공한다.

![](iOS/WWDC/WWDC%2021/Explore%20structured%20concurrency%20in%20Swift/Resources/Pasted%20image%2020250319113431.png)

직접 생성된 작업은 여전히 실행된 컨텍스트의 액터(있다면)를 상속받는다. 또한 우선 순위나 다른 속성들도 원본 작업과 동일하게 상속된다. 이는 그룹 작업이나 async-let이 작업을 생성할 때와 마찬가지다.

하지만 새로운 작업은 **비구조적(unscoped)** 이다. 즉, 그 작업의 생애 주기는 작업이 생성된 범위에 묶이지 않으며, 원본 작업이 비동기적이지 않아도 새로운 작업을 생성할 수 있다. 

대신, 이 모든 유연성을 제공하는 대신, 비구조적 작업은 구조적 동시성이 자동으로 처리하는 것들을 우리가 직접 관리해야 한다. 예를 들어, **취소(Cancellation)** 와 **오류 처리(Errors)** 는 자동으로 전파되지 않으며, 작업의 결과는 **명시적으로 대기(await)** 하지 않으면 자동으로 기다리지 않는다.

우리는 컬렉션 뷰 아이템이 표시될 때 썸네일을 가져오는 작업을 시작했다. 이제 썸네일이 준비되기 전에 아이템이 뷰에서 스크롤 아웃되면 그 작업을 취소해야 한다. 비구조적 작업을 사용하고 있기 때문에 그 취소는 자동으로 이루어지지 않는다.

```swift
@MainActor
class MyDelegate: UICollectionViewDelegate {
    var thumbnailTasks: [IndexPath: Task<Void, Never>] = [:]
    
    func collectionView(_ view: UICollectionView, willDisplay cell: UICollectionViewCell, forItemAt item: IndexPath) {
        let ids = getThumbnailIDs(for: item)
        thumbnailTasks[item] = Task {
            defer { thumbnailTasks[item] = nil }
            let thumbnails = await fetchThumbnails(for: ids)
            display(thumbnails, in: cell)
        }
    }
    
    func collectionView(_ view: UICollectionView, didEndDisplay cell: UICollectionViewCell, forItemAt item: IndexPath) {
        thumbnailTasks[item]?.cancel()
    }
}
```

우리는 비구조적(unscoped) 작업을 사용하고 있기 때문에, 작업 취소는 자동으로 이루어지지 않는다. 

이제 이를 구현해봅시다. 작업을 생성한 후, 얻은 값을 저장해두겠다. 이 값을 딕셔너리에 행 인덱스를 키로 하여 넣어두면, 나중에 그 작업을 취소할 때 사용할 수 있다. 작업이 완료되면, 딕셔너리에서 해당 작업을 제거해야 한다. 그렇지 않으면 이미 완료된 작업을 취소하려고 시도할 수 있기 때문이다. 

또한, 중요한 점은 이 딕셔너리를 **비동기 작업 안팎에서 안전하게 접근**할 수 있다는 것이다. 컴파일러에서 데이터 레이스를 경고하지 않는다.

우리의 델리게이트 클래스는 메인 액터에 바인딩되어 있으며, 새로 생성된 작업은 그 액터를 상속받기 때문에 둘이 병렬로 실행되지 않는다. 우리는 이 작업 안에서 메인 액터에 바인딩된 클래스의 저장된 속성에 접근할 때 데이터 레이스에 대한 걱정 없이 안전하게 접근할 수 있다. 한편, 나중에 델리게이트가 같은 테이블 행이 화면에서 제거되었다는 정보를 받으면, 값에 대해 cancel 메서드를 호출하여 작업을 취소할 수 있다.

이제 우리는 범위와 독립적으로 실행되는 비구조적 작업을 어떻게 생성할 수 있는지 살펴보았다. 이 작업은 여전히 해당 작업의 원래 컨텍스트에서 특성을 상속받는다. 그러나 때때로 원래의 컨텍스트에서 아무것도 상속받고 싶지 않을 때가 있다.
## Detached tasks
![](iOS/WWDC/WWDC%2021/Explore%20structured%20concurrency%20in%20Swift/Resources/Pasted%20image%2020250319115742.png)

최대한 유연성을 제공하기 위해, Swift는 분리된(detached) 작업을 제공한다. 이름에서 알 수 있듯이, 분리된 작업은 원래의 컨텍스트와 독립적이다. 이 작업은 여전히 비구조적 작업이며, 원래 범위에 의해 수명이 제한되지 않습니다. 그러나 분리된 작업은 원래의 범위에서 아무 것도 상속하지 않는다. 

기본적으로 동일한 액터에 의해 제약을 받지 않으며, 시작된 위치와 동일한 우선순위에서 실행될 필요도 없다. 분리된 작업은 기본적으로 우선순위와 같은 것들에 대해 일반적인 기본값으로 실행되지만, 새로운 작업이 어떻게 실행될지, 어디서 실행될지를 제어할 수 있는 선택적인 매개변수들을 사용하여 시작할 수도 있다.

서버에서 썸네일을 가져온 후, 나중에 다시 가져올 때 네트워크를 다시 호출하지 않도록 로컬 디스크 캐시에 저장하고 싶다고 가정해보자. 이 캐싱 작업은 메인 액터에서 이루어질 필요는 없으며, 썸네일을 모두 가져오는 작업을 취소하더라도 이미 가져온 썸네일을 캐시하는 것은 여전히 유용하다.

```swift
@MainActor
class MyDelegate: UICollectionViewDelegate {
    var thumbnailTasks: [IndexPath: Task<Void, Never>] = [:]
    
    func collectionView(_ view: UICollectionView, willDisplay cell: UICollectionViewCell, forItemAt item: IndexPath) {
        let ids = getThumbnailIDs(for: item)
        thumbnailTasks[item] = Task {
            defer { thumbnailTasks[item] = nil }
            let thumbnails = await fetchThumbnails(for: ids)
            Task.detached(priority: .background) {
                writeToLocalCache(thumbnails)
            }
            display(thumbnails, in: cell)
        }
    }
}
```

우리가 작업을 분리(detach)할 때, 새로운 작업이 어떻게 실행될지 설정하는 데 훨씬 더 많은 유연성을 얻는다. 

캐싱은 주요 UI와 방해되지 않도록 낮은 우선순위에서 이루어져야 하며, 우리는 이 새로운 작업을 분리할 때 백그라운드 우선순위를 지정할 수 있다. 

이제 잠시 미래를 계획해 봅시다. 만약 썸네일에 대해 여러 개의 백그라운드 작업을 수행해야 한다면 어떻게 해야 할까? 더 많은 백그라운드 작업을 분리할 수도 있지만, 분리된 작업 내에서 구조화된 동시성(structured concurrency)을 활용할 수도 있다.

우리는 다양한 종류의 작업을 결합하여 각각의 강점을 활용할 수 있다. 모든 백그라운드 작업에 대해 독립적인 작업을 분리하는 대신, 작업 그룹을 설정하고 각 백그라운드 작업을 그 그룹에 자식 작업으로 생성할 수 있다.

```swift
@MainActor
class MyDelegate: UICollectionViewDelegate {
    var thumbnailTasks: [IndexPath: Task<Void, Never>] = [:]
    
    func collectionView(_ view: UICollectionView, willDisplay cell: UICollectionViewCell, forItemAt item: IndexPath) {
        let ids = getThumbnailIDs(for: item)
        thumbnailTasks[item] = Task {
            defer { thumbnailTasks[item] = nil }
            let thumbnails = await fetchThumbnails(for: ids)
            Task.detached(priority: .background) {
                withTaskGroup(of: Void.self) { g in
                    g.async { writeToLocalCache(thumbnails) }
                    g.async { log(thumbnails) }
                    g.async { ... }
                }
            }
            display(thumbnails, in: cell)
        }
    }
}
```

이렇게 하는 데는 여러 가지 이점이 있다. 만약 나중에 백그라운드 작업을 취소해야 한다면, 작업 그룹을 사용하면 최상위 분리된 작업만 취소해도 모든 자식 작업을 취소할 수 있다. 이 취소는 자동으로 자식 작업에 전파되며, 우리는 핸들의 배열을 추적할 필요가 없다. 또한, 자식 작업은 자동으로 부모 작업의 우선순위를 상속한다. 모든 작업을 백그라운드에서 처리하려면, 분리된 작업만 백그라운드로 설정하면 되고, 이 설정은 자동으로 모든 자식 작업에 전파되므로, 백그라운드 우선순위를 설정하는 것을 잊어버려 UI 작업이 방해받는 상황을 걱정할 필요가 없다.

![](iOS/WWDC/WWDC%2021/Explore%20structured%20concurrency%20in%20Swift/Resources/Pasted%20image%2020250319142847.png)

---

![](iOS/WWDC/WWDC%2021/Explore%20structured%20concurrency%20in%20Swift/Resources/Pasted%20image%2020250319143009.png)
지금까지 우리는 Swift에서 사용 가능한 모든 주요 형태의 작업을 살펴보았다. async-let은 고정된 수의 자식 작업을 변수 바인딩으로 생성할 수 있게 하며, 바인딩이 범위를 벗어나면 자동으로 취소와 오류 전파를 관리한다. 

만약 동적으로 자식 작업의 수가 필요하고 여전히 범위에 제한이 있어야 한다면, 우리는 작업 그룹(task groups)을 사용할 수 있다. 

잘 정의된 범위가 없지만 여전히 원래 작업과 관련된 작업을 분리하고 싶다면, 구조화되지 않은 작업(unstructured tasks)을 생성할 수 있지만, 이 경우 수동으로 관리해야 한다. 

그리고 최대한의 유연성을 위해, 원래 작업과 아무것도 상속하지 않는 수동 관리 작업인 분리된 작업(detached tasks)을 사용할 수 있다.


  