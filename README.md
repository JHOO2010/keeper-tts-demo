# Keeper TTS

Keeper 키오스크 도우미에 붙이기 전에, 한국어 음성이 잘 나오는지 터미널에서 테스트하는 코드입니다.

Supertonic 3를 사용합니다. GPU 없이 CPU로 돌아가고, 처음에 모델을 받아두면 인터넷 없이 쓸 수 있습니다. 지금은 Windows에서 실행을 확인한 상태입니다.

## 처음 실행할 때

Windows 64비트와 Python 3.13이 필요합니다. PowerShell을 열고 아래 명령을 순서대로 입력하세요.

```powershell
git clone https://github.com/JHOO2010/keeper-tts-demo.git
cd keeper-tts-demo
.\setup-tts.ps1
```

필요한 패키지와 모델을 설치합니다. 모델은 약 400MB라 처음에는 조금 기다려야 합니다. 설치가 끝나면 테스트 음성도 하나 만들어집니다.

## 문장 읽어보기

```powershell
.\run-voice.ps1 -Text "안녕하세요. 주문을 도와드리겠습니다." -Play
```

`-Play`를 붙이면 만든 음성을 바로 들을 수 있습니다. 빼면 파일만 저장됩니다.

목소리나 속도를 바꾸고 싶으면 이렇게 실행하면 됩니다.

```powershell
.\run-voice.ps1 -Text "결제가 완료되었습니다." -Voice M1 -Speed 0.9 -Play
```

- `-Voice`: F1~F5, M1~M5 중 선택. 기본값은 F1입니다.
- `-Speed`: 1이 기본 속도입니다. 작게 하면 느려지고, 크게 하면 빨라집니다. 0.7~2 사이로 설정할 수 있습니다.
- `-Output`: 저장할 파일 이름을 직접 정할 때 씁니다.

```powershell
.\run-voice.ps1 -Text "이용해 주셔서 감사합니다." -Output outputs/thanks.wav -Play
```

파일 이름을 따로 정하지 않으면 실행 시간으로 이름을 붙여 `outputs` 폴더에 저장합니다. 같은 이름의 파일이 있으면 덮어쓰지 않습니다.

터미널에서 음성 생성에 걸린 시간과 저장 위치를 확인할 수 있고, 측정 결과는 WAV 옆의 JSON 파일에도 남습니다.

## 다른 PC로 옮길 때

이 저장소에는 코드만 들어 있습니다. 가상환경, 모델, 생성된 음성은 포함하지 않았습니다.

인터넷이 되는 PC라면 `setup-tts.ps1`을 다시 실행하면 됩니다. 인터넷이 없는 PC에는 모델 폴더(`models/supertonic-3`)와 그 PC에 맞게 준비한 Python 실행 환경을 함께 옮겨야 합니다. 설치 후 일반 실행에서는 모델을 다시 다운로드하지 않습니다.

Android는 아직 실기기에서 테스트하지 않았습니다. 현재 스크립트는 Windows용이고, Android 연동은 별도로 작업해야 합니다.

## 사용한 모델

- [Supertonic 3 모델](https://huggingface.co/Supertone/supertonic-3)
- [Supertonic Python 라이브러리](https://github.com/supertone-inc/supertonic-py)

모델 라이선스는 OpenRAIL-M입니다. 자세한 조건은 내려받은 모델 폴더의 라이선스 파일에서 확인할 수 있습니다.
