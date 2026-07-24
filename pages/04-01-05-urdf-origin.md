# Origin 이해

앞 장에서는 Link와 Link를 연결하고 로봇의 움직임을 정의하는 Joint에 대해 알아보았습니다.

Joint를 정의할 때 Parent Link와 Child Link의 연결 관계만 지정해서는 로봇의 정확한 구조를 표현할 수 없습니다. Child Link를 Parent Link의 어느 위치에 연결할 것인지, 그리고 어떤 방향으로 배치할 것인지도 함께 정의해야 합니다.

이러한 위치와 방향을 정의하는 요소가 **Origin**입니다.

Origin은 두 좌표계 사이의 상대적인 위치와 회전을 나타냅니다.

```
Origin = 위치(Translation) + 회전(Rotation)
```

URDF에서는 Origin을 이용하여 Link, Joint, Mesh, 충돌 형상 및 관성 정보의 위치와 방향을 정의합니다.

---

#### Origin의 기본 구조

Origin의 기본 구조는 다음과 같습니다.

```xml
<origin xyz="0 0 0.1" rpy="0 0 0"/>
```

Origin은 `xyz`와 `rpy`라는 두 가지 속성으로 구성됩니다.

- `xyz`: X, Y, Z축 방향의 상대적인 위치
- `rpy`: Roll, Pitch, Yaw로 표현한 상대적인 회전

```
origin = xyz + rpy
```

`origin` 태그를 생략하면 일반적으로 다음과 같이 위치와 회전이 모두 0인 것으로 처리됩니다.

```xml
<origin xyz="0 0 0" rpy="0 0 0"/>
```

즉, 두 좌표계의 원점과 방향이 서로 일치한다는 의미입니다.

---

#### Position과 xyz

`xyz`는 기준 좌표계에 대한 상대적인 위치를 나타냅니다.

```xml
<origin xyz="0 0 0.1" rpy="0 0 0"/>
```

각 값은 다음과 같은 의미를 가집니다.

```
x = 0m
y = 0m
z = 0.1m
```

따라서 기준 좌표계에서 Z축 방향으로 `0.1m` 떨어진 위치에 새로운 좌표계를 배치한다는 의미입니다.

```
기준 좌표계
    │
    │ Z축 방향 0.1m
    ↓
새로운 좌표계
```

URDF에서 길이의 기본 단위는 **미터(m)**입니다.

| URDF 값 | 실제 길이 |
| --- | --- |
| `0.001` | 1mm |
| `0.01` | 1cm |
| `0.1` | 10cm |
| `0.5` | 50cm |
| `1.0` | 1m |

예를 들어 CAD 프로그램에서 확인한 길이가 `150mm`라면 URDF에는 다음과 같이 `0.15m`로 입력해야 합니다.

```xml
<origin xyz="0 0 0.15" rpy="0 0 0"/>
```

밀리미터 값을 그대로 입력하면 로봇이 실제 크기보다 1,000배 크게 표현될 수 있으므로 단위에 주의해야 합니다.

---

#### X, Y, Z 방향

ROS2와 URDF에서는 오른손 좌표계를 사용합니다.

로봇의 기준 좌표계는 일반적으로 다음 방향을 사용합니다.

```
X축 = Forward, 앞쪽
Y축 = Left, 왼쪽
Z축 = Up, 위쪽
```

정리하면 다음과 같습니다.

| 축 | 양의 방향 |
| --- | --- |
| X | 앞쪽 |
| Y | 왼쪽 |
| Z | 위쪽 |

X축 방향으로 `0.2m` 이동한 위치는 다음과 같이 표현합니다.

```xml
<origin xyz="0.2 0 0" rpy="0 0 0"/>
```

Y축 방향으로 `0.05m` 이동한 위치는 다음과 같습니다.

```xml
<origin xyz="0 0.05 0" rpy="0 0 0"/>
```

Z축 방향으로 `0.1m` 이동한 위치는 다음과 같습니다.

