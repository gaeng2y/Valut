## XCConfig란?

XCConfig(Xcode Configuration File)는 Xcode 프로젝트의 빌드 설정을 외부화하여 관리할 수 있게 해주는 텍스트 파일입니다[1][2]. 주로 프로젝트나 타겟 파일 외부에 빌드 설정 및 구성을 저장하는 데 사용됩니다.

## XCConfig의 주요 장점

### 1. 설정의 분리와 가독성 향상
- 코드와 빌드 설정을 분리하여 관리
- Xcode UI보다 더 명확하고 일목요연한 설정 관리[3]

### 2. 환경별 설정 관리
- 개발, 테스트, 프로덕션 등 다양한 빌드 환경 설정 용이
- 공통 설정 관리 및 중복 최소화

### 3. 버전 관리 용이성
- 텍스트 파일 형식으로 Git 등 버전 관리 시스템에서 변경 이력 추적 가능
- 팀 내 설정 변경 사항 명확한 공유

### 4. 자동화 및 유지보수성
- CI/CD 파이프라인과 쉬운 통합
- 중앙 집중식 설정 관리
- 특정 설정의 일괄 변경 가능

## XCConfig 파일 생성 방법

1. Xcode에서 **File** > **New** > **File** 선택
2. **Configuration Settings File** 옵션 선택
3. 파일 이름과 위치 지정
4. 프로젝트 설정의 **Info** 탭에서 해당 xcconfig 파일 연결

## 사용 예시

```
// 공통 설정 파일 (App.shared.xcconfig)
CLANG_ENABLE_MODULES = YES
CLANG_ENABLE_OBJC_ARC = YES

// 개발 환경 설정 파일 (App.debug.xcconfig)
#include "App.shared.xcconfig"
CODE_SIGN_IDENTITY = iPhone Developer
```

## 주의사항
- xcconfig 파일은 변수를 순차적으로 쌓아올리는 방식이 아니라 한 번에 정의되는 사전(dictionary)으로 봐야 합니다
- 여러 환경에 대해 단일 타겟으로 다양한 설정 구성 가능

Citations:
[1] https://hongz-developer.tistory.com/244
[2] https://lafortune.tistory.com/22
[3] https://leviblog.tistory.com/44
[4] https://minsone.github.io/ios/mac/xcode-xcconfig
[5] https://github.com/asharov/xcconfig-sample