### Reference
https://medium.com/@darrenthiores/automatic-dependency-injection-di-for-your-swift-application-to-make-your-code-clean-911a8b59cb8a

https://medium.com/@darrenthiores/automatic-dependency-injection-di-in-swift-6-to-make-your-code-clean-4e0160c049ce
### 1) 첫 번째 글 요약 & 번역

**제목** : *Automatic Dependency Injection (DI) for your Swift application to make your code clean* (2024-08-27)([Medium][1])

| 영문 핵심                                                                                                                                                                         | 한글 번역 & 해설                                                                                                                                                                                                                                                                                                                                                     |
| ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| “Dependency injection (DI) is the most popular design-pattern … an object depends on one or more dependencies, then the dependencies are injected into the dependent object.” | **DI 정의** : 객체 A가 객체 B에 의존한다면, A 안에서 B를 직접 만들지 않고 “외부에서 주입”한다.                                                                                                                                                                                                                                                                                                 |
| Benefits : *loosely-coupled*, *testable*, *scalable*                                                                                                                          | **장점** : 결합도 감소, 테스트 용이(모킹), 구현 교체 용이                                                                                                                                                                                                                                                                                                                          |
| Problem : in large apps manual DI becomes boilerplate & memory-management headache.                                                                                           | **문제점** : 큰 코드베이스에서 일일이 생성자 주입을 관리하다 보면 코드가 장황해지고 메모리 관리도 난해해진다.                                                                                                                                                                                                                                                                                               |
| “Automatic Dependency Injection … the injection process is automated.”                                                                                                        | **Automatic DI**: 의존성 등록과 주입을 자동화해 개발자가 신경 쓸 부분을 최소화한다.                                                                                                                                                                                                                                                                                                        |
| **Implementation** (Swift): `DependencyInjector`에 의존성 딕셔너리 저장 → `@Provider`가 등록, `@Inject`가 꺼내 씀.                                                                             | **구현 요지** : <br>`swift<br>struct DependencyInjector { static var list:[String:Any]=[:] … }<br>@propertyWrapper struct Provider<T>{ init(wrappedValue:T){ DependencyInjector.register(…) }}<br>@propertyWrapper struct Inject<T>{ var wrappedValue:T = DependencyInjector.resolve() }<br>`<br>`AppModule.inject()`에서 한번에 등록하고, ViewModel/뷰에서는 `@Inject`만 붙이면 끝. |
| Conclusion : “When working on a big application, you will feel how helpful it is …”.                                                                                          | **결론** : 대형 프로젝트일수록 자동 DI가 보일러플레이트를 줄이고 생산성을 높인다.                                                                                                                                                                                                                                                                                                              |

---

### 2) 두 번째 글 요약 & 번역

**제목** : *Automatic Dependency Injection (DI) in Swift 6 to make your code clean* (2025-04-15)([Medium][2])

| Swift 6 변경점                                                                                        | 한글 번역 & 해설                                                                                                                                                           |
| -------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| “This article is a continuation … update the implementation to follow Swift 6 Strict Concurrency.” | **배경** : Swift 6의 *Strict Concurrency* 규칙에 맞춰 자동 DI 구현을 수정한다.                                                                                                        |
| Injector now marked `@MainActor`; keeps a mutable singleton `shared`.                              | **주요 수정** : 모든 등록·해제·조회가 메인 스레드에서 이뤄지도록 `@MainActor`로 래핑. <br>`swift<br>@MainActor struct DependenciesInjector { … static var shared = DependenciesInjector() }<br>` |
| Property wrappers `@Inject`, `@Provider` 역시 `@MainActor`                                           | **안전성** : 주입 로직도 메인 스레드에서 실행돼 데이터 경쟁(Data Race) 위험 감소.                                                                                                               |
| Tutorial app : Counter protocol & `SingleCounter` implementation.                                  | **예제** : 카운터 앱 ― Repository 대신 `Counter` 의존성 주입.                                                                                                                     |
| `deinit`에서 Main-actor-isolated 프로퍼티 접근 시 `DispatchQueue.main.sync` 필요.                             | **주의** : 뷰모델 소멸 시점에 배우체 접근하려면 동기 메인 큐 호출로 격리 우회.                                                                                                                     |
| Wrap-up : 구현은 이전 글과 유사하나 Swift 6 규칙 준수.                                                            | **마무리** : 큰 틀은 동일하지만, 스레딩 제약이 추가됐다.                                                                                                                                  |

---

### 공통 핵심 포인트 정리

1. **Provider/Inject 패턴** : 의존성을 *등록*(`@Provider`)하고 *주입*(`@Inject`)하는 두 개의 property-wrapper가 자동 DI의 심장부이다.
2. **초기 한 번의 부트스트랩** : `AppModule.inject()`를 앱 시작 시 호출해 의존성 그래프를 완성한다. 이후 객체는 생성자 파라미터 없이 `@Inject`만으로 의존성을 획득한다.
3. **Swift 6 대응** : 동시성 강화로 인해 모든 DI 동작을 `@MainActor`에 묶어 안전성을 확보했다.([Medium][1], [Medium][2])

---

### 개인적인 견해

* **교육적 가치** : 두 글 모두 *DI 개념 → 문제 → 실전 코드* 순으로 서술해 입문자가 맥락을 이해하기 좋다. 특히 property-wrapper를 활용한 간결함은 “Swift-다운” 맛을 잘 살렸다.
* **한계** :

  * 전역 싱글턴 + 가변 딕셔너리는 **스코프/라이프사이클 구분**이 어렵다. 객체 범위(트랜지언트·스코프·싱글턴) 관리가 필요한 복잡한 그래프에는 Swinject·Needle·Resolver 같은 전문 DI 프레임워크가 더 안전하다.
  * `fatalError`로 예외를 처리해 **런타임 크래시** 위험이 있다. 컴파일 타임 검증이나 옵셔널 리턴으로 방어 코드를 추가하면 실서비스 안정성이 올라간다.
  * Swift 6에서 모든 등록·조회가 메인 스레드에 고정되면 **백그라운드-액터** 환경(예: 비동기 작업)에서는 오히려 병목이 될 수 있다. actor-isolated 딕셔너리나 `Sendable` 제약을 적용해 다중 스레드 안전성을 확보할 여지가 있다.
* **확장 아이디어** :

  * Swift Macros(매크로)가 정식 도입되면 `@Inject`/`@Provider` 매크로화로 **컴파일-타임 의존성 그래프 생성**이 가능해질 전망이다.
  * 의존성 **스코프 정책**(e.g., `@Scoped(.view)`), **라이프사이클 후킹**(deinit clean-up), **실행 환경별 모듈 분리**(테스트 / 프로덕션) 등을 추가하면 실무 활용도가 한층 높아질 것이다.

> 총평 : 블로그 예제는 “작고 깨끗한” SwiftUI 앱에서 즉시 써먹기 좋다. 다만 팀 규모·코드베이스·멀티스레딩 요구가 커질수록 **전문 DI 프레임워크**나 더 견고한 설계 패턴을 검토하길 권한다.

[1]: https://medium.com/%40darrenthiores/automatic-dependency-injection-di-for-your-swift-application-to-make-your-code-clean-911a8b59cb8a "Automatic Dependency Injection (DI) for your Swift application to make your code clean | by Darren Thiores | Medium"
[2]: https://medium.com/%40darrenthiores/automatic-dependency-injection-di-in-swift-6-to-make-your-code-clean-4e0160c049ce "Dependency Injection (DI) in Swift 6 | Medium"