```xml
<origin xyz="0 0 0.1" rpy="0 0 0"/>
```

음수 값을 사용하면 반대 방향을 나타냅니다.

```xml
<origin xyz="-0.1 0 0" rpy="0 0 0"/>
```

위 코드는 X축의 반대 방향으로 `0.1m` 이동한 위치를 의미합니다.

다만 X, Y, Z 방향은 항상 **기준이 되는 좌표계**를 기준으로 해석해야 합니다. 기준 좌표계 자체가 회전되어 있다면 실제 화면에서 보이는 축의 방향도 달라질 수 있습니다.

---

#### Rotation과 rpy

`rpy`는 좌표계의 회전을 나타냅니다.

```
rpy = Roll + Pitch + Yaw
```

각 값은 다음 축을 기준으로 한 회전을 의미합니다.

| 항목 | 회전축 |
| --- | --- |
| Roll | X축 |
| Pitch | Y축 |
| Yaw | Z축 |

다음 코드는 Z축을 기준으로 약 90° 회전한 좌표계를 나타냅니다.

```
<originxyz="0 0 0"rpy="0 0 1.5708"/>
```

URDF에서 회전의 기본 단위는 도(Degree)가 아니라 **라디안(Radian)**입니다.

| 라디안 | 각도 |
| --- | --- |
| `0` | 0° |
| `0.7854` | 약 45° |
| `1.5708` | 약 90° |
| `3.1416` | 약 180° |
| `-1.5708` | 약 -90° |

라디안과 각도의 관계는 다음과 같습니다.

$$
\text{radian} = \text{degree} \times \frac{\pi}{180}
$$

$$
\text{degree} = \text{radian} \times \frac{180}{\pi}
$$

90°를 라디안으로 변환하면 다음과 같습니다.

$$
90 \times \frac{\pi}{180} = \frac{\pi}{2} \approx 1.5708
$$

---

#### Roll, Pitch, Yaw

`rpy`에는 Roll, Pitch, Yaw 순서로 값을 작성합니다.

```xml
<origin rpy="roll pitch yaw"/>
```

**X축을 기준으로 90° 회전**

```xml
<origin xyz="0 0 0" rpy="1.5708 0 0"/>
```

**Y축을 기준으로 90° 회전**

```xml
<origin xyz="0 0 0" rpy="0 1.5708 0"/>
```

**Z축을 기준으로 90° 회전**

```xml
<origin xyz="0 0 0" rpy="0 0 1.5708"/>
```

회전 방향은 각 축의 양의 방향을 기준으로 오른손 법칙을 따릅니다.

오른손 엄지손가락을 축의 양의 방향으로 향하게 했을 때, 나머지 손가락이 감기는 방향이 양의 회전 방향입니다.

RPY 값이 잘못되면 다음과 같은 문제가 발생할 수 있습니다.

- Link가 옆으로 눕거나 뒤집혀 표시됨
- Joint의 회전축이 예상과 다르게 보임
- STL Mesh의 방향이 실제 로봇과 다르게 표시됨
- Collision 형상과 Visual 형상이 서로 어긋남
- TF 좌표계의 방향이 실제 로봇과 일치하지 않음

---

#### Joint Origin

Joint 안에 작성하는 Origin은 Parent Link를 기준으로 Joint와 Child Link가 연결되는 위치와 방향을 정의합니다.

```xml
<joint name="shoulder_joint" type="revolute">
    <parent link="base_link"/>
    <child link="shoulder_link"/>

    <origin xyz="0 0 0.15" rpy="0 0 0"/>
    <axis xyz="0 0 1"/>
</joint>
```

위 코드의 의미는 다음과 같습니다.

- Parent Link: `base_link`
- Child Link: `shoulder_link`
- Joint 위치: `base_link` 기준 Z축 방향으로 `0.15m`
- Joint 방향: 회전 없음
- Joint 회전축: Z축

구조적으로 표현하면 다음과 같습니다.

```
base_link
    │
    │ Z축 방향 0.15m
    ↓
shoulder_joint
    ↓
shoulder_link
```

