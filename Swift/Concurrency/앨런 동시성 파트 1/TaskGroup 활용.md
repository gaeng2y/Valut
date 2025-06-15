### 커스텀 구현을 통해 병렬처리 갯수 조절하기
group에 자식 작업을 min을 통해 추가한 후 비동기 반복문에서 병렬 작업이 끝날 때마다 다른 자식 작업 추가하도록

```swift
func fetchImagesWith4Max(urlArray: [String]) async throws -> [UIImage] {
    
    let imageArray = try await withThrowingTaskGroup(of: UIImage.self, returning: [UIImage].self) { group in
        
        let maxConcurrent = min(4, urlArray.count)
        
        for index in 0..<maxConcurrent {   /// 일단은 4개의 작업만 시작
            group.addTask { [index] in
                let image = try await fetchImage(urlString: urlArray[index])
                return image
            }
        }
        
        var images = [UIImage]()
        
        var nextTaskIndex = maxConcurrent   /// 4, 5, 6, ....
        
        /// 비동기 반복문
        for try await downloadedImage in group {    /// 병렬 작업이 끝날때마다 다른 자식 작업 추가 ⭐️⭐️⭐️
            if nextTaskIndex < urlArray.count {
                              /// [nextTaskIndex] 변수 주소 캡처 방지 ==> 값 캡처 사용
                group.addTask { [nextTaskIndex] in
                    let image = try await fetchImage(urlString: urlArray[nextTaskIndex])
                    return image
                }
                
                nextTaskIndex += 1  /// 5, 6, 7...
            }
            
            print("이미지 다운로드 완료: \(downloadedImage)")
            images.append(downloadedImage)
        }
        return images
    }
    return imageArray
}
```