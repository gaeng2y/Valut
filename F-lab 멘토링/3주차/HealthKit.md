사용자의 개인 정보 보호 및 통제를 유지하면서 건강 및 피트니스 데이터에 액세스하고 공유하세요.
## Overview
HealthKit은 iPhone과 Apple Watch의 건강 및 피트니스 데이터를 위한 중앙 저장소를 제공합니다. 사용자의 허가를 받아, 앱은 HealthKit 스토어와 통신하여 이 데이터에 액세스하고 공유합니다.

완전하고 개인화된 건강 및 피트니스 경험을 만드는 것은 다양한 작업을 포함한다:

- 건강 및 피트니스 데이터 수집 및 저장
- 데이터 분석 및 시각화
- 사회적 상호 작용 활성화

HealthKit 앱은 이 경험을 구축하기 위해 협력적인 접근 방식을 취합니다. 당신의 앱은 이러한 모든 기능을 제공할 필요가 없습니다. 대신, 당신은 당신이 가장 관심 있는 작업의 하위 집합에만 집중할 수 있습니다.

예를 들어, 사용자는 개인적인 필요에 따라 조정된 자신이 좋아하는 체중 추적, 스텝 카운팅 및 건강 챌린지 앱을 선택할 수 있습니다. HealthKit 앱은 (사용자 허가 하에) 데이터를 자유롭게 교환하기 때문에, 결합된 제품군은 단일 앱보다 더 맞춤화된 경험을 제공합니다. 예를 들어, 친구 그룹이 일일 스텝 카운팅 챌린지에 참여할 때, 각 사람은 선호하는 하드웨어 장치와 앱을 사용하여 스텝을 추적할 수 있으며, 그룹의 모든 사람들은 챌린지에 동일한 소셜 앱을 사용합니다.

HealthKit은 또한 여러 소스의 데이터를 관리하고 병합하도록 설계되었습니다. 예를 들어, 사용자는 데이터 추가, 데이터 삭제, 앱의 권한 변경을 포함하여 건강 앱에서 모든 데이터를 보고 관리할 수 있습니다. 따라서, 앱은 앱 외부에서 발생하는 경우에도 이러한 변경 사항을 처리해야 합니다.

## HealthKit에서 데이터 읽기
## Overview
HealthKit Store에서 데이터에 액세스하는 세 가지 주요 방법이 있습니다:

