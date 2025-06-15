1.  동시성 프로그래밍이 필요한 이유를 설명하세요. 다음 중 올바른 설명은 무엇인가요?
- CPU의 자원을 최대한 활용하기 위해 (O)
- 사용자의 작업을 순차적으로 처리하기 위해
- 프로그램의 복잡도를 줄이기 위해
- 메모리 사용량을 줄이기 위해

2. 동시성과 병렬성(Parallelism)의 차이를 간단히 설명하고, 각각의 장단점을 비교하세요
- 동시성은 작업들이 빠르게 전환하면서 실행되어 동시에 실행되는 것처럼 보이는 것
- 병렬성은 물리적인 시간에 작업을 동시에 수행하는 것.(동시성과 차이는 실제로 동시에 수행하는 것)
- 동시성은 주로 단일 CPU에서의 작업 처리에 초점을 맞추는 반면, 병렬성은 여러 CPU나 코어를 활용하여 성능을 향상시키는 데 초점

3. 다음 상황에서 동시성 프로그래밍이 필요한 이유를 설명하고, 개선 방법을 코드로 작성하세요.
- 대용량 이미지를 다운로드 한다고하면 동시성으로 처리하지 않는 다면 실제로 코어에서 이미지 다운로드가 끝날때까지 다른 처리를 하지 못한다.
- 네트워크 요청에 대해 동기적으로 처리하려면 각 요청이 완료될 때까지 기다려야 하므로 시간이 오래걸리기 때문에 동시성을 이용해서 처리해야한다.
```swift
func downloadImage(with urlString: String) async throws {
  guard let url = URL(string: urlString) else {
    throw URLError(.badURL)
  }
  let (data, _) = try await URLSession.shared.data(from: url)
  let image = UIImage(data: data)
}

func fetchData(from urlString: String) async throws -> String {
    guard let url = URL(string: urlString) else {
        throw URLError(.badURL)
    }

    let (data, _) = try await URLSession.shared.data(from: url)
    let responseString = String(data: data, encoding: .utf8) ?? "No Data"
    print("데이터 가져오기 완료: \(urlString)")
    return responseString
}
// for문을 통해 처리
```

4. 다음 코드의 출력 순서를 예측하고, 그 이유를 설명하세요.
```swift
DispatchQueue.global(qos: .userInitiated).async {
    print("Task 1")
    
    DispatchQueue.main.async {
        print("Task 2")
    }
}

print("Task 3")

// Task 3
// Task 1
// Task 2
```

5. 다음 코드의 문제점을 찾고, 올바른 수정 방법을 제시하세요.
```swift
func updateUserInterface() {
    DispatchQueue.global().async {
        // UI 업데이트 코드
        label.text = "Updated"
    }
}

// label이 UILabel이라면 메인 스레드에서 접근해야한다.
// global() -> main
```

6. 아래 코드에서 notify 블록이 실행되는 시점과 조건을 설명하세요.
```swift
let group = DispatchGroup()

group.enter()
DispatchQueue.global().async {
    // 네트워크 작업 시뮬레이션
    sleep(2)
    print("Network Task 1 completed")
    group.leave()
}

group.enter()
DispatchQueue.global().async {
    // 이미지 다운로드 시뮬레이션
    sleep(3)
    print("Image Download completed")
    group.leave()
}

group.notify(queue: .main) {
    print("All tasks completed")
}

// group.enter()가 두 번 호출 되었기 때문에 각 global 큐 비동기 처리에서 group.leave()가 두 번 모두 호출 될 떄 notify 블록이 실행될 것이다.
```

7. 다음 코드의 실행 결과와 cancel() 메서드의 역할을 설명하세요.
```swift
let workItem = DispatchWorkItem {
    for i in 1...5 {
        print("Working: \(i)")
        sleep(1)
    }
}

DispatchQueue.global().async(execute: workItem)

// 2초 후 작업 취소
DispatchQueue.main.asyncAfter(deadline: .now() + 2) {
    workItem.cancel()
}

// Working: 1
// Working: 2
// Working: 3
// Working: 4
// Working: 5
// 로 나오며 cancel()메소드가 호출되면 workItem의 isCancelled가 true로 됨 그래서 for문 내에서 workItem.isCancelled를 체크하여서 break 해야함.
```

