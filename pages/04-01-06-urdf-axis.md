# Axis 이해

앞 장에서는 Joint와 Link의 위치 및 방향을 정의하는 Origin에 대해 알아보았습니다.

Origin을 이용하면 Joint가 Parent Link의 어느 위치에 연결되고, Joint Frame이 어떤 방향으로 배치되는지를 정의할 수 있습니다.

하지만 Joint의 위치와 방향만 정의해서는 로봇의 움직임이 완성되지 않습니다. Joint가 어느 축을 기준으로 회전하거나 어느 방향으로 직선 이동할 것인지도 정의해야 합니다.

이 역할을 담당하는 요소가 **Axis**입니다.

```
Axis = Joint가 움직이는 축의 방향
```

회전형 Joint에서는 Axis가 회전축이 되고, 직선형 Joint에서는 이동 방향이 됩니다.

같은 위치에 설치된 Joint라도 Axis를 다르게 설정하면 로봇의 움직임이 완전히 달라집니다. 따라서 Axis는 실제 로봇과 URDF 모델의 움직임을 일치시키기 위해 정확하게 정의해야 하는 중요한 요소입니다.

---

#### Axis의 기본 구조

Axis의 기본 구조는 다음과 같습니다.

```xml
<axis xyz="0 0 1"/>
```

`xyz`에는 X, Y, Z축 방향을 나타내는 벡터를 작성합니다.

```xml
axis xyz="X Y Z"
```

다음 Axis는 각각 X축, Y축, Z축을 나타냅니다.

```xml
<axis xyz="1 0 0"/>
```

```xml
<axis xyz="0 1 0"/>
```

```xml
<axis xyz="0 0 1"/>
```

Axis는 위치를 나타내는 값이 아니라 방향을 나타내는 벡터입니다.

일반적으로 방향을 명확하게 표현하기 위해 길이가 1인 단위 벡터를 사용합니다.

| Axis | 의미 |
| --- | --- |
| `1 0 0` | X축의 양의 방향 |
| `0 1 0` | Y축의 양의 방향 |
| `0 0 1` | Z축의 양의 방향 |
| `-1 0 0` | X축의 음의 방향 |
| `0 -1 0` | Y축의 음의 방향 |
| `0 0 -1` | Z축의 음의 방향 |

Axis를 생략하면 일부 URDF 처리 도구에서는 X축인 `1 0 0`을 기본값으로 사용할 수 있습니다. 하지만 로봇의 움직임을 명확하게 표현하고 오류를 방지하려면 Axis를 직접 작성하는 것이 좋습니다.

---

#### Axis와 좌표계

ROS2와 URDF에서는 오른손 좌표계를 사용합니다.

로봇의 기준 좌표계는 일반적으로 다음 방향으로 정의합니다.

```
X축 = Forward, 앞쪽
Y축 = Left, 왼쪽
Z축 = Up, 위쪽
```

따라서 기본적인 Joint Frame에서 다음 Axis는 X축을 기준으로 한 움직임을 의미합니다.

```xml
<axis xyz="1 0 0"/>
```

Y축을 기준으로 한 움직임은 다음과 같습니다.

```xml
<axis xyz="0 1 0"/>
```

Z축을 기준으로 한 움직임은 다음과 같습니다.

```xml
<axis xyz="0 0 1"/>
```

다만 Axis는 로봇 전체의 `base_link`가 아니라 **해당 Joint Frame을 기준으로 해석**됩니다.

Joint의 Origin에서 `rpy`를 사용하여 Joint Frame을 회전시키면 Axis의 실제 방향도 함께 회전합니다.

```xml
<origin xyz="0 0 0.1" rpy="0 0 1.5708"/>
<axis xyz="1 0 0"/>
```

위 코드에서 Axis는 Joint Frame의 X축입니다. 하지만 Joint Frame 자체가 Z축을 기준으로 90° 회전되어 있으므로 Parent Link에서 바라본 실제 Axis 방향은 달라질 수 있습니다.

따라서 Axis를 확인할 때는 다음 두 가지를 함께 확인해야 합니다.