Joint Origin은 Joint가 0인 상태에서 Parent Link Frame과 Child Link Frame 사이의 기본적인 위치와 방향 관계를 정의합니다.

따라서 Joint Origin이 잘못되면 Child Link뿐만 아니라 그 아래에 연결된 모든 Link의 위치도 함께 잘못됩니다.

```
base_link
    ↓ 잘못된 Origin
shoulder_link
    ↓
upper_arm_link
    ↓
forearm_link
```

상위 Joint에서 발생한 위치 오차는 하위 Link 전체에 영향을 주므로 실제 로봇의 관절 위치를 기준으로 정확하게 작성해야 합니다.

---

#### Visual Origin

Link의 `<visual>` 안에서도 Origin을 사용할 수 있습니다.

```xml
<link name="arm_link">

    <visual>
        <origin xyz="0 0 0.1" rpy="0 0 0"/>

        <geometry>
            <box size="0.05 0.05 0.2"/>
        </geometry>
    </visual>

</link>
```

Visual Origin은 Link Frame을 기준으로 화면에 표시되는 형상의 위치와 방향을 조정합니다.

```
Link Frame
    ↓ Visual Origin
Visual Geometry
```

다음 두 Origin은 서로 역할이 다릅니다.

- Joint Origin: Parent Link와 Child Link의 연결 위치
- Visual Origin: Link Frame과 화면에 표시되는 외형의 상대 위치

Joint Origin을 변경하면 Child Link와 그 아래의 전체 구조가 이동합니다. 반면 Visual Origin을 변경하면 Link Frame은 그대로 유지되고 화면에 표시되는 형상만 이동합니다.

---

#### Mesh와 Visual Origin

CAD 프로그램에서 만든 STL 파일의 원점과 방향이 URDF의 Link Frame과 일치하지 않을 수 있습니다.

```xml
<link name="wrist_link">

    <visual>
        <origin xyz="0 0 0.02" rpy="1.5708 0 0"/>

        <geometry>
            <mesh filename="package://so_arm101_description/meshes/wrist.stl"/>
        </geometry>
    </visual>

</link>
```

위 코드는 다음 작업을 수행합니다.

- Mesh를 Z축 방향으로 `0.02m` 이동
- Mesh를 X축 기준으로 약 90° 회전
- Link Frame에 맞게 STL의 위치와 방향 보정

STL의 방향이 잘못되었다고 해서 Joint Origin부터 수정하면 로봇의 전체 연결 구조가 틀어질 수 있습니다.

먼저 다음을 구분해야 합니다.

- Link 연결 위치가 잘못된 경우: Joint Origin 수정
- Link Frame은 정확하지만 Mesh만 어긋난 경우: Visual Origin 수정

이 구분은 URDF를 작성할 때 매우 중요합니다.

---

#### Collision Origin

`<collision>` 안에서도 Origin을 사용할 수 있습니다.

```xml
<link name="arm_link">

    <collision>
        <origin xyz="0 0 0.1" rpy="0 0 0"/>

        <geometry>
            <box size="0.05 0.05 0.2"/>
        </geometry>
    </collision>

</link>
```

Collision Origin은 Link Frame을 기준으로 충돌 검사에 사용하는 형상의 위치와 방향을 정의합니다.

Visual과 Collision은 서로 다른 Origin과 형상을 사용할 수 있습니다.

예를 들어 Visual에는 정교한 STL Mesh를 사용하고, Collision에는 계산이 간단한 박스를 사용할 수 있습니다.

```
Link Frame
├── Visual Origin → STL Mesh
└── Collision Origin → Box
```

Collision Origin이 Visual Origin과 크게 다르면 화면에 보이는 로봇과 실제 충돌이 발생하는 위치가 달라질 수 있으므로 주의해야 합니다.

---

#### Inertial Origin

`<inertial>` 안의 Origin은 Link Frame을 기준으로 질량 중심의 위치와 관성 좌표계의 방향을 정의합니다.

