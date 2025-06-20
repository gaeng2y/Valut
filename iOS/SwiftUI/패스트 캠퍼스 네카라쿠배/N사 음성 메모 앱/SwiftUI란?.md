### SwiftUI의 특징
1. Swift 언어로 모든 플랫폼에서 앱에 대한 UI와 동작을 선언해주는 프레임워크
2. 상태 중심 프레임워크
### 장점과 단점
**장점**
- 선언적 구문
- 간결한 코드로 가독성 향상 및 유지보수 용이
- 손쉬운 View 조합을 통한 구현
- Modifier Chaining을 통한 편리한 구현
- Preview의 강력한 기능
**단점**
- 아직 UIKit을 전부 대체하지 못함
- 낮은 버전에서 사용 시 버그가 많음
- 아직도 매 버전마다 변경되는 부분들이 많음
### SwiftUI의 View Layout 결정 원리
```swift
struct ContentView: View {
    var body: some View {
        Text("Hello World!")
            .padding(20)
            .background(.red)
    }
}
```
![](iOS/SwiftUI/패스트%20캠퍼스%20네카라쿠배/N사%20음성%20메모%20앱/Pasted%20image%2020250225203111.png)
**부모의 크기 제안 (상단에서 하단으로):**
- 부모 뷰(이 경우 암시적인 루트 뷰)가 ContentView에 제안 크기를 제공합니다.
**크기 계산 (하단에서 상단으로):**
- Text 뷰는 문자열 내용과 현재 폰트에 기반하여 이상적인 크기를 계산합니다.
- .padding(20) 수정자는 텍스트 뷰를 감싸는 새로운 뷰를 생성하고 모든 측면에 20포인트의 공간을 추가합니다.
- .background(.red) 수정자는 패딩된 텍스트 뷰의 크기를 가진 또 다른 래퍼 뷰를 생성합니다.
**최종 위치 결정 (상단에서 하단으로):**
- 부모가 자신의 좌표 공간 내에 ContentView를 배치합니다.
- ContentView는 자신의 경계 내에 내용(수정된 텍스트)을 배치합니다.
- 배경은 패딩된 텍스트의 전체 영역을 채웁니다.
- 패딩은 텍스트와 빨간색 배경의 가장자리 사이에 20포인트의 공간을 보장합니다.