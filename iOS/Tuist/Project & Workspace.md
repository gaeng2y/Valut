### Project
어플리케이션 개발의 기본 단위이다.

Project에는 어플리케이션을 빌드하는데 필요한 코드, 리소스, 설정 파일, 메타데이터 등을 포함한다.

![image](https://docs-assets.developer.apple.com/published/70722bf743fd6069c50523c993f20e24/project-navigator-overview~dark@2x.png)

Project에 연결되는 파일들을 일반적인 파일 시스템과 비슷하게 계층적인 구조로 이루어진다.

하지만 Project는 실제 파일 시스템 구조에 영향을 미치지 않고도, Project 내에서 자체적인 파일 구조를 가질 수 있다.

Project를 통해 내외부 라이브러리 등의 의존성 관계를 유지할 수 있으며 다른 Project를 통해 빌드된 라이브러리, 프레임워크 의존성을 설정할 수 있다.

### Workspace
여러 개의 Project를 하나의 환경에서 관리할 수 있는 공간이다.

