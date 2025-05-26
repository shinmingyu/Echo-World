## AVFoundation 요약

### 개요

**AVFoundation**은 iOS 및 macOS에서 **미디어 처리, 제어, 가져오기 등의 핵심 기능을 제공하는 프레임워크**입니다. 이 프레임워크는 오디오 및 비디오 기능을 구현하는 데 필수적입니다. **AVKit**은 이러한 AVFoundation을 기반으로 하여 좀 더 **고수준의 기능을 제공**하는 프레임워크입니다.

### 주요 구성 요소 (재생 관련)

AVFoundation에서 미디어 재생을 위해 주로 사용되는 객체들은 **AVPlayer, AVPlayerItem, AVAsset**입니다.

- **AVAsset**는 비디오나 오디오 파일과 같은 **미디어 데이터를 나타내는 객체**입니다. 미디어의 메타데이터와 트랙 정보를 포함하며, 실제 미디어 데이터를 직접 다루기보다는 미디어의 구조와 속성에 대한 정보를 제공합니다. 비디오, 오디오뿐만 아니라 자막 정보도 포함될 수 있습니다. AVAsset은 미디어의 **형식과 위치로부터 독립적인 인터페이스를 제공**합니다. 로컬 URL 또는 원격 서버의 스트림(예: HLS) URL로 초기화할 수 있으며, 프레임워크가 미디어를 효율적으로 검색하고 로드하는 작업을 대신 수행합니다.
- **AVPlayerItem**은 **재생할 미디어의 타이밍과 프레젠테이션 상태를 모델링**하며, **AVPlayer가 재생할 수 있는 에셋(asset)을 나타냅니다**. 즉, AVAsset과 AVPlayer 사이의 **중간 계층 역할**을 합니다.
- **AVPlayer**는 **미디어의 재생을 관리하는 객체**입니다. AVPlayerItem 객체를 받아 미디어를 재생합니다.

이 세 클래스의 관계는 AVAsset 객체를 AVPlayerItem에 포함시키고, 이 AVPlayerItem을 AVPlayer가 재생하는 형태입니다. 코드로 보면, URL로 AVAsset을 생성하고, 이 AVAsset으로 AVPlayerItem을 생성한 뒤, 최종적으로 AVPlayerItem을 AVPlayer에 할당하여 재생을 시작합니다.

```
let url = URL(string: "<https://example.com/audio.mp3>") /// 미디어 데이터를 나타내는 AVAsset 객체
let asset = AVAsset(url: url!)
/// 에셋을 포함하고, 재생 상태 및 정보를 관리하는 AVPlayerItem 객체
let playerItem = AVPlayerItem(asset: asset)
/// 미디어 재생을 관리하는 객체
let player = AVPlayer(playerItem: playerItem)
player.play()
```

AVAsset의 속성 (예: 재생 가능 여부, 기간, 메타데이터)은 요청될 때 지연 로딩됩니다. UI 중단이나 미디어 서비스 종료를 방지하기 위해 **AVAsset의 속성은 비동기적으로 로드해야 합니다**.

AVPlayer와 AVPlayerItem은 상태가 자주 변경되는 동적 객체이며, 상태 변화에 대응하기 위해 **Key-Value Observing (KVO)**을 사용할 수 있습니다. 특히 AVPlayerItem의 `status` 속성을 관찰하여 재생 준비 상태(.readyToPlay), 실패 상태(.failed), 알 수 없는 상태(.unknown) 등을 파악할 수 있습니다.

미디어 재생은 시간 기반 활동이므로 **시간 기반 작업** (예: 탐색)이 중요합니다. AVFoundation은 시간 표현에 부동 소수점 `NSTimeInterval` 대신 **Core Media 프레임워크의 `CMTime` 데이터 타입을 사용**합니다. `CMTime`는 시간의 유리수(분수) 표현으로, `value` (분자)와 `timescale` (분모) 필드를 가집니다. 이는 미디어의 프레임 속도나 샘플 속도에 기반한 시간을 정확하게 표현하는 데 유용합니다. `CMTime` 생성 및 연산은 Core Media에서 제공하는 함수나 Swift의 확장 기능을 통해 수행할 수 있습니다.

AVPlayer는 시간 변화를 관찰하기 위해 **주기적 관찰 (periodic observations)** 및 **경계 관찰 (boundary observations)** 두 가지 방법을 제공합니다. KVO는 연속적인 상태 변화 관찰에 적합하지 않으므로 AVPlayer 시간 관찰에 직접 사용되지 않습니다. 주기적 시간 관찰은 `addPeriodicTimeObserver` 메서드를 사용하여 특정 시간 간격으로 콜백을 받도록 설정합니다. 관찰을 중지하려면 `removeTimeObserver`를 호출하고 토큰을 해제해야 합니다.

