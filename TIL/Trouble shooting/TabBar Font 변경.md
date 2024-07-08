```swift
// 탭바 폰트 설정
let appearance = UITabBarItem.appearance()
let attributes = [NSAttributedString.Key.font:UIFont(name: "폰트 이름", size: 폰트사이즈)]
appearance.setTitleTextAttributes(attributes as [NSAttributedString.Key: Any], for: .normal)
```