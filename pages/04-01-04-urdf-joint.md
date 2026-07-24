# Joint 이해

앞 장에서는 로봇을 구성하는 물리적인 몸체인 Link에 대해 알아보았습니다.

하지만 Link만 정의해서는 로봇의 각 몸체가 어떻게 연결되고 움직이는지 알 수 없습니다. Link와 Link 사이의 연결 관계와 움직임을 정의하는 요소가 필요하며, 이 역할을 담당하는 것이 **Joint**입니다.

Joint는 두 개의 Link를 연결하고 회전, 직선 이동 또는 고정과 같은 움직임의 규칙을 정의합니다.

쉽게 정리하면 다음과 같습니다.

> Link는 로봇의 몸체이고, Joint는 몸체 사이의 연결과 움직임입니다.
> 

사람의 팔에 비유하면 팔뚝은 Link이고, 팔꿈치는 Joint라고 볼 수 있습니다. 팔꿈치가 위팔과 아래팔을 연결하고 움직임을 만드는 것처럼 로봇도 Joint를 통해 Link가 연결되고 움직입니다.

---

#### Joint의 기본 구조

Joint의 가장 기본적인 구조는 다음과 같습니다.

```xml
<joint name="joint1" type="revolute">
    <parent link="base_link"/>
    <child link="arm_link"/>
</joint>
```

각 항목의 의미는 다음과 같습니다.

- `name`: Joint의 이름
- `type`: Joint의 움직임 종류
- `parent`: 기준이 되는 부모 Link
- `child`: Joint에 연결되는 자식 Link

구조적으로 표현하면 다음과 같습니다.

```
base_link
    ↓
joint1
    ↓
arm_link
```

`joint1`은 `base_link`와 `arm_link`를 연결하고, `arm_link`가 `base_link`를 기준으로 회전할 수 있도록 정의합니다.

실제로 회전축과 움직임 범위를 지정하려면 `origin`, `axis`, `limit` 등의 정보도 추가해야 합니다.

```xml
<joint name="joint1" type="revolute">
    <parent link="base_link"/>
    <child link="arm_link"/>

    <origin xyz="0 0 0.1" rpy="0 0 0"/>
    <axis xyz="0 0 1"/>

    <limit
        lower="-1.57"
        upper="1.57"
        effort="10"
        velocity="1.0"/>
</joint>
```

---

#### Joint 이름

Joint도 Link와 마찬가지로 `name` 속성을 사용하여 식별합니다.

```xml
<joint name="shoulder_pan" type="revolute">
```

하나의 로봇 안에서 Joint 이름은 서로 중복되지 않아야 합니다. 또한 이름만 보고 관절의 역할을 파악할 수 있도록 작성하는 것이 좋습니다.

**권장하는 이름**

```
shoulder_pan
shoulder_lift
elbow_flex
wrist_flex
wrist_roll
gripper_joint
```

**역할을 파악하기 어려운 이름**

```
joint1
joint2
joint3
```

`joint1`, `joint2`와 같은 이름도 문법적으로는 사용할 수 있지만 로봇이 복잡해지면 각 Joint의 역할을 구분하기 어렵습니다.

일반적으로 Joint 이름은 소문자와 언더스코어를 사용하는 스네이크 표기법으로 작성합니다.

---

#### Parent Link와 Child Link

Joint는 반드시 하나의 Parent Link와 하나의 Child Link를 연결해야 합니다.

```xml
<parent link="base_link"/>
<child link="shoulder_link"/>
```

각 항목의 역할은 다음과 같습니다.

- Parent Link: 위치와 방향을 계산할 때 기준이 되는 Link
- Child Link: Joint를 통해 Parent Link에 연결되는 Link

연결 관계는 다음과 같습니다.

```
Parent Link → Joint → Child Link
```

예를 들어 다음 코드는 `base_link`와 `shoulder_link`를 연결합니다.

```xml
<link name="base_link"/>

<link name="shoulder_link"/>

<joint name="shoulder_pan" type="revolute">
    <parent link="base_link"/>
    <child link="shoulder_link"/>
</joint>
```

Joint가 회전하면 Child Link의 위치와 방향이 Parent Link를 기준으로 변경됩니다. Child Link 아래에 다른 Link가 연결되어 있다면 하위 구조도 함께 움직입니다.

```
base_link
    ↓ shoulder_pan
shoulder_link
    ↓ shoulder_lift
upper_arm_link
```

`shoulder_pan`이 회전하면 `shoulder_link`뿐만 아니라 그 아래에 연결된 `upper_arm_link`도 함께 영향을 받습니다.

---

#### Joint 타입

URDF에서는 로봇의 움직임을 표현하기 위해 여러 종류의 Joint를 제공합니다.

| Joint 타입 | 동작 | 주요 사용 예 |
| --- | --- | --- |
| `revolute` | 제한된 범위에서 회전 | 로봇 팔 관절 |
| `continuous` | 회전 범위 제한 없이 회전 | 바퀴, 롤러 |
| `prismatic` | 지정된 축을 따라 직선 이동 | 실린더, 리니어 액추에이터 |
| `fixed` | 움직임 없이 고정 | 카메라, 센서, 툴 장착 |
| `planar` | 평면 위에서 이동 | 특수 이동 구조 |
| `floating` | 3차원 공간에서 자유롭게 이동 | 자유 물체 또는 이동체 |

