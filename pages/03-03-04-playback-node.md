# so_arm101_pick_place 패키지: playback_node

이번 절에서는 Teaching 과정에서 저장한 CSV 파일을 불러와 SO-ARM101을 저장된 위치로 이동시키는 Playback 노드를 작성합니다.

`playback_node`는 키보드 입력을 받으면 CSV에 저장된 목표 위치를 `/so_arm101/joint_goal` 토픽으로 발행합니다. 실제 모터 제어는 앞 절에서 작성한 `feetech_driver_node`가 담당합니다.

> 제공한 소스 코드는 `teaching_node.py`가 다시 들어가 있어 Playback 기능을 수행하지 않습니다. 아래에서는 CSV를 읽고 목표 위치를 발행하는 Playback 코드로 수정했습니다.
> 

---

#### Playback 노드의 역할

`playback_node.py`는 다음 기능을 담당합니다.

1. Teaching 과정에서 생성한 CSV 파일 읽기
2. Teaching Point 1, 2, 3 불러오기
3. Grip 및 Ungrip 위치 불러오기
4. 키보드 입력 처리
5. 목표 위치를 `/so_arm101/joint_goal` 토픽으로 발행
6. `q` 키 입력 시 노드 종료

노드 파일은 다음 경로에 생성합니다.

```
~/project/ros2_ws/src/so_arm101_pick_place/
└── so_arm101_pick_place/
    └── playback_node.py
```

![image.png](../assets/so_arm101_pick_placeplayback_node/image.png)

---

#### 소스 코드 작성 프롬프트

AI로 기본 코드를 작성할 때는 다음 프롬프트를 사용할 수 있습니다.

```
이전에 작성한 motor_teach.py, motor_run.py,
motor_auto_run.py를 참고해 주세요.

SO-ARM101의 Pick & Place 동작을 ROS2로 구현하려고 합니다.

teaching_node.py에서 저장한
so_arm101_teaching.csv 파일을 읽어 다음 기능을 수행하는
playback_node.py를 작성해 주세요.

1. 숫자 키 1, 2, 3을 누르면 저장된 Teaching Point로 이동합니다.
2. + 키를 누르면 저장된 Grip 위치로 Gripper를 이동합니다.
3. - 키를 누르면 저장된 Ungrip 위치로 Gripper를 이동합니다.
4. 목표 위치는 sensor_msgs/msg/JointState 메시지로 구성합니다.
5. /so_arm101/joint_goal 토픽으로 목표 위치를 발행합니다.
6. q 키를 누르면 노드를 종료합니다.
7. CSV 파일이나 필요한 데이터가 없으면 오류 메시지를 출력합니다.
```

---

#### playback_node.py 작성

#### 전체 소스 코드

