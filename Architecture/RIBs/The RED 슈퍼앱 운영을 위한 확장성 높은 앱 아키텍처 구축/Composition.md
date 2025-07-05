객체와 데이터타입을 조합하여 더 복잡한 요소를 만드는 방법

### map
**A -(map)-> B**
`map` 메소드를 사용하는 경우를 살펴보면
1. Array
2. Optional
3. Result
4. Publisher

이 네 타입에서 공통점은
1. Generic 타입
	- `Optional<Wrapped>`
	- `Sequence` 프로토콜 내의 `associatedtype Element`
	- `Result<Success, Failure>`
	- `Publsiher` 프로토콜 내의 `associatedtype Output`
2. `transform` 함수를 인자로 받음

### flatMap
