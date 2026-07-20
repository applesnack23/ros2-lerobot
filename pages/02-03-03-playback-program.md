# 구동 프로그램

이번에는 Teaching Program에서 저장한 데이터를 이용하여 SO-ARM101을 실제로 움직이는 Playback Program을 만들어 보겠습니다.

Teaching Program과 Playback Program의 차이는 다음과 같습니다.

| Program | 역할 | Torque |
| --- | --- | --- |
| Teaching Program | 현재 Joint 위치 저장 | Disable |
| Playback Program | 저장된 위치로 이동 | Enable |

Playback Program에서는 `so_arm101_teaching.csv` 파일을 불러온 후 사용자가 입력한 Key에 따라 Robot을 움직입니다.

#### Prompt 4

```python
이번에는 앞에서 저장한 Teaching 데이터를 이용하여
SO-ARM101을 움직이는 Playback Program을 만들어주세요.

so_arm101_teaching.csv 파일에서
Teaching Point와 Gripper 위치를 읽어주세요.

Key에 따른 동작은 다음과 같습니다.

1 : Teaching Point 1로 이동
2 : Teaching Point 2로 이동
3 : Teaching Point 3으로 이동
+ : Grip 위치로 이동
- : Ungrip 위치로 이동
q : 프로그램 종료

Robot을 움직일 때는 Torque를 활성화하고,
종료할 때는 Torque를 비활성화한 후
Motor Bus 연결을 해제해주세요.
```

이 Prompt의 목적은 앞에서 저장한 so_arm101_teaching.csv 파일을 불러와 Keyboard 입력에 따라 Robot을 지정한 위치로 이동시키는 것입니다.

---

#### 결과

