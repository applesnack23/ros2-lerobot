# TF Tree 이해

앞 장까지 URDF를 구성하는 주요 요소를 하나씩 살펴보았습니다.

- Link
- Joint
- Origin
- Axis
- Limit
- Visual
- Collision
- Inertial

각 요소는 개별적인 정보를 정의하지만, 실제 로봇에서는 서로 연결되어 하나의 구조를 만듭니다.

예를 들어 Link는 로봇의 몸체를 정의하고, Joint는 Link와 Link 사이의 연결 관계를 정의합니다. 이 연결 관계를 좌표계 관점에서 표현한 구조가 `TF Tree`입니다.

TF는 `Transform`을 의미하며, 두 좌표계 사이의 위치와 방향 관계를 나타냅니다.

> **TF Tree는 로봇을 구성하는 좌표계의 부모·자식 연결 구조입니다.**
> 

URDF는 단순히 로봇의 외형만 정의하는 파일이 아닙니다. Link와 Joint의 연결 관계를 통해 로봇 전체의 좌표계 구조도 함께 정의합니다.

---

#### TF가 필요한 이유

로봇은 여러 개의 Link로 구성됩니다.

예를 들어 SO-ARM101은 다음과 같은 구조를 가질 수 있습니다.

- Base
- Shoulder
- Upper Arm
- Forearm
- Wrist
- Gripper

각 Link는 독립적인 좌표계를 가집니다.

로봇을 제어하려면 다음과 같은 정보를 알아야 합니다.

- Shoulder는 Base에서 얼마나 떨어져 있는가?
- Elbow는 현재 어디에 있는가?
- Wrist는 어느 방향을 바라보고 있는가?
- Gripper 끝단은 Base를 기준으로 어디에 있는가?
- Camera는 작업물을 어느 방향에서 바라보고 있는가?

이러한 위치와 방향의 관계를 계산하고 관리하기 위해 TF가 필요합니다.

TF는 각 좌표계에 대해 다음 정보를 나타냅니다.

- 부모 좌표계
- 자식 좌표계
- 부모 기준 자식의 위치
- 부모 기준 자식의 방향
- 해당 관계가 측정되거나 계산된 시간

---

#### Frame과 Transform

TF를 이해하려면 `Frame`과 `Transform`을 구분해야 합니다.

**Frame**

Frame은 위치와 방향을 표현하기 위한 좌표계입니다.

```
base_link Frame
shoulder_link Frame
upper_arm_link Frame
forearm_link Frame
wrist_link Frame
gripper_link Frame
```

각 Frame은 X, Y, Z축을 가지며 로봇의 특정 위치와 방향을 나타내는 기준이 됩니다.

**Transform**

Transform은 두 Frame 사이의 위치와 방향 관계입니다.

```
Parent Frame
    ↓ Transform
Child Frame
```

Transform에는 일반적으로 다음 정보가 포함됩니다.

- Translation: X, Y, Z 위치
- Rotation: Quaternion 또는 회전 행렬로 표현되는 방향

URDF에서는 Joint의 `origin`과 `axis`, 현재 Joint 값이 Transform을 계산하는 기반이 됩니다.

---

#### URDF 구조와 TF Tree

URDF는 Link와 Joint가 연결된 트리 구조를 가집니다.

```
base_link
└── shoulder_link
    └── upper_arm_link
        └── forearm_link
            └── wrist_link
                └── gripper_link
```

좌표계 관점에서 보면 다음과 같습니다.

```
base_link Frame
└── shoulder_link Frame
    └── upper_arm_link Frame
        └── forearm_link Frame
            └── wrist_link Frame
                └── gripper_link Frame
```

일반적으로 URDF의 각 Link 이름은 TF Frame 이름으로 사용됩니다.

하지만 URDF 파일을 작성했다고 해서 TF가 자동으로 발행되는 것은 아닙니다. `robot_state_publisher` 노드가 URDF와 JointState를 읽고 각 Link 사이의 Transform을 계산하여 TF Topic으로 발행합니다.