- **직접 메소드 호출.** HealthKit 저장소는 특성 데이터에 직접 액세스하는 방법을 제공합니다. 이 방법들은 특성 데이터에만 접근한다. 자세한 내용은 [`HKHealth`](https://developer.apple.com/documentation/healthkit/hkhealthstore) 참조하십시오.
- **질문.** 쿼리는 HealthKit 저장소에서 요청된 데이터의 현재 스냅샷을 반환합니다.
- **장기 실행 쿼리.** 이러한 쿼리는 백그라운드에서 계속 실행되며 시스템이 HealthKit 저장소의 변경 사항을 감지할 때마다 앱을 업데이트합니다.
## Query
쿼리는 HealthKit 저장소에 있는 데이터의 현재 스냅샷을 반환합니다. 모든 쿼리는 익명의 백그라운드 대기열에서 실행됩니다. 쿼리가 완료되면, 백그라운드 큐에서 결과 핸들러를 실행합니다. HealthKit은 다양한 유형의 쿼리를 제공하며, 각각 HealthKit 저장소에서 다양한 유형의 데이터를 반환하도록 설계되었습니다.

- **샘플 쿼리.** 이것은 범용 쿼리입니다. 샘플 쿼리를 사용하여 모든 유형의 샘플 데이터에 액세스하세요. 샘플 쿼리는 결과를 정렬하거나 반환된 총 샘플 수를 제한할 때 특히 유용합니다. 자세한 내용은 [`HKSample`](https://developer.apple.com/documentation/healthkit/hksamplequerydescriptor) 참조하십시오.
- **고정된 객체 쿼리.** 이 쿼리를 사용하여 HealthKit 스토어의 변경 사항을 검색하세요. 앵커 쿼리를 처음 실행할 때, 현재 상점에 있는 모든 일치하는 샘플을 반환합니다. 후속 실행 시, 마지막 실행 이후 추가되거나 삭제된 항목만 반환합니다. 자세한 내용은 [`HKAnchored`](https://developer.apple.com/documentation/healthkit/hkanchoredobjectquerydescriptor) 참조하십시오.
- **통계 쿼리**. 이 쿼리를 사용하여 일치하는 샘플 세트에 대한 통계 계산을 수행하십시오. 통계 쿼리를 사용하여 세트의 합계, 최소값, 최대값 또는 평균값을 계산할 수 있습니다. 자세한 내용은 HKStatisticsQueryDescriptor를 참조하십시오.
- **통계 수집 쿼리.** 이 쿼리를 사용하여 일련의 고정 길이 시간 간격에 걸쳐 여러 통계 쿼리를 수행하십시오. 그래프를 만들 때 이러한 쿼리를 자주 사용합니다. 그들은 매일 소비되는 총 칼로리 수 또는 5분 간격으로 취한 걸음 수와 같은 것을 계산하는 간단한 방법을 제공한다. 자세한 내용은 [`HKStatistics`](https://developer.apple.com/documentation/healthkit/hkstatisticscollectionquerydescriptor) 참조하십시오.
- **상관 관계 쿼리.** 이 쿼리를 사용하여 상관 관계에 포함된 데이터의 복잡한 검색을 수행하십시오. 이러한 쿼리는 상관 관계에 저장된 각 샘플 유형에 대한 개별 조건자를 포함할 수 있습니다. 상관 관계 유형을 일치시키고 싶다면, 대신 샘플 쿼리를 사용하세요. 자세한 내용은 [`HKCorrelation`](https://developer.apple.com/documentation/healthkit/hkcorrelationquery) 참조하십시오.
- **소스 쿼리.** 이 쿼리를 사용하여 HealthKit 스토어에 일치하는 샘플을 저장한 소스(앱 및 장치)를 검색하십시오. 소스 쿼리는 특정 샘플 유형을 저장하는 모든 소스를 나열합니다. 자세한 내용은 [`HKSource`](https://developer.apple.com/documentation/healthkit/hksourcequerydescriptor) 참조하십시오.
- **활동 요약 쿼리.** 이 쿼리를 사용하여 사용자의 활동 요약 정보를 검색하세요. 각 활동 요약 객체에는 주어진 날의 사용자 활동 요약이 포함되어 있습니다. 하루 또는 며칠 동안 쿼리할 수 있습니다. 자세한 내용은 [`HKActivity`](https://developer.apple.com/documentation/healthkit/hkactivitysummaryquerydescriptor) 참조하십시오.
- **문서 쿼리.** 이 쿼리를 사용하여 건강 문서를 검색하세요. 자세한 내용은 [`HKDocument`](https://developer.apple.com/documentation/healthkit/hkdocumentquery) 참조하십시오.
## 오래 지속되는 쿼리
오래 실행되는 쿼리는 익명의 백그라운드 대기열을 계속 실행하고, 시스템이 HealthKit 스토어의 변경 사항을 감지할 때마다 앱을 업데이트합니다. 또한, 관찰자 쿼리는 백그라운드 딜리버리에 등록할 수 있습니다. 이를 통해 HealthKit은 업데이트가 발생할 때마다 백그라운드에서 앱을 깨울 수 있습니다.

HealthKit은 다음과 같은 장기 실행 쿼리를 제공합니다:

- **관찰자 질의.** 이 장기 실행 쿼리는 HealthKit 저장소를 모니터링하고 일치하는 샘플의 변경 사항을 알려줍니다. 시스템이 상점의 변경 사항을 알려주기를 원할 때 관찰자 쿼리를 사용하세요. 백그라운드 딜리버리를 위해 관찰자 쿼리를 등록할 수 있습니다. 자세한 내용은 [`HKObserver`](https://developer.apple.com/documentation/healthkit/hkobserverquery) 참조하십시오.
- **고정된 객체 쿼리.** 수정된 데이터의 현재 스냅샷을 반환하는 것 외에도, 고정된 객체 쿼리는 장기 실행 쿼리 역할을 할 수 있습니다. 활성화되면, 백그라운드에서 계속 실행되며, 상점에서 일치하는 샘플을 추가하거나 제거할 때 업데이트를 제공합니다. 관찰자 쿼리와 달리, 이러한 업데이트에는 변경된 항목 목록이 포함되어 있지만, 백그라운드 전달을 위해 앵커된 개체 쿼리를 등록할 수 없습니다. 더 많은 정보를 원하시면, 보세요.[`HKAnchoredObjectQueryDescriptor`](https://developer.apple.com/documentation/healthkit/hkanchoredobjectquerydescriptor)
- **통계 수집 쿼리.** 통계 컬렉션의 현재 스냅샷을 계산하는 것 외에도, 이 쿼리는 장기 실행 쿼리 역할을 할 수 있습니다. 상점에서 일치하는 샘플을 추가하거나 제거하는 경우, 이 쿼리는 통계 컬렉션을 다시 계산하고 앱을 업데이트합니다. 백그라운드 딜리버리에 대한 통계 컬렉션 쿼리를 등록할 수 없습니다. 자세한 내용은 [`HKStatistics`](https://developer.apple.com/documentation/healthkit/hkstatisticscollectionquerydescriptor) 참조하십시오.
- **활동 요약 쿼리.** 사용자의 활동 요약의 현재 스냅샷을 계산하는 것 외에도, 이 쿼리는 장기 실행 쿼리 역할을 할 수 있습니다. 사용자의 활동 요약 데이터가 변경되면, 이 쿼리는 활동 요약을 다시 계산하고 앱을 업데이트합니다. 백그라운드 전달을 위해 활동 요약 쿼리를 등록할 수 없습니다. 자세한 내용은 [`HKActivity`](https://developer.apple.com/documentation/healthkit/hkactivitysummaryquerydescriptor) 참조하십시오.
## HealthKit에 데이터 저장하기
### Overview
앱은 새로운 샘플을 만들고 HealthKit 스토어에 추가할 수 있습니다. 모든 샘플 유형은 비슷한 절차를 따르지만, 각 유형에는 고유한 변형이 있습니다.

1. 데이터의 유형 식별자를 찾아보세요. 예를 들어, 사용자의 체중을 기록하려면, bodyMass 유형 식별자를 사용합니다. 유형 식별자의 전체 목록은 데이터 유형을 참조하십시오.
2. HKObjectType 클래스의 편의 방법을 사용하여 데이터에 대한 올바른 개체 유형을 만드세요. 예를 들어, 사용자의 가중치를 저장하려면, quantityType(forIdentifier:) 메소드를 사용하여 HKQuantityType 객체를 만들어야 합니다. 편의 방법 목록은 HKObjectType을 참조하십시오.
3. 객체 유형을 사용하여 일치하는 HKSample 서브클래스의 객체를 인스턴스화하세요.
4. `save(_:withCompletion:)` 메소드를 사용하여 객체를 HealthKit 저장소에 저장하세요.

각 HKSample 서브클래스에는 위의 목록에 설명된 단계를 수정하는 샘플 객체를 인스턴스화하는 고유한 편리한 방법이 있습니다.

![](Pasted%20image%2020240610222014.png)

수량 샘플의 경우, HKQuantity 클래스의 인스턴스를 만드세요. 수량의 단위는 유형 식별자의 문서에 설명된 허용 단위와 일치해야 합니다. 예를 들어, 높이 문서에는 길이 단위를 사용한다고 명시되어 있다. 따라서, 당신의 수량은 센티미터, 미터, 피트, 인치 또는 다른 호환 가능한 단위를 사용해야 합니다. 자세한 내용은 HKQuantitySample을 참조하십시오.

![](Pasted%20image%2020240610222050.png)

카테고리 샘플의 경우, 샘플의 값은 유형 식별자의 문서에 설명된 열거형과 일치해야 합니다. 예를 들어, sleepAnalysis 문서는 HKCategoryValueSleepAnalysis 열거형을 사용한다고 명시되어 있다. 따라서, 이 샘플을 만들 때 이 열거형에서 값을 전달해야 합니다. 자세한 내용은 HKCategorySample을 참조하십시오.

![](Pasted%20image%2020240610222120.png)

상관 관계를 위해, 먼저 상관 관계가 포함할 모든 샘플 객체를 만들어야 합니다. 상관 관계의 유형 식별자는 포함할 수 있는 객체의 유형과 수를 모두 설명한다. 포함된 개체를 HealthKit 저장소에 저장하지 마세요. 그것들은 상관관계의 일부로 저장된다. 자세한 내용은 HKCorrelation을 참조하십시오.

> ![info]
> iOS 17 이상에서 저널 앱은 사람들이 운동을 포함한 신체적 성취와 같은 일상적인 경험을 반영하도록 장려한다. 앱이 HealthKit에 데이터를 저장하는 경우, 해당 데이터의 높은 수준의 요약은 저널 앱이나 저널링 제안 프레임워크를 사용하는 다른 앱에서 제안으로 나타날 수 있습니다.

