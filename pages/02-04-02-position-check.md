# 위치 확인

Leader Arm 과 Follower Arm을 연동하려면 먼저 각 Motor가 움직일 수 있는 위치 범위를 확인해야 합니다.

위치 확인 Program은 각 Motor의 현재 위치를 반복해서 읽으면서 지금까지 측정된 최소값과 최대값을 기록합니다.

### Prompt 6

```
SO-ARM101의 6개 STS3215 Motor 위치를 확인하는
Python Program을 만들어주세요.

연결된 /dev/ttyACM*와 /dev/ttyUSB* Port를 검색하고,
사용자가 사용할 Port를 선택할 수 있게 해주세요.

사용자가 Robot을 손으로 움직일 수 있도록
Motor Bus에 연결한 후 Torque를 비활성화해주세요.

각 Motor의 Present_Position을 반복해서 읽고
현재 위치와 지금까지 측정된 Min/Max 값을 화면에 표시해주세요.

Enter를 누르면 측정을 종료하고,
저장 여부와 Leader/Follower 여부를 확인한 후
다음 CSV 파일 중 하나로 저장해주세요.

leader_arm_min_max.csv
follower_arm_min_max.csv

프로그램 종료 시 Motor Bus 연결을 해제해주세요.
```

이 Prompt의 목적은 Leader Arm과 Follower Arm의 각 Motor가 실제로 움직이는 위치 범위를 측정해 최대값과 최소값을 CSV 파일로 저장하는 것입니다.

전체 Source Code는 다음 GitHub에서 확인할 수 있습니다.

