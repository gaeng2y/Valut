빌드 맥북을 마이그레이션 한 이후부터 맛이 간거 같다,,,

그래서 맥북을 초기화하고 CI/CD 문서화 겸 이 문서를 작성한다,,,

우선 맥북을 초기화 후에 최신 OS 버전으로 업데이트해준다.(Xcode를 최신 버전을 사용하기 위해)
## Xcode 설치
앱스토어 가서 하세요!
## Homebrew 설치
[homebrew](https://brew.sh)를 설치한다.
```shell
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```
homebrew 설치하려면 xcode command line 설치하라고 나올거임,,,,

## Cocoapods 설치
[Cocoapods](https://cocoapods.org)도 설치
```shell
sudo gem install cocoapods
```
여기까지하면 밑작업이 끝났다.

이제 그러면 우리가 해야할 작업은 Jenkins와 Fastlane을 설치해야한다.
## Jenkins 설치
[Jenkins 다운로드 사이트](https://www.jenkins.io/download/)로 가서 OS를 맞게 선택해주면 예를 들면 macOS 

`macOS Installers for Jenkins LTS`라는 제목을 가진 페이지가 나올텐데
```shell
brew install jenkins-lts
```
명령어로 설치한다.

설치가 끝났다면 아래의 명령어를 입력하여 실행하자.
```shell
brew services restart jenkins-lts
```

그리고 기본으로 추천 플러그인 설치하고
- Jira
- Generic Webhook Trigger
- Bitbucket
## Fastlane 설치
