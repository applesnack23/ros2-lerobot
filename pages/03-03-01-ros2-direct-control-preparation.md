# 준비

#### 가상 환경과 의존성 문제

Linux에서 Python이나 ROS2를 사용하다 보면 자주 만나게 되는 문제가 있습니다. 바로 의존성(Dependency) 문제입니다.

대부분의 프로그램은 모든 기능을 처음부터 직접 만드는 것이 아니라 다른 개발자가 만든 오픈 소스나 라이브러리를 활용합니다. 이 교재에서 사용하는 ROS2와 LeRobot도 다양한 외부 라이브러리를 기반으로 동작합니다.

외부 라이브러리를 사용하면 다음과 같은 장점이 있습니다.

- 개발 시간 단축
- 개발 비용 절감
- 검증된 기능 재사용
- 복잡한 기능의 빠른 적용

하지만 프로젝트마다 필요한 라이브러리와 버전이 다르기 때문에 충돌이 발생할 수 있습니다.

예를 들면 다음과 같습니다.

- LeRobot이 요구하는 Python 버전
- ROS2가 요구하는 Python 버전
- PyTorch가 요구하는 NumPy 버전
- 다른 라이브러리가 요구하는 OpenCV 버전

Python이나 JavaScript처럼 빠르게 발전하는 환경에서는 라이브러리의 업데이트 주기가 짧기 때문에 이러한 문제가 더욱 자주 발생합니다.

이 문제를 줄이기 위해 사용하는 것이 가상 환경(Virtual Environment)입니다.

---

#### ROS2와 LeRobot의 설치 방식

우리는 LeRobot을 설치할 때 `venv` 가상 환경을 사용했습니다. 반면 ROS2는 Ubuntu 시스템 전체에 설치했습니다.

두 프로그램의 설치 방식이 다른 이유는 각각의 목적과 구조가 다르기 때문입니다.

**ROS2**

ROS2는 다음과 같이 운영체제와 밀접하게 연결된 기능을 사용합니다.

- DDS 통신
- C++ 라이브러리
- 시스템 패키지
- 네트워크 미들웨어
- RViz와 같은 그래픽 도구
- Gazebo와 같은 시뮬레이터

따라서 ROS2는 일반적으로 Python 가상 환경 안에 설치하지 않고 Ubuntu 시스템에 한 번 설치하여 사용합니다.

```bash
/opt/ros/lyrical
```

---

**LeRobot**

LeRobot은 Python 기반의 AI·Robot Framework이며 다음과 같은 외부 라이브러리를 사용합니다.

- PyTorch
- NumPy
- OpenCV
- Transformers
- Datasets
- Hugging Face 관련 라이브러리
- Serial 통신 라이브러리

이러한 라이브러리는 버전 변화가 빠르고 프로젝트마다 필요한 버전이 다를 수 있습니다. 따라서 LeRobot은 별도의 가상 환경에 설치하는 것이 일반적입니다.

이번 실습에서는 다음과 같이 환경을 구성합니다.

| 구분 | 설치 위치 |
| --- | --- |
| ROS2 | Ubuntu 시스템 |
| LeRobot | Python 가상 환경 |
| ROS2 Workspace | `~/project/ros2_ws` |
| LeRobot 가상 환경 | `~/project/rosws/lerobot/venv` |

---

#### ROS2와 LeRobot의 연결 구조

이번 실습에서는 LeRobot 가상 환경에서 시스템에 설치된 ROS2를 참조합니다.

```
LeRobot 가상 환경
        ↓
시스템에 설치된 ROS2
```

반대로 시스템에 설치된 ROS2가 LeRobot 가상 환경을 자동으로 찾아서 사용하는 구조는 아닙니다.

```
시스템에 설치된 ROS2
  ✕
LeRobot 가상 환경 자동 참조
```

따라서 LeRobot 라이브러리와 ROS2 Python 라이브러리를 하나의 노드에서 사용하려면 다음 환경을 모두 활성화해야 합니다.

1. 시스템에 설치된 ROS2 환경
2. LeRobot 가상 환경
3. 직접 만든 ROS2 Workspace 환경

---

#### 실행 환경 활성화

새로운 터미널을 열었을 때 다음 순서로 환경을 활성화합니다.

