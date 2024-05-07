Session: https://developer.apple.com/videos/play/wwdc2023/10155/

이전까지는 앱을 현지화하려면 Strings와 StringDict 파일을 관리해야 했다.
이를 위해 모든 문자열을 코드와 수동으로 동기화하느라 종종 현지화 컨텐츠를 빠트리는 적이 있을 것이다.

이러한 문제를 Xcode 15의 String Catalog를 통해 해결할 수 있다.

![image1](https://github.com/gaeng2y/Valut/blob/main/𐂮%20Resources/Pasted%20image%2020240403000737.png?raw=true)

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
	  ![image2](https://github.com/gaeng2y/Valut/blob/main/𐂮%20Resources/Pasted%20image%2020240403002216.png?raw=true)
	- 반면 String Catalog는 전체 문자열 테이블 단일 파일로 구성한다. 테이블 내에 현지화 가능한 문자열의 번역은 물론이고 추가 메타데이터도 포함한다. 
	- 문자열을 여러 문자열 테이블로 구성하고 싶다면 다중 String Catalog를 생성할 수 있다. 각 카탈로그에는 그 테이블에 속하는 문자열 키와 앱이 지원하는 모든 언어의 번역이 포함된다. 키는 포함된 테이블 내에서는 고유해야 하지만 다른 테이블과 중복은 가능하다. 앱 내 다른 맥락에서도 표시될 수 있기 때문이다.
	  ![[Pasted image 20240403002435.png]]
### Extract
그렇다면 Xcode는 현지화 가능한 문자열을 어디서 찾을까? 현지화 가능한 문자열의 위치는 다양하다. Xcode는 소스 코드와 IB 파일, Info.plist에서도 찾아서 String Catalog에 추가한다.

앱 현지화 경험이 많다면 많은 부분이 익숙할 것이다. 

![image4](https://github.com/gaeng2y/Valut/blob/main/𐂮%20Resources/Pasted%20image%2020240403003029.png?raw=true)

뷰에서 문자열 리터럴을 지정할 때마다 해당 스트링을 자동으로 현지화 가능한 것으로 간주한다. 현지화가 가능하다고 간주된 문자열은 xcstrings라는 이름의 String Catalog에 추가된다. LocalizedStringKey를 허용하는 모든 매개변수에 작동하게 된다. 

 클라이언트가 현지화하려는 문자열들을 사용하는 커스텀 뷰를 정의할 수도 있다.

![image5](https://github.com/gaeng2y/Valut/blob/main/𐂮%20Resources/Pasted%20image%2020240403003237.png?raw=true)

여기선 `LocalizedStringResource`를 문자열 유형으로 사용한다. Xcode는 호출 사이트에서 문자열 리터럴이 `LocalizedStringResource`의 인스턴스화에 사용됨을 확인하면 현지화 가능한 문자열임을 인식한다. 

![](Pasted%20image%2020240507121513.png)

`LocalizedStringResource`는 현지화 가능한 문자열의 표현과 전달에 권장되는 유형이다. 문자열 리터럴을 사용한 초기화를 지원하면서도 주석과 테이블 이름은 물론 문자열 키와 다른 기본값을 제공할 수도 있다.

![](Pasted%20image%2020240507121538.png)

위 코드는 `String`과 `AttributedString`에 `localized:` 이니셜라이저를 사용해 실행 중에 사용자에게 표시될 문자열을 지정하고 있다. 또한 `Foundation`을 임포트한 모든 곳에서 `LocalizedStringResource`를 직접 사용할 수도 있다.

빌드 세팅에서 `Use Complier to Extract Swift Strings`를 활성화하면 컴파일러가 알아서 String Catalog에 추가해줄 것이다.

String Catalog에서 Swift 코드 이외의 문자열도 추출할 수 있다.

`NSLocalizedString`을 사용한 Obj-C 코드 예제를 봐보자.

![](Pasted%20image%2020240507121555.png)

`NSLocalizedString` 매크로 내의 모든 문자열 리터럴은 자동으로 현지화 가능한 것으로 간주되며 이를 감지할 수 있는 유사한 매크로를 직접 정의할 수도 있다.

C코드에서도 비슷하게 `CFCopyLocalizedString`을 통해 사용할 수 있다.

![](Pasted%20image%2020240507121642.png)

C 또는 Obj-C에서 직접 만든 현지화 문자열 매크로를 써려면 Localized String Macro Names에 추가해라!

![](Pasted%20image%2020240507121651.png)

코드에서의 방법을 살펴봤으니 이제 IB에서 현지화 가능한 문자열을 살펴보자.

지정된 문자열은 자동으로 현지화 가능하다고 간주된다.

![](Pasted%20image%2020240507121659.png)

인스펙터를 사용해 이런 문자열에 주석을 지정하여 해당 스트링이 표시될 위치 콘텍스트를 번역가에게 제공한다.

![](Pasted%20image%2020240507121708.png)

String Catalog를 스토리보드 또는 xib와 페어링하면 IB에서 현지화 가능한 모든 문자열이 Catalog에 표시된다.

![](Pasted%20image%2020240507121717.png)

이 프로세스는 Info.plist 파일에서도 유사하게 작동한다.

InfoPlist.xcstrings 파일을 프로젝트에 추가하고 원하는 대상에 추가하지만 하면 된다. 빌드할 때마다 Xcode는 현지화 가능한 문자열 Info.plist 키 세트를 Catalog에 추가하며 필요에 따라 수동으로 더 추가할 수 있다.
#### Syncing localizations
Xcode에서 현지화 가능한 문자열을 찾을 수 있는 곳들을 살펴보았으니 이제 이런 문자열이 String Catalog로 어떻게 이동하는지 좀 더 알아보자.

빌드할 때마다 Xcode는 현재 스키마와 플랫폼에서 현지화 가능한 문자열을 찾는다.

![](Pasted%20image%2020240507121726.png)

소스 코드 문자열은 현지화 가능한 문자열의 소스 역할을 하는 반면 String Catalog의 소스 문자열은 동기화가 유지된다.

![](Pasted%20image%2020240507121734.png)

새 문자열이 추가되면 Xcode는 String Catalog에 문자열을 추가한다.

![](Pasted%20image%2020240507121747.png)

이 시점에서 문자열은 번역할 준비가 완료된다. 앞서 설명했듯이 현지화 가능한 문자열의 코드에 기본 소스값이 지정될 수 있다. 그런 경우 Catalog는 코드의 새 값으로 업데이트된다.

![](Pasted%20image%2020240507121758.png)

Xcode는 코드에서 문자열을 제거해도 검색할 수 있다. 문자열이 아직 번역되지 않았다면 Xcode가 문자열을 제거해 주지만 문자열에 번역을 제공한 다음에 문자열을 지웠다면 Xcode는 이를 그대로 두고 STALE이라고 표시한다.

이는 해당 문자열을 이제 코드에서 찾을 수 없음을 나타낸다. 또한 인스펙터를 통해 특정 문자열을 수동으로 관리할 수도 있다.

![](Pasted%20image%2020240507121808.png)

수동 관리 문자열은 빌드 후 현지화를 동기화할 때 Xcode가 업데이트하거나 제거하지 않는다. 코드에서 동적으로 구성된 코드나 데이터베이스에서 유래한 문자열에 유용한 기능이다.
### Edit
String Catalog Editor를 이용해 번역을 쉽게 관리하는 방법을 알아보자. String Catalog는 앱을 현지화할 때 상태와 번역 진행 상황을 추적하기 위한 수준 높은 기능을 지원한다. 
![](Pasted%20image%2020240507121825.png)

코드에서 문자열을 더 이상 찾을 수 없을 때 STALE 표시를 한다고 알려주었지만 그 외에도 알아두면 좋을 현지화 상태가 세 가지 더 있다.

![](Pasted%20image%2020240507121904.png)

- NEW는 문자열이 아직 선택한 언어로 번역되지 않았음을 
- NEEDS REVIEW는 값이 변경될 수도 있는 문자열이므로 번역가의 주의가 필요하다는 의미다.
	-> 사용하려면 우클릭해서 `Mark as Reviewed`를 선택하여 검토할 문자열을 표시할 수도 있다.
- 선택한 언어로 번역이 완료된 문자열에는 녹색 체크 표시가 나타난다.

개발자의 또 다른 현지화 고충은 바로 복수화이다.

예를 들어, 예제에서 추가된 문자열은 `1 %lld Recent visitor`  /  `* %lld Recent visitors` 영어에서는 문법에 따라 대상의 단수, 복수에 맞게 문자열을 변경해야 한다. 하지만 우크라이나 같은 언어는 고려할 경우의 수가 더 많다. 이를 해결하려면 대상의 숫자에 따라 문자열을 변경할 방법이 필요하다.

![](Pasted%20image%2020240507121914.png)

기존에는 많은 언어에서 이 문제를 해결하기 위해 stringdict 파일이 필요했다. 이러한 plist 형식은 제대로 사용하기가 어렵고 문자열 복수화 같은 간단한 작업에도 상당한 어려움을 초래할 수 있다. 

![](Pasted%20image%2020240507121923.png)

하지만 이제 String Catalog Editor에서 문자열 변형 워크플로를 기본 지원한다. 문자열에서 콘텍스트 메뉴를 열면 `Vary by Plural` 을 클릭하면 옵션별로 만들어 질 것이다..

![](Pasted%20image%2020240507121953.png)

두 개의 변수로 복수 변형을 사용해야 하는 더 복잡한 예시를 봐보자. 런타임에서 몇 가지 다른 시나리오가 발생할 것이다. 아래와 같은 경우에 숫자 주변의 문자열을 조금씩 다르게 번역해야 한다.

![](Pasted%20image%2020240507122013.png)

이때 치환이 필요하다. 여기서는 문자열의 두 인수를 모두 복수형으로 변형했다. 앞에 @ 기호가 붙은 치환에 복수형 딕셔너리와 그 값을 지정한다.

![](Pasted%20image%2020240507134408.png)

치환은 일반적으로 문자열에 전달된 인수에 대응하며 종종 문자열 보간을 사용한다. 인스펙터에서 Xcode는 숫자에 사용할 인수의 위치와 전달되는 유형의 C 스타일 서식 지정자에 대한 정보를 표시한다. 아래의 사진에서 yards 치환은 소스 코드에서 사용된 두 번째 문자열 보간이므로 인수 2에 해당한다.

![](Pasted%20image%2020240507134416.png)
## Export
종종 번역가와 협력해 현지화해야 할 때가 있다. xliff 파일로 협업할 수 있다.
## Build
String Catalog는 Xcode 프로젝트 간 상호 작용을 위해 맞춤 설계되었다. 내부 구성은 JSON 파일이므로 소스 제어에서도 쉽게 변경할 수 있어야 한다.

![](Pasted%20image%2020240507134427.png)

빌드 시에는 .strings 및 .stringdict 파일로 컴파일된다. OS에서 수년간 지원해 왔던 파일 형식이기 때문에 최소 배포 대상을 업데이트할 필요 없이 바로 사용할 수 있다.

![](Pasted%20image%2020240507134435.png)

## Migrate
기존 프로젝트에서 String Catalog를 시작하는 방법을 살펴보자.

Xcode를 쓰면 기존 프로젝트에서 String Catalog로 쉽게 마이그레이션 할 수 있대요,,,

