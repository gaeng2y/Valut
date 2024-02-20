관리 지속성 및 자동 iCloud 동기화를 추가하려면 모델 코드를 선언적으로 작성하세요.

**데이터모델링 및 관리를 위한 강력한 프레임워크**

외부 파일 형식 없이 **전적으로 코드에 집중**하고 Swift의 새로운 매크로 시스템을 사용하여 **원할한 API 경험**을 제공

## Overview
Core Data의 입증된 지속성 기술과 Swift의 모던 동시성 기능을 결합한 SwiftData를 사용하면 최소한의 코드와 외부 종속성 없이 신속하게 앱에 지속성을 추가할 수 있다.

Swift macros, SwiftData와 같은 현대 언어 기능을 사용하는 것은 빠르고 효율적이고 안전한 코드를 작성할 수 있며 앱의 전체 모델 계층(도는 객체 그래프)을 설명할 수 있다.

프레임워크는 기본 모델 데이터 저장을 처리하고 선택적으로 여러 장치에서 해당 데이터를 동기화한다.

![[Pasted image 20240220200147.png]]

### CoreData의 러닝커브
![[Pasted image 20240220200159.png]]

위의 이미지에서 나온 객체에 대한 이해를 다 해야 사용할 수 있기 때문에 러닝커브가 꽤 가파르다.

## 구성요소
![[Pasted image 20240220200444.png]]
#### @Model
Swift 클래스를 SwiftData에서 관리하는 저장된 모델로 변환하는 매크로

class -> PersistentModel

Entity -> Schema

#### PersistentModel
SwiftData가 Swift 클래스를 저장된 모델로 관리할 수 있게 해주는 인터페이스

```swift
protocol PersistentModel : AnyObject, Observable, Hashable, Identifiable
```