```
URDF
  └── Link와 Joint 구조

/joint_states
  └── 현재 Joint 위치

robot_state_publisher
  └── Transform 계산

/tf, /tf_static
  └── TF Tree 발행
```

---

#### Parent와 Child 관계

TF Tree는 부모와 자식 관계로 연결됩니다.

URDF에서는 Joint가 Parent Link와 Child Link를 연결합니다.

```xml
<joint name="elbow_joint" type="revolute">
    <parent link="upper_arm_link"/>
    <child link="forearm_link"/>

    <origin xyz="0 0 0.15" rpy="0 0 0"/>
    <axis xyz="0 1 0"/>
</joint>
```

위 구조는 다음과 같은 관계를 의미합니다.

```
upper_arm_link
       ↓
  elbow_joint
       ↓
forearm_link
```

TF 관점에서는 다음과 같이 표현할 수 있습니다.

```
Parent Frame: upper_arm_link
Child Frame:  forearm_link
```

`forearm_link`의 위치와 방향은 `upper_arm_link`를 기준으로 계산됩니다.

Joint의 `origin`은 Joint 위치가 0일 때 Parent Frame에서 Child Frame으로 이동하는 기본 Transform을 정의합니다.

Joint가 움직이면 `axis`를 기준으로 현재 Joint 값이 적용되어 Child Frame의 Transform이 변경됩니다.

---

#### TF Tree의 규칙

TF Tree는 이름 그대로 Tree 구조를 가져야 합니다.

일반적으로 다음 규칙을 따릅니다.

- 하나의 Root Frame에서 시작합니다.
- 각 Child Frame은 하나의 Parent Frame만 가집니다.
- 하나의 Parent는 여러 Child를 가질 수 있습니다.
- 부모·자식 관계가 순환해서는 안 됩니다.
- 모든 Frame 이름은 TF Tree 안에서 고유해야 합니다.

다음 구조는 정상적인 Tree입니다.

```
base_link
├── arm_link
│   └── gripper_link
└── camera_link
```

`base_link`는 `arm_link`와 `camera_link`라는 두 개의 Child를 가질 수 있습니다.

반면 다음과 같은 순환 구조는 사용할 수 없습니다.

```
link_a
  ↓
link_b
  ↓
link_c
  ↓
link_a
```

이 구조는 시작점과 끝점을 명확하게 결정할 수 없기 때문에 TF Tree로 구성할 수 없습니다.

---

#### Root Frame

모든 TF Tree에는 시작점이 되는 Frame이 필요합니다.

이것을 `Root Frame`이라고 합니다.

로봇 모델에서는 일반적으로 `base_link`가 Root Frame으로 사용됩니다.

```
base_link
└── shoulder_link
    └── upper_arm_link
        └── forearm_link
```

Root Frame은 URDF 내부에서 Parent Joint를 가지지 않는 Link입니다.

다만 `base_link`라는 이름을 사용했다고 해서 자동으로 바닥이나 세계 좌표계에 고정되는 것은 아닙니다.

실제 시스템에서는 다음과 같은 상위 Frame이 추가될 수 있습니다.

```
world
└── base_link
    └── shoulder_link
```

또는 이동 로봇에서는 다음과 같은 구조를 사용할 수 있습니다.

```
map
└── odom
    └── base_link
        └── sensor_link
```

따라서 Root Frame은 현재 TF Tree에서 가장 위에 있는 기준 좌표계이며, 항상 실제 세계에 고정된 좌표계라는 의미는 아닙니다.

---

#### 고정 Transform과 동적 Transform

TF는 움직임 여부에 따라 고정 Transform과 동적 Transform으로 구분할 수 있습니다.

**고정 Transform**

`fixed` Joint처럼 시간이 지나도 변하지 않는 좌표 관계입니다.

```xml
<joint name="camera_joint" type="fixed">
    <parent link="base_link"/>
    <child link="camera_link"/>
    <origin xyz="0.1 0 0.2" rpy="0 0 0"/>
</joint>
```

이 경우 `base_link`와 `camera_link` 사이의 위치와 방향은 항상 같습니다.

