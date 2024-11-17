![roadmap](https://minsone.github.io/image/flickr/23455429071_f9f5356729_z.jpg)
## Reference smantics
참조 의미론입니다. Swift 세상에서 참조 의미론에 들어가는 방법은 클래스를 정의하는 것입니다.

`Temperature`라는 클래스는 살펴보자.
![temp](https://minsone.github.io/image/flickr/22910786413_b407586179_z.jpg)

이제 `home`인스턴스와 `temperature` 인스턴스를 만들어보자.
`thermostat`을 화씨 75도로 설정한다. 이제 저녁에 연어를 굽고자 `oven`에 화씨 425로 설정하고 구웠다.
![image3](https://minsone.github.io/image/flickr/23511786656_85484cd3af_z.jpg)

근데 집이 너무 더워졌어요,,,,,,,,,,,, 의도치 않은 공유 사례에 부딪혔습니다.

![image4](https://minsone.github.io/image/flickr/22909609394_e55c87a5e8_z.jpg)

객체 그래프입니다. temp를 `house.thermostat`, `oven.temperature`에서 서로 사용하기 때문에...

참조 의미론이 있는 세상에서 공유를 막기 위해 복사해보자.

![image5](https://minsone.github.io/image/flickr/22910786683_e2e8715cf5_z.jpg)

`temperature`를 화씨 75도 설정을 다시 합니다. `thermostat`의 `temperature`을 설정할 때 복사합니다. 새로운 객체를 얻습니다.

그것은 `thermostate`의 `temperature`를 가리키는 것입니다. `temp` 변수의 `temperature`를 변경해도 영향 미치지 않습니다.

![image6](https://minsone.github.io/image/flickr/22910786833_07c12923c7_z.jpg)

좋습니다. `oven`의 `temperature`를 설정하고, 또 다른 사본을 만듭니다.

이제는, 기술적으로, 마지막 사본은 필요 없습니다. 추가 사본을 만들기 위해 힙에 메모리 할당하는 시간을 비효율적으로 낭비합니다.

그러나 지난번에 사본을 태워 잃어버렸기 때문에, 안전하게 하고자 합니다. 어서요, 금요일 세션이잖아요, 쉬자고요!

![image7](https://minsone.github.io/image/flickr/23429357502_347aacb54f_z.jpg)

우리는 방어 복사로 이것을 참조하고, 여기서 우리는 필요한 것을 알고 있으므로 복사할 수 없지만, 만약에 지금 또는 언젠가 필요하면, 이 문제를 디버깅하는데 정말로 어렵습니다. 그래서 `.copy()`로 언제든지 `oven`에서 `temperature`를 어딘가에 할당했다는 것을 너무 쉽게 잊습니다. 대신, `oven`에다 올바른 방법으로 구울 것입니다.

![image8](https://minsone.github.io/image/flickr/23455429671_190720faff_z.jpg)

그리고 Cocoa와 Objective-C에서 복사를 참조하는 곳에 관해 이야기합시다. Cocoa에서, 여러분은 `NSCopying` 프로토콜의 개념을 알고 있습니다. 그리고 복사를 의미하는 성문화(codifies), 그리고 많은 데이터 타입과 `NSString`,` NSArray`,` NSURLRequest`, 기타 등등을 알고 있습니다. 이것들은 복사하여 안전하므로 `NSCopying`에 적합합니다.

복사가 필요한 시스템에 있으며, 그리고 매우 매우 타당한 이유로 많은 방어 복사를 봅니다. 그래서 `NSDictionary`는 dictionary에 있는 key를 방어 복사(defensively copy)합니다. 왜 그럴까요? NSDictionary key를 얻어 삽입하고 나서 hash 값이 바뀌는 방법으로 변경된다면, 모든 `NSDictionary`의 내부 변수는 깨지고 버그는 우리에게 책임을 지웁니다. 우린 이런 것을 정말로 원치 않습니다.

우리가 진짜로 하는 것은 `NSDictionary`에서 방어 복사(defensive copy)입니다. 이 시스템에서는 정답이지만, 추가 복사하므로 운 나쁘게도 성능 하락이 있습니다. 물론 모든 방법을 Objective-C에 copy 속성과 언어 레벨로 끌어내리며 Objective-C는 모든 단일 할당에서 이들 문제를 막으려고 노력하기 위해 방어 복사를 수행하고, 모든 방어 복사는 이를 돕지만, 충분치 않습니다. 여전히 많은 버그가 있습니다.

![image9](https://minsone.github.io/image/flickr/23169907919_0ea76e75c1_z.jpg)

그래서 이들 참조 의미론인 문제와 변화가 있습니다. 아마도 여기 그 문제는 참조 의미론이 아니지만, 변화 그 자신일 수 있습니다. 아마 우리는 불변 데이터 구조와 참조 의미론의 세상으로 이동해야 할지도 모릅니다.

![image10](https://minsone.github.io/image/flickr/23511787176_cb427e9624_z.jpg)

함수형 프로그래밍 커뮤니티에 누군가와 이야기한다면, ‘우리는 수십 년간 이것을 해왔다’라고 말할 것입니다. 그리고 이것은 향상하게 시킵니다. 그래서 부작용이 없는 세상에서 의도치 않은 부작용은 있을 수 없으며, 불변 참조 의미론은 시스템에서 일관적인 방법으로 동작합니다.

`temperature` 예제를 실행해서 의도하지 않은 결과는 없었습니다. 불변성은 몇 가지 단점이 있습니다. 이는 몇 가지 어색한 인터페이스로 이끌 수 있고, 일부는 단지 언어 작업 방법이고, 일부는 변경할 수 있는 세상에서 살고 있는데 익숙한 것이고, 그리고 우리는 상태와 변경 상태에 관해 생각하고, 순수한 불변 세상에서의 생각은 때론 우리를 조금 이상하게 할 수 있습니다.

또한, 머신 모델에 이르기까지 효율적인 매칭 문제가 있습니다. 언젠가는 여러분은 기계어를 파야 하고, 안정적인 register와 cache, memory 그리고 storage가 있는 CPU에서 실행합니다.

불변의 알고리즘에서 효율적인 수준(level efficiently)까지 대응하는 것이 항상 쉽지 않습니다. 몇 가지 이야기합시다.

![image11](https://minsone.github.io/image/flickr/22910787073_88f46d8f25_z.jpg)

`Temperature` 클래스의 `celsius` 프로퍼티를 불변인 값으로 만들고 `fahrenheit` 프로퍼티의 setter를 없애자.
아무리 애써봐도 `Temperature` 상태를 변경할 수 없습니다. 편의(convenience)를 위한 여러 initializer를 추가합니다.