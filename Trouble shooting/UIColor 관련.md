## Problem
현재 컬러를 plist를 통해 가져오도록 정의되어있다.

하지만 관리가 힘들다는 것으로 인해 xcasset을 통해 가져오려고 작업을 하고 있던 중

`UIColor(named:)`를 통해 assets에서 가져올 수 있다는 것을 파악해서 적용하려했지만 해당 메소드에서 가져오는 컬러는 OS의 mode로 컬러를 라이트, 다크로 가져오고 있었다.

그래서 같은 방법으로 내가 원하는 모드의 컬러를 어떻게 가져올 수 있을까?

## Solution
### 1. UIColor(naemd:in:compatibleWith:) 
https://developer.apple.com/documentation/uikit/uicolor/2877381-init 에 있는 생성자를 통해 가져오면 어떨까?
마지막 파라미터인 compatibleWith: 는 [`UITraitCollection`](https://developer.apple.com/documentation/uikit/uitraitcollection)타입으로 생성자 중 [`UIUserInterfaceStyle`](https://developer.apple.com/documentation/uikit/uiuserinterfacestyle)타입을 받는 것이 있어서 여기서 내가 지정하여서 가져올 수 있는지 파악하기