- Joint Origin의 `rpy`
- Joint Axis의 `xyz`

---

#### SO-ARM101의 Axis 구성

SO-ARM101과 같은 일반적인 로봇 팔은 다음과 같은 관절 구조를 가집니다.

```
Base
    ↓ Shoulder Pan
Shoulder
    ↓ Shoulder Lift
Upper Arm
    ↓ Elbow Flex
Forearm
    ↓ Wrist Flex
Wrist
    ↓ Wrist Roll
Gripper
```

각 Joint Frame의 방향이 로봇의 기준 좌표계와 일치한다고 단순화하면 다음과 같은 Axis를 생각할 수 있습니다.

| Joint | 주요 움직임 | Axis 예시 |
| --- | --- | --- |
| Shoulder Pan | 몸체 좌우 회전 | Z축 |
| Shoulder Lift | 팔의 상승·하강 | Y축 |
| Elbow Flex | 팔꿈치 굽힘·펼침 | Y축 |
| Wrist Flex | 손목 굽힘·펼침 | Y축 |
| Wrist Roll | 손목 회전 | X축 |

```
Shoulder Pan  → Z축
Shoulder Lift → Y축
Elbow Flex    → Y축
Wrist Flex    → Y축
Wrist Roll    → X축
```

하지만 실제 SO-ARM101 URDF에서는 각 Joint Origin의 `rpy`와 STL 모델의 좌표 방향에 따라 Axis 값이 다르게 보일 수 있습니다.

따라서 위 표는 움직임을 이해하기 위한 일반적인 예이며, 실제 Axis는 SO-ARM101의 URDF와 Joint Frame을 기준으로 확인해야 합니다.

---

#### Revolute Joint의 Axis

`revolute` Joint에서 Axis는 회전축을 정의합니다.

```xml
<joint name="elbow_flex" type="revolute">

    <parent link="upper_arm_link"/>
    <child link="forearm_link"/>

    <origin xyz="0 0 0.15" rpy="0 0 0"/>

    <axis xyz="0 1 0"/>

    <limit
        lower="-1.57"
        upper="1.57"
        effort="10"
        velocity="1.0"/>

</joint>
```

위 Joint의 의미는 다음과 같습니다.

- `upper_arm_link`와 `forearm_link` 연결
- Parent Link에서 Z축 방향으로 `0.15m` 위치에 Joint 배치
- Joint Frame의 Y축을 기준으로 회전
- 회전 범위는 약 -90°부터 90°
- 팔꿈치를 굽히고 펴는 움직임 구현

구조적으로 표현하면 다음과 같습니다.

```
upper_arm_link
    ↓
elbow_flex
    ↻ Y축 회전
    ↓
forearm_link
```

Axis가 `0 1 0`이면 Y축을 기준으로 회전하고, `0 -1 0`으로 변경하면 관절값의 양수와 음수에 따른 회전 방향이 반대로 바뀝니다.

---

#### Continuous Joint의 Axis

`continuous` Joint도 `revolute`와 마찬가지로 Axis를 중심으로 회전합니다.

```xml
<joint name="wheel_joint" type="continuous">

    <parent link="base_link"/>
    <child link="wheel_link"/>

    <origin xyz="0 0.2 0" rpy="0 0 0"/>
    <axis xyz="0 1 0"/>

    <limit
        effort="5"
        velocity="10"/>

</joint>
```

위 Joint는 Y축을 중심으로 계속 회전할 수 있습니다.

`revolute`와 `continuous`의 차이는 회전축이 아니라 회전 범위입니다.

- `revolute`: `lower`와 `upper` 범위 안에서 회전
- `continuous`: 위치 범위 제한 없이 연속 회전

---

#### Prismatic Joint의 Axis

`prismatic` Joint에서 Axis는 회전축이 아니라 직선 이동 방향을 나타냅니다.

```xml
<joint name="slider_joint" type="prismatic">

    <parent link="base_link"/>
    <child link="slider_link"/>

    <origin xyz="0 0 0.1" rpy="0 0 0"/>

    <axis xyz="0 0 1"/>

    <limit
        lower="0.0"
        upper="0.3"
        effort="100"
        velocity="0.2"/>

</joint>
```

