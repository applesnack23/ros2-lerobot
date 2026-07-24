# RViz 활용

#### JointState와 TF 확인

앞 절에서는 SO-ARM101의 URDF를 ROS2 패키지로 구성하고, `joint_state_publisher_gui`, `robot_state_publisher`, RViz2를 실행하여 로봇 모델을 확인했습니다.

이번 절에서는 Joint 값을 변경했을 때 다음 데이터가 어떻게 생성되고 연결되는지 확인합니다.

- `/joint_states`: 각 Joint의 현재 값
- `/tf`: 움직이는 Frame 사이의 좌표 관계
- `/tf_static`: 변하지 않는 고정 Frame 관계

전체 데이터 흐름은 다음과 같습니다.

```
joint_state_publisher_gui
        ↓
   /joint_states
        ↓
robot_state_publisher
        ↓
  /tf, /tf_static
        ↓
      RViz2
```

`joint_state_publisher_gui`에서 Joint 값을 변경하면 `/joint_states` Topic으로 값이 발행됩니다.

`robot_state_publisher`는 이 값과 URDF를 이용해 각 Link Frame의 위치와 방향을 계산하고 `/tf`로 발행합니다. RViz2는 TF를 이용하여 로봇 모델의 자세를 화면에 반영합니다.

---

#### SO-ARM101 Joint 구성

SO-ARM101은 다음과 같은 여섯 개의 주요 Joint를 사용합니다.

- `shoulder_pan`
- `shoulder_lift`
- `elbow_flex`
- `wrist_flex`
- `wrist_roll`
- `gripper`

각 Joint의 역할은 다음과 같습니다.

| Joint | 역할 |
| --- | --- |
| `shoulder_pan` | Arm 전체를 좌우 방향으로 회전 |
| `shoulder_lift` | Shoulder를 들어 올리거나 내림 |
| `elbow_flex` | Elbow를 굽히거나 펼침 |
| `wrist_flex` | Wrist를 굽히거나 펼침 |
| `wrist_roll` | Wrist 또는 Tool을 회전 |
| `gripper` | Gripper 열기 및 닫기 |

`joint_state_publisher_gui`에서 각 Joint의 슬라이더를 움직이면 다음 동작을 확인할 수 있습니다.

1. Gripper Open 및 Close
2. Wrist Roll 회전
3. Wrist Flex 회전
4. Elbow Flex 회전
5. Shoulder Lift 회전
6. Shoulder Pan 회전

Joint를 검증할 때는 여러 Joint를 동시에 움직이기보다 한 번에 하나씩 움직이는 것이 좋습니다.

이를 통해 다음 항목을 쉽게 확인할 수 있습니다.

- 해당 Joint가 올바른 Link를 움직이는가?
- 회전축이 올바른가?
- 양수와 음수 방향이 실제 로봇과 일치하는가?
- Joint Limit가 올바르게 적용되는가?
- 하위 Link들이 함께 움직이는가?

---

#### 현재 Joint 값 확인

현재 Joint 값은 `/joint_states` Topic으로 확인할 수 있습니다.

새 터미널을 열고 ROS2 환경을 적용합니다.

```bash
source /opt/ros/lyrical/setup.bash
source ~/project/so_arm101_ws/install/setup.bash
```

다음 명령을 실행합니다.

```bash
ros2 topic echo /joint_states
```

`joint_state_publisher_gui`의 슬라이더를 움직이면 현재 Joint 값이 실시간으로 출력됩니다.

출력 형식은 다음과 같습니다.

```
header:
  stamp:
    sec: 1782273667
    nanosec: 548198246
  frame_id:''
name:
- shoulder_pan
- shoulder_lift
- elbow_flex
- wrist_flex
- wrist_roll
- gripper
position:
- 0.0
- 0.5
- -0.3
- 0.2
- 0.0
- 0.4
velocity: []
effort: []
```

한 번만 확인하려면 `--once` 옵션을 사용할 수 있습니다.

```bash
ros2 topic echo /joint_states --once
```

---

#### JointState 메시지 구조

`/joint_states`는 `sensor_msgs/msg/JointState` 타입을 사용합니다.

메시지 구조는 다음 명령으로 확인할 수 있습니다.

```bash
ros2 interface show sensor_msgs/msg/JointState
```

JointState에는 다음과 같은 데이터가 포함됩니다.

| 필드 | 의미 |
| --- | --- |
| `header` | 메시지가 생성된 시간과 기준 Frame |
| `name` | Joint 이름 목록 |
| `position` | 각 Joint의 현재 위치 |
| `velocity` | 각 Joint의 현재 속도 |
| `effort` | 각 Joint에 작용하는 힘 또는 토크 |