고정 Transform은 일반적으로 `/tf_static` Topic으로 발행됩니다.

**동적 Transform**

`revolute`, `continuous`, `prismatic` Joint처럼 Joint 값에 따라 계속 변하는 좌표 관계입니다.

```xml
<joint name="shoulder_pan" type="revolute">
    <parent link="base_link"/>
    <child link="shoulder_link"/>
    <origin xyz="0 0 0.1" rpy="0 0 0"/>
    <axis xyz="0 0 1"/>
    <limit lower="-1.57" upper="1.57" effort="5.0" velocity="1.0"/>
</joint>
```

`shoulder_pan`의 각도가 변하면 `shoulder_link`의 위치와 방향도 변합니다.

동적 Transform은 일반적으로 `/tf` Topic으로 계속 발행됩니다.

| 구분 | 대표 Joint | 발행 Topic | 변화 여부 |
| --- | --- | --- | --- |
| 고정 Transform | `fixed` | `/tf_static` | 변하지 않음 |
| 동적 Transform | `revolute`, `continuous`, `prismatic` | `/tf` | Joint 값에 따라 변화 |

---

#### robot_state_publisher의 역할

URDF는 로봇 구조를 정의하지만, 직접 TF를 발행하지는 않습니다.

실제로 TF를 계산하고 발행하는 대표적인 노드가 `robot_state_publisher`입니다.

`robot_state_publisher`는 다음 정보를 사용합니다.

- URDF의 Link 구조
- URDF의 Joint 연결 관계
- Joint Origin
- Joint Axis
- `/joint_states`의 현재 Joint 위치

처리 과정은 다음과 같습니다.

```
URDF에서 로봇 구조 확인
           ↓
/joint_states에서 현재 관절값 수신
           ↓
각 Parent와 Child 사이의 Transform 계산
           ↓
/tf와 /tf_static으로 발행
```

예를 들어 `/joint_states`에 다음 값이 들어오면,

```
name:
- shoulder_pan
- shoulder_lift
position:
- 0.5
- 0.8
```

`robot_state_publisher`는 URDF 구조를 참고하여 `shoulder_link`와 `upper_arm_link`의 현재 Transform을 계산합니다.

---

#### Transform 전파

TF Tree에서는 부모 Frame의 변화가 아래에 연결된 모든 Child Frame에 영향을 줍니다.

예를 들어 다음 구조를 생각해 보겠습니다.

```
base_link
└── shoulder_link
    └── elbow_link
        └── wrist_link
            └── gripper_link
```

Shoulder Joint가 움직이면 다음 Frame의 위치와 방향이 모두 변경됩니다.

- shoulder_link
- elbow_link
- wrist_link
- gripper_link

Elbow Joint가 움직이면 Elbow 아래에 있는 Frame만 변경됩니다.

- elbow_link
- wrist_link
- gripper_link

이처럼 상위 Transform의 변화가 하위 Frame에 연속해서 적용되는 것을 Transform 전파라고 볼 수 있습니다.

예를 들어 각 Joint가 다음 값을 가진다고 가정해 보겠습니다.

```
Shoulder = +30°
Elbow    = +20°
Wrist    = -10°
```

Gripper의 최종 위치와 방향은 각 Joint를 독립적으로 적용하는 것이 아니라 Root Frame부터 Gripper까지 연결된 모든 Transform을 순서대로 결합하여 계산합니다.

---

#### Transform의 누적

`base_link`에서 `gripper_link`까지의 Transform을 구하려면 중간 Transform을 모두 결합해야 합니다.

```
base_link
    ↓ T₁
shoulder_link
    ↓ T₂
elbow_link
    ↓ T₃
wrist_link
    ↓ T₄
gripper_link
```

이를 수식으로 표현하면 다음과 같습니다.

$$
{}^{base}T_{gripper}
=
{}^{base}T_{shoulder}
{}^{shoulder}T_{elbow}
{}^{elbow}T_{wrist}
{}^{wrist}T_{gripper}
$$

여기서 T는 위치와 회전을 함께 표현하는 변환 행렬입니다.