위 Joint는 Z축 방향으로 `0m`부터 `0.3m`까지 직선 이동합니다.

```
base_link
    ↑ Z축 직선 이동
slider_link
```

Prismatic Joint는 다음과 같은 장치에 사용할 수 있습니다.

- 공압 실린더
- 전동 실린더
- 리니어 액추에이터
- 승강 장치
- 직선 이동 스테이지

Axis가 `0 0 -1`이면 Z축의 반대 방향을 양의 이동 방향으로 사용합니다.

---

#### Fixed Joint의 Axis

`fixed` Joint는 움직이지 않는 고정 연결이므로 Axis를 정의하지 않습니다.

```xml
<joint name="camera_mount_joint" type="fixed">

    <parent link="wrist_link"/>
    <child link="camera_link"/>

    <origin xyz="0.05 0 0.03" rpy="0 0 0"/>

</joint>
```

Fixed Joint에는 회전이나 직선 이동이 없으므로 일반적으로 `axis`와 `limit`가 필요하지 않습니다.

---

#### 오른손 법칙

회전형 Joint의 양의 회전 방향은 **오른손 법칙**을 따릅니다.

오른손 법칙은 다음과 같이 확인할 수 있습니다.

```
오른손 엄지손가락 = Axis의 양의 방향
나머지 손가락이 감기는 방향 = 양의 회전 방향
```

예를 들어 다음 Axis를 사용한다고 가정해 보겠습니다.

```xml
<axis xyz="0 0 1"/>
```

오른손 엄지손가락을 Z축의 양의 방향인 위쪽으로 향하게 했을 때, 나머지 손가락이 감기는 방향이 관절값이 증가하는 양의 회전 방향입니다.

XY 평면을 Z축의 양의 방향에서 바라보면 양의 회전은 일반적으로 반시계 방향으로 보입니다.

```
+Z축 → 반시계 방향 회전
-Z축 → 시계 방향 회전
```

하지만 회전 방향을 판단할 때는 항상 어느 방향에서 축을 바라보는지도 함께 고려해야 합니다. 화면의 카메라 방향이 바뀌면 같은 회전도 반대로 보일 수 있습니다.

---

#### Axis의 반대 방향

Axis 벡터에 음수를 사용하면 양의 움직임 방향을 반대로 설정할 수 있습니다.

```xml
<axis xyz="0 1 0"/>
```

```xml
<axis xyz="0 -1 0"/>
```

두 Axis는 같은 Y축 위에 있지만 방향이 반대입니다.

```
0  1 0 = +Y 방향
0 -1 0 = -Y 방향
```

따라서 같은 양의 Joint 값을 입력해도 Child Link는 서로 반대 방향으로 움직입니다.

Axis의 방향을 반대로 설정하는 경우는 다음과 같습니다.

- 실제 서보 모터의 설치 방향이 반대인 경우
- 실제 로봇과 시뮬레이션의 양의 회전 방향이 다른 경우
- 대칭으로 배치된 두 관절을 표현하는 경우
- CAD 모델의 좌표 방향과 URDF 좌표 방향이 다른 경우

다만 로봇의 움직임이 반대라고 해서 Axis의 부호부터 바로 변경해서는 안 됩니다. 다음 항목을 함께 확인해야 합니다.

- Joint Origin의 `rpy`
- Axis의 `xyz`
- 모터의 양의 회전 방향
- JointState에 사용되는 관절값의 부호
- ros2_control의 하드웨어 변환 설정

---

#### Joint Origin과 Axis의 관계

Axis는 Joint Origin으로 만들어진 Joint Frame을 기준으로 정의됩니다.

```xml
<joint name="example_joint" type="revolute">

    <parent link="base_link"/>
    <child link="arm_link"/>

    <origin xyz="0 0 0.1" rpy="0 0 1.5708"/>
    <axis xyz="1 0 0"/>

</joint>
```

