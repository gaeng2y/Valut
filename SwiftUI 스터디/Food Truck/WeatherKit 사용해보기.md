![WeatherKit](https://velog.velcdn.com/images/kimdaehee0824/post/869eb5d2-d4ec-4ea5-8bd5-0f0a672df36c/image.png)

WeatherKit는 Apple이 WWDC2022에서 공개한 AP입니다. Apple이 말하길 정확한 기상 예측에 접근하는 것이 그 어느 때보다도 중요하다고 생각해서 WeatherKit을 만들었다 합니다. 이번 WeatherKit의 특별한 점이라면 REST API를 지원해서 애플 기기가 아닌 다른 디바이스에서도 사용이 가능하다는 점이 있습니다.

**iOS 16부터 가능**
#### 가격
- 월 50만 호출 건수: 멤버십에 포함
- 월 100만 호출 건수: 미화 49.99달러
- 월 200만 호출 건수: 미화 99.99달러
- 월 500만 호출 건수: 미화 249.99달러
- 월 1,000만 호출 건수: 미화 499.99달러
- 월 2,000만 호출 건수: 미화 999.99달러
- 월 5,000만 호출 건수: 미화 2,499.99달러
- 월 1억 호출 건수: 미화 4,999.99달러
- 월 1억 5,000만 호출 건수: 미화 7,499.99달러
- 월 2억 호출 건수: 미화 9,999.99달러

## [Docs](https://developer.apple.com/documentation/weatherkit/)
사용자에게 기상 조건과 알림을 전달합니다.

WeatherKit은 현재 상태, 분별 강수량, 시간별, 일일 예보를 포함한 시기적절한 날씨 정보를 제공합니다. 또한 악천후 경보도 제공합니다.

### WWDC 분석해보기
https://developer.apple.com/videos/play/wwdc2022/10003/

Apple Weather Service 를 사용함

* 고해상도 날씨 모델
* 예측 알고리즘
* 근접 지역의 기상 예뽀를 전세계에 제공

Privacy

개인 정보 보호에 대한 애플의 전념에 일치하도록 WeatherKit은 사용자 정보를 위태롭게 하지 않고 근접 지역 예보를 전달하도록 설계되었다.

위치는 기상 예보를 제공하는 데만 사용되고 어떤 개인 식별 정보도 연관되어 있지 않으며 결코 공유되거나 판매되지 않는다.

WWDC 영상에서는 이용 가능한 데이터 세트를 다룸. 그것은 Apple Weather Service가 뒷받침함.(줄여서 AWS라 그냥 부르겠음,,,)

WeatherKit 프레임워크는 REST Api를 사용해 기상 정보를 요청하는 방법으로 어느 플랫폼에서든 기상 데이터를 얻을 수 있다.

구현 요구 사항들과 모범 사례들을 설명.

## Available datasets
![[Pasted image 20240204233057.png]]
### Weather data
* **Current weather**
* **Minute forecast**
* **Hourly forecast**
* **Daily forecast**
* **Weather alerts**
* **Hisorical weather**