이 계산을 통해 `base_link`를 기준으로 `gripper_link`의 최종 위치와 방향을 구할 수 있습니다.

사용자가 직접 모든 변환 행렬을 계산할 수도 있지만, ROS2의 TF2 시스템은 이러한 계산을 자동으로 처리합니다.

---

#### Forward Kinematics

TF Tree는 Forward Kinematics의 기반이 됩니다.

Forward Kinematics는 현재 Joint 값을 이용하여 로봇 끝단의 위치와 방향을 계산하는 과정입니다.

```
Joint 값
   ↓
Link Transform 계산
   ↓
Transform 순차 결합
   ↓
End-Effector 위치와 방향 계산
```

예를 들어 다음 Joint 값이 주어졌다고 가정해 보겠습니다.

```
shoulder_pan  = 30°
shoulder_lift = 45°
elbow_flex    = 20°
```

Forward Kinematics는 이 값을 사용하여 Gripper의 위치와 방향을 계산합니다.

```
Joint Position → End-Effector Pose
```

여기서 Pose는 다음 두 정보를 포함합니다.

- Position: X, Y, Z 위치
- Orientation: Roll, Pitch, Yaw 또는 Quaternion 방향

TF Tree를 따라 Transform을 결합하는 과정이 바로 Forward Kinematics의 기본 원리입니다.

---

#### End-Effector

End-Effector는 로봇이 실제 작업을 수행하는 끝단을 의미합니다.

예를 들면 다음과 같습니다.

- Gripper
- 용접 토치
- 흡착 패드
- 드릴
- 카메라
- 절삭 공구

SO-ARM101에서는 일반적으로 Gripper 또는 Gripper 끝단을 End-Effector로 사용할 수 있습니다.

```
base_link
└── shoulder_link
    └── upper_arm_link
        └── forearm_link
            └── wrist_link
                └── gripper_link
                    └── tool_frame
```

여기서 실제 작업 기준점은 `gripper_link` 자체가 아니라 별도로 정의한 `tool_frame`이 될 수 있습니다.

따라서 Tree에서 가장 마지막에 있는 Link가 항상 정확한 작업 기준점인 것은 아닙니다. 로봇의 용도에 따라 별도의 Tool Frame을 정의하는 것이 일반적입니다.

---

#### Tool Frame

Tool Frame은 End-Effector의 실제 작업 기준점을 나타냅니다.

예를 들어 Gripper의 Link Frame이 손목 연결부에 있고, 실제 물체를 잡는 지점이 그보다 0.1 m 앞에 있다면 다음과 같이 고정 Joint를 추가할 수 있습니다.

```xml
<link name="tool_frame"/>

<joint name="tool_frame_joint" type="fixed">
    <parent link="gripper_link"/>
    <child link="tool_frame"/>
    <origin xyz="0.1 0 0" rpy="0 0 0"/>
</joint>
```

그러면 TF Tree는 다음과 같이 확장됩니다.

```
gripper_link
└── tool_frame
```

MoveIt에서 목표 위치를 지정하거나 Pick & Place 작업을 수행할 때는 `tool_frame`을 작업 기준으로 사용할 수 있습니다.

---

#### TF와 URDF의 관계

URDF와 TF Tree는 밀접하게 관련되어 있지만 같은 것은 아닙니다.

| 구분 | URDF | TF Tree |
| --- | --- | --- |
| 역할 | 로봇의 구조 정의 | 좌표계 관계 표현 |
| 주요 정보 | Link, Joint, Origin, Axis | Parent, Child, Translation, Rotation |
| 형태 | XML 또는 Xacro 파일 | 실행 중 관리되는 좌표계 데이터 |
| 변화 여부 | 기본적으로 정적 파일 | Joint 상태에 따라 실시간 변화 |
| 사용 노드 | `robot_state_publisher`가 읽음 | 여러 ROS2 노드가 조회하고 사용 |

관계를 정리하면 다음과 같습니다.

```
URDF
  + JointState
        ↓
robot_state_publisher
        ↓
TF Tree
```

