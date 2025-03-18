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

![](iOS/WWDC/WWDC%2021/Explore%20structured%20concurrency%20in%20Swift/Pasted%20image%2020250318120627.png)

하지만 `result` 의 실제 값이 필요한 표현식에 도달하면, 부모 작업은 자식 작업의 완료를 기다리게 된다. 

그러면 자식 작업이 result에 대한 자리 표시자를 채우게 된다. 이 예제에서 URLSession 호출은 오류를 던질 수도 있다. 즉, 결과를 기다리면 오류가 발생할 수 있다. 

그래서 이를 처리하기 위해 try를 작성해야 한다. 그리고 result의 값을 다시 읽어도 그 값을 다시 계산하지는 않는다.

![](iOS/WWDC/WWDC%2021/Explore%20structured%20concurrency%20in%20Swift/Pasted%20image%2020250318120849.png)

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

![](iOS/WWDC/WWDC%2021/Explore%20structured%20concurrency%20in%20Swift/Pasted%20image%2020250318150239.png)

부모 Task는 자식 Task들이 모두 끝나야 끝낼 수 있다. 이 규칙은 비정상적인 제어 흐름으로 인해 자식 작업이 await 되지 못하더라도 여전히 유효하다. 

예를 들어, 위 코드에서 metadata를 먼저 await를 하고 imageData를 await 한다. 만약, metadata 작업이 에러를 던지며 종료되면, fetchOneThumbnail 메소드는 오류를 던지며 종료돼야 한다. 하지만 두 번째 다운로드 작업을 수행하는 작업은 어떻게 될까? 비정상 종료가 발생하면 Swift는 완료되지 않은 작업을 자동으로 취소하고, 함수가 종료되기 전에 해당 작업이 끝날 때까지 기다린다.

Task를 취소한다고 해서 그 작업이 즉시 중단되는 것은 아니다. 취소는 단지 해당 Task에게 **결과가 더 이상 필요 없다**라고 알리는 것이다. 실제로 작업이 취소되면, 그 작업의 **모든 하위 작업**도 자동으로 취소된다.

![](iOS/WWDC/WWDC%2021/Explore%20structured%20concurrency%20in%20Swift/Pasted%20image%2020250318151451.png)

그래서 만약 URLSession의 구현이 이미지를 다운로드하기 위해 자체적으로 구조화된 작업들을 생성했다면, 그 작업들은 취소로 표시될 것이다.

함수 `fetchOneThumbnail` 은 자신이 직접 또는 간접적으로 생성한 모든 구조화된 작업들이 완료된 후, 오류를 던지며 종료된다. 이 보장은 구조화된 동시성에서 매우 중요한 원칙이다.