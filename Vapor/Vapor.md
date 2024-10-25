Vapor는 Swift를 사용하여 백엔드, 웹 앱 API 및 HTTP 서버를 작성할 수 있는 웹 프레임워크

Apple의 SwiftNIO를 기반으로 구축된 비동기 이벤트 기반 아키텍처

Fluent, Leaf와 같은 추가 패키지를 사용해 더 쉽고 빠르게 백엔드 구성이 가능

## 설치

조건
- Swift 5.6 이상
- 
#### 1. Install Xcode

Xcode 설치 후 `swift --version` 명령어 입력하여 Swift 버전 확인하기
#### 2. Instlal Toolbox

Swift를 설치했다면 [Vapor Toolbox](https://github.com/vapor/toolbox)를 설치하자. `brew install vapor`를 이용해서 설치
#### 3. New Project

```shell
vapor new {프로젝트 이름}
```
#### 4. Folder Strcuture

이 구조는 SPM의 폴더 구조를 기반으로 하므로, 이전에 SPM을 사용해 본 적이 있다면 익숙할 것

```
.
├── Public
├── Sources
│   ├── App
│   │   ├── Controllers
│   │   ├── Migrations
│   │   ├── Models
│   │   ├── configure.swift 
│   │   ├── entrypoint.swift
│   │   └── routes.swift
│       
├── Tests
│   └── AppTests
└── Package.swift
```

**Public**
