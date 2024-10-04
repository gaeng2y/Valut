각 섹션에 대한 선택적 헤더 및 푸터 보기가 있는 그리드로 항목을 구성하는 레이아웃 객체이다.

```swift
@MainActor
class UICollectionViewFlowLayout : UICollectionViewLayout
```

# Overview
Flow layout은 콜렉션 뷰의 한 종류이다. 콜렉션 뷰의 항목은 한 행 또는 열(스크롤 방향에 따라 다름)에서 다음 행 또는 열로 흐르며, 각 행에는 맞는 만큼의 셀이 포함된다. 셀은 크기가 같거나 다를 수 있다.

Flow layout은 컬렉션 뷰의 델리게이트 객체와 함께 작동하여 각 섹션과 그리드의 항목, 헤더 및 푸터 크기를 결정한다. 해당 델리게이트 객체는 반드시 [`UICollectionViewDelegateFlowLayout`](https://developer.apple.com/documentation/uikit/uicollectionviewdelegateflowlayout) 프로토콜을 준수해야한다. 델리게이트를 사용하면 레이아웃 정보를 동적으로 조정할 수 있다.

Flow 레이아웃은 한 방향으로 고정된 거리를 사용하고, 다른 방향으로는 스크롤 가능한 거리를 사용하여 콘텐츠를 배치한다.

각 섹션은 커스텀 헤더와 푸터를 가질 수 있으며, 이를 설정하려면 헤더나 푸터의 크기를 0이 아닌 값으로 지정해야 한다. 적절한 델리게이트 메소드를 구현하거나 headerReferenceSize와 footerReferenceSize 속성에 값을 할당하여 이를 구성할 수 있다. 만약 헤더나 푸터 크기가 0으로 설정되면, 해당 뷰는 컬렉션 뷰에 추가되지 않는다.