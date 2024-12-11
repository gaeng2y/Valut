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