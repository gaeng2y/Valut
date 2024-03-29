[`Date`](https://developer.apple.com/documentation/foundation/date) 타입은 달력이나 시간대와 무관한 특정 시점이다.
## Overview
`Date` 값은 특정 달력 시스템이나 시간대에 관계없이 단일 시점을 캡슐화한다. `Date`값은 절대 참조 날짜에 상대적인 시간 간격을 나타낸다.

`Date` 구조체는 날짜를 비교하고, 두 날짜 사이의 시간 간격을 계산하고, 다른 날짜에 상대적인 시간 간격에서 새로운 날짜를 만드는 방법을 제공한다. `DateFormatter` 인스턴스와 함께 날짜 값을 사용하여 날짜와 시간의 현지화된 표현을 만들고 달력 인스턴스를 사용하여 달력 산술을 수행하라.

`NSDate` 클래스로 가는 `Date` 브릿지. Objective-C API와 상호 작용하는 코드에서 이것들을 서로 바꿔서 사용할 수 있습니다.