`name`과 `position`은 같은 인덱스를 사용합니다.

예를 들어 다음 데이터가 있다고 가정해 보겠습니다.

```yaml
name:
- shoulder_pan
- shoulder_lift
position:
- 0.5
- 0.8
```

이 데이터는 다음을 의미합니다.

```
shoulder_pan  = 0.5 rad
shoulder_lift = 0.8 rad
```

회전 Joint의 `position` 단위는 라디안입니다.

Gripper가 Revolute Joint로 정의되어 있다면 Gripper 값도 라디안으로 표현됩니다. Prismatic Joint로 정의되어 있다면 미터 단위를 사용합니다.

---

#### JointState와 RViz2의 관계

RViz2가 JointState를 이용해 직접 Link 위치를 계산하는 것은 아닙니다.

실제 처리 과정은 다음과 같습니다.

```
joint_state_publisher_gui
        ↓
JointState 발행
        ↓
robot_state_publisher
        ↓
URDF + JointState로 TF 계산
        ↓
RViz2가 TF를 이용해 모델 표시
```

`joint_state_publisher_gui`는 Joint 값을 생성하는 테스트용 도구입니다.

실제 SO-ARM101을 연결한 경우에는 모터 Driver Node가 현재 모터 위치를 읽고 `/joint_states`를 발행해야 합니다.

```
RViz2 검증 환경
joint_state_publisher_gui → /joint_states

실제 로봇 환경
feetech_driver_node → /joint_states
```

두 환경에서 동일한 `/joint_states` 인터페이스를 사용하면 RViz2와 `robot_state_publisher`는 데이터가 어디에서 생성되었는지 알 필요 없이 동일하게 동작할 수 있습니다.

---

#### TF 데이터 확인

TF는 각 Frame의 부모·자식 관계와 상대적인 위치 및 방향을 나타냅니다.

현재 발행되는 동적 TF를 확인하려면 다음 명령을 사용합니다.

```bash
ros2 topic echo /tf --once
```

출력은 여러 개의 Transform으로 구성됩니다.

```
transforms:
- header:
    stamp:
      sec: 1782273667
      nanosec: 548198246
    frame_id: upper_arm_link
  child_frame_id: lower_arm_link
  transform:
    translation:
      x: -0.11257
      y: -0.028
      z: 1.73763e-16
    rotation:
      x: -4.732286827821173e-16
      y: 1.441762319358764e-17
      z: 0.41194266517412853
      w: 0.9112097676217239
```

이 Transform의 부모와 자식은 다음과 같습니다.

```
Parent Frame = upper_arm_link
Child Frame  = lower_arm_link
```

즉, `upper_arm_link`를 기준으로 `lower_arm_link`의 위치와 방향을 나타냅니다.

---

#### TF 메시지 구조

각 Transform은 다음과 같은 구조를 가집니다.

```
TransformStamped
├── header
│   ├── stamp
│   └── frame_id
├── child_frame_id
└── transform
    ├── translation
    │   ├── x
    │   ├── y
    │   └── z
    └── rotation
        ├── x
        ├── y
        ├── z
        └── w
```

각 항목의 의미는 다음과 같습니다.

| 항목 | 의미 |
| --- | --- |
| `stamp` | Transform이 계산된 시간 |
| `frame_id` | Parent Frame |
| `child_frame_id` | Child Frame |
| `translation` | Parent를 기준으로 한 Child의 위치 |
| `rotation` | Parent를 기준으로 한 Child의 방향 |

---

#### Translation

Translation은 Parent Frame에서 Child Frame까지의 위치 차이를 나타냅니다.

```yaml
translation:
  x: -0.11257
  y: -0.028
  z: 1.73763e-16
```

이를 해석하면 다음과 같습니다.

```
X = -0.11257 m
Y = -0.028 m
Z = 약 0 m
```

`1.73763e-16`과 같이 매우 작은 값은 부동소수점 계산 오차로 발생한 값입니다.

```
1.73763e-16
= 0.000000000000000173763
≈ 0
```

따라서 실질적으로 0으로 해석할 수 있습니다.

---

#### Rotation

Rotation은 Quaternion 형식으로 표현됩니다.

```
rotation:
  x: -4.732286827821173e-16
  y: 1.441762319358764e-17
  z: 0.41194266517412853
  w: 0.9112097676217239
```

