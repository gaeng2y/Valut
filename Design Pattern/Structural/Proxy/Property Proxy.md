실제 시스템에서 마주치는 요구 사항 중 하나는 클래스에 있는 프로퍼티를 제거하여 별도의 클래스에 포함시키는 것이다.

따라서 단순히 Int나 String을 나타내는 속성을 일반적인 속성으로 가지는 것이 아니라 실제로는 별도의 컨테이너에 추가적인 일이 발생하는 것입니다.

예제에서 

```swift
class Property<T: Equatable>
{
  private var _value: T

  public var value: T {
    get {
      return _value
    }
    set(value) {
      if (value == _value) {
        return
      }
      print("Setting value to \(value)")
      _value = value
    }
  }

  init(_ value: T)
  {
    self._value = value
  }
}
```
객체를 따로 두는 것이 아닌 프로퍼티로 실제 값을 접근하지 않고 프록시를 둬 사용할 수 있다.