8. 다음 시나리오를 해결하는 코드를 작성하세요
- 최대 3개의 동시 다운로드만 허용
- 모든 다운로드가 완료되면 UI 업데이트
- 다운로드 실패 시 에러 처리
```swift
let urls = [
    "https://github.com/IQAndreas/sample-images/blob/gh-pages/100-100-color/00.jpg?raw=true",
    "https://github.com/IQAndreas/sample-images/blob/gh-pages/100-100-color/01.jpg?raw=true",
    "https://github.com/IQAndreas/sample-images/blob/gh-pages/100-100-color/02.jpg?raw=true"
]
private let downloadQueue = DispatchQueue(label: "downloadQueue", attributes: .concurrent)
private let semaphore = DispatchSemaphore(value: 3)
private let group = DispatchGroup()

func download() {
    for urlString in urls {
        group.enter()
        semaphore.wait()
        downloadQueue.async {
            let url = URL(string: urlString)!
            let request = URLRequest(url: url)
            URLSession.shared.dataTask(with: request) { data, response, error in
                defer {
                    semaphore.signal()
                    group.leave()
                }
                if let error {
                    return
                }
                let image = UIImage(data: data!)
                print(image!.size)
            }
        }
    }
}
```

9. 다음 클로저에서 발생할 수 있는 메모리 순환 참조를 해결하는 코드를 작성하세요.
```swift
class NetworkManager {
    func fetchData(completion: @escaping () -> Void) {
        DispatchQueue.global().async {
            // 네트워크 작업
            self.processData()
            completion()
        }
    }
}

// async 클로저 내에 [weak self] 혹은 [unowned self]로 레퍼런스 카운트 증가하지 않도록 적용
```

10. Race condition의 주요 원인 세 가지를 나열하고, 각각의 문제를 해결할 수 있는 동기화 메커니즘을 제시하세요.
- 공유 자원에 대한 동시 접근
- 작업 순서의 부재
- 자원 접근에 대한 제한 메커니즘 부재
- 동기화 도구: 세마포, 뮤텍스, 락

11. 다음 코드에서 발생할 수 있는 문제를 설명하고, 이를 해결하기 위한 방법을 제안하세요
    ```swift
var sharedCounter = 0
let queue = DispatchQueue.global(qos: .userInitiated)

for _ in 1...10 {
    queue.async {
        sharedCounter += 1
    }
}

print("Final Counter: \(sharedCounter)")
```
- 비동기로 queue에서 sharedCounter에 접근하므로 경쟁 상태 발생 가능. async 를 sync로 바꾸면 경쟁 상태 해결
  
12. 다음 코드에서 Deadlock이 발생하는 이유를 설명하고, 이를 방지하기 위한 방법을 제안하세요
    ```swift
let lock1 = NSLock()
let lock2 = NSLock()

DispatchQueue.global().async {
    lock1.lock()
    sleep(1)
    lock2.lock()
    print("Task 1 completed")
    lock2.unlock()
    lock1.unlock()
}

DispatchQueue.global().async {
    lock2.lock()
    sleep(1)
    lock1.lock()
    print("Task 2 completed")
    lock1.unlock()
    lock2.unlock()
}

// task1에선 lock1을 획득하고 lock2을 대기, task2에선 lock2를 획득하고 lock1을 대기하기 때문에 서로 해제하기를 기다리므로 데드락 발생
```