미디어 탐색(seeking)은 `AVPlayer`의 `seek(to:)` 메서드를 사용하여 수행할 수 있습니다. `seek(to:)`는 빠르지만 정밀도가 떨어질 수 있으며, 정확한 탐색이 필요하면 `seek(to:toleranceBefore:toleranceAfter:)` 메서드를 사용하여 허용 오차를 지정할 수 있습니다. 샘플 단위의 정확한 탐색을 위해 허용 오차를 0으로 설정할 수 있지만, 이는 디코딩 지연을 유발할 수 있습니다.

### 오디오 처리

오디오 기능을 구현하기 위해 AVFoundation의 여러 클래스가 사용됩니다.

#### AVAudioSession

**AVAudioSession**은 **앱의 오디오 동작을 시스템에 알리고 오디오 하드웨어를 제어하는 객체**입니다. 이를 통해 백그라운드 재생, 녹음 권한 등을 관리할 수 있습니다. 음성 메모 앱 개발 시, 앱 실행 후 최초 녹음이 되지 않는 문제가 AVAudioSession을 초기화하지 않아서 발생했으며, 녹음 전에 세션을 초기화하여 문제를 해결했습니다.

AVAudioSession은 `sharedInstance()`를 통해 싱글톤 객체를 제공하며, `setCategory` 메서드를 사용하여 오디오 세션의 활용 범주(카테고리)와 옵션을 설정할 수 있습니다.

주요 오디오 카테고리:

- `.playback`: 녹음된 음악이나 사운드를 재생하는 데 사용됩니다.
- **.playAndRecord**: 오디오를 녹음하고 재생하기 위한 카테고리입니다 (예: VOIP 앱). 재생과 녹음이 자주 번갈아 사용될 때 적합합니다.
- `.record`: 재생 오디오를 음소거하면서 오디오를 녹음하는 카테고리입니다.

주요 옵션:

- `.duckOthers`: 이 세션의 오디오 재생 중 다른 오디오 세션의 볼륨을 줄입니다.
- `.allowBluetooth`: 블루투스 핸즈프리 장치를 사용 가능한 입력 경로로 표시할지 결정합니다 (.playAndRecord, .record 카테고리에서 사용 가능).
- `.defaultToSpeaker`: 세션 오디오를 기본적으로 내장 스피커로 출력할지 결정합니다.

AVAudioSession의 `setActive(true)`는 다른 오디오 세션과의 간섭을 시작하게 합니다. `AVAudioPlayer`나 `AVAudioRecorder`를 통한 재생/녹음 시작 시 `setActive`는 자동으로 `true`가 되고, 중지 시 자동으로 `false`가 됩니다. 한 앱에서 하나의 오디오 모드만 사용할 경우 앱 시작 시 카테고리만 설정하고 `setActive`는 자동 관리에 맡기는 경우가 많습니다.

마이크와 같은 하드웨어 접근 권한 요청 및 확인에 사용됩니다. 코드에서 `AVAudioSession.sharedInstance().requestRecordPermission { granted in ... }` 메서드를 사용하여 마이크 권한을 사용자에게 요청할 수 있습니다. 이 요청은 비동기적으로 처리되며, 결과(`granted`)는 클로저를 통해 전달받습니다.

#### AVAudioRecorder

**AVAudioRecorder**는 디바이스의 마이크로 **오디오를 녹음할 수 있는 객체**입니다. 주요 기능:

- 디바이스로부터 음성 녹음
- 녹음 중지 및 재개
- 녹음되는 음량 측정

AVAudioRecorder 인스턴스 생성 후 사용하기 위해서는 초기화가 필요합니다. 초기화 방법은 두 가지가 있습니다:

1. `init(url: URL, settings: [String : Any])`: dictionary 설정과 함께 초기화.
2. `init(url: URL, format: AVAudioFormat)`: AVAudioFormat을 사용하여 초기화.

`init(url:...)`에서 **URL은 녹음된 데이터를 로컬에 저장할 경로를 지정**합니다. 일반적으로 `FileManager.default.urls(for: .documentDirectory, in: .userDomainMask)` 또는 `.cachesDirectory` 등을 사용하여 앱의 샌드박스 내 디렉토리에 저장 경로를 구성합니다.

`settings` dictionary를 통해 오디오 녹음 시 관련 옵션을 전달합니다. 주요 설정 키는 다음과 같습니다:

