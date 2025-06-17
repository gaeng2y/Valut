EventStorage.swift 파일을 상세히 분석해드리겠습니다.

## EventStorage.swift 상세 분석

### **핵심 설계 철학**
이 파일은 **Thread-Safe한 영구 저장소**를 구현하여 다음 문제들을 해결합니다:
1. **데이터 영속성** → 앱 종료/재시작 시에도 이벤트 보존
2. **동시성 안전** → 멀티스레드 환경에서 데이터 무결성 보장
3. **상태 관리** → 이벤트별 전송 상태 추적

---

### **1. 프로토콜 설계 (추상화)**

```swift
protocol EventStorage {
    func save(_ event: Event)           // 새 이벤트 저장
    func loadPendingEvents() -> [Event] // 미전송 이벤트 조회
    func markAsSent(_ event: Event)     // 전송 완료 표시
    func deleteEvent(_ event: Event)    // 이벤트 삭제
    var count: Int { get }              // 미전송 이벤트 개수
}
```

**설계 장점:**
- **단일 책임**: 순수하게 저장소 기능만 정의
- **상태 기반**: 이벤트의 생명주기 관리 (저장 → 전송 중 → 완료 → 삭제)
- **확장성**: 다른 저장소 구현체 (Core Data, SQLite 등) 교체 가능

---

### **2. 동시성 아키텍처 (Reader-Writer Lock 패턴)**

```swift
private let queue = DispatchQueue(label: "userdefaults.storage", attributes: .concurrent)
```

#### **읽기 작업 (동시 실행 허용):**
```swift
func loadPendingEvents() -> [Event] {
    return queue.sync {  // sync로 즉시 결과 반환
        return loadStoredEvents()
            .filter { !$0.isSent }
            .map { $0.toEvent() }
    }
}

var count: Int {
    return queue.sync {  // 읽기 전용이므로 동시 접근 허용
        return loadStoredEvents().count { !$0.isSent }
    }
}
```

#### **쓰기 작업 (독점적 실행):**
```swift
func save(_ event: Event) {
    queue.async(flags: .barrier) {  // barrier로 독점적 접근
        var events = self.loadStoredEvents()
        let storedEvent = StoredEventData(event: event, isSent: false)
        events.append(storedEvent)
        self.saveStoredEvents(events)
    }
}
```

**동시성 전략:**
- **Concurrent Queue**: 여러 읽기 작업을 동시에 처리하여 성능 향상
- **Barrier Flag**: 쓰기 작업 시에만 모든 다른 작업을 차단하여 데이터 무결성 보장
- **async vs sync**: 쓰기는 비동기, 읽기는 동기로 적절한 성능 균형

---

### **3. 데이터 모델링 (상태 관리)**

```swift
// 암시적으로 존재하는 StoredEventData 구조
struct StoredEventData {
    let event: Event
    var isSent: Bool  // 핵심: 전송 상태 추적
    
    func toEvent() -> Event {
        return event
    }
}
```

**상태 기반 설계:**
- **저장 시**: `isSent = false` (미전송 상태)
- **전송 완료**: `isSent = true` (전송 완료 상태)
- **필터링**: 미전송 이벤트만 조회 가능

---

### **4. 이벤트 매칭 알고리즘**

#### **전송 완료 처리:**
```swift
func markAsSent(_ event: Event) {
    queue.async(flags: .barrier) {
        var events = self.loadStoredEvents()
        
        // 동일한 이벤트 중 가장 오래된 미전송 이벤트를 찾기
        if let index = events.firstIndex(where: {
            !$0.isSent &&                    // 미전송 상태이면서
            $0.category == event.category && // 카테고리가 같고
            $0.action == event.action &&     // 액션이 같고  
            $0.label == event.label          // 라벨이 같은 것
        }) {
            events[index].isSent = true      // 전송 완료 표시
            self.saveStoredEvents(events)
        }
    }
}
```

#### **이벤트 삭제:**
```swift
func deleteEvent(_ event: Event) {
    queue.async(flags: .barrier) {
        var events = self.loadStoredEvents()
        
        // 동일한 이벤트 중 가장 오래된 것을 삭제 (전송 상태 무관)
        if let index = events.firstIndex(where: {
            $0.category == event.category &&
            $0.action == event.action &&
            $0.label == event.label
        }) {
            events.remove(at: index)
            self.saveStoredEvents(events)
        }
    }
}
```

**매칭 전략:**
- **FIFO 원칙**: `firstIndex`로 가장 오래된 이벤트 우선 처리
- **내용 기반 매칭**: category, action, label로 동일성 판단
- **상태 고려**: markAsSent는 미전송만, deleteEvent는 전체 대상

---

### **5. 영구 저장소 구현 (UserDefaults)**

