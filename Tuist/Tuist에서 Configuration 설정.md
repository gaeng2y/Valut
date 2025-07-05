Configuration을 통해 개발 환경을 분리하여 설정할 수 있다.
### xcconfigs 디렉토리 구성
```bash
xcconfigs
  ㄴ Shared.xcconfig
  ㄴ DevelopmentTeam.xcconfig
  ㄴ DEV.xcconfig
  ㄴ PROD.xcconfig
```

위와 같이 구성되어있는데 DevelopmentTeam.xcconfig는 Team ID가 들어가기 때문에 `.gitignore`에 추가하기 위해 따로 분리하였고 `Shared.xcconfig`에서 `#include "DevelopmentTeam.xcconfig"` 으로 가져온다.

그리고 `Shared.xcconfig`는 DEV, PROD에서 같은 방식으로 import를 한다.

그리고 `tuist edit` 을 실행하고 Project에 settings를 구성하면 된다.

```swift
.target(
    name: "DDay",
    destinations: .iOS,
    product: .app,
    productName: "DDay",
    bundleId: "com.gaeng2y.DDay",
    deploymentTargets: .multiplatform(iOS: "17.0"),
    infoPlist: .extendingDefault(with: [:]),
    sources: ["Sources/**"],
    resources: ["Resources/**"],
    dependencies: [
        .project(target: "Foundations", path: .relativeToRoot("Foundations"))
    ],
    settings: .settings(
        configurations: [
            .release(name: .release, xcconfig: .relativeToRoot("xcconfigs/PROD.xcconfig"))
        ]
    )
),
.target(
    name: "DDay-DEV",
    destinations: .iOS,
    product: .app,
    productName: "DDay",
    bundleId: "com.gaeng2y.DDay",
    deploymentTargets: .multiplatform(iOS: "17.0"),
    infoPlist: .extendingDefault(with: [:]),
    sources: ["Sources/**"],
    resources: ["Resources/**"],
    dependencies: [
        .project(target: "Foundations", path: .relativeToRoot("Foundations"))
    ],
    settings: .settings(
        configurations: [
            .debug(name: .debug, xcconfig: .relativeToRoot("xcconfigs/DEV.xcconfig"))
        ]
    )
),
```

위와 같이 debug, release를 나눌 수 있다.