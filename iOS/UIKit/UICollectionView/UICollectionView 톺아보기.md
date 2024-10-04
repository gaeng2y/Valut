`UICollectionView`는 정렬된 데이터 항목 모음을 관리하고 사용자 정의 가능한 레이아웃을 사용하여 제공하는 객체이다.

![](Pasted%20image%2020240417143700.png)

컬렉션뷰는 테이블뷰와 마찬가지로, UICollectionViewCell을 사용하여 데이터를 화면에 표현합니다. 
그 외에 Supplementary view(Section header, footer)를 지원 함으로써 다음과 같이 셀을 구분하여 표현 가능 합니다.
## Overview
사용자 인터페이스에 컬렉션 뷰를 추가할 때, 앱의 주요 작업은 해당 컬렉션 뷰와 관련된 데이터를 관리하는 것입니다. 컬렉션 뷰는 컬렉션 뷰의 [`dataSource`](https://developer.apple.com/documentation/uikit/uicollectionview/1618091-datasource) 프로퍼티에 저장된 데이터 소스 개체에서 데이터를 가져옵니다. 데이터 소스의 경우, 컬렉션 뷰의 데이터 및 사용자 인터페이스에 대한 업데이트를 간단하고 효율적으로 관리하는 데 필요한 동작을 제공하는 [`UICollectionViewDiffableDataSource`](https://developer.apple.com/documentation/uikit/uicollectionviewdiffabledatasource) 객체를 사용할 수 있습니다. 또는 [`UICollectionViewDataSource`](https://developer.apple.com/documentation/uikit/uicollectionviewdatasource) 프로토콜을 채택하여 사용자 지정 데이터 소스 객체를 만들 수 있습니다.


![example1](https://docs-assets.developer.apple.com/published/a84db79dea/50390428-f9f2-4cbc-bd99-1cacca4f0617.png)

