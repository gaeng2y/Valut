1.  동시성 프로그래밍이 필요한 이유를 설명하세요. 다음 중 올바른 설명은 무엇인가요?
- CPU의 자원을 최대한 활용하기 위해 (O)
- 사용자의 작업을 순차적으로 처리하기 위해
- 프로그램의 복잡도를 줄이기 위해
- 메모리 사용량을 줄이기 위해
2. 동시성과 병렬성(Parallelism)의 차이를 간단히 설명하고, 각각의 장단점을 비교하세요.
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
global() -> main
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
private let downloadQueue = DispatchQueue(label: "downloadQueue", attributes: .concurrent)
private let semaphore = DispatchSemaphore(value: 3)
private let group = DispatchGroup()

func download(from: urls: [String]) {
  for url in urls {
    group.enter()
  }
}
```