Quaternion은 네 개의 값으로 3차원 회전을 표현합니다.

```
x, y, z, w
```

Quaternion은 RPY보다 짐벌락 문제가 없고, 회전을 안정적으로 계산할 수 있기 때문에 TF에서 기본 회전 표현 방식으로 사용됩니다.

직접 각도를 확인하려면 `tf2_echo`를 사용하는 것이 편리합니다.

---

#### SO-ARM101 TF 관계 분석

출력된 `/tf` 데이터를 Parent와 Child 관계로 정리하면 다음과 같습니다.

| Parent Frame | Child Frame |
| --- | --- |
| `base_link` | `shoulder_link` |
| `shoulder_link` | `upper_arm_link` |
| `upper_arm_link` | `lower_arm_link` |
| `lower_arm_link` | `wrist_link` |
| `wrist_link` | `gripper_link` |
| `gripper_link` | `moving_jaw_so101_v1_link` |

이를 하나의 Tree로 표현하면 다음과 같습니다.

```
base_link
└── shoulder_link
    └── upper_arm_link
        └── lower_arm_link
            └── wrist_link
                └── gripper_link
                    └── moving_jaw_so101_v1_link
```

이 구조에서는 다음 항목을 확인할 수 있습니다.

- Link Frame: 7개
- Parent·Child 연결: 6개
- 움직이는 Joint: 6개

여기서 중요한 점은 `/tf` 메시지에 Joint 이름이 직접 표시되지 않는다는 것입니다.

TF에는 다음 정보만 표시됩니다.

- Parent Frame
- Child Frame
- Translation
- Rotation

Joint 이름과 타입을 확인하려면 URDF를 함께 살펴봐야 합니다.

예를 들어 다음 TF 관계가 있다고 가정해 보겠습니다.

```
base_link → shoulder_link
```

두 Link를 연결하는 Joint 이름이 `shoulder_pan`인지 확인하려면 URDF에서 다음 내용을 찾아야 합니다.

```xml
<joint name="shoulder_pan" type="revolute">
    <parent link="base_link"/>
    <child link="shoulder_link"/>
</joint>
```

따라서 다음과 같이 구분해야 합니다.

```
TF Tree = Link Frame 연결 관계
URDF    = Link를 연결한 Joint의 이름과 속성
```

---

#### world와 base_link 확인

앞 절의 Launch 파일에서는 다음과 같은 고정 Transform을 추가했습니다.

```
world
└── base_link
```

`world`와 `base_link` 관계는 움직이지 않는 고정 Transform이므로 일반적으로 `/tf_static`으로 발행됩니다.

고정 TF를 확인하려면 다음 명령을 사용합니다.

```bash
ros2 topic echo /tf_static --once
```

출력 예시는 다음과 같습니다.

```
transforms:
- header:
    frame_id: world
  child_frame_id: base_link
  transform:
    translation:
      x: 0.0
      y: 0.0
      z: 0.0
    rotation:
      x: 0.0
      y: 0.0
      z: 0.0
      w: 1.0
```

전체 TF Tree는 다음과 같습니다.

```
world
└── base_link
    └── shoulder_link
        └── upper_arm_link
            └── lower_arm_link
                └── wrist_link
                    └── gripper_link
                        └── moving_jaw_so101_v1_link
```

정리하면 다음과 같습니다.

- `world → base_link`: 고정 Transform
- `base_link` 아래의 회전 관절: Joint 값에 따라 변하는 Transform
- `base_link`: 움직이지 않는 기준 Link
- 나머지 Child Link: Joint 값에 따라 위치와 방향이 변경됨

---

#### /tf와 /tf_static의 차이

| 구분 | `/tf` | `/tf_static` |
| --- | --- | --- |
| 용도 | 움직이는 Transform | 변하지 않는 Transform |
| 대표 대상 | Revolute, Continuous, Prismatic Joint | Fixed Joint |
| 발행 방식 | 계속 반복해서 발행 | 한 번 발행 후 유지 가능 |
| 예시 | `base_link → shoulder_link` | `world → base_link` |

URDF의 Fixed Joint는 `robot_state_publisher`에 의해 `/tf_static`으로 발행될 수 있습니다.

움직이는 Joint의 Transform은 `/joint_states` 값이 변경될 때마다 다시 계산되어 `/tf`로 발행됩니다.

#### JointState와 TF의 차이

JointState와 TF는 서로 관련되어 있지만 같은 데이터는 아닙니다.

