[참조](https://hanulyun.medium.com/ios-multiple-configurations-%EB%A1%9C-debug-release-%EA%B5%AC%EB%B6%84%ED%95%B4-%EB%B0%B0%ED%8F%AC%ED%95%98%EA%B8%B0-1-43a6f8bd1b5b)
1. **Multiple Configurations 생성**: 'Debug', 'Staging', 'Release' 세 가지 설정을 만듭니다. 'Staging'은 Testflight용 Debug 환경, 'Release'는 Testflight 및 AppStore용 Production 환경입니다​[](https://hanulyun.medium.com/ios-multiple-configurations-%EB%A1%9C-debug-release-%EA%B5%AC%EB%B6%84%ED%95%B4-%EB%B0%B0%ED%8F%AC%ED%95%98%EA%B8%B0-1-43a6f8bd1b5b#:~:text=,AppStore%20%EC%97%90%20%EC%98%AC%EB%9D%BC%EA%B0%80%EB%8A%94%20Production%20%ED%99%98%EA%B2%BD)​.
    
2. **Build Settings에서 User-Defined Setting 생성**: 앱 아이콘을 구분하기 위한 사용자 정의 설정을 추가합니다. 이 설정은 Bundle Identifier의 일부가 됩니다​[](https://hanulyun.medium.com/ios-multiple-configurations-%EB%A1%9C-debug-release-%EA%B5%AC%EB%B6%84%ED%95%B4-%EB%B0%B0%ED%8F%AC%ED%95%98%EA%B8%B0-1-43a6f8bd1b5b#:~:text=,Identifier%20%EB%92%A4%EC%97%90%20%EB%B6%99%EB%8A%94%20%EB%B6%80%EB%B6%84%EC%9D%B4%20%EB%90%A0%EA%B1%B0%EC%98%88%EC%9A%94)​.
    
3. **Packaging Setting**: 'Product Bundle Identifier' 설정에서 Bundle Identifier의 앞부분과 Custom으로 생성한 `${BUNDLE_ID_SUFFIX}`를 결합합니다. 'Product Name'을 수정하면 나머지 설정이 자동으로 적용됩니다​[](https://hanulyun.medium.com/ios-multiple-configurations-%EB%A1%9C-debug-release-%EA%B5%AC%EB%B6%84%ED%95%B4-%EB%B0%B0%ED%8F%AC%ED%95%98%EA%B8%B0-1-43a6f8bd1b5b#:~:text=,%EB%A7%8C%20%EC%88%98%EC%A0%95%ED%95%98%EB%A9%B4%20%EB%82%98%EB%A8%B8%EC%A7%80%EB%8A%94%20%EC%9E%90%EB%8F%99%EC%9C%BC%EB%A1%9C%20%EC%9E%85%EB%A0%A5%EB%90%98%EB%8D%94%EB%9D%BC%EA%B5%AC%EC%9A%94)​.
    
4. **Multiple App Icon 설정**: 'Asset Catalog App Icon Set Name'에 앱 아이콘 설정을 입력하고, Assets 파일에 해당하는 앱 아이콘을 배치합니다​[](https://hanulyun.medium.com/ios-multiple-configurations-%EB%A1%9C-debug-release-%EA%B5%AC%EB%B6%84%ED%95%B4-%EB%B0%B0%ED%8F%AC%ED%95%98%EA%B8%B0-1-43a6f8bd1b5b#:~:text=,%ED%95%B4%EB%8B%B9%ED%95%98%EB%8A%94%20%EC%95%B1%20%EC%95%84%EC%9D%B4%EC%BD%98%EC%9D%84%20%EB%84%A3%EC%9C%BC%EB%A9%B4%20%EB%81%9D)​.
    
5. **Info.plist Setting**: Bundle Identifier가 변경되었으므로 Info.plist에서도 이를 반영합니다. 이를 통해 Debug와 Release가 각각 다른 이름과 아이콘으로 빌드됩니다​[](https://hanulyun.medium.com/ios-multiple-configurations-%EB%A1%9C-debug-release-%EA%B5%AC%EB%B6%84%ED%95%B4-%EB%B0%B0%ED%8F%AC%ED%95%98%EA%B8%B0-1-43a6f8bd1b5b#:~:text=,%EC%97%86%EC%96%B4%EC%84%9C%20%EC%A0%81%EC%9A%A9%ED%95%98%EC%A7%80%20%EB%AA%BB%ED%96%88%EC%A7%80%EB%A7%8C%20%EB%B6%84%EB%AA%85%ED%9E%88%20%EB%90%A0%EA%B1%B0%EC%98%88%EC%9A%94)​.