# 티칭 프로그램

앞에서 작성한 Motor 제어 Program을 바탕으로 SO-ARM101의 자세를 저장하는 Teaching Program을 만들어 보겠습니다.

Teaching은 사용자가 Robot을 원하는 자세로 직접 움직인 후 각 Joint의 현재 위치를 저장하는 작업입니다.

이번 Program에서는 다음 데이터를 저장합니다.

| 입력 Key | 저장할 데이터 |
| --- | --- |
| `1` | Teaching Point 1 |
| `2` | Teaching Point 2 |
| `3` | Teaching Point 3 |
| `+` | Grip Position |
| `-` | Ungrip Position |
| `q` | 종료 및 저장 여부 확인 |

Teaching Point 1, 2, 3에는 Gripper를 제외한 5개 Arm Joint의 위치를 저장합니다. Grip과 Ungrip에는 Gripper Motor의 위치만 저장합니다.

#### Prompt 1

앞에서 작성한 Motor 한 개 제어 Program을 AI에 입력하고 다음 Prompt를 추가합니다.

```
이 Source Code는 SO-ARM101의 Motor 한 개를 제어하는 Program입니다.
이 Source Code를 이요하여 Teaching Program을 만들어 주세요.

그리퍼를 제외한 5개 Arm Joint를 사용자가 손으로 움직인 후,
다음 Key를 누르면 현재 위치를 저장하도록 만들어주세요.

1 : Teaching Point 1 저장
2 : Teaching Point 2 저장
3 : Teaching Point 3 저장

그리퍼를 사용자가 손으로 움직인 후,
다음 Key를 누르면 현재 위치를 저장하도록 만들어주세요.

+ : Grip Position 저장
- : Ungrip Position 저장

Q를 누르면 종료할 때 저장할지를 물어보고,
Y를 누르면 티칭 포인트를 파일로 저장해주세요.
```

이 Prompt의 목적은 Motor 한 개를 제어하는 기존 Program을 여러 Joint의 현재 위치를 저장할 수 있는 Program으로 확장하는 것입니다.

처음 생성된 Source Code에서는 Motor 이름이 `m1`, `m2`와 같이 정의되었고 Teaching 데이터가 JSON 파일로 저장되었습니다.

Program은 정상적으로 동작할 수 있지만 실제 SO-ARM101의 Joint 구조를 쉽게 파악하고 데이터를 편리하게 확인하기 위해 다음과 같이 추가로 요청했습니다.

#### Prompt 2

```python
모터 이름을 SO ARM 101 에 맞춰서 변경해주세요.
```

#### Prompt 3

```python
json 보다 그냥 csv 형태로 저장하는게 보기 좋을것 같습니다.
```

---

#### 결과

Source Code 를 teaching_program.py 라는 이름으로 저장합니다.

