Task는 중단 지점이 없는 긴 실행 작업의 중간에서 자발적으로 자신을 중단시켜 다른 작업이 잠시 실행될 수 있도록 한 후에 이 작업으로 실행이 돌아오게 할 수 있음
이 Task가 시스템 내에서 가장 높은 우선수위 작업이라면, 실행자는 동일한 작업의 실행을 즉시 재개함

```swift
func getImagesArray() async -> [UIImage?] {
    let image1 = await getImage()
    await Task.yield()                    // 예를 들어서..  오래걸리는 작업이 있다면.. 중단 지점을 만들어 줄 수도 있음
    let image2 = await getImage()
    let image3 = await getImage()
    
    return [image1, image2, image3]
}
```

(참고) 후에 액터(actor)와 연관지어 자세히 다룰 예정 

원자적 연산(atomic operation): 시작부터 끝까지 중단 없이 실행되어야만 하는 작업(함수)
1) 디스크에 파일을 쓰거나
2) 메모리 캐시의 딕셔너리를 변경하거나
3) 데이터베이스에 쓰는 것은 중단시키면 안되는 작업들의 예임

이런 원자적으로(atomically) 실행되어야 하는 작업은 await와 같은 방식으로(중간에 잠시 중단 될 수 있는 형태로) 설계되면 안됨 
비동기(async)함수로 설계하거나, Task.yield()를 호출하면 안됨(쓰레드가 한번에 해당 작업의 실행부터 종료까지 쭈욱 이어서 완료 시켜야 함)