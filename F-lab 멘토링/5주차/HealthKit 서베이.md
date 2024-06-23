### 사용하기 전
1. HealthKit capability 추가하기
2. Info.plist에 
   `Privacy - Health Share Usage Description` : We need access to your health data to display your water intake.
   `Privacy - Health Update Usage Description` : We need access to your health data to log your water intake.
로 값을 준다.

### HealthKit 처리
### 권한 요청
```swift
func requestAuthorization() async throws {
        guard let waterType = HKObjectType.quantityType(forIdentifier: .dietaryWater) else {
            throw HealthKitError.failedQuantityType
        }
        
        try await healthStore.requestAuthorization(toShare: [waterType], read: [waterType])
    }
```

로 `HKObjectType` 배열을 통해 원하는 오브젝트에 대한 공유 및 읽기 권한 가져오기

### Save
```swift
func saveWaterIntake(amount: Double, date: Date) async throws {
    guard let waterType = HKObjectType.quantityType(forIdentifier: .dietaryWater) else {
        throw HealthKitError.failedQuantityType
    }
        
    let waterQuantity = HKQuantity(unit: .liter(), doubleValue: amount)
    let waterSample = HKQuantitySample(type: waterType, quantity: waterQuantity, start: date, end: date)
        
    try await healthStore.save(waterSample)
}
```

마찬가지로 원하는 오브젝트 타입에 대해 값 데이터를 적용할 수 있다.

### Read
```swift
func readWaterIntake(startDate: Date, endDate: Date) async throws -> Double {
    return try await withCheckedThrowingContinuation { continuation in
        guard let waterType = HKObjectType.quantityType(forIdentifier: .dietaryWater) else {
            continuation.resume(throwing: HealthKitError.failedQuantityType)
            return
        }
        
        let predicate = HKQuery.predicateForSamples(withStart: startDate, end: endDate, options: .strictStartDate)
        let query = HKStatisticsQuery(quantityType: waterType, quantitySamplePredicate: predicate, options: .cumulativeSum) { query, result, error in
            if let error = error {
                continuation.resume(throwing: error)
                return
            }
            
            guard let result = result, let sum = result.sumQuantity() else {
                continuation.resume(throwing: HealthKitError.failedFetchResult)
                return
            }
            
            let totalWater = sum.doubleValue(for: .liter())
            continuation.resume(returning: totalWater)
        }
        
        healthStore.execute(query)
    }
}
```

[`NSPredicate`](doc://com.apple.documentation/documentation/foundation/nspredicate) 을 인자로 사용하는 `HKStatisticsQuery` 타입으로 건강 데이터 가져올 수 있다.