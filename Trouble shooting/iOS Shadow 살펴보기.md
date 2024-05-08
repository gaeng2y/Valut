iOS에서 그림자를 적용하기 위해서는 `CALayer` 에 있는 5개의 프로퍼티를 사용해서 구현한다.

![](Pasted%20image%2020240508114925.png)

하나하나 살펴보면

- shadowOpacity: 층의 그림자의 불투명도. 애니메이션 가능.
- shadowRadius: 레이어의 그림자를 렌더링하는 데 사용되는 흐림 반경(점). 애니메이션 가능.
- shadowOffset: 레이어의 그림자의 오프셋(점). 애니메이션 가능.
- shadowColor: 레이어의 그림자의 색깔. 애니메이션 가능.
- shadowPath: 층의 그림자 모양. 애니메이션 가능.

여기서 `shadowPath`를 주지 않고 4개의 프로퍼티만 준다면 런타임 중에 보라색 경고인 `Optimization Oppertunities` 경고를 만나게 될 것이다. 경고를 살펴보면

>[!warning]
>The layer is using dynamic shadows which are expensive to render.
>If possible try setting `shadowPath`, or pre-rendering the shadow into an image and putting it under the layer.

이런 내용의 경고일 것이다. 해석해보면 

```
그 레이어는 렌더링 비용이 많이 드는 동적 그림자를 사용하고 있다.

가능하다면 `shadowPath`를 설정하거나, 그림자를 이미지로 미리 렌더링하여 레이어 아래에 넣으세요.
```

라는 것이다. 그래서 `shadowPath`를 사용하라는 의미이다...

그렇다면 `shadowPath` 값을 넣어주기 위해서는 `CGPath`라는 값을 넣어줘야하는데 이것은 `UIBezierPath` 타입의 `cgPath`라는 프로퍼티를 넣어주면된다.

그러면 `UIBezierPath`를 알아봐야하는데,,,

## UIBezierPath
- documentation: https://developer.apple.com/documentation/uikit/uibezierpath
![](Pasted%20image%2020240508120301.png)

**커스텀 뷰에서 렌더링할 수 있는 직선 및 곡선 세그먼트로 구성된 경로**라고 한다...

어떻게 쓰는지 궁금하다면

애플에서 친히 가이드를 작성해둬서,,, 아래를 참고하면 좋을 것 같다.
https://developer.apple.com/library/archive/documentation/2DDrawing/Conceptual/DrawingPrintingiOS/BezierPaths/BezierPaths.html#//apple_ref/doc/uid/TP40010156-CH11-SW2

