![](iOS/Swift/UIBezierPath/define.png)

path인데 CustomView에서 렌더링할 수 있는 직선과 곡선 세그먼트로 구성된 거

## Overview
`UIBezierPath` 클래스는 경로의 기하학적 구조와 렌더링 속성을 결합하여 모양을 그릴 수 있게 해준다. 경로는 직사각형, 타원, 호와 같은 간단한 모양부터 직선과 곡선 세그먼트를 혼합한 복잡한 다각형까지 정의할 수 있다. 생성, 구성, 렌더링 과정이 별개로 이루어져 코드에서 쉽게 재사용할 수 있다.
### 주요 메서드:
- `move(to:)`: 현재 점을 설정하여 그리기 없이 이동합니다.
- `addLine(to:)`: 현재 점에서 지정한 점까지 직선 세그먼트를 추가합니다.
- `addArc(withCenter:radius:startAngle:endAngle:clockwise:)`: 원호 세그먼트를 추가합니다.
- `close()`: 현재 점에서 하위 경로의 첫 점까지 직선 세그먼트를 추가하여 하위 경로를 닫습니다.
- `stroke()`: 현재 획 색상을 사용하여 경로의 윤곽을 따라 그립니다.
- `fill()`: 현재 채우기 색상을 사용하여 경로로 둘러싸인 영역을 채웁니다.
- `addClip()`: 경로 객체가 나타내는 모양과 그래픽 컨텍스트의 현재 클리핑 영역을 교차시켜 새로운 클리핑 영역을 정의합니다.

이 클래스는 동일한 경로 객체를 여러 번 렌더링하거나, 각 렌더링 사이에 속성을 변경하여 사용할 수 있다.

## BezierPath Basic
`UIBezierPath` 객체는 `CGPathRef` 데이터 타입을 감싸는 래퍼이다. 각 세그먼트는 하나 이상의 점(현재 좌표계 내)과 이러한 점이 해석되는 방식을 정의하는 그리기 명령으로 구성된다.

연결된 선과 곡선 세그먼트의 각 세트는 하위 경로라고 한다. 하위 경로에서 한 선 또는 곡선 세그먼트의 끝은 다음 세그먼트의 시작을 정의한다. 단일 `UIBezierPath` 객체는 하나 이상의 하위 경로를 포함할 수 있으며, 이 하위 경로는 `moveToPoint:` 명령으로 분리되어 사실상 그리기 펜을 들어 새로운 위치로 이동시킨다.

경로 객체를 구성하고 사용하는 과정은 별개다. 경로를 구성하는 첫 번째 과정은 다음 단계를 포함한다:

1. 경로 객체를 생성한다.
2. `lineWidth`나 `lineJoinStyle` 속성과 같은 `UIBezierPath` 객체의 관련 그리기 속성을 설정한다. 이러한 그리기 속성은 전체 경로에 적용된다.
3. `moveToPoint:` 메서드를 사용하여 초기 세그먼트의 시작점을 설정한다.
4. 선 및 곡선 세그먼트를 추가하여 하위 경로를 정의합니다.
5. 선택적으로 `closePath`를 호출하여 하위 경로를 닫는다. 이는 마지막 세그먼트의 끝에서 첫 번째 세그먼트의 시작까지 직선 세그먼트를 그린다.
6. 선택적으로 3, 4, 5 단계를 반복하여 추가 하위 경로를 정의한다.

경로를 구성할 때 경로의 점을 원점(0, 0)을 기준으로 배치해야한다. 이렇게 하면 나중에 경로를 이동하기가 더 쉬워진다. 그리기 동안 경로의 점은 현재 그래픽 컨텍스트의 좌표계에 그대로 적용된다. 경로가 원점을 기준으로 배치된 경우, 현재 그래픽 컨텍스트에 평행 이동 인자가 있는 아핀 변환을 적용하기만 하면 경로를 재배치할 수 있다. 그래픽 컨텍스트를 수정하는 장점은 변환을 저장하고 그래픽 상태를 복원하여 쉽게 실행 취소할 수 있다는 것이다.

경로 객체를 그리려면 `stroke` 및 `fill` 메서드를 사용한다. 이러한 메서드는 현재 그래픽 컨텍스트에서 경로의 선과 곡선 세그먼트를 렌더링한다. 렌더링 과정은 경로 객체의 속성을 사용하여 선과 곡선 세그먼트를 래스터화한다. 래스터화 과정은 경로 객체 자체를 수정하지 않는다. 따라서 동일한 경로 객체를 현재 컨텍스트나 다른 컨텍스트에서 여러 번 렌더링할 수 있다.

# Reference
https://developer.apple.com/documentation/uikit/uibezierpath

https://developer.apple.com/library/archive/documentation/2DDrawing/Conceptual/DrawingPrintingiOS/BezierPaths/BezierPaths.html#//apple_ref/doc/uid/TP40010156-CH11-SW2

https://zeddios.tistory.com/814