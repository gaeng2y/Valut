## 정의
메소드를 뒤섞다 라는 의미인데...

> [!quote] Runtime 시점에 기존 메서드를 다른 메서드로 바꾸어 실행하는 것

## 언제 사용?
이미 정해진 iOS의 특정 메소드가 실행될 때, 해당 메소드 대신 다른 메소드가 실행되도록 바꿀 때 사용한다.

예를 들면, `viewDidAppear`될 때마다 로그를 출력하고 싶을 수 있다.

일반적으로 override 해서 적지만 VC가 많아질 수록 보일러플레이트가 발생한다.

또는 UIViewController를 상속받는 서브 클래스를 만들어주고, 거기에서 override를 한 후, 이후 ViewController에서 서브 클래스를 상속받게 될 수도 잇다.

이럴 때 사용하는게 **Method Swizzling**! 🔀

viewDidAppear -. swizzleViewDidAppear

말 그대로, **기존에 있는 메소드**를 **런타임**에 **원하는 메소드로 바꿔**주는 것이다.

일반적으로는 앱에 분석기능을 통합할 때, 특정 기능을 클래스&서브 클래스 모두 한 번에 적용시키고 싶을 때

사용한다고 한다.

## 사용 방법
```swift
extension UIViewController {
    @objc public func swizzleViewWillAppear(animated: Bool) {
        print("🌞 swizzleViewWillAppear")
    }
}
```
이렇게 확장하고 Method Swizzling을 적용하는 타입 메소드를 만들어준다.
```swift
extension UIViewController {
    class func swizzleMethod() {
        let originalSelector = #selector(viewWillAppear)
        let swizzleSelector = #selector(swizzleViewWillAppear)
    }

    @objc public func swizzleViewWillAppear(animated: Bool) {
        print("🌞 swizzleViewWillAppear")
    }
}
```
**Selector**를 이용해 가져옴...!

그리고 다음과 같이 **인스턴스 메서드를 반환하게 해주는 메서드**를 guard문을 통해 작성하는데,
```swift
extension UIViewController {
    class func swizzleMethod() {
        let originalSelector = #selector(viewWillAppear)
        let swizzleSelector = #selector(swizzleViewWillAppear)

        guard
            let originMethod = class_getInstanceMethod(UIViewController.self, originalSelector),
            let swizzleMethod = class_getInstanceMethod(UIViewController.self, swizzleSelector)
            else { return }
    }

    @objc public func swizzleViewWillAppear(animated: Bool) {
        print("🌞 swizzleViewWillAppear")
    }
}
```
**Selector라는 것은 함수 자체를 가리키는 것이 아니기 때문**에, 진짜 **UIViewController** 클래스 안의 **ViewWillAppear**, **swizzleViewWillAppear**라는 **인스턴스 메서드**를 받는 것이`class_getInstanceMethod` 라는 함수를 이용해서 가져온는 것이다.
```swift

extension UIViewController {
    class func swizzleMethod() {
        let originalSelector = #selector(viewWillAppear)
        let swizzleSelector = #selector(swizzleViewWillAppear)

        guard
            let originMethod = class_getInstanceMethod(UIViewController.self, originalSelector),
            let swizzleMethod = class_getInstanceMethod(UIViewController.self, swizzleSelector)
            else { return }

            method_exchangeImplementations(originMethod, swizzleMethod)
    }

    @objc public func swizzleViewWillAppear(animated: Bool) {
        print("🌞 swizzleViewWillAppear")
    }
}
```
`**method_exchangeImplementations**`를 실행해 주면, Swizzling 작업은 끝

그리고 `AppDelegate`같은 곳에서 `UIViewController.swizzleMethod()` 를 호출해주면된다.
