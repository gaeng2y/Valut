현재 나는 UIKit으로 만들어진 프로젝트에
SwiftUI로 구성된 ToastView라는 패키지를 추가한 상태이다.

```swift
@MainActor
    static func showToast(with message: String, duration: TimeInterval = 0.5, delay: TimeInterval = 2) {
        guard let window = UIApplication.shared.windows.first else { return }
        
        let toastView = ExodusToastView(message: message)
        let hostingController = UIHostingController(rootView: toastView)
        hostingController.view.frame = window.frame
        hostingController.view.backgroundColor = .clear
        guard let hostingView = hostingController.view else { return }
        
        window.addSubview(hostingView)
        hostingView.translatesAutoresizingMaskIntoConstraints = false
        NSLayoutConstraint.activate([
            hostingView.leadingAnchor.constraint(equalTo: window.leadingAnchor, constant: 20),
            hostingView.centerXAnchor.constraint(equalTo: window.centerXAnchor),
            hostingView.bottomAnchor.constraint(equalTo: window.bottomAnchor, constant: -22)
        ])
        
        UIView.animate(withDuration: duration,
                       delay: delay - duration,
                       options: .curveEaseOut) {
            hostingView.alpha = 0.0
        } completion: { _ in
            hostingView.removeFromSuperview()
        }
    }
```

위와 같이 View를 불러오고 UIHostingController에 rootView로 넣는다.