처리 순서를 개념적으로 정리하면 다음과 같습니다.

1. Parent Link Frame을 기준으로 Joint Origin의 `xyz`만큼 이동합니다.
2. Joint Origin의 `rpy`만큼 Joint Frame을 회전합니다.
3. 회전된 Joint Frame을 기준으로 Axis를 적용합니다.
4. Axis를 기준으로 Child Link가 움직입니다.

따라서 Axis 값만 보고 실제 회전 방향을 판단하면 안 됩니다. Origin의 회전값까지 함께 확인해야 정확한 움직임 방향을 알 수 있습니다.

---

#### Axis 확인 방법

Axis가 올바르게 설정되었는지는 RViz2에서 확인할 수 있습니다.

JointState Publisher GUI를 실행한 뒤 관절값을 양의 방향으로 조금씩 증가시키면서 다음 항목을 확인합니다.

- 실제로 원하는 축을 중심으로 회전하는가?
- 회전 방향이 실제 로봇과 일치하는가?
- 회전할 때 Child Link가 정상적으로 연결되어 있는가?
- 상위 Joint가 움직일 때 하위 Link가 함께 움직이는가?
- Joint Limit 안에서 정상적으로 움직이는가?

Axis가 잘못되었다면 다음과 같은 현상이 나타날 수 있습니다.

- 팔이 옆으로 비틀리며 회전함
- 굽혀져야 하는 관절이 좌우로 움직임
- 실제 로봇과 RViz2 모델이 반대 방향으로 움직임
- Link가 관절 중심이 아닌 위치를 기준으로 회전함

Link가 잘못된 위치를 중심으로 회전한다면 Axis가 아니라 Joint Origin의 `xyz`를 먼저 확인해야 합니다.

회전축의 위치는 올바르지만 회전 방향이나 회전축 자체가 잘못되었다면 Origin의 `rpy`와 Axis의 `xyz`를 확인해야 합니다.

---

#### Axis 문제 구분

Axis와 관련된 문제를 다음과 같이 구분할 수 있습니다.

| 문제 현상 | 확인할 항목 |
| --- | --- |
| 회전 중심의 위치가 잘못됨 | Joint Origin의 `xyz` |
| 회전축 방향이 잘못됨 | Joint Origin의 `rpy`, Axis의 `xyz` |
| 양수와 음수 방향이 반대 | Axis 부호, 모터 방향 |
| Mesh만 잘못된 방향으로 표시됨 | Visual Origin의 `rpy` |
| 실제 로봇과 시뮬레이션 방향이 다름 | Axis, JointState 부호, 하드웨어 변환 |
| 이동 범위가 잘못됨 | Joint Limit |

문제의 원인을 구분하지 않고 Axis만 변경하면 다른 좌표 관계까지 잘못될 수 있으므로 각 요소의 역할을 정확하게 이해해야 합니다.

---

#### 정리

Axis는 Joint가 회전하거나 직선 이동하는 방향을 정의하는 핵심 요소입니다.

주요 내용을 정리하면 다음과 같습니다.

- Axis는 방향 벡터로 표현합니다.
- `revolute`와 `continuous`에서는 회전축을 정의합니다.
- `prismatic`에서는 직선 이동 방향을 정의합니다.
- `fixed` Joint에는 일반적으로 Axis가 필요하지 않습니다.
- Axis는 해당 Joint Frame을 기준으로 해석됩니다.
- Joint Frame의 방향은 Joint Origin의 `rpy`에 의해 결정됩니다.
- 양의 회전 방향은 오른손 법칙을 따릅니다.
- Axis의 부호를 반대로 설정하면 양의 움직임 방향도 반대로 바뀝니다.
- 실제 로봇과 URDF 모델의 움직임 방향을 일치시켜야 합니다.

Axis는 단순한 방향 값처럼 보이지만 로봇 관절의 실제 동작을 결정하는 중요한 정보입니다. 다음 장에서는 Joint의 움직임 범위와 최대 힘 및 속도를 제한하는 Limit에 대해 자세히 알아보겠습니다.
