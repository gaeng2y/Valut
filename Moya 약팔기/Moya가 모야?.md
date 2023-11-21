Alamofire를 사용하여 URLSession에 대한 액세스와 별로 신경 쓰지 않는 모든 불쾌한 세부 정보를 추상화할 수 있습니다. 

그러나 많은 똑똑한 개발자들처럼 임시 네트워크 추상화 계층을 작성합니다. 아마도 "APIManager" 또는 "NetworkModel"이라고 불리며 항상 눈물로 끝납니다.

![moya1](https://github.com/Moya/Moya/raw/master/web/diagram.png)

Moya의 몇 가지 놀라운 기능:

* 올바른 API 엔드포인트 액세스를 컴파일 타임에 확인합니다.
* 연관된 열거형 값을 사용하여 다양한 끝점의 명확한 사용법을 정의할 수 있습니다.
- 테스트 스텁을 일류 시민으로 취급하므로 단위 테스트가 매우 쉽습니다.

### [Example](https://github.com/Moya/Moya/tree/master/docs/Examples)