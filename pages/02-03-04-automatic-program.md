# 자동 프로그램

이번에는 앞에서 만든 Playback Program을 바탕으로 Pick & Place 동작을 자동으로 실행해 보겠습니다.

Playback Program에서는 사용자가 `1`, `2`, `3`, `+`, `-` Key를 직접 입력했습니다. 하지만 Automatic Pick & Place Program에서는 동작 순서를 Source Code에 저장하고 Program을 실행하면 순서대로 동작하게 만듭니다.

#### Prompt 5

```python
앞에서 만든 Playback Program을 Automatic Pick & Place Program으로 변경해주세요.

프로그램을 실행하면 다음 순서로 한 번 동작하게 만들어주세요.

Teaching Point 1
→ Ungrip
→ Teaching Point 2
→ Grip
→ Teaching Point 1
→ Teaching Point 3
→ Ungrip
→ Teaching Point 1

각 동작 사이에는 Robot이 이동할 수 있도록 대기 시간을 적용해주세요.

프로그램이 끝나면 Torque를 비활성화하고
Motor Bus 연결을 해제해주세요.
```

이 Prompt의 목적은 사용자가 Key를 하나씩 입력하지 않오도 Program을 실행하면 정해진 순서대로 Pick & Place를 수행하도록 만드는 것입니다.

#### 결과

