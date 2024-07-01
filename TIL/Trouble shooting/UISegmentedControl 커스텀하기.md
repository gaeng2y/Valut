ref: https://medium.com/ios-app開發之世界這麼大/how-to-custom-uisegmentcontrol-71ac2bd69499

### UISegmentControl의 보기 계층 구조를 분석하기
![](https://miro.medium.com/v2/resize:fit:720/format:webp/1*Ahg-tJeDNrcDkKxU7yUiXw.png)


기본적으로 Background가 있고 각각 요소들에 대한 이미지뷰 그리고 선택된 부분에 대한 이미지뷰 등 이렇게 나눠져있다.

```swift
override func layoutSubviews() {
	super.layoutSubviews()
	
	self.backgroundColor = .systemOrange
	self.layer.cornerRadius = self.radius
	self.layer.masksToBounds = true
	
	// 선택된 이미지뷰 찾기
	let selectedImageViewIndex = self.numberOfSegments
	guard let selectedImageView = self.subviews[selectedImageViewIndex] as? UIImageView else {
		return
	}
	selectedImageView.backgroundColor = .green
	selectedImageView.image = nil
    }
```

이렇게 `layoutSubviews()`에서 변경해줄 수 있으나

클릭하거나 스와이프할 때 선택한 이미지 보기 센터 색상이 변경되고 모서리 반경이 변경된 것을 찾을 수 있습니다. 

```swift
private var segmentInset: CGFloat = 0.1 {
	didSet {
		guard self.segmentInset == .zero else {
			return
		}
		self.segmentInset = 0.1
	}
}

selectedImageView.bounds = selectedImageView.bounds.insetBy(dx: self.segmentInset, dy: self.segmentInset)
        
selectedImageView.layer.masksToBounds = true
selectedImageView.layer.cornerRadius = self.radius
```

이렇게 바꿔주니 정상적으로 된다!