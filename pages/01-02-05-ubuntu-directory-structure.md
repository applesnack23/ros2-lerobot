# Ubuntu 기본 폴더 구조

Linux는 Windows와 달리 하나의 최상위 디렉터리(`/`) 아래에 모든 파일과 장치가 연결되는 구조를 가지고 있습니다.

가장 먼저 알아야 할 것은 **Home 디렉터리**와 **Root 디렉터리**의 차이입니다.

```bash
# Home Folder
username@lt:~$ 

# Root Folder
username@lt:/$
```

여기서 `~`는 현재 사용자의 Home 디렉터리를 의미합니다.

예를 들어 사용자 이름이 `username` 라면:

```bash
username@lt:~$ = username@lt:/home/username$
```

와 같은 의미입니다. (~ = /home/username)

반면 `/` 는 Linux 시스템의 최상위 디렉터리인 Root 디렉터리를 의미합니다.

Windows의 경우:

```bash
C:\
D:\
E:\
```

처럼 여러 개의 드라이브가 존재하지만,

Linux는 모든 장치와 저장장치를 하나의 디렉터리 구조 아래에서 관리합니다.

---

#### 자주 사용되는 폴더들

Root 디렉터리(`/`) 안에는 매우 많은 폴더가 존재합니다.

하지만 일반적인 ROS2 개발 환경에서 자주 보게 되는 폴더는 아래 정도입니다.

| **No** | 폴더 | 설명 |
| --- | --- | --- |
| 1 | `home` | 일반 사용자 계정과 사용자 데이터가 저장되는 폴더 |
| 2 | `media` | USB, 외장하드 같은 외부 저장장치가 자동으로 마운트되는 위치
윈도우와 비교하면 E드라이브 등인데, 리눅스에서는 파일로 관리를 한다. |
| 3 | `mnt` | 수동으로 저장장치를 임시 마운트할 때 사용하는 폴더 |
| 4 | `dev` | 하드웨어 장치를 나타내는 가상 파일 시스템(sda, I2C, UART, GPIO 등) |
| 5 | `proc` | 커널과 현재 실행 중인 프로세스 정보를 담고 있는 가상 파일 시스템 |
| 6 | `sys` | 하드웨어 상태와 커널 상태를 보여주는 폴더, 저수준 하드웨어 제어에 사용 |
| 7 | `opt` | 서드파이 프로그램 설치 위치 (예: Visual Studio Code) |
| 8 | `var` | 로그, 캐시, 데이터베이스 같은 자주 변경되는 데이터 저장 폴더 |
| 9 | `tmp` | 임시 파일 저장 폴더 (재부팅하면 대부분 삭제됨) |

**media**

USB 메모리나 외장하드를 연결하면 대부분 이 위치에 자동으로 마운트됩니다.

예를 들어:

```bash
/media/username/USB
```

Windows의 `E:` 드라이브와 비슷한 역할이라고 생각하면 이해하기 쉽습니다.

다만 Linux에서는 별도의 드라이브 문자가 존재하지 않고, 디렉터리 구조 안에 연결되는 형태로 동작합니다.

---

**dev**

ROS2와 로봇 개발을 시작하면 가장 자주 보게 되는 폴더 중 하나입니다.

예를 들어:

```bash
/dev/ttyUSB0
/dev/ttyACM0
/deb/video0
```

등과 같은 장치 파일이 생성됩니다.

- USB Serial 장치
- Arduino
- Dynamixel Controller
- USB Camera

등이 연결되면 대부분 이 위치에서 확인할 수 있습니다.

Linux 에서는 이렇게 대부분의 연결을 파일과 동일하게 다루는 특징이 있습니다.

---

**opt**

ROS2를 설치하면 가장 먼제 보게 되는 폴더 중 하나입니다.

예를 들어 ROS2 Lyrical을 설치하면:

```bash
/opt/ros/lyrical
```

위치에 ROS2가 설치됩니다.

ROS2 환경 설정 시 자주 사용하게 되는 명령 역시 이 경로를 사용합니다.

```bash
source /opt/ros/lyrical/setup.bash
```

---

#### 어디에서 작업해야 할까?

물론 Linux에는 매우 많은 디렉터리가 존재하지만,

우리가 ROS2 프로젝트나 Python 프로젝트를 만들 때는 대부분 **home 디렉터리** 안에서 작업하게 됩니다.

예를 들어:

```bash
/home/username/Project
/home/username/ros2_ws
/home/username/so_arm101_ws
```

와 같은 구조입니다.

필자 역시 대부분의 프로젝트를home 디렉터리 안에서 관리하고 있습니다.

---

개인적으로는 프로젝트 성격을 구분할 수 있도록 별도의 폴더를 만들어 관리하는 것을 추천합니다.

예를 들어:

```bash
/home/twiniex/Project
├── Python
├── ROS2
├── Unreal
├── Unity
└── Docker
```

또는:

```bash
/home/twiniex/Project
├── ros2_ws
├── so_arm101_ws
└── lerobot_ws
```

와 같이 관리하면 프로젝트가 많아지더라도 정리하기 쉽습니다.

---

앞으로 이 책에서도 대부분의 작업은 home 디렉터리 안에서 진행하게 됩니다.

따라서 지금 단계에서는 **home 디렉터리와 Root 디렉터리의 차이만 이해해도 충분합니다.**
