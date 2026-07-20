# Leader 추격

이번에는 Leader Arm을 손으로 움직이면 Follower Arm이 실시간으로 따라 움직이는 Program을 만들어 보겠습니다.

### Prompt 7

```
앞에서 저장한 다음 두 CSV 파일을 이용하여
SO-ARM101 Teleoperation Program을 만들어주세요.

leader_arm_min_max.csv
follower_arm_min_max.csv

연결된 /dev/ttyACM*와 /dev/ttyUSB* Port를 검색하고,
Leader Arm과 Follower Arm의 Port를 각각 선택할 수 있게 해주세요.

Leader Arm은 사용자가 손으로 움직여야 하므로
Torque를 비활성화해주세요.

Follower Arm은 목표 위치로 움직여야 하므로
Torque를 활성화해주세요.

Leader Arm의 6개 Motor Present_Position을 반복해서 읽고,
Leader의 Min/Max 범위에서 현재 위치 비율을 계산해주세요.

계산된 비율을 Follower의 Min/Max 범위에 적용하여
Follower Motor의 Goal_Position으로 전달해주세요.

Leader와 Follower의 현재 위치와 변환된 목표 위치를
Terminal 화면에 표시해주세요.

Enter 또는 Ctrl+C를 누르면 동작을 종료하고,
Follower Torque를 비활성화한 후
두 Motor Bus의 연결을 모두 해제해주세요.
```

이 Prompt의 목적은 Leader Arm의 현재 위치를 읽어 Follower Arm의 위치 범위에 맞게 변환하고, Follower Arm이 Leader Arm의 움직임을 실시간으로 따라가도록 만드는 것입니다.

전체 Source Code는 다음 GitHub에서 확인할 수 있습니다.