> GitHub Link: [https://github.com/applesnack23/ros2-lerobot-code/blob/main/chapter2/auto_pick_place_program.py](https://github.com/applesnack23/ros2-lerobot-code/blob/main/chapter2/auto_pick_place_program.py)
> 

Source Code 를 auto_pick_place_program.py 라는 이름으로 저장합니다.

```python
import csv
import time

from lerobot.motors.feetech.feetech import FeetechMotorsBus
from lerobot.motors.motors_bus import Motor

CSV_FILE = "so_arm101_teaching.csv"
AUTO_DELAY = 1.0

motors = {
    "shoulder_pan": Motor(id=1, model="sts3215", norm_mode="position"),
    "shoulder_lift": Motor(id=2, model="sts3215", norm_mode="position"),
    "elbow_flex": Motor(id=3, model="sts3215", norm_mode="position"),
    "wrist_flex": Motor(id=4, model="sts3215", norm_mode="position"),
    "wrist_roll": Motor(id=5, model="sts3215", norm_mode="position"),
    "gripper": Motor(id=6, model="sts3215", norm_mode="position"),
}

arm_motor_names = [
    "shoulder_pan",
    "shoulder_lift",
    "elbow_flex",
    "wrist_flex",
    "wrist_roll",
]

grip_motor_name = "gripper"

def load_teaching_data_csv(filename):
    teaching_data = {
        "points": {},
        "gripper": {
            "grip": None,
            "ungrip": None,
        }
    }

    with open(filename, "r", encoding="utf-8-sig") as f:
        reader = csv.DictReader(f)

        for row in reader:
            row_type = row["type"]
            name = row["name"]

            if row_type == "point":
                teaching_data["points"][name] = {
                    "shoulder_pan": int(row["shoulder_pan"]),
                    "shoulder_lift": int(row["shoulder_lift"]),
                    "elbow_flex": int(row["elbow_flex"]),
                    "wrist_flex": int(row["wrist_flex"]),
                    "wrist_roll": int(row["wrist_roll"]),
                }

            elif row_type == "gripper":
                teaching_data["gripper"][name] = int(row["gripper"])

    return teaching_data

def check_auto_data(teaching_data):
    for point in ["1", "2", "3"]:
        if point not in teaching_data["points"]:
            raise ValueError(f"Teaching Point {point} is not saved.")

    if teaching_data["gripper"]["grip"] is None:
        raise ValueError("Grip position is not saved.")

    if teaching_data["gripper"]["ungrip"] is None:
        raise ValueError("Ungrip position is not saved.")

def move_arm_to_point(bus, point_data):
    for motor_name in arm_motor_names:
        bus.write(
            "Goal_Position",
            motor_name,
            point_data[motor_name],
            normalize=False
        )

    print("Move arm:", point_data)

def move_gripper(bus, position):
    bus.write(
        "Goal_Position",
        grip_motor_name,
        position,
        normalize=False
    )

    print("Move gripper:", position)

def run_auto_sequence(bus, teaching_data):
    sequence = [
        ("point", "1"),
        ("gripper", "ungrip"),
        ("point", "2"),
        ("gripper", "grip"),
        ("point", "1"),
        ("point", "3"),
        ("gripper", "ungrip"),
        ("point", "1"),
    ]

    print("Auto sequence start")

    for action_type, name in sequence:
        if action_type == "point":
            move_arm_to_point(bus, teaching_data["points"][name])

        elif action_type == "gripper":
            move_gripper(bus, teaching_data["gripper"][name])

        time.sleep(AUTO_DELAY)

    print("Auto sequence finished")

teaching_data = load_teaching_data_csv(CSV_FILE)
check_auto_data(teaching_data)

bus = FeetechMotorsBus(
    port="/dev/ttyACM0",
    motors=motors
)

try:
    bus.connect()
    bus.enable_torque()

    run_auto_sequence(bus, teaching_data)

finally:
    try:
        bus.disable_torque()
    except Exception:
        pass

    try:
        bus.disconnect()
    except Exception:
        pass

    print("Motor bus disconnected.")
```

---

#### 자동 동작 순서

Automatic Pick & Place Program은 다음 순서로 동작합니다.

```
Teaching Point 1
→ Ungrip
→ Teaching Point 2
→ Grip
→ Teaching Point 1
→ Teaching Point 3
→ Ungrip
→ Teaching Point 1
```

각 동작의 의미는 다음과 같습니다.

| 순서 | 동작 | 의미 |
| --- | --- | --- |
| 1 | Teaching Point 1 | 대기 위치로 이동 |
| 2 | Ungrip | Gripper를 열어 물체를 잡을 준비 |
| 3 | Teaching Point 2 | Pick 위치로 이동 |
| 4 | Grip | Gripper를 닫아 물체를 잡음 |
| 5 | Teaching Point 1 | 물체를 들어 대기 위치로 이동 |
| 6 | Teaching Point 3 | Place 위치로 이동 |
| 7 | Ungrip | Gripper를 열어 물체를 내려놓음 |
| 8 | Teaching Point 1 | 대기 위치로 복귀 |

Pick 위치에서 Place 위치로 바로 이동하지 않고 중간에 대기 위치를 거치기 때문에 물체나 Robot이 작업대와 충돌할 가능성을 줄일 수 있습니다.

---

#### 자동 동작 Sequence

자동 동작의 핵심은 Robot이 수행할 작업을 순서대로 저장한 `sequence`입니다.

```python
sequence = [
    ("point", "1"),
    ("gripper", "ungrip"),
    ("point", "2"),
    ("gripper", "grip"),
    ("point", "1"),
    ("point", "3"),
    ("gripper", "ungrip"),
    ("point", "1"),
]
```

각 항목은 다음과 같이 구성됩니다.

```python
("동작 종류","동작 이름")
```

예를 들어 다음 항목은 Teaching Point 1로 이동하라는 의미입니다.

```python
("point","1")
```

다음 항목은 Gripper를 저장된 Ungrip 위치로 이동시키라는 의미입니다.

```python
("gripper","ungrip")
```

`sequence`를 사용하면 Robot의 동작 순서를 Source Code에서 쉽게 확인하고 변경할 수 있습니다.

---

#### Sequence 실행

`for` 반복문을 이용하여 `sequence`에 저장된 동작을 하나씩 실행합니다.

```python
for action_type, name in sequence:
    if action_type == "point":
        move_arm_to_point(
            bus,
            teaching_data["points"][name]
        )

    elif action_type == "gripper":
        move_gripper(
            bus,
            teaching_data["gripper"][name]
        )

    time.sleep(AUTO_DELAY)
```

`action_type`이 `point`이면 Arm을 해당 Teaching Point로 이동시킵니다.

`action_type`이 `gripper`이면 Gripper를 Grip 또는 Ungrip 위치로 이동시킵니다.

각 동작 후에는 다음 동작으로 넘어가기 전에 일정 시간 동안 기다립니다.

```python
AUTO_DELAY=2.0
```

위 설정은 각 동작 후 2초 동안 기다린다는 의미입니다.

Robot의 이동 거리가 길거나 속도가 느리면 목표 위치에 도착하기 전에 다음 동작이 실행될 수 있습니다. 처음에는 충분히 긴 대기 시간을 사용하고 실제 동작을 확인하면서 조정해야 합니다.

> `time.sleep()`은 Robot이 목표 위치에 도착했는지 확인하는 기능이 아니라 지정된 시간 동안 기다리는 기능입니다.
> 

---

#### Teaching 데이터 확인

자동 동작을 시작하려면 다음 데이터가 모두 필요합니다.

- Teaching Point 1
- Teaching Point 2
- Teaching Point 3
- Grip Position
- Ungrip Position

Program에서는 `check_atuo_data()` 함수를 이용하여 필요한 데이터가 저장되어 있는지 확인합니다.

```python
def check_auto_data(teaching_data):
    for point in ["1", "2", "3"]:
        if point not in teaching_data["points"]:
            raise ValueError(
                f"Teaching Point {point} is not saved."
            )

    if teaching_data["gripper"]["grip"] is None:
        raise ValueError("Grip position is not saved.")

    if teaching_data["gripper"]["ungrip"] is None:
        raise ValueError("Ungrip position is not saved.")
```

필요한 데이터가 하나라도 없으면 자동 동작을 시작하지 않고 Error를 발생시킵니다.

실제 자동화 Program에서는 동작을 시작하기 전에 필요한 데이터와 Hardware 상태를 확인하는 과정이 중요합니다.

---

#### Motor 연결과 자동 실행

Teaching 데이터를 확인한 후 Motor Bus에 연결하고 Torque를 활성화합니다.

```
bus.connect()bus.enable_torque()
```

이후 자동 동작 함수를 호출합니다.

```
run_auto_sequence(bus,teaching_data)
```

`run_auto_sequence()`가 호출되면 `sequence`에 저장된 동작이 처음부터 끝까지 한 번 실행됩니다.

자동 동작이 완료되거나 실행 중 Error가 발생하면 Torque를 비활성화하고 Motor Bus 연결을 해제합니다.

```
finally:bus.disable_torque()bus.disconnect()
```

---

#### Program 실행

Terminal에서 작업 폴더로 이동합니다.

```
cd ~/project/rosws/control
```

LeRobot 가상환경을 활성화합니다.

```
source ~/project/rosws/lerobot/venv/bin/activate
```

자동 Program을 실행합니다.

```
python auto_pick_place_program.py
```

Program을 실행하면 별도의 Key 입력 없이 다음 순서가 한 번 실행됩니다.

```
대기 위치
→ Gripper Open
→ Pick 위치
→ Gripper Close
→ 대기 위치
→ Place 위치
→ Gripper Open
→ 대기 위치
```

---

#### 실행 전 확인

Automatic Pick & Place는 Program을 실행하는 즉시 Robot이 움직이므로 다음 항목을 먼저 확인해야 합니다.

- Motor Bus가 정상적으로 연결되어 있는가
- USB Port가 올바른가
- Motor ID가 실제 Joint와 일치하는가
- `so_arm101_teaching.csv` 파일이 존재하는가
- Teaching Point 1, 2, 3이 모두 저장되어 있는가
- Grip과 Ungrip 위치가 저장되어 있는가
- Robot 주변에 사람이나 장애물이 없는가
- Joint와 Link가 서로 충돌하지 않는가
- Cable이 Joint에 걸리지 않는가
- Gripper와 물체가 작업대에 충돌하지 않는가
- 즉시 전원을 차단할 수 있는가

처음에는 물체 없이 전체 동작을 확인하는 것이 좋습니다. 정상적으로 움직이는 것을 확인한 후 물체를 놓고 Pick & Place를 실행합니다.

---

#### 자동 Program의 한계

현재 Program은 각 동작 후 `time.sleep()`을 이용하여 일정 시간 동안 기다립니다.

따라서 다음 내용을 직접 확인하지는 않습니다.

- Motor가 목표 위치에 도착했는가
- 이동 중 Error가 발생했는가
- Gripper가 물체를 정상적으로 잡았는가
- 물체가 Pick 위치에 존재하는가
- Place 위치에 물체를 놓을 공간이 있는가

현재 단계에서는 정해진 위치와 순서를 이용하여 Automatic Pick & Place의 기본 구조를 경험하는 것이 목적입니다.

---
