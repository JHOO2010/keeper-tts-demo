# Android에서 테스트하기

이 문서는 앞으로 Android용 테스트 앱을 만들 때 따라갈 순서입니다. 현재 저장소에는 Android 프로젝트나 APK가 없고, Android 실기기에서 음성을 생성해 본 상태도 아닙니다.

## 먼저 확인할 것

테스트할 실제 키오스크의 Android 버전, CPU 종류, RAM과 남은 저장 공간을 확인해 둡니다. 같은 Android라도 기기마다 생성 속도와 사용할 수 있는 메모리가 다릅니다.

개발 PC에는 Android Studio와 Android SDK Platform-Tools가 필요합니다. 모델은 Windows에서 설치한 `models/supertonic-3` 폴더를 사용하면 됩니다. 모델과 임시 파일이 차지할 공간도 따로 확보하세요.

기기에서 USB 디버깅을 켜고 PC에 연결한 뒤, 기기에 표시되는 연결 허용을 직접 확인합니다. 이후 PC 터미널에서 확인합니다.

```powershell
adb devices
adb shell getprop ro.build.version.release
adb shell getprop ro.product.cpu.abilist
```

`adb`를 찾지 못하면 Android SDK의 `platform-tools` 폴더에서 실행하거나 PATH에 추가하세요. `unauthorized`가 나오면 기기의 USB 디버깅 허용 화면을 확인합니다. 여러 기기를 연결했다면 `adb -s 기기번호 ...`로 대상을 지정하세요.

## 1. 스피커부터 확인하기

Windows에서 음성 파일을 하나 만듭니다.

```powershell
.\run-voice.ps1 -Text "안녕하세요. 주문을 도와드리겠습니다." -Output outputs/android-speaker.wav
adb push outputs/android-speaker.wav /sdcard/Download/
```

키오스크의 파일 앱에서 WAV를 열어 소리가 나오는지 확인합니다. 스피커, 볼륨, USB 오디오 같은 기본 문제를 먼저 확인하는 단계입니다. **이것만으로 Android에서 모델이 돌아간 것은 아닙니다.**

## 2. 작은 테스트 앱 만들기

처음부터 Keeper 전체에 붙이기보다, 문장 입력·생성 버튼·재생 버튼·소요 시간만 있는 앱으로 시작하면 원인을 찾기 쉽습니다.

구현 방향은 **Kotlin + ONNX Runtime Android + CPU 실행**입니다. 앱에 넣을 라이브러리는 `com.microsoft.onnxruntime:onnxruntime-android`이며, 사용할 버전을 정해 Gradle에 고정합니다. Windows의 Python용 패키지를 Android 앱에 복사해서 쓰는 방식은 아닙니다.

모델을 앱의 assets에 포함하는 경우에는 아래처럼 준비할 수 있습니다. 이 경로는 앞으로 만들 Android 프로젝트의 예시입니다.

```text
app/src/main/assets/supertonic-3/
├── onnx/
│   ├── duration_predictor.onnx
│   ├── text_encoder.onnx
│   ├── vector_estimator.onnx
│   ├── vocoder.onnx
│   ├── tts.json
│   └── unicode_indexer.json
└── voice_styles/
    └── F1.json
```

다른 목소리도 쓸 거라면 해당 JSON을 함께 넣습니다. 모델의 라이선스 파일도 배포본에 포함하세요.

기기에서 모델 경로로 세션을 열려면 첫 실행 때 assets의 파일을 앱 내부 저장소로 복사한 후 사용할 수 있습니다. 이미 복사한 파일은 매번 다시 복사하지 않도록 하고, 모델 버전이 바뀔 때만 교체합니다. USB로 모델을 별도 배포한다면 앱이 그 파일을 내부 저장소로 가져오는 기능도 필요합니다.

APK 크기와 설치·복사 중의 임시 저장 공간까지 확인해야 합니다. 모든 Android 기기에서 설치된다고 보장하는 사양은 아직 정하지 않았습니다.

## 3. 음성 생성 연결하기

처리 순서는 아래와 같습니다.

```text
입력 문장
→ 한국어 태그와 Unicode 전처리
→ duration_predictor / text_encoder
→ vector_estimator 반복 실행
→ vocoder
→ PCM 음성
→ AudioTrack 재생 또는 WAV 저장
```

ONNX 파일 하나만 여는 것으로 끝나지 않습니다. 공식 Java 예제를 참고해 텍스트 전처리, 네 모델의 입출력, 목소리 데이터 로딩, 반복 추론을 함께 옮겨야 합니다. 한국어 언어 코드는 `ko`로 맞춥니다.