- **AVFormatIDKey**: 오디오 데이터 형식을 지정합니다. 예시 값: `kAudioFormatMPEG4AAC` (높은 압축률/품질), `kAudioFormatLinearPCM` (무압축), `kAudioFormatAppleLossless` 등.
- **AVSampleRateKey**: 오디오 샘플링 레이트(초당 샘플링 횟수)를 지정합니다. 값이 높을수록 품질이 좋아지지만 파일 크기가 커집니다. 8 kHz (전화 품질)부터 192 kHz까지 설정 가능합니다.
- **AVNumberOfChannelsKey**: 오디오 데이터의 채널 수를 지정합니다. 1 (모노) 또는 2 (스테레오) 등으로 설정합니다.
- **AVEncoderAudioQualityKey**: 오디오 인코더의 품질 정도를 지정합니다. `AVAudioQuality` enum 값(min, low, medium, high, max)의 rawValue를 사용합니다.

녹음 제어 메서드:

- `record()`: 녹음 시작.
- `stop()`: 녹음 중단.
- `pause()`: 녹음 일시 중지.
- `resume()`: 일시 중지된 녹음 재개.
- `deleteRecording()`: 녹음된 오디오 파일 삭제.

AVAudioRecorder는 **AVAudioRecorderDelegate** 프로토콜을 통해 이벤트를 처리할 수 있습니다. 주요 Delegate 메서드:

- `audioRecorderDidFinishRecording(_:successfully:)`: 녹음 완료 시 호출.
- `audioRecorderEncodeErrorDidOccur(_:error:)`: 인코딩 오류 발생 시 호출.

녹음 중 오디오 볼륨(사운드 레벨)을 측정하려면 `isMeteringEnabled` 속성을 `true`로 설정하고, 주기적으로 `updateMeter()` 메서드를 호출합니다. `averagePower(forChannel:)` 및 `peakPower(forChannel:)` 메서드를 사용하여 오디오 신호의 평균 및 피크 파워를 데시벨(dB) 단위로 가져올 수 있습니다. 이 값은 일반적으로 -160 dB (거의 무음)에서 0 dB (최대 파워) 사이로 반환되며, 시각적으로 표현하기 위해 정규화 과정이 필요할 수 있습니다.

#### AVAudioPlayer

**AVAudioPlayer**는 **오디오 데이터를 파일 또는 버퍼에서 재생하는 객체**입니다. 간단한 로컬 오디오 파일 재생에는 `AVAudioPlayer`로도 충분합니다. `AVPlayer`가 더 고수준의 기능(비디오 재생 등)을 제공하는 반면, `AVAudioPlayer`는 오디오 파일/버퍼 재생에 특화되어 있습니다.

AVAudioPlayer 기능:

- 파일 또는 버퍼로부터 오디오 재생.
- 볼륨, 속도, 패닝, 반복 여부 설정.
- 오디오 볼륨 정보 실시간 가져오기 (시각적 표현 가능).

AVAudioPlayer는 `init(contentsOf: URL)` 또는 `init(data: Data)` 등을 사용하여 초기화합니다. 재생을 시작하려면 `play()` 메서드를 사용합니다.

재생 제어 메서드:

- `play()`: 재생 시작.
- `stop()`: 재생 중단.
- `pause()`: 재생 일시 중지.
- `prepareToPlay()`: 오디오 재생을 위한 준비 (예: 버퍼링).

AVAudioPlayer는 **AVAudioPlayerDelegate** 프로토콜을 통해 재생 완료 등의 이벤트를 처리할 수 있습니다. `audioPlayerDidFinishPlaying(_:successfully:)` 메서드는 오디오 재생이 성공적으로 완료되었을 때 호출됩니다.

AVAudioPlayer도 `updateMeter()`, `averagePower(forChannel:)`, `peakPower(forChannel:)` 메서드를 사용하여 오디오 레벨을 측정할 수 있습니다.

### 권한 설정 (마이크)

마이크와 같은 하드웨어에 접근하려면 사용자에게 권한을 요청해야 합니다.

1. **Info.plist 설정 (필수)**: 앱이 마이크 접근이 필요한 이유를 설명하는 메시지를 `Info.plist` 파일에 추가해야 합니다. Xcode 13부터는 프로젝트 타겟의 "Info" 탭에 있는 "Custom iOS Target Properties" (또는 유사 인터페이스)에서 설정합니다.
    
    - **Key**: `Privacy - Microphone Usage Description` (`NSMicrophoneUsageDescription`).
    - **Type**: `String`.
    - **Value**: 사용자에게 표시될 마이크 사용 목적에 대한 설명 메시지. 예: "음성 녹음을 위해 마이크 접근 권한이 필요합니다.".
2. **코드에서 권한 요청 및 상태 확인**: `AVFoundation` 프레임워크를 사용하여 마이크 권한 상태를 확인하고 필요 시 요청합니다.
    
    - `AVAudioSession.sharedInstance().recordPermission`으로 현재 권한 상태를 확인합니다 (.granted, .denied, .undetermined).
    - `AVAudioSession.sharedInstance().requestRecordPermission { granted in ... }` 메서드를 사용하여 사용자에게 권한 요청 팝업을 표시합니다. 이 메서드는 비동기적으로 실행되며, 결과는 메인 스레드에서 처리하는 것이 좋습니다.
    - 또는 `AVCaptureDevice.requestAccess(for: .audio)` 메서드를 사용할 수도 있습니다.

