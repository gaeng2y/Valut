Ref: https://blog.devgenius.io/repository-pattern-in-swift-a8eda25b515d

앱을 개발하면서 **데이터**를 저장하거나 가져와야하는 일이 빈번하게 발생한다.

**예**
- iOS라면 CoreData/SwiftData를 이용해서 번들/인메모리에 저장
- 백엔드와 통신을 위해서 Api 송수신

이런 데이터들을 구조체 혹은 클래스로 가지고 ViewModel에서 받아서 사용하거나 ViewController에서 모델로 갖고 있다.

### 문제

그렇다면 이렇게 ViewController와 ViewModel에 서버로부터 받은 데이터를 직접 저장하는 것이 어떤 문제가 있기에 그러는 걸까?

왜 이게 문제가 있고 굳이 **Repository Pattern**이라는 것을 써야한다고 할까?

문제는 유지보수가 힘들다는 점이다.

로컬 DB에 저장하기 위해 CoreData를 사용했던 앱을 Realm으로 갈아탄다고 한다면 NSManagedObject를 상속한 객체들이 많을텐데 이걸 다 고친다면 큰 리소스가 발생할 것이다.

또한 다 바꿀 수 있다고 장담할 수 도 없고

비슷하게 네트워크를 통해 API call로 받아온 JSON Data를 가지고 있는 경우에 앱을 고친다면 어떤 일이 발생할까

아래 그림을 예로 보자.

![uml](https://miro.medium.com/v2/resize:fit:720/format:webp/1*PvzaurL9CeM7oGphavBHUw.png)

Data Access Class를 통해 API/CoreData에 접근하고 있다.

그리고 여기서 CRUD되는 데이터들을 직접 각각의 ViewModel과 ViewController에서 사용하게 되는 예시다.

### Domain Object와 Repository 분리
###### Domain Object
해당 앱에서 정의하는 데이터 구조
도메인 객체는 앱에서 정말 보여질 때 사용하는 객체라고 생각해도 무방

Repository를 사용하여 도메인 객체에 Mapping

위에서 제시한 여러가지 문제상황들이 왔을 때, Repository 부분만 바꿔주면 되고, 이것을 우리가 정의해 놓았던 Domain Objects에 Mapping만 해주면 되는 일

> 이게 바로 Repository Pattern의 핵심

##### 요약 
Repository는 데이터 소스에 접근하기 위해 캡슐화된 구성요소. 데이터에 접근하는 기능을 한군데에 집중시켜서, 유지보수에 용이하고, 데이터 구조에 대한 의존성 낮춤. 이로써 앱이 데이터를 다루는 곳과 이 데이터를 표현하는 곳으로 명확하게 나뉘게 됨

![repository-pattern-uml](https://miro.medium.com/v2/resize:fit:4800/format:webp/1*6M0WnNJpaKKSccN4ICI1MA.png)
