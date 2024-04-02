Session: https://developer.apple.com/videos/play/wwdc2023/10155/

이전까지는 앱을 현지화하려면 Strings와 StringDict 파일을 관리해야 했다.
이를 위해 모든 문자열을 코드와 수동으로 동기화하느라 종종 현지화 컨텐츠를 빠트리는 적이 있을 것이다.

이러한 문제를 Xcode 15의 String Catalog를 통해 해결할 수 있다.

![[Pasted image 20240403000737.png]]

시간이 지나면 이 새 형식이 Xcode의 Strings와 StringsDict파일을 대체할 것이다.

String Catalog로 모든 문자열을 한 곳에서 쉽게 관리할 수 있고 출시 전 모든 콘텐츠가 현지화되었는지 확인할 수 있다.

내가 프로젝트에서 문자열을 넣으면 자동으로 추가된다.

또한 기기별로 다르게 보여야하는 문자열은 xcstrings 파일에서 해당 키에 우클릭을 해서 `Vary by Device`
를 통해 해당 키값으로 기기별로 다른 문자열을 보여줄 수 있다.

## Deeper to String Catalog
### Agenda
- **Extract**
- **Edit**
- **Export**
- **Build**
- **Migrate**
### Localizable Strings
- **Key** (required): 문자열의 고유 식별자로 보통 문자열 그 자체에 해당한다. 이 값은 런타임에 표시할 적절한 값을 조회하는 데 사용된다.
- **Value** (optional, default: Key): 필요에 따라 명시적으로 지정할 수 있지만 그렇지 않으면 기본 현지화 키로 돌아간다.
- **Comment** (recommended): 사용자 인터페이스에서 문자열의 위치와 사용에 관한 맥락을 번역가에게 제공. 번역가가 번역에 모호하게 할 것 같다면 문자열에 주석을 다는 게 좋을 것이다.
- **Table** (optional, default: `"Localizable"`): 현지화 가능한 문자열은 번역이 저장될 파일에 해당하는 문자열 테이블에 속한다. 기본값으로 코드의 문자열은 Localizable 테이블에 배치된다. 다른 방법으로 정리하고 싶다면 사용자화도 가능하다. 
	- 자세히 살펴보면 .strings 파일을 사용하는 기존 애플리케이션은 단일 문자열 테이블에는 지원하는 언어의 `lproj` 폴더 내에 .strings 및 .stringsdict 파일이 포함되어 있다. 여기에 나타나는 모든 파일이 Localizable 문자열 테이블을 구성한다.
	  ![[Pasted image 20240403002216.png]]
	- 반면 String Catalog는 전체 문자열 테이블 단일 파일로 구성한다. 테이블 내에 현지화 가능한 문자열의 번역은 물론이고 추가 메타데이터도 포함한다. 
	- 문자열을 여러 문자열 테이블로 구성하고 싶다면 다중 String Catalog를 생성할 수 있다. 각 카탈로그에는 그 테이블에 속하는 문자열 키와 앱이 지원하는 모든 언어의 번역이 포함된다. 키는 포함된 테이블 내에서는 고유해야 하지만 다른 테이블과 중복은 가능하다. 앱 내 다른 맥락에서도 표시될 수 있기 때문이다.
	  ![[Pasted image 20240403002435.png]]
### Extract
그렇다면 Xcode는 현지화 가능한 문자열을 어디서 찾을까? 현지화 가능한 문자열의 위치는 다양하다. Xcode는 소스 코드와 IB 파일, Info.plist에서도 찾아서 String Catalog에 추가한다.

앱 현지화 경험이 많다면 많은 부분이 익숙할 것이다. 

![[Pasted image 20240403003029.png]]

뷰에서 문자열 리터럴을 지정할 때마다 해당 스트링을 자동으로 현지화 가능한 것으로 간주한다. 현지화가 가능하다고 간주된 문자열은 xcstrings라는 이름의 String Catalog에 추가된다. LocalizedStringKey를 허용하는 모든 매개변수에 작동하게 된다. 

 클라이언트가 현지화하려는 문자열들을 사용하는 커스텀 뷰를 정의할 수도 있다.

![[Pasted image 20240403003237.png]]

여기선 `LocalizedStringResource`를 문자열 유형으로 사용한다. Xcode는 호출 사이트에서 문자열 리터럴이 `LocalizedStringResource`의 인스턴스화에 사용됨을 확인하면 현지화 가능한 문자열임을 인식한다. 

![[Pasted image 20240403003401.png]]

`LocalizedStringResource`는 현지화 가능한 문자열의 표현과 전달에 권장되는 유형이다. 문자열 리터럴을 사용한 초기화를 지원하면서도 주석과 테이블 이름은 물론 문자열 키와 다른 기본값을 제공할 수도 있다.

![[Pasted image 20240403003555.png]]

위 코드는 `String`과 `AttributedString`에 `localized:` 이니셜라이저를 사용해 실행 중에 사용자에게 표시될 문자열을 지정하고 있다. 또한 `Foundation`을 임포트한 모든 곳에서 `LocalizedStringResource`를 직접 사용할 수도 있다.

빌드 세팅에서 `Use Complier to Extract Swift Strings`를 활성화하면 컴파일러가 알아서 String Catalog에 추가해줄 것이다.

String Catalog에서 Swift 코드 이외의 문자열도 추출할 수 있다.

`NSLocalizedString`을 사용한 Obj-C 코드 예제를 봐보자.

![[Pasted image 20240403003855.png]]

`NSLocalizedString` 매크로 내의 모든 문자열 리터럴은 