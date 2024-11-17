https://voracious-pigment-aaf.notion.site/3613bac852aa4528b3b1c916078add1e#1cf1cb2c38ac44458ae0b0b1a094117e

해당 문서를 참고

문제가 생겼음 처음에는 프레임워크로 개발하였음.

근데? 프레임워크에서 UIColor로 에셋에 접근하면 Color를 잘 가져옴.

하지만 asset에 직접 접근하려면 문제가 생김.

https://zeddios.tistory.com/1018

그래서 일단 패키지로 바꿈

왜냐면 스태틱, 다이나믹 신경쓰기 싫어서,,,

그래서
https://minsone.github.io/programming/bundling-resources-with-a-swift-package

해당 내용 참고하면서 진행 중

Package.swift
파일에 taget에 `resources: [.process("Resources")` 추가
resource에는 .process, .copy가 있는데 process는 뎁스를 다 없애버리고, copy는 뎁스 그대로 카피해버림.

ChatGPT 말로는 직접 Asset.car에 접근하지말래서 그냥 UIColor로 선회,,,

이제 Color를 어떻게 적용할 지? Foudnation에 직접 컬러가 들어갈지 혹은 앱 || 컴포넌트에서 컬러를 갖고있어서 전달할지?

### Lottie 의존성 추가하기
Lottie에 대한 의존성을 DesignSystemFoundation에 추가하려했다.

그래서 우선..
```swift
dependencies: [
        .package(url: "https://github.com/airbnb/lottie-spm.git", .upToNextMajor(from: "4.4.0"))
    ],
```

를 추가해줬고

이후에 taget에 추가하려했다.
```swift
dependencies: ["Lottie"]
```
이렇게 추가하니... 컴파일 에러가 떴다.
`product 'Lottie' required by package 'designsystemfoundation' target 'DesignSystemFoundation' not found.`
음... 대충 DesignSystemFoundation에 Lottie 패키지가 없다. 라고 하는데

그래서 우리의 채찍피티에서 물어봤다.

`.product(name: "Lottie", package: "lottie-ios")`

이렇게 넣어보셈~

해서 넣어보니까 오우 잘됨...

## IdolUISystem
App'sUISystem으로 DesignSystemFoundation에 대해 의존성을 가지게 하려고 한다.

1. 디렉토리에서 `swift package init`을 실행하면 패키지 설정 완료
2. 프로젝트 새로 만들어서 프레임워크 추가하기
3. 폴더 정리

## 의존성 구조 만들기
DesignSystemFoundation

IdolUISystem

idolapp

이러한 구조를 가질텐데

idolapp 에서 DesignSystemFoundation에서 선언한 ColorAsset을 IdolUISystem만 import해서는 알 수 없다.

> ![note] 그래서 ChatGPT에 물어보니 
Swift 컴파일러가 `idolMain` 타입 프로퍼티가 어떤 타입에 속하는지 알 수 없을 때 발생합니다. 이는 보통 해당 타입이 현재 스코프 내에 없거나, 해당 타입에 대한 정보가 명확하지 않을 때 나타나는 문제입니다.
여기서 `ColorAsset` 구조체는 A 프레임워크에 선언되어 있고, `idolMain` 타입 프로퍼티 확장은 B 프레임워크에서 이루어졌습니다. B 프레임워크를 C 프로젝트에 import 했음에도 불구하고 `ColorAsset`에 대한 접근이나 확장된 프로퍼티를 찾을 수 없다는 것은, `ColorAsset` 타입이 C 프로젝트의 스코프 내에서 명확히 인식되지 않았음을 의미합니다.
B 프레임워크가 A 프레임워크에 의존하고 있다면, B 프레임워크 내에서 `ColorAsset` 타입을 사용하려면, A 프레임워크도 C 프로젝트에서 직접 접근 가능해야 합니다. 그러나 A 프레임워크를 직접 import하지 않고 B 프레임워크만을 통해 `ColorAsset` 타입에 접근하고 싶다면, B 프레임워크에서 `ColorAsset` 타입을 재익스포트하는 방법을 사용할 수 있습니다.
Swift 5.5 이상부터는 `@_exported import` 키워드를 사용하여 모듈 재익스포트를 할 수 있습니다. 이 방법을 사용하면, B 프레임워크 내부에서 A 프레임워크를 재익스포트하면, B 프레임워크를 import하는 C 프로젝트에서도 A 프레임워크의 공개 인터페이스에 접근할 수 있게 됩니다.
`@_exported import DesignSystemFoundation`
그러나 주의할 점은, `@_exported import`는 공식적으로 Swift 언어 사양에 포함된 기능이 아니라 내부적으로 사용되는 기능이므로, 향후 버전의 Swift에서 변경되거나 제거될 가능성이 있습니다. 따라서 이 방법을 사용할 때는 해당 리스크를 고려해야 합니다.
만약 `@_exported import` 사용이 적절하지 않다고 판단되거나, Swift 버전이 낮아 사용할 수 없는 경우, C 프로젝트에서 A 프레임워크를 직접 import하는 것이 가장 확실한 해결 방법입니다. 이렇게 하면 `ColorAsset` 타입과 그에 대한 모든 확장에 직접 접근할 수 있게 됩니다.

라고 한다...

근데 막상해보니 안됨

그래서 그냥

DSF(DesignSystemFoundation) -> UISystem(Pacakge.swift에 DSF에 대한 의존성 추가) -> App

이렇게 진행 될 것 같다.

그렇다면 실제로 App에서는 DSF, UISystem 두 개 모두 의존하게 된다.

근데 지금 실제로 사용해보니 UISystem에 static으로 선언한 색들을 제대로 못가져온다.