사용자 경험을 위해 마이크 기능이 **실제로 필요한 시점에 권한을 요청**하는 것이 좋으며, 사용자가 권한을 거부했을 경우 해당 기능을 사용할 수 없음을 알리고 설정으로 이동하도록 안내하는 UI를 제공해야 합니다.

### AVCaptureDevice

**AVCaptureDevice**는 카메라나 마이크와 같은 **하드웨어 또는 가상 캡처 장치를 나타내는 클래스**입니다. 이 클래스는 미디어 데이터를 AVCaptureSession에 연결된 캡처 세션 입력에 제공합니다. 음성 기능 구현 시 마이크 사용 권한 요청 및 확인에 사용될 수 있습니다.

### AVKit과 AVFoundation의 관계 (UI 관점)

**AVKit**은 AVFoundation을 기반으로 **고수준의 기능을 제공**합니다. 특히 AVKit의 **VideoPlayer** 뷰는 SwiftUI에서 비디오 재생을 위한 **표준 사용자 인터페이스 (재생/일시정지 버튼, 타임라인 슬라이더, 전체 화면 버튼 등)를 편리하게 제공**합니다. VideoPlayer는 인자로 AVPlayer를 받습니다.

만약 기본 제공 UI가 아닌 **커스텀한 플레이어를 만들려면**, AVKit의 VideoPlayer 대신 **AVFoundation의 요소들을 직접 사용하고 SwiftUI 등으로 UI를 커스텀하게 구현**해야 합니다. 즉, 사용자 인터페이스 커스터마이징 요구사항이 높다면 AVFoundation을 선택하는 것이 더 적절합니다.

### SwiftUI와의 연동

AVFoundation 객체를 SwiftUI 뷰와 연동하기 위해 **`ObservableObject` 프로토콜을 채택하는 클래스에 AVFoundation 객체를 래핑**하고, 뷰에서는 이 클래스의 인스턴스를 **`@StateObject` 또는 `@ObservedObject`로 주입**하여 사용합니다. `@Published` 속성을 사용하여 녹음 중 여부 (`isRecording`), 재생 중 여부 (`isPlaying`), 녹음 시간 (`recordingDuration`) 등 상태 변화를 뷰에 자동으로 반영할 수 있습니다.

예시: `AudioRecorder` 또는 `AudioRecorderManager` 클래스를 만들어 `AVAudioRecorder`, `AVAudioPlayer`, `AVAudioSession` 인스턴스를 관리하고 `@Published` 변수로 상태를 노출합니다. SwiftUI 뷰에서는 `@StateObject`로 이 관리자 객체를 생성하고, `@ObservedObject`로 하위 뷰에 전달하여 상태에 따라 UI를 업데이트하고 버튼 액션에서 관리자 객체의 메서드를 호출합니다.

뷰가 나타날 때 (`.onAppear`) 오디오 세션을 설정하는 로직을 추가하는 것이 일반적입니다.

### 앱 생명주기 및 백그라운드 오디오

**iOS 앱의 생명주기**는 앱이 실행되고 종료될 때 시스템이 발생시키는 이벤트에 따라 앱의 상태가 전환되는 과정입니다. 주요 상태는 **Not Running, Inactive, Active, Background - Running, Background - Suspended**가 있습니다.

앱이 **Background - Running 상태**일 때, 화면에 보이지 않지만 제한된 시간 동안 작업을 수행할 수 있으며, **음악 재생과 같은 백그라운드 동작이 가능**합니다. 앱 실행 후 홈 화면으로 나가면 Active -> Inactive -> Background 상태로 전환됩니다. 이 과정에서 `applicationWillResignActive(_:)` 및 `applicationDidEnterBackground(_:)` 등의 `AppDelegate` 메서드가 호출됩니다.

백그라운드에서 작업을 완료하기 위한 방법으로는 **Background Tasks** 및 **Background Modes** 설정이 있습니다. Background Modes는 앱이 백그라운드에서 실행 가능한 특정 기능을 설정하는 옵션입니다. Background Tasks API는 이러한 Background Modes로 승인된 기능을 구현하는 데 사용될 수 있습니다. iOS는 성능 문제로 백그라운드 실행 시간을 제한하며, `BackgroundTaskAPI`로 시간 제한을 늘릴 수 있지만 강제 종료될 수 있습니다. (소스에서는 AVFoundation을 사용한 백그라운드 오디오 작업의 구체적인 구현 방법을 상세히 다루지는 않습니다.)