```bash
# 1. 시스템 ROS2 환경 활성화
source /opt/ros/lyrical/setup.bash

# 2. LeRobot 가상 환경 활성화
source ~/project/rosws/lerobot/venv/bin/activate

# 3. 직접 만든 Workspace 환경 활성화
source ~/project/ros2_ws/install/setup.bash
```

세 번째 명령은 ROS2 Workspace를 한 번 이상 빌드한 후에 사용할 수 있습니다.

환경이 활성화되면 터미널에서 ROS2 명령과 LeRobot Python 라이브러리를 함께 사용할 수 있습니다.

---

#### 가상 환경에 추가 의존성 설치

시스템에 ROS2가 설치되어 있어도 ROS2가 사용하는 일부 Python 라이브러리가 LeRobot 가상 환경에는 없을 수 있습니다.

예를 들면 다음과 같은 라이브러리입니다.

- `empy`
- `lark`

필요한 모듈이 없다는 오류가 발생하면 LeRobot 가상 환경을 활성화한 상태에서 설치할 수 있습니다.

```bash
source ~/project/rosws/lerobot/venv/bin/activate

pip install empy
pip install lark
```

설치된 패키지는 현재 가상 환경에만 적용되며 시스템 전체나 다른 Python 프로젝트에는 영향을 주지 않습니다.

다만 ROS2 배포판에 따라 필요한 라이브러리 버전이 다를 수 있으므로 오류가 발생했을 때 요구되는 버전을 확인하여 설치하는 것이 안전합니다.

또한 ROS2와 가상 환경에서 사용하는 Python의 주요 버전이 호환되어야 합니다. Python 버전이 서로 다르면 ROS2 Python 모듈을 불러오는 과정에서 오류가 발생할 수 있습니다.

---

#### 환경 구성 정리

이번 실습 환경의 핵심은 다음과 같습니다.

- ROS2는 Ubuntu 시스템에 설치합니다.
- LeRobot은 Python 가상 환경에 설치합니다.
- LeRobot 가상 환경에서 시스템 ROS2 환경을 참조합니다.
- 필요한 ROS2 Python 의존성은 가상 환경에 추가로 설치할 수 있습니다.
- 가상 환경에 설치한 패키지는 다른 프로젝트에 영향을 주지 않습니다.
- 실행 전에는 ROS2, LeRobot 가상 환경, Workspace 환경을 모두 활성화합니다.

이러한 환경 구성은 앞으로 ROS2, AI, Vision, Robot 제어 기능을 하나의 시스템에서 함께 사용할 때 자주 사용됩니다.

---

#### ROS2 기반 SO-ARM101 제어

앞에서는 Feetech에서 제공하는 라이브러리를 Python 코드에서 직접 사용하여 SO-ARM101을 제어했습니다.

이번에는 같은 기능을 ROS2 패키지와 노드로 구성해보겠습니다.

다음 두 개의 패키지를 만듭니다.

- `so_arm101_driver`
- `so_arm101_pick_place`

두 패키지는 역할을 분리하여 구성합니다.

| 패키지 | 역할 |
| --- | --- |
| `so_arm101_driver` | 실제 모터와 ROS2 사이의 통신 |
| `so_arm101_pick_place` | 티칭, 구동 및 자동 Pick & Place |

---

#### 전체 제어 구조

```
so_arm101_pick_place
        │
        │ /so_arm101/joint_goal
        ▼
so_arm101_driver
        │
        │ Feetech 통신
        ▼
SO-ARM101 모터
        │
        │ 현재 위치
        ▼
so_arm101_driver
        │
        │ /joint_states
        ▼
so_arm101_pick_place
```

`so_arm101_driver`는 실제 하드웨어와 통신하고, `so_arm101_pick_place`는 Driver가 제공하는 Topic을 사용해 로봇 동작을 구성합니다.

---

#### so_arm101_driver 패키지

`so_arm101_driver`는 ROS2 명령과 실제 Feetech 모터 사이를 연결하는 Driver 패키지입니다.

주요 노드는 다음과 같습니다.

```
feetech_driver_node.py
```

---

**feetech_driver_node.py의 역할**

1. `/dev/ttyACM0` 포트에 연결합니다.
2. SO-ARM101의 Feetech 모터 6개를 등록합니다.
3. 각 모터의 현재 위치를 읽습니다.
4. 현재 관절 위치를 `/joint_states` Topic으로 발행합니다.
5. `/so_arm101/joint_goal` Topic을 구독합니다.
6. 수신한 목표 위치를 모터의 `Goal_Position`으로 전달합니다.