#### **데이터 로드:**
```swift
private func loadStoredEvents() -> [StoredEventData] {
    guard let data = userDefaults.data(forKey: eventsKey),
          let events = try? JSONDecoder().decode([StoredEventData].self, from: data) else {
        return []  // 데이터가 없거나 디코딩 실패 시 빈 배열 반환
    }
    return events
}
```

#### **데이터 저장:**
```swift
private func saveStoredEvents(_ events: [StoredEventData]) {
    if let data = try? JSONEncoder().encode(events) {
        userDefaults.set(data, forKey: eventsKey)
    }
    // 인코딩 실패 시 조용히 무시 (현재 상태 유지)
}
```

**저장소 특성:**
- **UserDefaults 선택 이유**: 
  - 간단한 설정과 자동 동기화
  - 앱 재시작 시 자동으로 데이터 복구
  - iOS 시스템이 백업 관리
- **JSON 직렬화**: Codable 프로토콜로 안전한 직렬화
- **에러 처리**: 실패 시 현재 상태 유지 (데이터 손실 방지)

---

### **6. 메모리 효율성과 성능 최적화**

#### **지연 로딩:**
```swift
// 매번 디스크에서 읽지 않고, 필요할 때만 로드
private func loadStoredEvents() -> [StoredEventData] {
    // UserDefaults는 내부적으로 캐싱하므로 효율적
}
```

#### **배치 연산:**
```swift
func save(_ event: Event) {
    queue.async(flags: .barrier) {
        var events = self.loadStoredEvents()  // 1. 전체 로드
        events.append(storedEvent)            // 2. 메모리에서 수정
        self.saveStoredEvents(events)         // 3. 일괄 저장
    }
}
```

**성능 고려사항:**
- **전체 로드/저장**: 이벤트 수가 적을 때 효율적 (UserDefaults 특성)
- **메모리 연산**: 디스크 I/O 최소화
- **큐 활용**: 백그라운드에서 비동기 처리로 UI 블록 방지

---

### **7. 에러 처리 전략**

#### **조용한 실패 (Graceful Degradation):**
```swift
// 디코딩 실패 시
guard let events = try? JSONDecoder().decode([StoredEventData].self, from: data) else {
    return []  // 빈 배열 반환으로 앱 크래시 방지
}

// 인코딩 실패 시  
if let data = try? JSONEncoder().encode(events) {
    userDefaults.set(data, forKey: eventsKey)
}
// 실패해도 아무것도 하지 않음 (현재 상태 유지)
```

**에러 처리 철학:**
- **앱 안정성 우선**: 저장 실패보다는 앱 크래시 방지
- **데이터 보존**: 기존 데이터를 삭제하지 않음
- **투명한 실패**: 사용자에게 에러를 노출하지 않음

---

### **8. 확장성과 유지보수성**

#### **프로토콜 기반 설계:**
```swift
// 다른 저장소 구현체로 쉽게 교체 가능
final class CoreDataEventStorage: EventStorage {
    // Core Data 구현
}

final class SQLiteEventStorage: EventStorage {
    // SQLite 구현  
}
```

#### **설정 분리:**
```swift
private let userDefaults = UserDefaults.standard
private let eventsKey = "pending_events"  // 키 변경 용이
private let queue = DispatchQueue(label: "userdefaults.storage", attributes: .concurrent)
```

---

### **9. 잠재적 개선점과 고려사항**

#### **현재 제한사항:**
1. **UserDefaults 크기 제한**: 대량 이벤트 시 성능 저하 가능
2. **선형 검색**: `firstIndex` 사용으로 O(n) 복잡도
3. **전체 로드/저장**: 일부 수정에도 전체 데이터 처리

#### **개선 방안:**
```swift
// 1. 딕셔너리 기반 인덱싱
private var eventIndex: [String: [Int]] = [:] // 카테고리별 인덱스

// 2. 페이징 처리
func loadPendingEvents(limit: Int = 50) -> [Event]

// 3. 압축 저장
private func compressEvents(_ events: [StoredEventData]) -> Data?
```

---

### **전체적인 설계 평가**

#### **장점:**
1. **Thread-Safe**: Reader-Writer Lock 패턴으로 완벽한 동시성 제어
2. **간단함**: UserDefaults + JSON으로 구현 복잡도 최소화
3. **안정성**: 에러 상황에서도 앱 크래시 없이 동작
4. **확장성**: 프로토콜 기반으로 다른 저장소 교체 용이

#### **적합한 사용 사례:**
- **중소규모 이벤트** (수십~수백 개)
- **단순한 구조**의 이벤트 데이터
- **빠른 개발**이 필요한 프로젝트

#### **대안이 필요한 경우:**
- **대량 이벤트** (수천 개 이상) → Core Data, SQLite
- **복잡한 쿼리** 필요 → 관계형 데이터베이스
- **고성능** 요구사항 → 메모리 캐시 + 비동기 저장

이 구현은 **실용적이고 안정적인 설계**로, 대부분의 모바일 분석 SDK에서 요구하는 기본적인 저장소 기능을 효과적으로 제공합니다.