| 구분 | JointState | TF |
| --- | --- | --- |
| 주요 Topic | `/joint_states` | `/tf`, `/tf_static` |
| 데이터 | Joint 이름과 현재 값 | Frame 사이 위치와 방향 |
| 기준 | 관절 상태 | 좌표계 관계 |
| 주요 생성 노드 | Joint Driver 또는 Joint State Publisher | `robot_state_publisher` |
| 단위 | rad 또는 m | m와 Quaternion |
| 사용 목적 | 현재 Joint 상태 전달 | Link 위치와 방향 계산 |

예를 들어 다음 JointState가 들어오면,

```
shoulder_pan = 0.5 rad
```

`robot_state_publisher`는 URDF에서 다음 정보를 함께 확인합니다.

- `shoulder_pan`의 Parent Link
- `shoulder_pan`의 Child Link
- Joint Origin
- Joint Axis
- Joint 타입

그리고 `base_link`에서 `shoulder_link`로 이어지는 Transform을 계산해 `/tf`로 발행합니다.

---

#### tf2_echo를 이용한 좌표 확인

`/tf` 전체 내용을 직접 읽는 것보다 두 Frame 사이의 관계만 확인하는 것이 더 편리할 수 있습니다.

명령 형식은 다음과 같습니다.

```bash
ros2 run tf2_ros tf2_echo <parent_frame> <child_frame>
```

예를 들어 `base_link`에서 `gripper_link`까지의 좌표 관계를 확인하려면 다음과 같이 실행합니다.

```bash
ros2 run tf2_ros tf2_echo base_link gripper_link
```

Gripper의 움직이는 Jaw까지 확인하려면 다음 명령을 사용할 수 있습니다.

```bash
ros2 run tf2_ros tf2_echo \
    base_link moving_jaw_so101_v1_link
```

출력에서는 다음 정보를 확인할 수 있습니다.

- Translation
- Quaternion
- RPY
- Degree
- 변환 행렬

`joint_state_publisher_gui`에서 Joint 슬라이더를 움직이면 값이 실시간으로 변경됩니다.

---

#### rqt_tf_tree

`/tf`를 터미널에서 직접 확인하면 Transform이 길게 출력되기 때문에 전체 구조를 한눈에 파악하기 어렵습니다.

`rqt_tf_tree`를 사용하면 TF Frame의 Parent와 Child 관계를 GUI 환경에서 확인할 수 있습니다.

> `rqt_tf_tree`에는 Joint 이름이 아니라 TF Frame 이름이 표시됩니다.
> 

---

#### rqt_tf_tree 설치

먼저 패키지 목록을 갱신합니다.

```bash
sudo apt update
```

`rqt_tf_tree`를 설치합니다.

```bash
sudo apt install ros-lyrical-rqt-tf-tree
```

설치는 처음 한 번만 수행하면 됩니다.

---

#### rqt_tf_tree 실행

ROS2 환경과 SO-ARM101 Workspace Overlay를 적용합니다.

```bash
source /opt/ros/lyrical/setup.bash
source ~/project/so_arm101_ws/install/setup.bash
```

다음 명령으로 실행합니다.

```bash
ros2 run rqt_tf_tree rqt_tf_tree
```

정상적으로 실행되면 SO-ARM101의 TF Tree가 GUI에 표시됩니다.

```
world
└── base_link
    └── shoulder_link
        └── upper_arm_link
            └── lower_arm_link
                └── wrist_link
                    └── gripper_link
                        └── moving_jaw_so101_v1_link
```

`rqt_tf_tree`를 실행한 뒤 구조가 바로 표시되지 않는다면 새로고침 버튼을 눌러 확인합니다.

---

#### 플러그인을 찾지 못하는 오류

다음과 같은 오류가 발생할 수 있습니다.

```
qt_gui_main() found no plugin matching
"rqt_tf_tree.tf_tree.RosTfTree"

try passing the option "--force-discover"
```

이 메시지는 rqt 내부의 플러그인 목록에서 `rqt_tf_tree` 플러그인을 찾지 못했다는 의미입니다.

패키지가 정상적으로 설치되었더라도 다음과 같은 이유로 발생할 수 있습니다.

- ROS2 환경이 적용되지 않음
- rqt 플러그인 검색 캐시가 갱신되지 않음
- Workspace Overlay가 적용되지 않음
- 설치 직후 기존 터미널을 계속 사용함

먼저 ROS2 환경을 다시 적용합니다.

```bash
source /opt/ros/lyrical/setup.bash
source ~/project/so_arm101_ws/install/setup.bash
```