따라서 URDF는 TF Tree를 만들기 위한 구조 정보이고, TF Tree는 실행 중인 로봇의 실제 좌표 관계입니다.

---

#### RViz2에서 TF 확인하기

RViz2에서는 TF Tree를 시각적으로 확인할 수 있습니다.

일반적인 확인 과정은 다음과 같습니다.

1. URDF 또는 Xacro를 준비합니다.
2. `robot_state_publisher`를 실행합니다.
3. 필요한 경우 `joint_state_publisher_gui`를 실행합니다.
4. RViz2를 실행합니다.
5. `TF` Display를 추가합니다.
6. `RobotModel` Display를 추가합니다.
7. Fixed Frame을 설정합니다.

TF Display에서는 다음 내용을 확인할 수 있습니다.

- 각 Frame의 위치
- 각 Frame의 X, Y, Z축 방향
- Parent와 Child 연결 관계
- Frame 이름
- 현재 Transform 상태

축은 일반적으로 다음 색상으로 표시됩니다.

```
X축 = 빨간색
Y축 = 초록색
Z축 = 파란색
```

RViz2를 이용하면 다음과 같은 URDF 오류를 쉽게 발견할 수 있습니다.

- Joint Origin 위치 오류
- Axis 방향 오류
- Parent와 Child 연결 오류
- Link 방향 오류
- 끊어진 TF 관계
- 잘못된 Root Frame
- End-Effector Frame 위치 오류

---

#### Fixed Frame

RViz2에는 `Fixed Frame` 설정이 있습니다.

Fixed Frame은 화면에 표시되는 모든 데이터를 변환할 기준 좌표계입니다.

로봇만 확인하는 경우 다음과 같이 설정할 수 있습니다.

```
Fixed Frame = base_link
```

시뮬레이션이나 이동 로봇 시스템에서는 다음과 같은 Frame을 사용할 수 있습니다.

```
world
map
odom
base_link
```

Fixed Frame으로 설정한 좌표계가 TF Tree에 존재하지 않으면 다음과 같은 오류가 발생할 수 있습니다.

```
Fixed Frame [world] does not exist
```

이 경우 TF Tree에 실제로 존재하는 Frame 이름으로 변경하거나 필요한 Transform을 추가해야 합니다.

---

#### view_frames로 TF Tree 확인하기

`view_frames`는 현재 발행되고 있는 TF 정보를 수집하여 전체 구조를 파일로 만들어주는 도구입니다.

다음 명령을 실행합니다.

```bash
ros2 run tf2_tools view_frames
```

명령을 실행하면 일정 시간 동안 TF 데이터를 수집한 뒤 일반적으로 `frames.pdf` 파일을 생성합니다.

```
frames.pdf
```

이 파일에서는 다음 내용을 확인할 수 있습니다.

- TF Tree 전체 구조
- Parent와 Child 관계
- Transform 발행 주기
- 최근 Transform 시간
- 각 Frame의 Broadcaster 정보

현재 폴더에 생성된 파일은 다음과 같이 확인할 수 있습니다.

```bash
ls
```

TF 구조가 끊어졌거나 같은 로봇의 Frame이 여러 개의 Tree로 분리되어 있다면 `frames.pdf`에서 쉽게 확인할 수 있습니다.

---

#### tf2_echo로 좌표 관계 확인하기

두 Frame 사이의 위치와 방향을 실시간으로 확인하려면 `tf2_echo`를 사용합니다.

기본 명령 형식은 다음과 같습니다.

```bash
ros2 run tf2_ros tf2_echo <source_frame> <target_frame>
```

예를 들어 `base_link`를 기준으로 `gripper_link`의 위치와 방향을 확인하려면 다음과 같이 실행합니다.

```bash
ros2 run tf2_ros tf2_echo base_link gripper_link
```

출력 예시는 다음과 같습니다.

```
Translation: [0.265, 0.000, 0.221]
Rotation: [0.249, -0.379, -0.489, 0.745]
```

여기서 Translation은 두 Frame 사이의 X, Y, Z 위치 차이를 의미하고, Rotation은 방향 차이를 나타냅니다.