SO-ARM101과 같은 일반적인 로봇 팔에서는 주로 `revolute`와 `fixed` 타입을 사용합니다.

---

#### Revolute Joint

`revolute`는 일정한 각도 범위 안에서 회전하는 Joint입니다.

```xml
<joint name="elbow_flex" type="revolute">
    <parent link="upper_arm_link"/>
    <child link="forearm_link"/>

    <axis xyz="0 1 0"/>

    <limit
        lower="-1.57"
        upper="1.57"
        effort="10"
        velocity="1.0"/>
</joint>
```

위 Joint는 Y축을 기준으로 `-1.57rad`부터 `1.57rad`까지 회전할 수 있습니다.

```
-1.57rad ≈ -90°
 1.57rad ≈  90°
```

`revolute`는 다음과 같은 로봇 관절에 주로 사용됩니다.

- Shoulder
- Elbow
- Wrist
- 회전형 Gripper Joint

서보 모터를 사용하는 SO-ARM101의 주요 관절도 대부분 `revolute` 타입으로 표현합니다.

---

#### Continuous Joint

`continuous`는 회전 각도의 제한 없이 계속 회전할 수 있는 Joint입니다.

```xml
<joint name="wheel_joint" type="continuous">
    <parent link="base_link"/>
    <child link="wheel_link"/>

    <axis xyz="0 1 0"/>

    <limit
        effort="5"
        velocity="10"/>
</joint>
```

`continuous`에는 회전 위치의 최소값과 최대값이 없으므로 일반적으로 `lower`와 `upper`를 작성하지 않습니다. 하지만 최대 힘과 속도를 나타내는 `effort`와 `velocity`는 정의할 수 있습니다.

주로 다음과 같은 구조에 사용됩니다.

- 이동 로봇의 바퀴
- 컨베이어 롤러
- 연속 회전축
- 무한 회전이 가능한 모터

---

#### Prismatic Joint

`prismatic`은 지정한 축을 따라 직선으로 이동하는 Joint입니다.

```xml
<joint name="slide_joint" type="prismatic">
    <parent link="base_link"/>
    <child link="slide_link"/>

    <axis xyz="1 0 0"/>

    <limit
        lower="0.0"
        upper="0.3"
        effort="100"
        velocity="0.2"/>
</joint>
```

위 Joint는 X축 방향으로 `0m`부터 `0.3m`까지 이동할 수 있습니다.

주로 다음과 같은 구조에 사용됩니다.

- 공압 실린더
- 전동 실린더
- 리니어 액추에이터
- 직선 이동 스테이지
- 승강 장치

`revolute`의 위치 단위는 라디안이지만 `prismatic`의 위치 단위는 미터입니다.

---

#### Fixed Joint

`fixed`는 두 Link를 움직이지 않도록 고정하는 Joint입니다.

```xml
<joint name="camera_mount_joint" type="fixed">
    <parent link="wrist_link"/>
    <child link="camera_link"/>

    <origin xyz="0.05 0 0.03" rpy="0 0 0"/>
</joint>
```

`fixed` Joint는 움직이지 않으므로 일반적으로 `axis`와 `limit`가 필요하지 않습니다.

주로 다음과 같은 구조에 사용됩니다.

- 카메라 장착
- 센서 장착
- 툴 장착
- 고정 브래킷 연결
- 로봇 Base 고정
- `world`와 `base_link` 연결

여러 부품이 서로 고정되어 있지만 좌표계를 따로 관리해야 할 때 Fixed Joint를 사용하면 유용합니다.

---

#### Joint Origin

`origin`은 Parent Link를 기준으로 Joint와 Child Link가 연결되는 위치와 방향을 정의합니다.

```xml
<origin xyz="0 0 0.1" rpy="0 0 0"/>
```

`origin`은 `xyz`와 `rpy` 속성으로 구성됩니다.

- `xyz`: X, Y, Z축 방향의 위치
- `rpy`: Roll, Pitch, Yaw 방향의 회전

URDF에서 위치의 기본 단위는 미터이며, 회전의 기본 단위는 라디안입니다.

```xml
<origin xyz="0 0 0.1" rpy="0 0 0"/>
```

위 코드는 Parent Link를 기준으로 Z축 방향으로 `0.1m`, 즉 `100mm` 떨어진 위치에 Joint를 배치한다는 의미입니다.

```
base_link
    │
    │ Z축 방향 0.1m
    ↓
shoulder_joint
    ↓
shoulder_link
```

Joint의 위치가 잘못되면 Child Link가 잘못된 위치에 연결되므로 실제 로봇의 관절 위치를 기준으로 정확하게 작성해야 합니다.

---

#### Joint Axis

`axis`는 Joint가 움직이는 축을 정의합니다.