시작할 때는 Windows와 동일하게 F1, 속도 1.0, 추론 5단계로 비교하세요. 출력 샘플레이트는 모델 설정에서 읽고 재생 쪽에도 동일하게 설정합니다.

모델 세션은 앱에서 한 번 로딩해 재사용합니다. 모델 로딩과 음성 생성은 UI 스레드 밖에서 실행하고, 연속 요청이 들어와도 메모리가 계속 늘지 않도록 세션과 텐서 수명을 관리합니다.

## 4. 실제 기기에서 실행하기

Android 프로젝트를 만든 다음 Android Studio에서 연결된 키오스크를 선택해 Run으로 설치할 수 있습니다.

Gradle wrapper가 있는 표준 Android 프로젝트라면 PC 터미널에서도 빌드·설치할 수 있습니다. **아래 명령은 Android 프로젝트 폴더에서 실행하는 예시이며, 현재 이 저장소 루트에서는 실행되지 않습니다.**

```powershell
.\gradlew.bat assembleDebug
adb install -r app/build/outputs/apk/debug/app-debug.apk
```

설치 후 앱을 열어 입력한 문장으로 새로운 음성을 만들어 봅니다. 미리 넣은 WAV를 재생하는 대신, 지금 입력한 문장이 실제로 들리는지 확인합니다.

## 5. 인터넷을 끊고 다시 확인하기

모델을 포함한 APK나 필요한 파일을 USB로 준비한 뒤 다음을 확인합니다.

1. Wi-Fi, 모바일 데이터, 유선망을 모두 끕니다. 비행기 모드에서도 Wi-Fi가 다시 켜져 있지 않은지 확인합니다.
2. 앱을 완전히 종료하고 다시 엽니다.
3. 이전에 생성하지 않았던 문장을 입력해 음성을 만듭니다.
4. 기기를 재부팅한 뒤 같은 테스트를 반복합니다.
5. 개발 PC와의 USB 연결도 끊고 단독으로 실행합니다.

최초 설치부터 인터넷 없이 써야 한다면, 깨끗한 테스트 설치에서도 모든 모델 파일이 APK 또는 별도 설치 매체만으로 준비되는지 확인해야 합니다. 개발 중 받아둔 캐시 덕분에 작동하는 경우와 구분하세요.

## 6. 확인할 문장과 기록

| 확인할 것 | 예시 |
| --- | --- |
| 짧은 안내 | 안녕하세요. 주문을 도와드리겠습니다. |
| 결제 안내 | 결제가 완료되었습니다. 카드를 챙겨 주세요. |
| 숫자와 가격 | 아메리카노 두 잔, 총 8,000원입니다. |
| 메뉴 이름과 영문 | ICE 아메리카노와 디카페인 라테를 선택하셨습니다. |
| 긴 안내 | 문장을 여러 개 이어서 입력하고 끊김이나 누락이 있는지 확인합니다. |

발음은 직접 들어서 확인하고, 숫자나 영문 표기가 어색하면 Keeper에서 읽기 좋은 문장으로 바꾸는 처리도 검토합니다.

기기별로 아래 내용을 남겨두면 Windows와 비교하기 좋습니다.

- 기기 모델, Android 버전, CPU, RAM
- 모델 로딩 시간과 음성 생성 시간
- 생성한 음성 길이와 사용한 목소리·속도·추론 단계
- 앱을 처음 켰을 때와 두 번째 요청의 차이
- 연속 30회 실행했을 때 멈춤, 발열, 메모리 증가 여부
- 네트워크 차단 후 재실행·재부팅 결과

지금 Windows에서 측정한 속도가 Android에서도 그대로 나오는 것은 아닙니다. 에뮬레이터는 기능 확인에 쓰고, 속도와 메모리는 실제 키오스크에서 판단하세요.

## 참고 자료

- [Supertonic Java 예제](https://github.com/supertone-oss-archive/supertonic/tree/main/java): 전처리와 모델 실행 흐름을 참고할 코드입니다. Android 완성 앱은 아닙니다.
- [ONNX Runtime Android 예제](https://onnxruntime.ai/docs/tutorials/mobile/deploy-android.html): Android 프로젝트에 런타임을 연결하는 방법입니다. 이미지 모델 예제이므로 TTS 처리는 별도로 구현해야 합니다.
- [Android 실제 기기 연결](https://developer.android.com/studio/run/device)
- [ADB 사용법](https://developer.android.com/tools/adb)
- [AudioTrack](https://developer.android.com/reference/android/media/AudioTrack)
