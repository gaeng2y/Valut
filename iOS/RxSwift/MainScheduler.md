- MainScheduler.instance는 바로 sync하게 동작하며, MainScheduler.asyncInstance는 바로 async하게 동작
- 이미 메인스레드인 경우, instance는 동기적으로 이벤트를 바로 처리할 수 있고, asyncInstance는 비동기 이벤트 처리를 보장
- 웬만하면 instance를 사용하되, 특이 케이스는 asyncInstance를 사용할 것 
  - 특이 케이스: 해당 스트림에서 재귀적인 접근이 동작할때, Rx 내부적으로 이런 동작은 async에서 처리하게끔 되어있어서 아래처럼 경고 메시지가 뜨는데 이 경우에 asyncIntance를 사용할 것

```
// https://github.com/RxSwiftCommunity/RxBiBinding/issues/20

⚠️ Reentrancy anomaly was detected.
Debugging: To debug this issue you can set a breakpoint in /Users/antonioscardigno/Documents/Sogetel/repository-forward/SOGETEL-RUMORS/DEV/iOSProjects/branches/newUI/Pods/RxSwift/RxSwift/Rx.swift:97 and observe the call stack.
Problem: This behavior is breaking the observable sequence grammar. next (error | completed)?
This behavior breaks the grammar because there is overlapping between sequence events.
Observable sequence is trying to send an event before sending of previous event has finished.
Interpretation: This could mean that there is some kind of unexpected cyclic dependency in your code,
or that the system is not behaving in the expected way.
Remedy: If this is the expected behavior this message can be suppressed by adding .observeOn(MainScheduler.asyncInstance)
or by enqueing sequence events in some other way.
```
