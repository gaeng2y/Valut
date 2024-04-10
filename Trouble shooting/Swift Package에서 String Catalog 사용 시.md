Pokedex 프로젝트 진행 중에 [Region](https://github.com/gaeng2y/Pokedex/blob/main/PokedexKit/Sources/Region.swift)이라는 열거형을 만들어서 사용하고 있는 중에 PokedexKit이라는 Swift package에 xcsstrings 파일을 만들어서 사용할 때 제대로 가져오지 못하는 문제가 있었다.

```swift
public var regionName: String {
        switch self {
        case .kanto: return "Kanto"
        case .johto: return "Johto"
        case .hoenn: return "Hoenn"
        case .sinnoh: return "Sinnoh"
        case .unova: return "Unova"
        case .kalos: return "Kalos"
        case .alola: return "Alola"
        case .galar: return "Galar"
        case .paldea: return "Paldea"
        }
    }
```

이런식으로 선언을 해두고 사용 중이었지만 실제로 빌드를 해서 런타임에 확인을 해보면 현지화 가능한 스트링으로 나오지 않은 문제가 있어서 확인해보니

```swift
public var regionName: String {
        let region: String.LocalizationValue = switch self {
        case .kanto: "Kanto"
        case .johto: "Johto"
        case .hoenn: "Hoenn"
        case .sinnoh: "Sinnoh"
        case .unova: "Unova"
        case .kalos: "Kalos"
        case .alola: "Alola"
        case .galar: "Galar"
        case .paldea: "Paldea"
        }
        
        return String(localized: region, bundle: .module)
    }
```

위와같이 bundle 파라미터를 적용한 메소드를 이용하여 인스턴스를 생성해주면 정상적으로 들어간다.

4월 10일 수정내용
이렇게 사용했었는데 앱 언어를 변경 시 제대로 적용되지 않아

```swift
private var regionNameResource: LocalizedStringResource {
        let region: String = switch self {
        case .kanto: "Kanto"
        case .johto: "Johto"
        case .hoenn: "Hoenn"
        case .sinnoh: "Sinnoh"
        case .unova: "Unova"
        case .kalos: "Kalos"
        case .alola: "Alola"
        case .galar: "Galar"
        case .paldea: "Paldea"
        }
        return LocalizedStringResource(stringLiteral: region)
    }
    
    public var regionName: String {
        String(localized: regionNameResource)
    }
```

이렇게 private 프로터리를 `LocalizedStringResource` 으로 만들어 사용