```xml
<link name="arm_link">

    <inertial>
        <origin xyz="0 0 0.08" rpy="0 0 0"/>

        <mass value="0.147"/>

        <inertia
            ixx="0.001"
            ixy="0.0"
            ixz="0.0"
            iyy="0.001"
            iyz="0.0"
            izz="0.001"/>
    </inertial>

</link>
```

Inertial Origin은 주로 Gazebo와 같은 물리 시뮬레이터에서 사용됩니다.

이 값이 실제 로봇의 무게중심과 크게 다르면 다음과 같은 문제가 발생할 수 있습니다.

- 로봇이 비정상적으로 흔들림
- Link가 예상하지 못한 방향으로 회전함
- 로봇이 쉽게 넘어짐
- 관절에 비정상적인 힘이 적용됨

---

#### Origin 종류 비교

URDF에서 사용하는 주요 Origin을 정리하면 다음과 같습니다.

| Origin 위치 | 기준 좌표계 | 역할 |
| --- | --- | --- |
| Joint Origin | Parent Link Frame | Joint와 Child Link의 연결 위치 및 방향 |
| Visual Origin | Link Frame | 화면에 표시되는 외형의 위치 및 방향 |
| Collision Origin | Link Frame | 충돌 형상의 위치 및 방향 |
| Inertial Origin | Link Frame | 질량 중심과 관성 좌표계의 위치 및 방향 |

각 Origin은 서로 독립적인 역할을 가집니다. 한 종류의 Origin을 수정하여 다른 문제를 해결하려고 하면 로봇 구조가 잘못될 수 있습니다.

---

#### Origin 작성 예제

다음은 Joint Origin과 Visual Origin을 함께 사용한 간단한 예제입니다.

```xml
<link name="base_link"/>

<link name="arm_link">

    <visual>
        <origin xyz="0 0 0.1" rpy="0 0 0"/>

        <geometry>
            <box size="0.05 0.05 0.2"/>
        </geometry>
    </visual>

</link>

<joint name="arm_joint" type="revolute">
    <parent link="base_link"/>
    <child link="arm_link"/>

    <origin xyz="0 0 0.15" rpy="0 0 0"/>
    <axis xyz="0 1 0"/>

    <limit
        lower="-1.57"
        upper="1.57"
        effort="10"
        velocity="1.0"/>
</joint>
```

이 구조에서 다음 두 가지 위치 관계가 적용됩니다.

1. `arm_joint`는 `base_link`에서 Z축 방향으로 `0.15m` 위치에 배치됩니다.
2. 박스 형상은 `arm_link`의 원점에서 Z축 방향으로 `0.1m` 위치에 표시됩니다.

따라서 Joint Origin과 Visual Origin을 각각 다른 기준으로 계산해야 합니다.

---

#### 정리

Origin은 URDF에서 좌표계 사이의 상대적인 위치와 방향을 정의하는 핵심 요소입니다.

주요 내용을 정리하면 다음과 같습니다.

- `xyz`는 X, Y, Z축 방향의 위치를 정의합니다.
- `rpy`는 Roll, Pitch, Yaw 회전을 정의합니다.
- 위치의 기본 단위는 미터입니다.
- 회전의 기본 단위는 라디안입니다.
- Joint Origin은 Parent Link와 Child Link의 연결 위치를 정의합니다.
- Visual Origin은 Link Frame을 기준으로 외형의 위치를 조정합니다.
- Collision Origin은 충돌 형상의 위치를 조정합니다.
- Inertial Origin은 질량 중심과 관성 좌표계의 위치를 정의합니다.
- Mesh만 어긋났다면 Joint Origin이 아니라 Visual Origin을 확인해야 합니다.

Origin을 정확하게 정의해야 실제 로봇, RViz2 모델, Gazebo 시뮬레이션 및 TF 좌표계가 서로 일치합니다. 다음 장에서는 Joint가 회전하거나 이동하는 방향을 결정하는 Axis에 대해 자세히 알아보겠습니다.
