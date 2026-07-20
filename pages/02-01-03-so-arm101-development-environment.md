# 개발환경 구축

#### LeRobot 개발환경 구축

이번 장에서는 Hugging Face에서 제공하는 LeRobot을 설치하고 SO-ARM101을 제어하기 위한 개발환경을 구성합니다.

LeRobot은 Python 기반으로 개발되었으며, 다른 Python Project와 Package가 충돌하지 않도록 가상환경을 사용합니다.

개발환경은 다음 순서로 구성합니다.

1. Workspace 생성
2. LeRobot Source Code Download
3. Python 가상환경 생성
4. LeRobot 및 Feetech Driver 설치
5. USB Port와 권한 확인

---

#### Workspace 생성

이제 LeRobot 프로젝트를 저장할 작업 공간을 생성합니다.

```bash
cd ~
mkdir project/rosws
cd project/rosws
```

여기서 `rosws` 라는 이름의 폴더를 사용합니다.

이름은 자유롭게 정할 수 있지만, ROS 및 Robot 관련 프로젝트를 관리하기 위한 Workspace라는 의미로 `rosws` 또는 `robot_ws` 와 같은 이름을 사용하는 경우가 많습니다.

현재 디렉터리 구조는 다음과 같습니다.

```bash
/home/username/project/rosws
```

---

#### LeRobot 다운로드

다음으로 Hugging Face 공식 GitHub 저장소에서 LeRobot 소스코드를 다운로드 합니다.

```bash
git clone https://github.com/huggingface/lerobot.git
```

다운로드가 완료되면 다음과 같은 Directory가 생성됩니다.

```bash
project
└── rosws
    └── lerobot
```

이제 LeRobot 프로젝트 디렉터리로 이동합니다.

```bash
cd lerobot
```

---

#### Python 가상환경 생성

LeRobot 에서 사용할 Python 가상환경을 생성합니다.

```bash
python3 -m venv venv
```

명령의 의미는 다음과 같습니다.

```bash
python3      → Python3 실행
-m venv      → venv 모듈 실행
venv         → venv 라는 이름의 가상환경 생성
```

명령 실행 후 다음과 같은 디렉터리가 생성됩니다.

```bash
lerobot
├── lerobot
├── examples
├── tests
├── pyproject.toml
└── venv
```

---

#### 가상환경 활성화

생성한 가상환경을 활성화합니다.

```bash
source venv/bin/activate
```

가상환경이 활성화되면 Terminal Prompt 앞에 `(venv)` 가 표시됩니다.

```bash
(venv) username@lt:~/project/rosws/lerobot$
```

이 상태에서 `pip`로 설치하는 Python Package는 현재 가상환경 안에 설치됩니다.

작업을 종료한 후 가상환경에서 빠져나오려면 다음 명령을 사용합니다.

```bash
deactivate
```

---

#### pip 업데이트

패키지 설치 과정에서 발생할 수 있는 문제를 줄이기 위해 가상환경의 `pip`를 Update합니다.

```bash
sudo apt update

pip install --upgrade pip
```

---

#### LeRobot 및 Feetech Driver 설치

SO-ARM101은 Feetech Bus Servo를 사용합니다.

LeRobot과 Feetech 관련 Package를 함께 설치합니다.

```bash
pip install -e ".[feetech]"
```

`-e`는 Editable Install을 의미합니다.

Editable 방식으로 설치하면 Download한 LeRobot Source Code를 수정했을 때 Package를 다시 설치하지 않아도 변경 내용이 반영됩니다.

---

#### 연결된 포트 확인

USB를 통해 연결된 Motor Controller의 Port를 확인합니다.

```bash
lerobot-find-port
```

실행 결과 예시는 다음과 같습니다.

```bash
/dev/ttyACM0
```

여러 개의 USB 장치가 연결되어 있다면 안내에 따라 USB Cable을 분리하고 다시 연결하면서 각 Motor Controller의 Port를 확인합니다.

예를 들어 다음과 같이 구분할 수 있습니다.

```python
Follower Arm → /dev/ttyACM0
Leader Arm   → /dev/ttyACM1
```

USB Port 이름은 Computer의 연결 상태와 실행 환경에 따라 달라질 수 있습니다.

---

#### USB 권한 확인

Linux에서는 USB Serial 장치의 접근 권한을 Group 단위로 관리합니다.

장치의 권한을 확인합니다.

```bash
ls -l /dev/ttyACM0
```

예시 출력:

```bash
crw-rw---- 1 root dialout 166, 0 ... /dev/ttyACM0
```

여기서 `dialout`은 해당 Device에 접근할 수 있는 Group입니다.

현재 사용자가 속한 Group을 확인합니다.

```bash
groups
```

현재 사용자가 dialout Group에 포함되어 있지 않다면 다음 명령을 실행해 현재 로그인한 사용자를 dialout 그룹에 추가합니다.

```bash
sudo usermod -a -G dialout $USER
```

옵션 의미는 다음과 같습니다.

| 옵션 | 의미 |
| --- | --- |
| -a | 기존 그룹 유지 후 추가 |
| -G | 추가할 그룹 지정 |

Group 변경 사항은 다시 로그인한 후 적용됩니다. 가장 간단한 방법은 System을 재부팅하는 것입니다.

```bash
reboot
```

---

#### 최종 디렉터리 구조

모든 과정이 완료되면 다음과 같은 구조가 만들어 집니다.

```bash
~/project
└── rosws
    └── lerobot
        ├── lerobot
        ├── examples
        ├── tests
        ├── pyproject.toml
        └── venv
```

---
