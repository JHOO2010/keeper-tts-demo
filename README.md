# Keeper 한국어 TTS 터미널 데모

현재 작업 범위는 모델 단독 실행 확인이다. 키오스크 UI나 APK를 만들지 않는다.
모델은 Supertonic 3, 실행 엔진은 ONNX Runtime CPU다. GPU, WSL, 외부 API가 필요 없다.

## 설치 및 실행

Windows 64비트 및 Python 3.13에서 확인했습니다. PowerShell에서 실행하세요.
최초 설치에는 인터넷과 모델 저장 공간 약 400MB가 필요합니다.

```powershell
git clone https://github.com/JHOO2010/keeper-tts-demo.git
cd keeper-tts-demo
.\setup-tts.ps1
```

설치 후에는 인터넷 없이 실행할 수 있습니다.

```powershell
.\run-voice.ps1 -Text '안녕하세요. 주문을 도와드리겠습니다.' -Play
.\run-voice.ps1 -Text '결제가 완료되었습니다.' -Voice M1
```

모델을 `models/supertonic-3`에서 읽고, WAV와 측정 결과 JSON을 `outputs`에 저장한다.
실행 중 네트워크 연결을 차단하고 자동 다운로드를 끈다.
터미널에는 모델 로딩 시간, 음성 생성 시간, 재생 길이, WAV 경로가 표시된다.
출력 파일은 기존 파일을 덮어쓰지 않는다.

새 개발 환경 초기 설치는 Python이 있는 상태에서 `setup-tts.ps1`을 실행한다.
패키지와 모델 다운로드는 이 설치 단계에만 인터넷이 필요하다.
설치가 완료된 PC에서는 `run-voice.ps1`만 실행하면 된다.

가상환경, 모델 가중치, 생성 음성은 Git에 포함하지 않는다.
오프라인 기기로 옮길 때는 별도로 설치한 실행 환경과 `models/supertonic-3`도 함께 준비해야 한다.

## 범위 및 출처
Windows에서 모델 단독 실행을 검증한다. Android 실기기 실행은 아직 검증하지 않았다.
향후 Android에서는 동일 모델을 ONNX Runtime Android 또는 웹 런타임으로 이식할 수 있으나
이 Python 스크립트 자체가 Android 배포본은 아니다.
모델은 OpenRAIL-M, 예제 코드는 MIT이며 라이선스 파일은 모델 폴더에 포함된다.

- https://huggingface.co/Supertone/supertonic-3
- https://github.com/supertone-inc/supertonic-py

이번 데모의 진입점은 `run-voice.ps1`이다.
