#### [data(for:delegate:)](https://developer.apple.com/documentation/foundation/urlsession/3767352-data)

```swift
func data(
    for request: URLRequest,
    delegate: (any URLSessionTaskDelegate)? = nil
) async throws -> (Data, URLResponse)
```

오래 걸리는 함수라는 것을 명시적으로 선언

함수 결과값을 직접 리턴 가능

함수에서 직접 에러 던지는 방식도 가능 (에러 처리 직관적)

잠시 멈추었다가 다시 실행될 수 있는 중단점을 표기

내부의 코드들은 순차적 실행
## URLSession (async/await)
##### GCD
- 콜백 방식
- resume 처리 필요
- 에러 처리 직접 던지지 못함
#### async/await
- 리턴 방식
- 에러 처리 가능