그다음 `--force-discover` 옵션을 추가하여 실행합니다.

```bash
ros2 run rqt_tf_tree rqt_tf_tree --force-discover
```

- `-force-discover`는 저장된 플러그인 목록만 사용하는 대신 설치된 rqt 플러그인을 다시 검색하도록 요청하는 옵션입니다.

강제 검색 후 정상적으로 실행되면 이후에는 일반 명령으로 실행할 수 있습니다.

```bash
ros2 run rqt_tf_tree rqt_tf_tree
```

---

#### 오류가 계속 발생하는 경우

- `-force-discover`를 사용해도 실행되지 않는다면 다음 순서로 확인합니다.

**패키지 설치 확인**

```abap
ros2 pkg list | grep rqt_tf_tree
```

다음과 같이 출력되어야 합니다.

```
rqt_tf_tree
```

**실행 파일 확인**

```bash
ros2 pkg executables rqt_tf_tree
```

**ROS2 환경 확인**

```bash
echo $AMENT_PREFIX_PATH
```

출력에 다음 경로가 포함되어 있는지 확인합니다.

```
/opt/ros/lyrical
```

**새 터미널에서 다시 실행**

설치 이전에 열어둔 터미널에서는 패키지 환경이 제대로 반영되지 않을 수 있습니다.

새 터미널을 열고 다음 명령을 실행합니다.

```bash
source /opt/ros/lyrical/setup.bash
source ~/project/so_arm101_ws/install/setup.bash
ros2 run rqt_tf_tree rqt_tf_tree --force-discover
```

---

#### rqt_tf_tree에서 확인할 내용

TF Tree가 표시되면 다음 항목을 확인합니다.

- `world`가 전체 Tree의 시작점인가?
- `world` 아래에 `base_link`가 연결되어 있는가?
- 모든 Link Frame이 하나의 Tree로 연결되어 있는가?
- Parent와 Child 순서가 URDF와 일치하는가?
- Frame 이름이 중복되지 않았는가?
- 연결되지 않은 별도의 Tree가 존재하지 않는가?
- `gripper_link` 아래에 Moving Jaw Frame이 연결되는가?

정상적인 TF Tree는 하나의 Root에서 시작하여 모든 Frame이 연결되어 있어야 합니다.

---

#### TF 확인 도구 비교

| 도구 | 특징 | 적합한 용도 |
| --- | --- | --- |
| `ros2 topic echo /tf` | TF 원본 메시지 확인 | 실제 Transform 데이터 분석 |
| `ros2 topic echo /tf_static` | 고정 TF 확인 | Fixed Joint 및 World 연결 확인 |
| `tf2_echo` | 두 Frame 사이 관계 확인 | End-Effector 좌표 확인 |
| `rqt_tf_tree` | GUI 기반 Tree 표시 | Parent·Child 구조 확인 |
| `view_frames` | TF Tree를 파일로 생성 | 전체 구조 기록 및 문서화 |
| RViz2 TF Display | 3차원 좌표축 표시 | Frame 위치와 축 방향 확인 |

---

#### 정리

이번 절에서는 SO-ARM101의 JointState와 TF 구조를 확인했습니다.

- SO-ARM101은 여섯 개의 주요 Joint를 사용합니다.
- `/joint_states`에는 각 Joint의 현재 위치가 발행됩니다.
- `robot_state_publisher`는 URDF와 JointState를 이용해 TF를 계산합니다.
- `/tf`에는 움직이는 Frame의 Transform이 발행됩니다.
- `/tf_static`에는 고정 Frame의 Transform이 발행됩니다.
- `/tf`에는 Joint 이름이 아니라 Parent와 Child Frame 이름이 표시됩니다.
- SO-ARM101의 구조는 7개의 Link Frame과 6개의 Joint 연결로 구성됩니다.
- `world`와 `base_link`는 고정 Transform으로 연결됩니다.
- `tf2_echo`를 이용하면 두 Frame 사이의 좌표를 확인할 수 있습니다.
- `rqt_tf_tree`를 이용하면 전체 TF Tree를 GUI로 확인할 수 있습니다.

가장 중요한 데이터 흐름은 다음과 같습니다.

> **JointState는 현재 관절값을 나타내고, TF는 그 관절값으로 계산된 각 Link의 위치와 방향을 나타냅니다.**
> 

`joint_state_publisher_gui`에서 Joint 값을 변경했을 때 `/joint_states`, `/tf`, RViz2 모델이 함께 변경된다면 URDF와 TF 구조가 정상적으로 연결된 것입니다.