13. 데드락 발생 조건 네 가지
- 상호 배제: 한 번에 하나의 스레드만 자원을 사용 가능
- 점유 및 대기: 자원을 점유하고 있는 스레드가 스스로 자원을 해제할 때까지 기다림
- 비선점: 자원을 점유하고 있는 스레드가 스스로 자원을 해제할 때까지 기다림
- 순환 대기: 스레드 간에 순환적인 자원 대기 상태가 발생

14. Deadlock을 방지하기 위한 일반적인 전략 두 가지를 제시하고, 각각의 장단점을 설명하세요
- 자원 획득 순서 고정
  - 모든 스레드가 자원을 획득할 때 고정된 순서로 자원을 요청하도록 강제
  - 장점: 간단하고 직관적
  - 단점: 확장성이 낮음
- 타임아웃 설정
  - 스레드가 자원을 요청할 때 시간 제한 설정
  - 장점: 확실히 방지
  - 단점: 복잡한 구현

15. Priority Inversion이 발생할 수 있는 상황을 설명하세요
```swift
let highPriorityQueue = DispatchQueue.global(qos: .userInteractive)
let lowPriorityQueue = DispatchQueue.global(qos: .background)
let semaphore = DispatchSemaphore(value: 1)

lowPriorityQueue.async {
    semaphore.wait()
    sleep(3)
    print("Low priority task completed")
    semaphore.signal()
}

highPriorityQueue.async {
    semaphore.wait()
    print("High priority task completed")
    semaphore.signal()
}
```
- lowPriorityQueue에서 세마포를 얻고 3초 지연
- 그 후 hightPriorityQueue에서 세마포에 접근하려하지만 이미 접근되어있어 우선순위가 높지만 먼저 실행할 수 없는 상황이 발생할 수 있음

16. iOS에서 GCD(Grand Central Dispatch)를 사용할 때 Priority Inversion 문제가 발생하지 않도록 하기 위해 개발자가 따라야 할 권장 사항은 무엇인가요?
- 우선 순위가 높은 작업이 낮은 작업에 의해 지연되지 않도록 설계해야 한다. (적절한 타임아웃, 우선순위 상속 등을 이용해서)

17. 다음 코드에서 Dispatch Barrier를 활용하여 위 코드를 수정하고, 수정된 코드가 thread-safe한 이유를 설명하세요.

```swift
class SharedResource {
    private var data: [String] = []
    private let queue = DispatchQueue(label: "com.example.concurrentQueue", attributes: .concurrent)

    func addItem(_ item: String) {
        queue.async {
            self.data.append(item)
        }
    }

    func getItems() -> [String] {
        return queue.sync {
            return self.data
        }
    }
}

let resource = SharedResource()
DispatchQueue.global().async {
    resource.addItem("Item 1")
}
DispatchQueue.global().async {
    print(resource.getItems())
}

// addItem 메소드 내의 queue에서 flag에 .barrier 추가
// 사용하게 되면 thread-safe한 이유
// 배리어를 이용하면 data 배열에 아이템을 추가할 때 작업이 다 끝나야 getItems()에서 data에 접근이 가능해지기 때문에 thread-safe 해진다.
```

18. 아래 코드는 AccessToken과 RefreshToken을 관리하는 클래스입니다. 이 코드를 thread-safe하게 수정하세요.

```swift
class TokenManager {
    private var accessToken: String = ""
    private var refreshToken: String = ""
    private let queue = DispatchQueue(label: "token", attributes: .concurrent)

    func updateAccessToken(_ token: String) {
            queue.async(flags: .barrier) {
            accessToken = token
          }
    }

    func updateRefreshToken(_ token: String) {
              queue.async(flags: .barrier) {
        refreshToken = token
        }
    }

    func getAccessToken() -> String {
              queue.sync {
          return accessToken
              }
    }

    func getRefreshToken() -> String {
        queue.sync {
            return refreshToken
        }
        
    }
}

let manager = TokenManager()

DispatchQueue.global().async() {
    manager.updateAccessToken("newAccessToken")
}

DispatchQueue.global().async {
    print(manager.getAccessToken())
}
```
