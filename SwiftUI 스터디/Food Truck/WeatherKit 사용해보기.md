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
	* 요청된 지역에서 지금의 상태를 묘사해준다.
	* 그건 하나의 시점을 나타내고 거기에는 자외선 지수, 기온, 풍속 같은 상태가 포함된다.
* **Minute forecast**
	* 가능한 곳의 다음 한 시간 동안 1분마다의 강수 상태가 들어있다.
	* 이 데이터 셋은 사람들이 밖으로 나갈 때 우산을 챙겨 가야 하는지 결정하는 데 유용하다.
* **Hourly forecast**
	* 현재 시간에 시작하는 예보의 모음이다.
	* 최대 240시간에 대한 데이터 제공
	* 각각의 시간에 포함되는 건 습도, 시계, 기아, 이슬점 등의 상태
* **Daily forecast**
	* 10일 예보 집합이 포함
	* 매일 하루 전체의 정보를 포함
* **Weather alerts**
	* 요청된 지역의 발표된 심각한 기상 주의 사항이 포함
	* 사용자들을 안전하게 잘 알게, 준비하게 해주는 중요한 정보가 들어있다.
* **Hisorical weather**
	* 과거의 기상 예보 중 저장된 걸 제공해서 기상 데이터의 동향을 볼 수 있다.
	* 매시간과 일일 요청에 시자고가 종료 일자를 지정하는 방식으로 역사 데이터에 접근할 수 있다.
## Swift 프레임워크를 이용해서 코드 작성해보기
![[Pasted image 20240204233954.png]]

App target에 WeatherKit 추가하기

```swift
private func fetchWeather() async throws {
        let weatherService = WeatherService()
        let seoul = CLLocation(latitude: 37.514575, longitude: 127.0495556)
        do {
            let weather = try await weatherService.weather(for: seoul)
            let temperature = weather.currentWeather.temperature
            let uvIndex = weather.currentWeather.uvIndex
            
            print(temperature, uvIndex)
        } catch {
            print(error)
        }
    }
```

그리고 호출해보니 
`Error Domain=WeatherDaemon.WDSJWTAuthenticatorServiceListener.Errors Code=2 "(null)"`
라는 에러가 나온다...

음 그래서 WWDC를 좀 더 보니
![[Pasted image 20240205201513.png]]

여기서 App Services 에서 WeatherKit을 추가해야 정상적으로 가져온다!

그 후 실행해보니

![[Pasted image 20240205201543.png]]

와 같은 결과 값을 가져올 수 있다.

WWDC 세션을 계속 보니 이런 식으로 attribution을 이용해서 링크나 로고를 가져오네요

![[Pasted image 20240205201935.png]]

그래서 해당 attribution이 뭔지 찾아봤습니다.

#### WeatherAttribution

![[Pasted image 20240205202401.png]]

날씨 데이터 제공자를 지정하는 데 필요한 정보를 정의하는 구조체이다.

WeatherKit을 사용하여 소프트웨어를 게시하려면 저작자 표시가 필요합니다.

라고 하네요. (킹작권은 중요하죠 암요...)

말 그대로 니들 이거 표시해야되니까 우리가 제공해줄게 라는거군요...

### Publishing requirements

* Display active link to attribution
* Display Apple Weather logo
* Provide link to Weather alert attribution