> GitHub Link: [https://github.com/applesnack23/ros2-lerobot-code/blob/main/chapter2/position_monitor.py](https://github.com/applesnack23/ros2-lerobot-code/blob/main/chapter2/position_monitor.py)
> 

전체 Source Code를 `position_monitor.py`라는 이름으로 저장합니다.

```python
from lerobot.motors.feetech.feetech import FeetechMotorsBus
from lerobot.motors.motors_bus import Motor
import time
import os
import sys
import select
import csv
import glob

motors = {
    "shoulder_pan": Motor(id=1, model="sts3215", norm_mode="position"),
    "shoulder_lift": Motor(id=2, model="sts3215", norm_mode="position"),
    "elbow_flex": Motor(id=3, model="sts3215", norm_mode="position"),
    "wrist_flex": Motor(id=4, model="sts3215", norm_mode="position"),
    "wrist_roll": Motor(id=5, model="sts3215", norm_mode="position"),
    "gripper": Motor(id=6, model="sts3215", norm_mode="position"),
}

min_pos = {name: None for name in motors}
max_pos = {name: None for name in motors}

def select_port():
    ports = sorted(glob.glob("/dev/ttyACM*") + glob.glob("/dev/ttyUSB*"))

    if not ports:
        print("No serial ports found.")
        print("Check USB connection.")
        sys.exit(1)

    print("Available serial ports:")
    print("-" * 30)

    for i, port in enumerate(ports, start=1):
        print(f"{i}. {port}")

    print("-" * 30)

    while True:
        choice = input("Select port number: ").strip()

        if choice.isdigit():
            index = int(choice) - 1

            if 0 <= index < len(ports):
                return ports[index]

        print("Invalid selection. Try again.")

def enter_pressed():
    return select.select([sys.stdin], [], [], 0)[0]

def select_arm_type():
    print("\nSelect arm type:")
    print("1. leader arm")
    print("2. follower arm")

    while True:
        choice = input("Select 1 or 2: ").strip()

        if choice == "1":
            return "leader_arm"

        if choice == "2":
            return "follower_arm"

        print("Invalid selection. Select 1 or 2.")

def save_min_max(arm_type):
    filename = f"{arm_type}_min_max.csv"

    with open(filename, "w", newline="") as f:
        writer = csv.writer(f)
        writer.writerow(["arm_type", "motor", "min", "max"])

        for name in motors:
            writer.writerow([
                arm_type,
                name,
                min_pos[name],
                max_pos[name]
            ])

    print(f"Saved: {filename}")

selected_port = select_port()

bus = FeetechMotorsBus(
    port=selected_port,
    motors=motors
)

try:
    bus.connect()

    print(f"\nConnected to {selected_port}")
    print("Motor position monitor started.")
    print("Press Enter to stop.")
    time.sleep(1)

    while True:
        if enter_pressed():
            sys.stdin.readline()
            break

        os.system("clear")

        print("SO-ARM101 / STS3215 Position Monitor")
        print(f"Port: {selected_port}")
        print("-" * 55)
        print(f"{'Motor':<8} {'Current':>10} {'Min':>10} {'Max':>10}")
        print("-" * 55)

        for name in motors:
            try:
                pos = bus.read(
                    "Present_Position",
                    name,
                    normalize=False
                )

                if hasattr(pos, "item"):
                    pos = pos.item()

                pos = int(pos)

                if min_pos[name] is None or pos < min_pos[name]:
                    min_pos[name] = pos

                if max_pos[name] is None or pos > max_pos[name]:
                    max_pos[name] = pos

                print(
                    f"{name:<8} "
                    f"{pos:>10} "
                    f"{min_pos[name]:>10} "
                    f"{max_pos[name]:>10}"
                )

            except Exception as e:
                print(f"{name:<8} READ ERROR: {e}")

        print("-" * 55)
        print("Press Enter to stop monitoring.")

        time.sleep(0.1)

    print("\nMonitoring stopped.")
    print('To save min/max values, type exactly "yes".')
    answer = input("Save min/max values? ").strip()

    if answer == "yes":
        arm_type = select_arm_type()
        save_min_max(arm_type)
    else:
        print("Not saved.")

except KeyboardInterrupt:
    print("\nStopped by Ctrl + C.")
    print('To save min/max values, type exactly "yes".')
    answer = input("Save min/max values? ").strip()

    if answer == "yes":
        arm_type = select_arm_type()
        save_min_max(arm_type)
    else:
        print("Not saved.")

finally:
    try:
        bus.disconnect()
    except:
        pass

    print("Bus disconnected.")
```

---

#### Serial Port tjsxor

Leader Arm과 Follower Arm은 각각 별도의 USB Motor Controller를 사용합니다.

Program에서는 현재 연결된 Serial Port를 검색합니다.

```python
ports = sorted(
    glob.glob("/dev/ttyACM*")
    + glob.glob("/dev/ttyUSB*")
)
```

검색된 Port는 다음과 같이 표시됩니다.

```python
Available serial ports:
------------------------------
1. /dev/ttyACM0
2. /dev/ttyACM1
------------------------------
Select port number:
```

Leader Arm 또는 Follower Arm에 해당하는 Port 번호를 선택합니다.

어떤 Port가 각 Arm에 연결되어 있는지 알 수 없다면 USB Cable을 하나씩 분리한 후 다음 명령으로 확인할 수 있습니다.

```bash
lerobot-find-port
```

---

#### Motor 구성

위치 확인 Program에서는 6개의 Motor를 다음과 같이 정의합니다.

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

`shoulder_pan` 부터 `gripper` 는 각각 Motor ID 1부터 6에 대응합니다.

위치 범위를 저장하기 위해 Motor별 최소값과 최대값을 준비합니다.

```python
min_pos = {name: None for name in motors}
max_pos = {name: None for name in motors}
```

---

#### 현재 위치와 범위 확인

Motor 의 현재 위치는 `Present_Position`을 읽어 확인합니다.

```python
pos = bus.read(
    "Present_Position",
    name,
    normalize=False
)
```

`normalize=False`를 사용했으므로 변환된 관절값이 아닌 Motor의 Raw Position을 읽습니다.

현재 위치가 기존 최소값보다 작으면 최소값을 변경합니다.

```python
if min_pos[name] is None or pos < min_pos[name]:
    min_pos[name] = pos
```

현재 위치가 기존 최대값보다 크면 최대값을 변경합니다.

```python
if max_pos[name] is None or pos > max_pos[name]:
    max_pos[name] = pos
```

Program을 실행한 상태에서 각 Joint를 천천히 움직이면 현재 위치와 측정된 범위가 표시됩니다.

```
Motor      Current        Min        Max
-------------------------------------------------------
m1            2048       1024       3072
m2            2100       1200       2900
m3            1950       1100       3000
m4            2200       1300       2850
m5            2050       1000       3100
m6            1800       1500       2400
```

표시되는 값은 예시이며 실제 결과는 Robot의 조립 상태와 Motor 설정에 따라 달라집니다.

---

#### 위치 측정 방법

Program을 실행한 후 각 Joint를 움직일 수 있도록 Torque를 비활성화합니다.

```
bus.connect()
bus.disable_torque()
```

Leader Arm 또는 Follower Arm 의 Joint를 하나씩 천천히 움직여 실제 사용 가능한 전체 범위를 측정합니다.

---

#### CSV 파일 저장

측정을 종료하면 저장 여부를 확인합니다.

```
To save min/max values, type exactly "yes".
Save min/max values?
```

저장하려면 다음과 같이 입력합니다.

```
yes
```

이후 측정한 Robot의 종류를 선택합니다.

```
Select arm type:
1. leader arm
2. follower arm
```

Leader Arm을 측정했다면 `1`, Follower Arm을 측정했다면 `2`를 입력합니다.

측정 결과는 다음 파일로 저장됩니다.

```
Leader Arm
→ leader_arm_min_max.csv

Follower Arm
→ follower_arm_min_max.csv
```

CSV 파일에는 각 Motor의 최소값과 최대값이 저장됩니다.

```
arm_type,motor,min,max
leader_arm,m1,1024,3072
leader_arm,m2,1200,2900
leader_arm,m3,1100,3000
leader_arm,m4,1300,2850
leader_arm,m5,1000,3100
leader_arm,m6,1500,2400
```

위의 숫자는 예시이며 실제 측정값은 Robot 마다 달라질 수 있습니다.

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

위치 확인 Program을 실행합니다.

```bash
python position_monitor.py
```

먼저 Leader Arm의 전체 위치 범위를 측정하여 저장하고, Program을 다시 실행하여 Follower Arm의 위치 범위를 측정합니다.

최종 Directory 구조는 다음과 같습니다.

```bash
~/project/rosws/control
├── position_monitor.py
├── leader_arm_min_max.csv
└── follower_arm_min_max.csv
```

두 CSV 파일은 다음 Leader 추격 Program에서 사용하므로 파일 이름을 변경하지 않습니다.

---