```
<axisxyz="0 0 1"/>
```

각 축의 기본 표현은 다음과 같습니다.

| Axis 설정 | 의미 |
| --- | --- |
| `1 0 0` | X축 방향 |
| `0 1 0` | Y축 방향 |
| `0 0 1` | Z축 방향 |
| `-1 0 0` | X축 반대 방향 |
| `0 -1 0` | Y축 반대 방향 |
| `0 0 -1` | Z축 반대 방향 |

`revolute`와 `continuous`에서는 해당 축을 중심으로 회전하고, `prismatic`에서는 해당 축을 따라 직선 이동합니다.

```xml
<axis xyz="0 1 0"/>
```

- `revolute`: Y축을 중심으로 회전
- `prismatic`: Y축 방향으로 직선 이동

Axis는 Joint Frame을 기준으로 해석됩니다. 따라서 `origin`의 `rpy`를 사용하여 Joint Frame을 회전시키면 Axis의 실제 방향도 함께 달라집니다.

축의 방향이 반대로 정의되면 같은 관절값을 입력해도 실제 로봇과 시뮬레이션이 서로 반대 방향으로 움직일 수 있습니다. 따라서 SO-ARM101의 실제 회전축과 URDF의 Axis 방향을 일치시키는 것이 중요합니다.

---

#### Joint Limit

`limit`는 Joint가 움직일 수 있는 범위와 최대 힘, 최대 속도를 정의합니다.

```xml
<limit
    lower="-1.57"
    upper="1.57"
    effort="10"
    velocity="1.0"/>
```

각 속성의 의미는 다음과 같습니다.

| 속성 | 의미 | 단위 |
| --- | --- | --- |
| `lower` | 최소 위치 | rad 또는 m |
| `upper` | 최대 위치 | rad 또는 m |
| `effort` | 최대 힘 또는 토크 | N 또는 N·m |
| `velocity` | 최대 이동 또는 회전 속도 | m/s 또는 rad/s |

회전형 Joint에서는 `lower`와 `upper`를 라디안으로 작성합니다.

```
-1.57rad ≈ -90°
 1.57rad ≈  90°
```

직선형 Joint에서는 미터 단위를 사용합니다.

```
0.0m ~ 0.3m
```

SO-ARM101의 `shoulder_pan` Joint가 다음과 같이 정의되어 있다고 가정해 보겠습니다.

```xml
<limit
    lower="-1.91986"
    upper="1.91986"
    effort="10"
    velocity="10"/>
```

각도 단위로 변환하면 약 다음과 같습니다.

```
-1.91986rad ≈ -110°
 1.91986rad ≈  110°
```

Joint Limit는 단순한 시각화 정보가 아닙니다. ros2_control, Gazebo 및 MoveIt에서 관절의 안전한 움직임 범위를 판단할 때 사용될 수 있으므로 실제 로봇의 기구적인 제한과 일치하도록 작성해야 합니다.

---

#### Joint의 전체 예제

다음은 SO-ARM101의 회전 관절을 단순화한 예제입니다.

```xml
<link name="base_link"/>

<link name="shoulder_link"/>

<joint name="shoulder_pan" type="revolute">

    <parent link="base_link"/>
    <child link="shoulder_link"/>

    <origin xyz="0 0 0.1" rpy="0 0 0"/>

    <axis xyz="0 0 1"/>

    <limit
        lower="-1.91986"
        upper="1.91986"
        effort="10"
        velocity="10"/>

</joint>
```

이 Joint의 의미는 다음과 같습니다.

- `base_link`와 `shoulder_link` 연결
- Joint 이름은 `shoulder_pan`
- Z축을 중심으로 회전
- `base_link`에서 Z축 방향으로 `0.1m` 위치에 배치
- 회전 범위는 약 `110°`부터 `110°`
- 최대 토크와 속도 제한 정의

---

#### 정리

Joint는 Link와 Link를 연결하고 로봇의 움직임을 정의하는 핵심 요소입니다.

주요 내용을 정리하면 다음과 같습니다.

- Joint는 하나의 Parent Link와 하나의 Child Link를 연결합니다.
- `name`은 Joint를 식별하는 고유한 이름입니다.
- `type`은 회전, 직선 이동 또는 고정과 같은 움직임의 종류를 정의합니다.
- `origin`은 Parent Link를 기준으로 Joint의 위치와 방향을 정의합니다.
- `axis`는 Joint가 회전하거나 이동하는 축을 정의합니다.
- `limit`는 Joint의 위치, 힘 및 속도 범위를 제한합니다.
- Child Link 아래에 연결된 모든 구조는 상위 Joint의 움직임에 영향을 받습니다.

가장 기본적인 연결 구조는 다음과 같습니다.

```
Parent Link
    ↓
Joint
    ↓
Child Link
```

이 구조가 반복되면서 로봇 전체의 Kinematic Chain과 TF Tree가 만들어집니다. 다음 장에서는 Joint와 Link의 위치 및 방향을 결정하는 Origin에 대해 자세히 알아보겠습니다.