> GitHub Link: [https://github.com/applesnack23/ros2-lerobot-code/blob/main/chapter3/playback_node.py](https://github.com/applesnack23/ros2-lerobot-code/blob/main/chapter3/playback_node.py)
> 

```python
import csv
import os
import sys
import termios
import tty

import rclpy
from rclpy.node import Node
from sensor_msgs.msg import JointState

CSV_FILE = os.path.expanduser(
    '~/project/ros2_ws/so_arm101_teaching.csv'
)

ARM_JOINT_NAMES = [
    'shoulder_pan',
    'shoulder_lift',
    'elbow_flex',
    'wrist_flex',
    'wrist_roll',
]

GRIPPER_JOINT_NAME = 'gripper'

def get_key():
    fd = sys.stdin.fileno()
    old_settings = termios.tcgetattr(fd)

    try:
        tty.setraw(fd)
        key = sys.stdin.read(1)
    finally:
        termios.tcsetattr(
            fd,
            termios.TCSADRAIN,
            old_settings
        )

    return key

class PlaybackNode(Node):

    def __init__(self):
        super().__init__('playback_node')

        self.teaching_points = {}
        self.gripper_positions = {}

        self.joint_goal_pub = self.create_publisher(
            JointState,
            '/so_arm101/joint_goal',
            10
        )

        self.load_teaching_data()

        self.get_logger().info(
            'SO-ARM101 Playback Node started.'
        )

        self.print_menu()

    def load_teaching_data(self):
        if not os.path.exists(CSV_FILE):
            raise FileNotFoundError(
                f'Teaching CSV file not found: {CSV_FILE}'
            )

        with open(
            CSV_FILE,
            'r',
            newline='',
            encoding='utf-8-sig'
        ) as file:
            reader = csv.DictReader(file)

            for row in reader:
                data_type = row.get('type', '').strip()
                data_name = row.get('name', '').strip()

                if data_type == 'point':
                    positions = {}

                    for joint_name in ARM_JOINT_NAMES:
                        value = row.get(joint_name, '').strip()

                        if value == '':
                            raise ValueError(
                                f'Missing {joint_name} value '
                                f'in Teaching Point {data_name}'
                            )

                        positions[joint_name] = float(value)

                    self.teaching_points[data_name] = positions

                elif data_type == 'gripper':
                    value = row.get(
                        GRIPPER_JOINT_NAME,
                        ''
                    ).strip()

                    if value == '':
                        raise ValueError(
                            f'Missing gripper value: {data_name}'
                        )

                    self.gripper_positions[data_name] = (
                        float(value)
                    )

        self.get_logger().info(
            f'Teaching data loaded: {CSV_FILE}'
        )

        self.get_logger().info(
            f'Teaching Points: '
            f'{list(self.teaching_points.keys())}'
        )

        self.get_logger().info(
            f'Gripper Positions: '
            f'{list(self.gripper_positions.keys())}'
        )

    def print_menu(self):
        print('====================================')
        print(' SO-ARM101 ROS2 Playback Program')
        print('====================================')
        print('1 : Move to Teaching Point 1')
        print('2 : Move to Teaching Point 2')
        print('3 : Move to Teaching Point 3')
        print('+ : Move to Grip Position')
        print('- : Move to Ungrip Position')
        print('q : Quit')
        print('====================================')

    def publish_arm_goal(self, point_name):
        if point_name not in self.teaching_points:
            self.get_logger().warning(
                f'Teaching Point {point_name} is not saved.'
            )
            return

        positions = self.teaching_points[point_name]

        msg = JointState()
        msg.header.stamp = self.get_clock().now().to_msg()
        msg.name = ARM_JOINT_NAMES.copy()
        msg.position = [
            positions[joint_name]
            for joint_name in ARM_JOINT_NAMES
        ]

        self.joint_goal_pub.publish(msg)

        self.get_logger().info(
            f'Move to Teaching Point {point_name}'
        )

        for name, position in zip(
            msg.name,
            msg.position
        ):
            self.get_logger().info(
                f'  {name}: {position}'
            )

    def publish_gripper_goal(self, position_name):
        if position_name not in self.gripper_positions:
            self.get_logger().warning(
                f'Gripper position is not saved: '
                f'{position_name}'
            )
            return

        position = self.gripper_positions[position_name]

        msg = JointState()
        msg.header.stamp = self.get_clock().now().to_msg()
        msg.name = [GRIPPER_JOINT_NAME]
        msg.position = [position]

        self.joint_goal_pub.publish(msg)

        self.get_logger().info(
            f'Move Gripper: {position_name} -> {position}'
        )

    def run(self):
        while rclpy.ok():
            key = get_key()

            if key in ['1', '2', '3']:
                self.publish_arm_goal(key)

            elif key == '+':
                self.publish_gripper_goal('grip')

            elif key == '-':
                self.publish_gripper_goal('ungrip')

            elif key.lower() == 'q':
                print('\nExit requested.')
                break

            else:
                print(f'\nUnknown key: {repr(key)}')

def main(args=None):
    rclpy.init(args=args)

    try:
        node = PlaybackNode()

    except (FileNotFoundError, ValueError) as error:
        print(f'Playback initialization failed: {error}')
        rclpy.shutdown()
        return

    try:
        node.run()

    except KeyboardInterrupt:
        pass

    finally:
        node.destroy_node()
        rclpy.shutdown()

if __name__ == '__main__':
    main()
```

---

#### 코드의 주요 구조

**CSV 파일 경로**

```python
CSV_FILE = os.path.expanduser(
    '~/project/ros2_ws/so_arm101_teaching.csv'
)
```

Teaching 노드와 Playback 노드가 동일한 CSV 파일을 사용하도록 경로를 고정합니다.

파일이 존재하지 않으면 Playback 노드는 다음과 같은 오류를 출력하고 종료합니다.

```bash
Playback initialization failed:
Teaching CSV file not found:
~/project/ros2_ws/so_arm101_teaching.csv
```

**Teaching 데이터 저장 구조**

CSV에서 읽은 데이터는 두 개의 딕셔너리로 구분해 저장합니다.

```python
self.teaching_points = {}
self.gripper_positions = {}
```

데이터를 모두 불러오면 다음과 같은 구조가 됩니다.

```python
self.teaching_points = {
    '1': {
        'shoulder_pan': 2071.0,
        'shoulder_lift': 885.0,
        'elbow_flex': 3081.0,
        'wrist_flex': 2953.0,
        'wrist_roll': 2076.0,
    },
    '2': {
        # Teaching Point 2
    },
    '3': {
        # Teaching Point 3
    },
}

self.gripper_positions = {
    'grip': 1800.0,
    'ungrip': 2300.0,
}
```

**목표 위치 Publisher 생성**

```python
self.joint_goal_pub = self.create_publisher(
    JointState,
    '/so_arm101/joint_goal',
    10
)
```

Playback 노드는 목표 관절 위치를 `/so_arm101/joint_goal` 토픽으로 발행합니다.

`feetech_driver_node`는 이 토픽을 구독하고 있으므로 메시지가 발행되면 해당 위치를 Feetech 모터에 전달합니다.

**Teaching Point 이동**

```python
def publish_arm_goal(self, point_name):
```

숫자 키를 누르면 선택한 Teaching Point에서 Arm 관절 다섯 개의 목표 위치를 가져옵니다.

```python
msg.name = ARM_JOINT_NAMES.copy()
msg.position = [
    positions[joint_name]
    for joint_name in ARM_JOINT_NAMES
]
```

예를 들어 `1` 키를 누르면 다음과 같은 메시지가 발행됩니다.

```yaml
name:
- shoulder_pan
- shoulder_lift
- elbow_flex
- wrist_flex
- wrist_roll
position:
- 2071.0
- 885.0
- 3081.0
- 2953.0
- 2076.0
```

**Gripper 이동**

```python
def publish_gripper_goal(self, position_name):
```

`+` 또는 `-` 키를 누르면 Gripper 관절 하나만 포함된 메시지를 발행합니다.

```yaml
name:
- gripper
position:
- 1800.0
```

이에 따라 Arm 자세는 유지되고 Gripper만 움직입니다.

#### 키보드 입력 구성

| 입력 키 | 실행 동작 |
| --- | --- |
| `1` | Teaching Point 1로 이동 |
| `2` | Teaching Point 2로 이동 |
| `3` | Teaching Point 3으로 이동 |
| `+` | Grip 위치로 이동 |
| `-` | Ungrip 위치로 이동 |
| `q` | Playback 노드 종료 |

---

#### setup.py 실행 파일 등록

![image.png](../assets/so_arm101_pick_placeplayback_node/image%201.png)

`so_arm101_pick_place/setup.py`의 `entry_points`를 다음과 같이 수정합니다.

```python
entry_points={
    'console_scripts': [
        'teaching_node = '
        'so_arm101_pick_place.teaching_node:main',

        'playback_node = '
        'so_arm101_pick_place.playback_node:main',
    ],
},
```

---

#### 패키지 빌드

ROS2 환경과 LeRobot 가상 환경을 적용합니다.

```bash
source /opt/ros/lyrical/setup.bash
source ~/project/rosws/lerobot/venv/bin/activate
```

워크스페이스로 이동해 필요한 패키지를 빌드합니다.

```bash
cd ~/project/ros2_ws

python -m colcon build \
  --packages-select \
  so_arm101_driver \
  so_arm101_pick_place
```

빌드가 완료되면 워크스페이스 환경을 적용합니다.

```bash
source ~/project/ros2_ws/install/setup.bash
```

빌드할 때마다 `build`, `install`, `log` 폴더를 삭제할 필요는 없습니다. 일반적인 소스 코드 변경은 해당 패키지만 다시 빌드하면 반영됩니다.

---

#### 노드 실행

두 개의 터미널을 준비합니다.

**1번 터미널: Feetech 드라이버 실행**

```bash
source /opt/ros/lyrical/setup.bash
source ~/project/rosws/lerobot/venv/bin/activate
source ~/project/ros2_ws/install/setup.bash

ros2 run so_arm101_driver feetech_driver_node
```

**2번 터미널: Playback 노드 실행**

```bash
source /opt/ros/lyrical/setup.bash
source ~/project/rosws/lerobot/venv/bin/activate
source ~/project/ros2_ws/install/setup.bash

ros2 run so_arm101_pick_place playback_node
```

---

#### 실행 결과

Playback 노드가 정상적으로 실행되면 다음과 비슷한 내용이 출력됩니다.

```
(venv) twiniex@lt:~/project/ros2_ws$ \
ros2 run so_arm101_pick_place playback_node

[INFO] [playback_node]:
Teaching data loaded:
/home/twiniex/project/ros2_ws/so_arm101_teaching.csv

[INFO] [playback_node]:
Teaching Points: ['1', '2', '3']

[INFO] [playback_node]:
Gripper Positions: ['grip', 'ungrip']

[INFO] [playback_node]:
SO-ARM101 Playback Node started.

====================================
 SO-ARM101 ROS2 Playback Program
====================================
1 : Move to Teaching Point 1
2 : Move to Teaching Point 2
3 : Move to Teaching Point 3
+ : Move to Grip Position
- : Move to Ungrip Position
q : Quit
====================================
```

`1` 키를 누르면 다음과 같이 목표 위치가 출력되면서 로봇이 Teaching Point 1로 이동합니다.

```
[INFO] [playback_node]:
Move to Teaching Point 1

[INFO] [playback_node]:
  shoulder_pan: 2071.0

[INFO] [playback_node]:
  shoulder_lift: 885.0

[INFO] [playback_node]:
  elbow_flex: 3081.0

[INFO] [playback_node]:
  wrist_flex: 2953.0

[INFO] [playback_node]:
  wrist_roll: 2076.0
```

`+` 키를 누르면 Grip 위치로 이동합니다.

```
[INFO] [playback_node]:
Move Gripper: grip -> 1800.0
```

- 키를 누르면 Ungrip 위치로 이동합니다.

```
[INFO] [playback_node]:
Move Gripper: ungrip -> 2300.0
```

---

#### 발행 토픽 확인

Playback 노드가 발행하는 메시지는 다음 명령으로 확인할 수 있습니다.

```bash
ros2 topic echo /so_arm101/joint_goal
```

다른 터미널에서 Playback 노드의 숫자 키를 누르면 다음과 같은 메시지가 출력됩니다.

```yaml
header:
  stamp:
    sec: 1782357000
    nanosec: 123456789
  frame_id: ''
name:
- shoulder_pan
- shoulder_lift
- elbow_flex
- wrist_flex
- wrist_roll
position:
- 2071.0
- 885.0
- 3081.0
- 2953.0
- 2076.0
velocity: []
effort: []
---
```

---

#### 전체 동작 구조

```
so_arm101_teaching.csv
          ↓
     playback_node
          ↓
/so_arm101/joint_goal
          ↓
feetech_driver_node
          ↓
Feetech Goal_Position
          ↓
    SO-ARM101 이동
```

---

#### 동작 확인 순서

1. 로봇 주변에 장애물이 없는지 확인합니다.
2. `feetech_driver_node`를 실행합니다.
3. `playback_node`를 실행합니다.
4. `1`, `2`, `3` 키를 한 번씩 눌러 각 자세로 이동하는지 확인합니다.
5. `+` 키로 Grip 동작을 확인합니다.
6.  키로 Ungrip 동작을 확인합니다.
7. `q` 키를 눌러 노드를 종료합니다.

---

#### 주의 사항

- 처음 실행할 때는 비상 정지가 가능한 상태에서 테스트합니다.
- 저장된 위치와 현재 위치의 차이가 크면 로봇이 빠르게 움직일 수 있습니다.
- 반드시 현재 위치와 가까운 Teaching Point부터 테스트합니다.
- 로봇의 작업 범위 안에 사람이나 물체가 없는지 확인합니다.
- 잘못된 CSV 값이 들어 있으면 관절이 예상하지 못한 방향으로 움직일 수 있습니다.
- 모터별 허용 위치 범위를 넘는 값을 사용하지 않습니다.
- Teaching 노드와 Playback 노드는 동일한 CSV 경로를 사용해야 합니다.

---

#### 마무리

이번 절에서는 Teaching 노드가 저장한 CSV 파일을 읽고, 키보드 입력에 따라 SO-ARM101을 저장된 위치로 이동시키는 Playback 노드를 작성했습니다.

Playback 노드는 Feetech 모터를 직접 제어하지 않습니다. 대신 `/so_arm101/joint_goal` 토픽으로 목표 위치를 발행하고, `feetech_driver_node`가 실제 모터 명령을 처리합니다.

다음 절에서는 Teaching Point와 Grip·Ungrip 위치를 정해진 순서로 자동 실행하는 `auto_pick_place_node`를 작성합니다.