> GitHub Link: [https://github.com/applesnack23/ros2-lerobot-code/blob/main/chapter2/teaching_program.py](https://github.com/applesnack23/ros2-lerobot-code/blob/main/chapter2/teaching_program.py)
> 

```python
import csv
import time
import termios
import tty
import sys
from datetime import datetime

from lerobot.motors.feetech.feetech import FeetechMotorsBus
from lerobot.motors.motors_bus import Motor

# =========================
# 1. Motor 설정
# =========================
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
# 3. 값 변환 함수
# =========================
def to_int(value):
    if isinstance(value, list):
        return int(value[0])

    try:
        return int(value)
    except TypeError:
        return int(value.item())

# =========================
# 4. 위치 읽기 함수
# =========================
def read_motor_position(bus, motor_name):
    pos = bus.read(
        "Present_Position",
        motor_name,
        normalize=False
    )
    return to_int(pos)

def read_arm_positions(bus):
    positions = {}

    for name in arm_motor_names:
        positions[name] = read_motor_position(bus, name)

    return positions

def read_grip_position(bus):
    return read_motor_position(bus, grip_motor_name)

# =========================
# 5. CSV 저장 함수
# =========================
def save_teaching_data_csv(teaching_data):
    filename = f"so_arm101_teaching.csv"

    headers = [
        "type",
        "name",
        "shoulder_pan",
        "shoulder_lift",
        "elbow_flex",
        "wrist_flex",
        "wrist_roll",
        "gripper",
    ]

    with open(filename, "w", newline="", encoding="utf-8-sig") as f:
        writer = csv.DictWriter(f, fieldnames=headers)
        writer.writeheader()

        for point_name, positions in teaching_data["points"].items():
            if positions is None:
                continue

            row = {
                "type": "point",
                "name": point_name,
                "shoulder_pan": positions["shoulder_pan"],
                "shoulder_lift": positions["shoulder_lift"],
                "elbow_flex": positions["elbow_flex"],
                "wrist_flex": positions["wrist_flex"],
                "wrist_roll": positions["wrist_roll"],
                "gripper": "",
            }

            writer.writerow(row)

        if teaching_data["gripper"]["grip"] is not None:
            writer.writerow({
                "type": "gripper",
                "name": "grip",
                "shoulder_pan": "",
                "shoulder_lift": "",
                "elbow_flex": "",
                "wrist_flex": "",
                "wrist_roll": "",
                "gripper": teaching_data["gripper"]["grip"],
            })

        if teaching_data["gripper"]["ungrip"] is not None:
            writer.writerow({
                "type": "gripper",
                "name": "ungrip",
                "shoulder_pan": "",
                "shoulder_lift": "",
                "elbow_flex": "",
                "wrist_flex": "",
                "wrist_roll": "",
                "gripper": teaching_data["gripper"]["ungrip"],
            })

    print(f"Saved: {filename}")

# =========================
# 6. 메인 프로그램
# =========================
teaching_data = {
    "points": {
        "1": None,
        "2": None,
        "3": None,
    },
    "gripper": {
        "grip": None,
        "ungrip": None,
    }
}

bus = FeetechMotorsBus(
    port="/dev/ttyACM0",
    motors=motors
)

try:
    bus.connect()

    # 손으로 티칭하기 위해 토크 OFF
    bus.disable_torque()

    print("====================================")
    print(" SO-ARM101 Teaching Program")
    print("====================================")
    print("1 : Teaching Point 1 저장")
    print("2 : Teaching Point 2 저장")
    print("3 : Teaching Point 3 저장")
    print("+ : Grip Position 저장")
    print("- : Ungrip Position 저장")
    print("q : 종료")
    print("====================================")

    while True:
        key = get_key()

        if key in ["1", "2", "3"]:
            positions = read_arm_positions(bus)
            teaching_data["points"][key] = positions

            print(f"\nTeaching Point {key} saved")
            print(positions)

        elif key == "+":
            grip_pos = read_grip_position(bus)
            teaching_data["gripper"]["grip"] = grip_pos

            print("\nGrip position saved")
            print("gripper:", grip_pos)

        elif key == "-":
            ungrip_pos = read_grip_position(bus)
            teaching_data["gripper"]["ungrip"] = ungrip_pos

            print("\nUngrip position saved")
            print("gripper:", ungrip_pos)

        elif key.lower() == "q":
            print("\nExit requested.")
            save = input("Save teaching data? (y/n): ")

            if save.lower() == "y":
                save_teaching_data_csv(teaching_data)
            else:
                print("Not saved.")

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

#### Motor 설정

SO-ARM101의 6개 Motor를 다음과 같이 정의합니다.

```python
motors = {
    "shoulder_pan": Motor(id=1, model="sts3215", norm_mode="position"),
    "shoulder_lift": Motor(id=2, model="sts3215", norm_mode="position"),
    "elbow_flex": Motor(id=3, model="sts3215", norm_mode="position"),
    "wrist_flex": Motor(id=4, model="sts3215", norm_mode="position"),
    "wrist_roll": Motor(id=5, model="sts3215", norm_mode="position"),
    "gripper": Motor(id=6, model="sts3215", norm_mode="position"),
}
```

각 Motor는 다음 정보로 구성됩니다.

- Dictionary에 사용하는 Motor 이름
- 실제 motor를 구분하는 ID
- Motor Model
- 위치 제어에 사용하는 Normalization Mode

Program에서는 Arm Joint와 Gripper를 구분하여 관리합니다.

```python
arm_motor_names = [
    "shoulder_pan",
    "shoulder_lift",
    "elbow_flex",
    "wrist_flex",
    "wrist_roll",
]

grip_motor_name = "gripper"
```

`arm_motor_names`에는 Robot Arm의 자세를 결정하는 5개 Joint가 들어 있습니다.

`gripper_motor_name`에는 물체를 잡거나 놓을 때 사용하는 Gripper가 정의되어 있습니다.

---

#### Key 입력 함수

```python
def get_key():
    fd = sys.stdin.fileno()
    old_settings = termios.tcgetattr(fd)

    try:
        tty.setraw(fd)
        key = sys.stdin.read(1)
    finally:
        termios.tcsetattr(fd, termios.TCSADRAIN, old_settings)

    return key
```

`get_key()`는 Enter를 누르지 않아도 사용자가 입력한 문자 한 개를 즉시 읽어오는 함수입니다.

함수 내부의 모든 문장을 외우거나 직접 작성할 필요는 없습니다. 이번 실습에서는 다음 역할만 이해하면 충분합니다.

> `get_key()`는 keyboard에서 입력한 문자 한 개를 즉시 반환합니다.
> 

---

#### Motor 위치 읽기

개별 Motor의 현재 위치는 `read_motor_position()` 을 읽어 확인합니다.

```python
def read_motor_position(bus, motor_name):
    pos = bus.read(
        "Present_Position",
        motor_name,
        normalize=False
    )
    return to_int(pos)
```

`motor_name`에는 위치를 읽을 Motor의 이름이 전달됩니다.

normalize=Fale를 사용했으므로 변환된 Joint값이 아닌 Motor의 Raw Position을 읽습니다.

5개 Arm Joint의 현재 위치는 다음 함수로 한 번에 읽습니다.

```python
def read_arm_positions(bus):
    positions = {}

    for name in arm_motor_names:
        positions[name] = read_motor_position(bus, name)

    return positions
```

`for` 반복문을 이용하여 `arm_motor_names`에 등록된 Motor의 위치를 하나씩 읽고 Dictionary에 저장합니다.

Gripper의 위치는 다음 함수로 별도로 읽습니다.

```python
def read_gripper_position(bus):
    return read_motor_position(bus, gripper_motor_name)
```

`read_gripper_position()`은 현재 Gripper 위치를 읽을 뿐 Grip과 Ungrip을 직접 구분하지는 않습니다.

읽어온 값을 Grip으로 저장할지 Ungrip으로 저장할지는 사용자가 입력한 Key에 따라 결정됩니다.

---

#### Torque 비활성화

Teaching에서는 사용자가 Robot의 Joint를 손으로 직접 움직여야 합니다.

따라서 Motor와 연결한 후 Torque를 비활성화합니다.

```python
bus.connect()
bus.disable_torque()
```

Torque가 활성화되어 있으면 Motor가 현재 위치를 유지하려고 힘을 사용하므로 Joint를 손으로 움직이기 어렵습니다.

Torque를 비활성화하면 Robot이 아래로 떨어질 수 있으므로 한 손으로 Robot을 지지하면서 자세를 변경해야 합니다.

---

#### Teaching 동작

Program은 `while` 반복문을 이용하여 사용자의 Key 입력을 계속 기다립니다.

```python
while True:
    key = get_key()

    if key in ["1", "2", "3"]:
        positions = read_arm_positions(bus)
        teaching_data["points"][key] = positions

    elif key == "+":
        grip_pos = read_gripper_position(bus)
        teaching_data["gripper"]["grip"] = grip_pos

    elif key == "-":
        ungrip_pos = read_gripper_position(bus)
        teaching_data["gripper"]["ungrip"] = ungrip_pos

    elif key.lower() == "q":
        break
```

입력한 Key에 따라 다음 작업을 수행합니다.

- `1`, `2`, `3`: 현재 5개 Arm Joint의 위치를 해당 Teaching Point로 저장
- `+`: 현재 Gripper 위치를 Grip으로 저장
- `-` : 현재 Gripper 위치를 Ungrip으로 저장
- `q`: Teaching을 종료하고 저장 여부 확인

`q`를 입력하면 다음과 같이 저장 여부를 확인합니다.

```python
save = input("Save teaching data? (y/n): ")

if save.lower() == "y":
    save_teaching_data_csv(teaching_data)
```

`y`를 입력하고 Enter를 누르면 지금까지 기록한 Teaching 데이터를 `so_arm101_teaching.csv` 파일로 저장합니다.

---

#### CSV 파일

CSV는 Comma-Separated Values의 약자로, 값을 쉼표로 구분하여 저장하는 Text File 형식입니다.

구조가 단순하고 Excel을 비롯한 다양한 Program에서 열 수 있으므로 Teaching 데이터를 확인하고 수정하기에 적합합니다.

Teaching 결과는 다음과 같은 형태로 저장됩니다.

| type | name | shoulder_pan | shoulder_lift | elbow_flex | wrist_flex | wrist_roll | gripper |
| --- | --- | --- | --- | --- | --- | --- | --- |
| point | 1 | 1981 | 1319 | 2673 | 2610 | 2075 |  |
| point | 2 | 1906 | 2039 | 2563 | 2226 | 2076 |  |
| point | 3 | 2204 | 2057 | 2456 | 2423 | 2072 |  |
| gripper | grip |  |  |  |  |  | 2211 |
| gripper | ungrip |  |  |  |  |  | 2840 |

Text Editor로 CSV 파일을 열면 다음과 같이 표시됩니다.

```
type,name,shoulder_pan,shoulder_lift,elbow_flex,wrist_flex,wrist_roll,gripper
point,1,1981,1319,2673,2610,2075,
point,2,1906,2039,2563,2226,2076,
point,3,2204,2057,2456,2423,2072,
gripper,grip,,,,,,2211
gripper,ungrip,,,,,,2840
```

`type`이 `point`인 행에는 5개 Arm Joint의 위치가 저장됩니다.

`type`이 `gripper`인 행에는 Grip 또는 Ungrip에 해당하는 Gripper 위치가 저장됩니다.

위의 숫자는 예시이며 실제 저장되는 값은 Teaching한 Robot 자세에 따라 달라집니다.

---

#### Program 실행

Terminal에서 작업 폴더로 이동합니다.

```bash
venv (또는 source ~/project/rosws/lerobot/venv/bin/activate)

cd ~/project/rosws/control

python teaching_program.py
```

Program이 실행되면 다음과 같은 사용 방법이 출력됩니다.

```
====================================
 SO-ARM101 Teaching Program
====================================
1 : Teaching Point 1 저장
2 : Teaching Point 2 저장
3 : Teaching Point 3 저장
+ : Grip Position 저장
- : Ungrip Position 저장
q : 종료
====================================
```

---

#### Teaching 실습

Pick & Place를 위해 다음 자세를 저장합니다.

**Teaching Point 1**

`1`을 눌러 대기 위치를 저장합니다.

대기 위치는 Pick 위치와 Place 위치 사이에서 Robot이 안전하게 이동할 수 있는 위치로 설정합니다.

**Teaching Point 2**

`2`를 눌러 Pick 위치를 저장합니다.

Pick 위치는 Gripper가 물체를 잡을 수 있는 위치입니다.

**Teaching Point 3**

`3`을 눌러 Place 위치를 저장합니다.

Place 위치는 Robot이 잡은 물체를 내려놓을 위치입니다.

**Ungrip Position**

Gripper를 물체보다 넓게 연 후 `-`를 눌러 Ungrip 위치를 저장합니다.

Ungrip은 물체를 잡기 전이나 내려놓을 때 사용하는 Gripper Open 위치입니다.

**Grip Position**

Gripper로 실제 물체를 잡은 상태에서 `+`를 눌러 Grip 위치를 저장합니다.

Gripper를 지나치게 닫으면 Motor에 계속 부하가 걸릴 수 있으므로 물체를 안정적으로 잡을 수 있는 정도로 설정합니다.

**종료 및 저장**

모든 위치를 저장한 후 `q`를 누릅니다.

```
Save teaching data? (y/n):
```

`y`를 입력하고 Enter를 누르면 다음 파일이 생성됩니다.

```
so_arm101_teaching.csv
```

최종 Directory 구조는 다음과 같습니다.

```
~/project/rosws/control
├── teaching_program.py
└── so_arm101_teaching.csv
```

---

#### Teaching 결과 확인

저장된 Teaching 데이터는 이후 다음 순서의 Pick & Place 동작에 사용합니다.

```
Teaching Point 1
→ 대기 위치

Teaching Point 2
→ Pick 위치

Teaching Point 3
→ Place 위치

Grip
→ Gripper Close

Ungrip
→ Gripper Open
```

이를 이용하면 다음과 같은 동작을 구성할 수 있습니다.

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

이번 장에서는 Robot의 자세를 직접 지정하고 CSV 파일에 저장했습니다.

다음 장에서는 저장된 `so_arm101_teaching.csv` 파일을 읽어 Robot을 각 Teaching Point로 이동시키는 Playback Program을 만들어 보겠습니다.
