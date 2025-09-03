## 개요

**AVCaptureSession**은 iOS와 macOS에서 모든 미디어 캡처의 기반이 되는 클래스입니다. 이 세션은 다음을 관리합니다:
- 앱의 OS 캡처 인프라 및 캡처 장치에 대한 독점 액세스
- 입력 장치에서 미디어 출력으로의 데이터 흐름
- 입력과 출력 간의 연결 구성을 통해 캡처 세션의 기능 정의

## 주요 구성 단계

### 1. 입력과 출력을 세션에 연결

모든 캡처 세션에는 최소한 하나의 **캡처 입력**과 **캡처 출력**이 필요합니다.

**비디오 입력 설정 (카메라 사용)**
```swift
captureSession.beginConfiguration()
let videoDevice = AVCaptureDevice.default(.builtInWideAngleCamera,
                                          for: .video, position: .unspecified)
guard
    let videoDeviceInput = try? AVCaptureDeviceInput(device: videoDevice!),
    captureSession.canAddInput(videoDeviceInput)
    else { return }
captureSession.addInput(videoDeviceInput)
```

**출력 설정 (사진 캡처용)**
```swift
let photoOutput = AVCapturePhotoOutput()
guard captureSession.canAddOutput(photoOutput) else { return }
captureSession.sessionPreset = .photo
captureSession.addOutput(photoOutput)
captureSession.commitConfiguration()
```

**중요한 규칙**:
- 세션의 입력이나 출력을 변경하기 전에 `beginConfiguration()` 호출
- 변경 후 `commitConfiguration()` 호출

### 2. 카메라 미리보기 표시

사용자가 사진을 찍거나 비디오 녹화를 시작하기 전에 카메라 입력을 볼 수 있도록 해야 합니다.

**PreviewView 클래스 정의**
```swift
class PreviewView: UIView {
    override class var layerClass: AnyClass {
        return AVCaptureVideoPreviewLayer.self
    }
    
    /// 레이어를 정적으로 알려진 타입으로 가져오는 편의 래퍼
    var videoPreviewLayer: AVCaptureVideoPreviewLayer {
        return layer as! AVCaptureVideoPreviewLayer
    }
}
```

**미리보기 레이어를 캡처 세션과 연결**
```swift
self.previewView.videoPreviewLayer.session = self.captureSession
```

**참고사항**:
- `AVCaptureVideoPreviewLayer`는 Core Animation 레이어이므로 다른 CALayer 서브클래스처럼 표시하고 스타일링 가능
- 다중 인터페이스 방향을 지원하는 경우, 미리보기 레이어의 `connection`을 사용하여 UI와 일치하는 `videoOrientation` 설정

### 3. 캡처 세션 실행

입력, 출력, 미리보기를 구성한 후 `startRunning()`을 호출하여 데이터가 입력에서 출력으로 흐르도록 합니다.

**캡처 출력 유형에 따른 동작**:
- **일부 캡처 출력**: 세션 실행만으로 미디어 캡처 시작 (예: `AVCaptureVideoDataOutput`)
- **기타 캡처 출력**: 세션 실행 후 캡처 출력 클래스를 사용하여 캡처 시작 (예: 사진 앱에서 `AVCapturePhotoOutput`의 `capturePhoto(with:delegate:)` 메서드 사용)

## 세션 구성 예시

**다중 입력/출력 가능**:
- 영화에서 비디오와 오디오 모두 녹화: 카메라와 마이크 장치 모두에 대한 입력 추가
- 동일한 카메라에서 사진과 영화 모두 캡처: `AVCapturePhotoOutput`과 `AVCaptureMovieFileOutput` 모두 세션에 추가

이 설정을 통해 커스텀 카메라 앱에서 사진 촬영, 비디오 녹화, 실시간 미리보기 등의 기능을 구현할 수 있습니다.