| 통신 방향 | Topic | 역할 |
| --- | --- | --- |
| Driver → 다른 노드 | `/joint_states` | 현재 관절 위치 발행 |
| 다른 노드 → Driver | `/so_arm101/joint_goal` | 목표 관절 위치 수신 |

Driver 패키지가 실제 하드웨어 제어를 담당하므로 Pick & Place 노드는 Feetech 통신 방식이나 Serial 포트를 직접 다룰 필요가 없습니다.

---

#### so_arm101_pick_place 패키지

`so_arm101_pick_place`는 Driver가 제공하는 Topic을 이용해 티칭과 Pick & Place 동작을 수행하는 패키지입니다.

다음 세 개의 노드를 작성합니다.

```
teaching_node.py
playback_node.py
auto_pick_place_node.py
```

**teaching_node.py**

현재 로봇의 관절 위치를 저장하는 노드입니다.

- `/joint_states` Topic 구독
- 현재 관절 위치 확인
- Teaching Point 저장
- Grip 및 Ungrip 위치 저장
- 저장한 데이터를 CSV 파일로 출력

**playback_node.py**

CSV 파일에 저장된 위치로 로봇을 이동시키는 노드입니다.

- CSV 파일 읽기
- 키보드 입력 처리
- Teaching Point 선택
- Grip 및 Ungrip 명령 선택
- `/so_arm101/joint_goal` Topic 발행

**auto_pick_place_node.py**

저장된 티칭 데이터를 이용해 Pick & Place 순서를 자동으로 실행하는 노드입니다.

- CSV 파일 읽기
- 필요한 Teaching Point 확인
- Pick & Place 동작 순서 구성
- `/so_arm101/joint_goal` Topic 발행
- 각 동작 사이의 대기 시간 처리
- 자동 동작 완료 후 안전하게 종료

---

#### Pick & Place 동작 흐름

자동 Pick & Place 노드는 다음 순서로 동작합니다.

```
대기 위치 이동
→ Gripper 열기
→ Pick 위치 이동
→ Gripper 닫기
→ 대기 위치 이동
→ Place 위치 이동
→ Gripper 열기
→ 대기 위치 이동
```

`auto_pick_place_node`는 목표 위치를 직접 모터에 전달하지 않습니다. `/so_arm101/joint_goal` Topic에 목표 관절 위치를 발행하고, `feetech_driver_node`가 해당 값을 받아 실제 모터를 움직입니다.

---

#### 패키지 구조

전체 Workspace는 다음과 같이 구성할 수 있습니다.

```
~/project/ros2_ws/
└── src/
    ├── so_arm101_driver/
    │   ├── package.xml
    │   ├── setup.py
    │   └── so_arm101_driver/
    │       ├── __init__.py
    │       └── feetech_driver_node.py
    │
    └── so_arm101_pick_place/
        ├── package.xml
        ├── setup.py
        └── so_arm101_pick_place/
            ├── __init__.py
            ├── teaching_node.py
            ├── playback_node.py
            └── auto_pick_place_node.py
```

---

#### 패키지 역할 정리

| 노드 | 입력 | 출력 | 주요 역할 |
| --- | --- | --- | --- |
| `feetech_driver_node` | `/so_arm101/joint_goal` | `/joint_states` | 실제 모터 제어 |
| `teaching_node` | `/joint_states` | CSV 파일 | 관절 위치 저장 |
| `playback_node` | CSV 및 키 입력 | `/so_arm101/joint_goal` | 저장 위치 수동 실행 |
| `auto_pick_place_node` | CSV 파일 | `/so_arm101/joint_goal` | Pick & Place 자동 실행 |

---

#### 마무리

이번 실습에서는 하드웨어 통신과 로봇 동작을 서로 다른 ROS2 패키지로 분리합니다.

`so_arm101_driver`는 실제 모터 연결과 위치 제어를 담당하고, `so_arm101_pick_place`는 Driver가 제공하는 Topic을 사용해 티칭과 Pick & Place 동작을 구현합니다.

이처럼 기능을 패키지와 노드 단위로 분리하면 하드웨어 제어 코드를 변경하지 않고도 Teaching, Playback, 자동화 기능을 독립적으로 개발하고 확장할 수 있습니다.