`tf2_echo`는 다음 내용을 확인할 때 유용합니다.

- End-Effector 현재 위치
- Camera와 Base 사이의 위치
- 센서 장착 방향
- Tool Frame 위치
- Joint 움직임에 따른 좌표 변화

---

#### rqt_tf_tree로 확인하기

TF Tree를 GUI 환경에서 확인하려면 `rqt_tf_tree`를 사용할 수 있습니다.

```bash
ros2 run rqt_tf_tree rqt_tf_tree
```

또는 `rqt`를 실행한 뒤 TF Tree 플러그인을 열 수 있습니다.

```bash
rqt
```

GUI에서는 Frame의 Parent와 Child 관계를 트리 형태로 확인할 수 있습니다.

다만 배포판이나 설치된 패키지 구성에 따라 플러그인이 기본 설치되어 있지 않을 수 있습니다.

---

#### TF 문제 확인

TF 관련 문제가 발생하면 다음 항목을 확인합니다.

| 문제 | 확인할 항목 |
| --- | --- |
| RViz2에서 로봇이 보이지 않음 | Fixed Frame과 TF 존재 여부 |
| 일부 Link가 보이지 않음 | URDF의 Parent·Child 관계 |
| Frame 연결이 끊어짐 | Joint 정의와 `robot_state_publisher` |
| 움직이는 Joint가 반영되지 않음 | `/joint_states` 발행 여부 |
| Link가 잘못된 위치에 표시됨 | Joint Origin의 `xyz` |
| Link 방향이 잘못됨 | Joint Origin의 `rpy` |
| Joint가 반대로 움직임 | Joint Axis 방향 |
| End-Effector 위치가 잘못됨 | 전체 Joint 연결 및 Tool Frame |
| `world` Frame 오류 발생 | `world` Transform 정의 여부 |
| 같은 Frame이 중복됨 | Frame 이름과 노드의 TF 발행 설정 |

---

#### TF Tree 확인 순서

URDF를 작성한 뒤에는 다음 순서로 TF Tree를 확인하는 것이 좋습니다.

1. Root Link를 확인합니다.
2. 모든 Link가 Joint로 연결되어 있는지 확인합니다.
3. Parent와 Child 방향을 확인합니다.
4. `robot_state_publisher`를 실행합니다.
5. `/joint_states`가 정상적으로 발행되는지 확인합니다.
6. RViz2에서 TF Display를 추가합니다.
7. 각 Frame의 위치와 축 방향을 확인합니다.
8. `view_frames`로 전체 Tree를 확인합니다.
9. `tf2_echo`로 End-Effector 좌표를 확인합니다.

---

#### 정리

TF Tree는 로봇의 전체 좌표계 연결 구조입니다.

- 각 Link는 하나의 Frame으로 사용됩니다.
- Joint가 Parent Frame과 Child Frame을 연결합니다.
- Root Frame에서 전체 구조가 시작됩니다.
- 고정 Joint는 `/tf_static`으로 발행됩니다.
- 움직이는 Joint는 `/tf`로 발행됩니다.
- `robot_state_publisher`가 URDF와 JointState를 이용해 TF를 계산합니다.
- 부모 Transform의 변화는 아래 Child Frame에 전달됩니다.
- TF Tree는 Forward Kinematics의 기반이 됩니다.
- End-Effector와 Tool Frame의 위치를 계산할 수 있습니다.
- RViz2, `view_frames`, `tf2_echo`로 TF를 확인할 수 있습니다.

가장 중요한 관계는 다음과 같습니다.

> **URDF는 로봇의 구조를 정의하고, TF Tree는 실행 중인 로봇의 좌표 관계를 표현합니다.**
> 

URDF를 정확하게 이해하고 작성하려면 Link와 Joint뿐 아니라 이들이 만들어내는 TF Tree를 함께 이해해야 합니다. TF Tree가 올바르게 구성되어야 RViz2 시각화, Gazebo 시뮬레이션, MoveIt 경로 계획 및 실제 로봇 제어가 정상적으로 동작할 수 있습니다.
