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
#### Jenkins credentials 추가
exodusAppleDev 구글 계정을 bitbucket에 추가
ssh-keygen을 통해서 private key 추가(BEGN-END 까지 다넣어야됨)
처음에 안될거임 뭐 pod이 어쩌고 저쩌고 해서 pod: comand not found라고 되어서 나옴
그래서 처음 한번은 디렉토리가서 pod install 해줘야됨
## Fastlane 설치

그리고 이제 빌드를 하면 path가 제대로 들어가지않아서 문제가 생긴다.

jenkins관리 -> 시스템 관리-> Global properties에

터미널에서 `echo $PATH`로 나온 값을 복사해서 PATH라는 이름으로 붙여주면된다.

## 빌드 에러
빌드가 자꾸 실패한다.
#### read asset error
에셋을 제대로 못읽는 이슈가 있어서 에셋을 지웠다가 재참조 시켰다.
