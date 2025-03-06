> [!summary]
> Work with audiovisual assets, control device cameras, process audio, and configure system audio interactions.

> 오디오 에셋, 디바이스 카메라 컨트롤, 오디오 처리 그리고 시스템 오디오 상호작용에 대한 역할을 해주는 프레임워크
![avfoundation](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FHTX4V%2Fbtrx0XiUv8S%2Fd2S5SpgY6HxDThcuLelewK%2Fimg.png)

- AVKit은 UI 구현을 위한 인터페이스 제공
### AVAsset
> [!summary]
> An object that models timed audiovisual media.

> 시간이 지정된 시청각 미디어를 모델링하는 객체
> QuickTime 동영상이나 MP3 오디오 파일과 같은 파일 기반 미디어와 HTTP 라이브 스트리밍(HLS)을 사용하여 스트리밍된 미디어를 모델링

![AVAsset](https://docs-assets.developer.apple.com/published/a01a16315e681312a0596a223db5b961/media-3845943@2x.png)
- 미디어에 대한 데이터 부분들을 트랙
- 트랙 속성을 비동기적으로 로드 (파일 사이즈가 크기 때문에)

### AVPlayer
> [!summary]
> An object that provides the interface to control the player’s transport behavior

> 미디어 Asset의 재생 및 타이밍을 관리하는 컨트롤러 객체로 AVPlayerQuickTime 동영상, MP3 오디오 파일 등의 로컬 및 원격 파일 기반 미디어 및 HLS를 사용해 제공되는 미디어까지 재생할 때 사용

![avplayer](https://blog.kakaocdn.net/dn/c1ZqGL/btrx2SorKVZ/SEzvpHwwAM5CKTnjVh5SFk/img.png)
- AVPlayer 자체는 비시각적인 객체
- 화면에 표시하기 위해서는 AVKit / AVPlayerLayer와 같은 방식을 사용
### AVPlayerItem
> [!summary]
> An object that models the timing and presentation state of an asset during playback

- 쉽게 말해서 시간과 그에 따른 현재 미디어 상태 정보를 가지고 있는 객체
- AVAsset은 그 자체로 미디어가 가진 모든 정보가 포함
- AVPlayerItem은 시간 경과에 따른 현재 상태 정보를 담음
### AVAudioRecorder
> [!summary]
> An object that records audio data to a file

1. 시스템 입력 장치를 통해 오디오 녹음
2. 중지하거나 지정된 시간이 될 때 까지 녹음
3. 녹음 일시 정지 및 재개
4. 녹음 수준 측정 데이터에 액세스

> 음성 변조 등 고급 기능들의 사용을 위해서는 AVAudioEngine을 사용

### AVAudioPlayer
> [!summary]
> An object that plays audio data from a file or buffer.

1. 파일이나 버퍼에서 원하는 기간의 오디오 재생
2. 재생되는 오디오의 볼륨, 패닝, 속도 및 반복 동작 등을 제어
3. 재생 수준 측정 데이터에 액세스
4. 여러 플레이어의 재생을 동기화하여 여러 사운드를 동시 재생

> 실시간으로 스트리밍하는 오디오 같은 경우 AVPlayer를, 로컬 파일들의 재생은AVAuidoPlayer를 이용

