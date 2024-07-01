## MVVM 패턴의 구성 요소
### Model

MVC와 MVP 패턴의 모델과 같다. 모델은 비즈니스 로직을 담당하고 앱 데이터의 저장, 조작, 접근을 담당하는 컴포넌트이다.

### View

UIView와 UIViewController
### ViewModel

View와 Model의 중간에 위치한다.
- ViewModel은 항상 View의 현재 상태를 표시해야 합니다.
- View를 제어하는 ​​Input/Output 정보 처리를 담당
- View가 ViewModel을 갖고 있고 ViewModel이 Model을 갖고 있다.
- 