> GitHub Link: [https://github.com/applesnack23/ros2-lerobot-code/blob/main/chapter2/leader_follower.py](https://github.com/applesnack23/ros2-lerobot-code/blob/main/chapter2/leader_follower.py)
> 

전체 Source Code를 `leader_follower.py`라는 이름으로 저장합니다.

```python
from lerobot.motors.feetech.feetech import FeetechMotorsBus
from lerobot.motors.motors_bus import Motor

import time
import os
import sys
import glob
import csv
import select

MOTOR_NAMES = [
    "shoulder_pan",
    "shoulder_lift",
    "elbow_flex",
    "wrist_flex",
    "wrist_roll",
    "gripper",
]

motors = {
    "shoulder_pan": Motor(id=1, model="sts3215", norm_mode="position"),
    "shoulder_lift": Motor(id=2, model="sts3215", norm_mode="position"),
    "elbow_flex": Motor(id=3, model="sts3215", norm_mode="position"),
    "wrist_flex": Motor(id=4, model="sts3215", norm_mode="position"),
    "wrist_roll": Motor(id=5, model="sts3215", norm_mode="position"),
    "gripper": Motor(id=6, model="sts3215", norm_mode="position"),
}

def select_port(title):
    ports = sorted(glob.glob("/dev/ttyACM*") + glob.glob("/dev/ttyUSB*"))

    if not ports:
        print("No serial ports found.")
        sys.exit(1)

    print(f"\nSelect {title} port")
    print("-" * 40)

    for i, port in enumerate(ports, start=1):
        print(f"{i}. {port}")

    print("-" * 40)

    while True:
        choice = input("Select port number: ").strip()

        if choice.isdigit():
            index = int(choice) - 1
            if 0 <= index < len(ports):
                return ports[index]

        print("Invalid selection. Try again.")

def load_min_max(filename):
    data = {}

    with open(filename, "r") as f:
        reader = csv.DictReader(f)

        for row in reader:
            motor = row["motor"]
            data[motor] = {
                "min": int(row["min"]),
                "max": int(row["max"]),
            }

    return data

def clamp(value, min_value, max_value):
    return max(min_value, min(max_value, value))

def map_position(leader_pos, leader_min, leader_max, follower_min, follower_max):
    if leader_max == leader_min:
        return follower_min

    ratio = (leader_pos - leader_min) / (leader_max - leader_min)
    ratio = clamp(ratio, 0.0, 1.0)

    follower_pos = follower_min + ratio * (follower_max - follower_min)
    return int(follower_pos)

def enter_pressed():
    return select.select([sys.stdin], [], [], 0)[0]

leader_range = load_min_max("leader_arm_min_max.csv")
follower_range = load_min_max("follower_arm_min_max.csv")

leader_port = select_port("LEADER ARM")
follower_port = select_port("FOLLOWER ARM")

if leader_port == follower_port:
    print("Leader and follower ports cannot be the same.")
    sys.exit(1)

leader_bus = FeetechMotorsBus(
    port=leader_port,
    motors=motors
)

follower_bus = FeetechMotorsBus(
    port=follower_port,
    motors=motors
)

try:
    leader_bus.connect()
    follower_bus.connect()

    # leader arm은 손으로 움직여야 하므로 torque OFF
    leader_bus.disable_torque()

    # follower arm은 명령을 받아 움직여야 하므로 torque ON
    follower_bus.enable_torque()

    print("\nLeader-Follower sync started.")
    print("Move the leader arm by hand.")
    print("Press Enter to stop.")
    time.sleep(1)

    while True:
        if enter_pressed():
            sys.stdin.readline()
            break

        os.system("clear")

        print("SO-ARM101 Leader → Follower Sync")
        print(f"Leader Port  : {leader_port}")
        print(f"Follower Port: {follower_port}")
        print("-" * 75)
        print(f"{'Motor':<8} {'Leader':>10} {'Target':>10} {'L-Min':>10} {'L-Max':>10} {'F-Min':>10} {'F-Max':>10}")
        print("-" * 75)

        for name in MOTOR_NAMES:
            try:
                leader_pos = leader_bus.read(
                    "Present_Position",
                    name,
                    normalize=False
                )

                if hasattr(leader_pos, "item"):
                    leader_pos = leader_pos.item()

                leader_pos = int(leader_pos)

                l_min = leader_range[name]["min"]
                l_max = leader_range[name]["max"]
                f_min = follower_range[name]["min"]
                f_max = follower_range[name]["max"]

                target_pos = map_position(
                    leader_pos,
                    l_min,
                    l_max,
                    f_min,
                    f_max
                )

                follower_bus.write(
                    "Goal_Position",
                    name,
                    target_pos,
                    normalize=False
                )

                print(
                    f"{name:<8} "
                    f"{leader_pos:>10} "
                    f"{target_pos:>10} "
                    f"{l_min:>10} "
                    f"{l_max:>10} "
                    f"{f_min:>10} "
                    f"{f_max:>10}"
                )

            except Exception as e:
                print(f"{name:<8} ERROR: {e}")

        print("-" * 75)
        print("Press Enter to stop.")

        time.sleep(0.05)

except KeyboardInterrupt:
    print("\nStopped by Ctrl + C.")

finally:
    try:
        follower_bus.disable_torque()
    except:
        pass

    try:
        leader_bus.disconnect()
    except:
        pass

    try:
        follower_bus.disconnect()
    except:
        pass

    print("Disconnected.")
```

Leader 추격 Program에서는 앞에서 저장한 다음 두 파일을 사용합니다.

```bash
leader_arm_min_max.csv
follower_arm_min_max.csv
```

두 파일은 leader_follower.py와 같은 폴더에 있어야 합니다.

```bash
~/project/rosws/control
├── position_monitor.py
├── leader_follower.py
├── leader_arm_min_max.csv
└── follower_arm_min_max.csv
```

---

#### 위치 범위 불러오기

먼저 Leader Arm과 Follower Arm의 위치 범위를 CSV 파일에서 불러옵니다.

```python
leader_range = load_min_max(
    "leader_arm_min_max.csv"
)

follower_range = load_min_max(
    "follower_arm_min_max.csv"
)
```

각 Motor의 최소값과 최대값은 다음과 같은 Dictionary 형태로 변환됩니다.

```python
{
    "m1": {
        "min": 1024,
        "max": 3072
    }
}
```

이 위치 범위를 이용하여 Leader Arm의 현재 위치를 Follower Arm의 목표 위치로 변환합니다.

---

#### Leader와 Follower Port 선택

Program을 실행하면 먼저 Leader Arm과 Follower Arm의 Serial Port를 차례대로 선택합니다.

```bash
Select LEADER ARM port
----------------------------------------
1. /dev/ttyACM0
2. /dev/ttyACM1
----------------------------------------
Select port number:
```

Leader ARm의 Port를 선택한 후 Follower Arm의 Port를 선택합니다.

두 Arm에 같은 Port를 선택하면 Program을 실행할 수 없습니다.

```python
if leader_port == follower_port:
    print("Leader and follower ports cannot be the same.")
    sys.exit(1)
```

예를 들어 다음과 같이 구성할 수 있습니다.

```python
Leader Arm   → /dev/ttyACM0
Follower Arm → /dev/ttyACM1
```

Port 번호는 USB 연결 순서와 Computer 환경에 따라 달라질 수 있습니다.

---

#### Leader와 Follower Bus

Leader Arm과 Follower Arm은 각각 별도의 Motor Bus 객체를 사용합니다.

```python
leader_bus = FeetechMotorsBus(
    port=leader_port,
    motors=motors
)

follower_bus = FeetechMotorsBus(
    port=follower_port,
    motors=motors
)
```

두 Bus에 연결한 후 각 Arm의 역할에 맞게 Torque를 설정합니다.

```python
leader_bus.connect()
follower_bus.connect()

leader_bus.disable_torque()
follower_bus.enable_torque()
```

Leader Arm은 사용자가 손으로 움직여야 하므로 Torque를 비활성화합니다.

Follower Arm은 전달받은 목표 위치로 이동해야 하므로 Torque를 활성화합니다.

| Robot | 역할 | Torque |
| --- | --- | --- |
| Leader Arm | 사용자가 손으로 조작 | Disable |
| Follower Arm | Leader 위치를 따라 이동 | Enable |

---

#### 위치 변환

Leader Arm과 Follower Arm은 Motor의 기어비와 위치 범위가 다를 수 있으므로 Leader 위치를 그대로 전달하지 않고 두 Arm의 범위를 기준으로 변환합니다.

위치 변환 과정은 다음과 같습니다.

```python
Leader 현재 위치
→ Leader 범위에서의 비율 계산
→ Follower 범위에 같은 비율 적용
→ Follower 목표 위치 계산
```

예를 들어 Leader Arm의 위치 범위가 다음과 같다고 가정하겠습니다.

```python
Leader Min = 1000
Leader Max = 3000
```

현재 위치가 `2000` 이라면 전체 범위의 중간인 약 50% 위치입니다.

Follower Arm의 위치 범위가 다음과 같다면,

```python
Follower Min = 1200
Follower Max = 2800
```

Follower Arm도 자신의 범위에서 약 50%에 해당하는 `2000`을 목표 위치로 사용합니다.

Source Code에서는 다음 함수가 위치를 변환합니다.

```python
def map_position(
    leader_pos,
    leader_min,
    leader_max,
    follower_min,
    follower_max
):
    ratio = (
        (leader_pos - leader_min)
        / (leader_max - leader_min)
    )

    ratio = clamp(ratio, 0.0, 1.0)

    follower_pos = (
        follower_min
        + ratio * (follower_max - follower_min)
    )

    return int(follower_pos)
```

ratio는 Leader Arm의 현재 위치가 전체 동작 범위에서 어느 정도에 있는지를 나타냅니다.

clamp() 함수는 계산된 비율이 `0.0`보다 작거나 `1.0`보다 커지지 않도록 제한합니다.

```python
def clamp(value, min_value, max_value):
    return max(
        min_value,
        min(max_value, value)
    )
```

이를 통해 Follower의 목표 위치가 측정된 범위를 벗어나지 않도록 제한합니다.

---

#### Leader 위치 추종

Program은 Leader ARm의 현재 위치를 계속 읽고 이를 Follower ARm의 목표 위치로 전달합니다.

```python
leader_pos = leader_bus.read(
    "Present_Position",
    name,
    normalize=False
)
```

Leader의 현재 위치와 두 Arm의 범위를 이용하여 Follower 목표 위치를 계산합니다.

```python
target_pos = map_position(
    leader_pos,
    l_min,
    l_max,
    f_min,
    f_max
)
```

계산한 목표 위치를 Follower Arm의 `Goal_Position`으로 전달합니다.

```python
follower_bus.write(
    "Goal_Position",
    name,
    target_pos,
    normalize=False
)
```

이 작업을 shoulder_pan부터 gripper까지 반복하여 Arm Joint와 Gripper가 모두 Leader Arm을 따라 움직이게 합니다.

```python
MOTOR_NAMES = [
    "shoulder_pan",
    "shoulder_lift",
    "elbow_flex",
    "wrist_flex",
    "wrist_roll",
    "gripper",
]

Leader shoulder_pan 현재 위치  → Follower shoulder_pan 목표 위치
Leader shoulder_lift 현재 위치 → Follower shoulder_lift 목표 위치
Leader elbow_flex 현재 위치    → Follower elbow_flex 목표 위치
Leader wrist_flex 현재 위치    → Follower wrist_flex 목표 위치
Leader wrist_roll 현재 위치    → Follower wrist_roll 목표 위치
Leader gripper 현재 위치       → Follower gripper 목표 위치
```

각 동작은 약 0.05초 간격으로 반복됩니다.

```python
time.sleep(0.05)
```

이;는 이론적으로 초당 약 20회 Leader 위치를 읽고 Follower 목표 위치를 갱신한다는 의미입니다.

실제 통신 주기는 Motor 통신 시간과 Computer 상태에 따라 달라질 수 있습니다.

---

#### Program 실행

Program을 실행하기 전에 Leader Arm과 Follower Arm을 서로 비슷한 자세로 맞춥니다.

두 Arm의 자세 차이가 큰 상태에서 Program을 실행하면 Follower Arm이 Leader Arm의 자세를 따라가기 위해 갑자기 움직일 수 있습니다.

Terminal에서 작업 폴더로 이동합니다.

```bash
cd ~/project/rosws/control
```

LeRobot 가상환경을 활성화합니다.

```bash
source ~/project/rosws/lerobot/venv/bin/activate
```

Leader 추격 Program을 실행합니다.

```bash
python leader_follower.py
```

Leader Arm과 Follower Arm의 Port를 순서대로 선택합니다.

Program이 시작되면 Leader Arm을 천천히 움직여 Follower Arm이 같은 방향으로 움직이는지 확인합니다.

```bash
Leader-Follower sync started.
Move the leader arm by hand.
Press Enter to stop.
```

Enter를 누르면 동작을 종료합니다. `Ctrl + C` 를 눌러 종료할 수도 있습니다.

Program이 종료되면 Follower Arm의 Torque를 비활성화하고 두 Motor Bus의 연결을 해제합니다.