> GitHub Link: [https://github.com/applesnack23/ros2-lerobot-code/blob/main/chapter2/playback_program.py](https://github.com/applesnack23/ros2-lerobot-code/blob/main/chapter2/playback_program.py)
> 

Source Code 를 playback_program.py 라는 이름으로 저장합니다.

```python
import csv
import time
import termios
import tty
import sys

from lerobot.motors.feetech.feetech import FeetechMotorsBus
from lerobot.motors.motors_bus import Motor

# =========================
# 1. 설정
# =========================
CSV_FILE = "so_arm101_teaching.csv"

MOVE_DELAY = 0.5

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

# =========================
# 2. 키 입력 함수
# =========================
def get_key():
    fd = sys.stdin.fileno()
    old_settings = termios.tcgetattr(fd)

    try:
        tty.setraw(fd)
        key = sys.stdin.read(1)
    finally:
        termios.tcsetattr(fd, termios.TCSADRAIN, old_settings)

    return key

# =========================
# 3. CSV 읽기
# =========================
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

# =========================
# 4. 모터 이동 함수
# =========================
def move_arm_to_point(bus, point_data):
    for motor_name in arm_motor_names:
        bus.write(
            "Goal_Position",
            motor_name,
            point_data[motor_name],
            normalize=False
        )

    print("Arm moved:")
    print(point_data)

def move_gripper(bus, position):
    bus.write(
        "Goal_Position",
        grip_motor_name,
        position,
        normalize=False
    )

    print("Gripper moved:", position)

# =========================
# 5. 메인 프로그램
# =========================
teaching_data = load_teaching_data_csv(CSV_FILE)

bus = FeetechMotorsBus(
    port="/dev/ttyACM0",
    motors=motors
)

try:
    bus.connect()

    # 실행 프로그램은 토크 ON
    bus.enable_torque()

    print("====================================")
    print(" SO-ARM101 Playback Program")
    print("====================================")
    print(f"Loaded CSV: {CSV_FILE}")
    print("------------------------------------")
    print("1 : Move to Teaching Point 1")
    print("2 : Move to Teaching Point 2")
    print("3 : Move to Teaching Point 3")
    print("+ : Grip")
    print("- : Ungrip")
    print("q : Quit")
    print("====================================")

    while True:
        key = get_key()

        if key in ["1", "2", "3"]:
            if key not in teaching_data["points"]:
                print(f"\nTeaching Point {key} is not saved.")
                continue

            move_arm_to_point(bus, teaching_data["points"][key])
            time.sleep(MOVE_DELAY)

        elif key == "+":
            grip_pos = teaching_data["gripper"]["grip"]

            if grip_pos is None:
                print("\nGrip position is not saved.")
                continue

            move_gripper(bus, grip_pos)
            time.sleep(MOVE_DELAY)

        elif key == "-":
            ungrip_pos = teaching_data["gripper"]["ungrip"]

            if ungrip_pos is None:
                print("\nUngrip position is not saved.")
                continue

            move_gripper(bus, ungrip_pos)
            time.sleep(MOVE_DELAY)

        elif key.lower() == "q":
            print("\nQuit requested.")
            break

        time.sleep(0.05)

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

#### Teaching 데이터 읽기

Playback Program에서는 먼저 CSV 파일에 저장된 Teaching 데이터를 읽습니다.

```python
CSV_FILE = "so_arm101_teaching.csv"

teaching_data = load_teaching_data_csv(CSV_FILE)
```

`load_teaching_data_csv()` 함수는 CSV 파일에서 다음 데이터를 읽어 Program에서 사용할 수 있는 형태로 변환합니다.

- Teaching Point 1
- Teaching Point 2
- Teaching Point 3
- Grip Position
- Ungrip Position

CSV 파일은 Playback Program과 같은 폴더에 저장되어 있어야 합니다.

```
~/project/rosws/control
├── teaching_program.py
├── playback_program.py
└── so_arm101_teaching.csv
```

파일이 없거나 파일 이름이 다르면 Teaching 데이터를 불러올 수 없습니다.

---

#### Motor 연결과 Torque 활성화

Motor Controller와 연결하기 위해 `FeetechMotorsBus` 객체를 생성합니다.

```python
bus = FeetechMotorsBus(
    port="/dev/ttyACM0",
    motors=motors
)
```

`/dev/ttyACM0`은 필자의 환경에서 확인된 Port입니다. 사용하는 Computer에서 다른 Port가 확인되었다면 자신의 환경에 맞게 수정해야 합니다.

Teaching Program에서는 Robot을 손으로 움직이기 위해 Torque를 비활성화했습니다.

Playback Program에서는 Motor가 저장된 위치로 직접 이동해야 하므로 Torque를 활성화합니다.

```python
bus.connect()
bus.enable_torque()
```

Torque가 활성화되면 Motor는 전달받은 목표 위치로 이동하고 그 위치를 유지합니다.

> Torque를 활성화하면 Robot이 갑자기 움직일 수 있으므로 Robot 주변의 장애물을 제거하고 즉시 전원을 차단할 수 있도록 준비해야 합니다.
> 

---

#### Arm 이동 함수

Teaching Point에는 5개 Arm Joint의 위치가 저장되어 있습니다.

```python
def move_arm_to_point(bus, point_data):
    for motor_name in arm_motor_names:
        bus.write(
            "Goal_Position",
            motor_name,
            point_data[motor_name],
            normalize=False
        )
```

`move_arm_to_point()` 함수는 저장된 위치를 각 Motor의 `Goal_Position`으로 전달합니다.

예를 들어 다음 코드는 Teaching Point 1에 저장된 위치로 Robot을 이동시킵니다.

```python
move_arm_to_point(
    bus,
    teaching_data["points"]["1"]
)
```

함수 내부에서는 다음 5개 Joint에 목표 위치를 전달합니다.

- `shoulder_pan`
- `shoulder_lift`
- `elbow_flex`
- `wrist_flex`
- `wrist_roll`

---

#### Gripper 이동 함수

Gripper는 Arm Joint와 별도로 제어합니다.

```python
def move_gripper(bus, position):
    bus.write(
        "Goal_Position",
        "gripper",
        position,
        normalize=False
    )
```

Grip 동작은 Teaching 데이터에 저장된 Grip Position을 전달합니다.

```python
move_gripper(
    bus,
    teaching_data["gripper"]["grip"]
)
```

Ungrip 동작은 Ungrip Position을 전달합니다.

```python
move_gripper(
    bus,
    teaching_data["gripper"]["ungrip"]
)
```

---

#### Key 입력에 따른 동작

Playback Program은 `while` 반복문을 이용하여 사용자의 Key 입력을 계속 기다립니다.

```python
while True:
    key = get_key()

    if key in ["1", "2", "3"]:
        move_arm_to_point(
            bus,
            teaching_data["points"][key]
        )

    elif key == "+":
        move_gripper(
            bus,
            teaching_data["gripper"]["grip"]
        )

    elif key == "-":
        move_gripper(
            bus,
            teaching_data["gripper"]["ungrip"]
        )

    elif key.lower() == "q":
        break
```

각 Key의 동작은 다음과 같습니다.

| 입력 Key | 동작 |
| --- | --- |
| `1` | Teaching Point 1로 이동 |
| `2` | Teaching Point 2로 이동 |
| `3` | Teaching Point 3으로 이동 |
| `+` | Grip 동작 |
| `-` | Ungrip 동작 |
| `q` | Program 종료 |

선택한 Teaching Point 또는 Gripper Position이 CSV 파일에 저장되어 있지 않으면 Robot을 움직이지 않고 오류 메시지를 출력합니다.

---

#### 종료 처리

Program을 종료할 때는 `q`를 입력합니다.

종료 과정에서는 `finally`를 이용하여 Torque를 비활성화하고 Motor Bus 연결을 해제합니다.

```python
finally:
    bus.disable_torque()
    bus.disconnect()
```

`finally`는 Program이 정상적으로 종료되거나 실행 중 Error가 발생해도 마지막에 실행됩니다.

이를 통해 Motor의 Torque와 통신 연결을 정리할 수 있습니다.

---

#### Program 실행

Terminal에서 작업 폴더로 이동합니다.

```bash
cd ~/project/rosws/control
```

LeRobot 가상환경을 활성화합니다.

```bash
source ~/project/rosws/lerobot/venv/bin/activate
```

Playback Program을 실행합니다.

```bash
python playback_program.py
```

Program이 실행되면 다음과 같은 사용 방법이 출력됩니다.

```
====================================
 SO-ARM101 Playback Program
====================================
1 : Move to Teaching Point 1
2 : Move to Teaching Point 2
3 : Move to Teaching Point 3
+ : Grip
- : Ungrip
q : Quit
====================================
```

---

#### Pick & Place 동작 순서

Playback Program을 이용하여 수동으로 Pick & Place를 실행해 보겠습니다.

앞에서 Teaching Point를 다음과 같이 저장했습니다.

| Teaching 데이터 | 역할 |
| --- | --- |
| Teaching Point 1 | 대기 위치 |
| Teaching Point 2 | Pick 위치 |
| Teaching Point 3 | Place 위치 |
| Grip | Gripper Close |
| Ungrip | Gripper Open |

다음 순서대로 Key를 입력합니다.

1. `1`을 눌러 대기 위치로 이동합니다.
2. 를 눌러 Gripper를 엽니다.
3. `2`를 눌러 Pick 위치로 이동합니다.
4. `+`를 눌러 Gripper를 닫고 물체를 잡습니다.
5. `1`을 눌러 대기 위치로 이동합니다.
6. `3`을 눌러 Place 위치로 이동합니다.
7. 를 눌러 Gripper를 열고 물체를 내려놓습니다.
8. `1`을 눌러 대기 위치로 이동합니다.

전체 동작 흐름은 다음과 같습니다.

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

처음 실행할 때는 물체 없이 각 Teaching Point로 정상적으로 이동하는지 확인하는 것이 좋습니다.

특히 다음 항목을 확인해야 합니다.

- Motor ID가 실제 Joint와 일치하는가
- Robot이 예상한 방향으로 움직이는가
- Joint와 Link가 서로 충돌하지 않는가
- Cable이 당겨지거나 Joint에 걸리지 않는가
- Pick 및 Place 위치가 작업대와 충돌하지 않는가
- Grip Position에서 Motor에 과도한 부하가 발생하지 않는가

이 Program은 저장된 Teaching 데이터를 이용하여 Robot을 움직일 수 있지만 사용자가 Key를 하나씩 입력해야 합니다.

다음 장에서는 이 Key 입력 순서를 Program에 저장하여 Pick & Place 동작을 자동으로 실행해 보겠습니다.
