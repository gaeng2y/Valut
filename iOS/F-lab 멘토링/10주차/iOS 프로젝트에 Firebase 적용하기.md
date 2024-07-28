## Cloud Firestore
1. Firebase 프로젝트를 아직 만들지 않았으면 [Firebase Console](https://console.firebase.google.com/?authuser=0&hl=ko)에서 **프로젝트 추가**를 클릭한 후 화면에 표시된 안내를 따라 Firebase 프로젝트를 만들거나 기존 GCP 프로젝트에 Firebase 서비스를 추가합니다.
    
2. [Firebase Console](https://console.firebase.google.com/project/_/firestore?authuser=0&hl=ko)의 **Cloud Firestore** 섹션으로 이동합니다. 기존 Firebase 프로젝트를 선택하라는 메시지가 표시됩니다. 데이터베이스 만들기 워크플로를 따릅니다.
    
3. Cloud Firestore 보안 규칙의 시작 모드를 선택합니다.

## Trouble Shooting
- trouble
	- 앱은 firebase로 동작하게 만들었는데 위젯이 Firebase를 못써서 문제다.
* solution
	- 그래서 gpt는 appgroup userdefault를 쓰라는데 publisher 붙여놓은데에 쓰면 되려나?
	- 찾아보니 https://hixfield.medium.com/ios-wiget-with-firebase-auth-and-realtime-database-471a42377838 이런게 있었다.

firestore ref:https://github.com/JK0369/ExFirestore/tree/main/ExFirestore