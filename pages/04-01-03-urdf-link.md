# Link 이해

앞 장에서는 URDF의 기본 구조를 살펴보았습니다.

URDF는 크게 Robot, Link, Joint로 구성되며, 이 요소들이 Parent–Child 관계로 연결되어 하나의 로봇 구조를 만듭니다.

이번 장에서는 로봇 구조의 기본 단위인 Link에 대해 자세히 알아보겠습니다.

---

#### Link란?

Link는 로봇을 구성하는 물리적인 몸체를 의미합니다.

SO-ARM101을 예로 들면 다음과 같은 구조물을 Link로 표현할 수 있습니다.

- Base
- Shoulder
- Upper Arm
- Forearm
- Wrist
- Gripper

쉽게 생각하면 Link는 로봇의 뼈나 몸체이고, Joint는 Link와 Link를 연결하여 움직임을 만드는 관절입니다.

```
Link = 로봇의 몸체
Joint = 몸체 사이의 연결과 움직임
```

다만 실제 로봇의 모든 부품을 반드시 각각의 Link로 정의할 필요는 없습니다. 볼트나 브래킷처럼 서로 단단하게 고정되어 함께 움직이는 여러 부품은 하나의 Link로 묶어서 표현할 수 있습니다.

따라서 Link는 단순히 부품 하나가 아니라 **서로 상대적으로 움직이지 않는 하나의 강체(Rigid Body)**라고 이해하는 것이 더 정확합니다.

---

#### Link의 기본 구조

가장 단순한 Link는 다음과 같이 작성합니다.

```xml
<link name="base_link"/>
```

이 코드는 `base_link`라는 이름을 가진 Link를 정의합니다. 아직 외형이나 물리적인 정보가 없으므로 RViz2나 Gazebo 화면에는 표시되지 않습니다.

Link에 외형과 물리 정보를 추가하려면 다음과 같은 하위 태그를 사용합니다.

```xml
<link name="base_link">

    <visual>
        ...
    </visual>

    <collision>
        ...
    </collision>

    <inertial>
        ...
    </inertial>

</link>
```

각 태그의 역할은 다음과 같습니다.

- `visual`: RViz2나 Gazebo 화면에 표시할 외형
- `collision`: 시뮬레이션에서 충돌을 검사할 영역
- `inertial`: 질량, 무게중심 및 관성 정보

---

#### Link 이름

모든 Link는 `name` 속성을 이용하여 식별합니다.

```xml
<link name="wrist_link"/>
```

Link 이름은 Joint의 Parent와 Child를 지정하거나 TF 좌표계를 확인할 때 사용됩니다.

```xml
<joint name="wrist_joint" type="revolute">
    <parent link="forearm_link"/>
    <child link="wrist_link"/>
</joint>
```

위 코드에서 `forearm_link`와 `wrist_link`는 미리 정의된 Link의 이름이어야 합니다.

Link 이름은 하나의 로봇 안에서 중복되면 안 되며, 역할을 쉽게 파악할 수 있도록 작성하는 것이 좋습니다.

**권장하는 이름**

```
base_link
shoulder_link
upper_arm_link
forearm_link
wrist_link
gripper_link
```

**의미를 파악하기 어려운 이름**

```
link1
link2
link3
```

`link1`, `link2`와 같은 이름도 문법적으로는 사용할 수 있지만 로봇이 복잡해질수록 각 Link의 역할을 구분하기 어려워집니다.

일반적으로 Link 이름은 소문자와 언더스코어를 사용하는 스네이크 표기법으로 작성합니다.

---

#### Root Link

모든 URDF 로봇에는 트리 구조의 시작점이 되는 Link가 하나 있어야 합니다. 이를 **Root Link**라고 합니다.

대부분의 로봇에서는 `base_link`를 Root Link로 사용합니다.

```xml
<link name="base_link"/>
```

Root Link는 다른 Joint의 Child로 연결되지 않는 최상위 Link입니다.

```
base_link
    ↓
shoulder_link
    ↓
upper_arm_link
    ↓
forearm_link
    ↓
wrist_link
    ↓
gripper_link
```

로봇 내부의 Link 위치와 방향은 일반적으로 Root Link를 기준으로 계산됩니다.

다만 `base_link`가 자동으로 Gazebo의 `world` 좌표계에 고정되는 것은 아닙니다. 시뮬레이션에서 로봇을 바닥에 고정하려면 `world`와 `base_link`를 Fixed Joint로 연결하거나 별도의 고정 설정을 추가해야 합니다.

---

#### Link와 Joint의 관계

Link는 로봇의 몸체를 나타내고, Joint는 두 Link의 연결 관계와 움직임을 정의합니다.

```
Parent Link → Joint → Child Link
```

예를 들어 다음 구조에서 `base_link`는 Parent Link이고 `shoulder_link`는 Child Link입니다.

```xml
<link name="base_link"/>

<link name="shoulder_link"/>

<joint name="shoulder_pan" type="revolute">
    <parent link="base_link"/>
    <child link="shoulder_link"/>
</joint>
```

구조적으로 표현하면 다음과 같습니다.

```
base_link
    ↓ shoulder_pan
shoulder_link
```

Link 자체에 회전축이나 이동 범위를 정의하는 것이 아니라 Link를 연결하는 Joint에 이러한 정보를 정의합니다.

---

#### 모터와 Link

실제 로봇의 모터를 URDF로 표현할 때는 모터 전체를 단순히 Joint 하나로 표현하는 것이 아닙니다.

- 움직이지 않는 모터 몸체와 브래킷: Link에 포함
- 모터의 회전축과 움직임: Joint로 표현
- 회전축에 연결되어 함께 움직이는 구조물: Child Link로 표현

예를 들어 Shoulder 모터의 몸체가 Base에 고정되어 있고 출력축에 Arm이 연결되어 있다면 다음과 같이 이해할 수 있습니다.

```
base_link
    ↓ shoulder_joint
upper_arm_link
```

여기서 모터의 회전 동작은 `shoulder_joint`로 표현하고, 모터 출력축과 함께 움직이는 구조물은 `upper_arm_link`로 표현합니다.

실제 모델에서는 모터 하우징이나 브래킷을 별도의 Link로 만들 수도 있고, 주변 구조물과 하나의 Link로 묶을 수도 있습니다. 이는 필요한 시각화 수준과 제어 목적에 따라 결정합니다.

---

#### Link의 위치와 좌표계

각 Link는 자신의 기준이 되는 좌표계인 **Link Frame**을 가집니다.

하지만 Child Link가 Parent Link를 기준으로 어디에 위치하는지는 Link 태그가 아니라 두 Link를 연결하는 Joint의 `origin`에서 정의합니다.

```xml
<joint name="shoulder_joint" type="revolute">
    <parent link="base_link"/>
    <child link="shoulder_link"/>

    <origin xyz="0 0 0.1" rpy="0 0 0"/>
</joint>
```

위 예제에서 `shoulder_link`의 기준 좌표계는 `base_link`를 기준으로 Z축 방향으로 `0.1m` 떨어진 위치에 배치됩니다.

```
base_link
    └── shoulder_joint의 origin
            └── shoulder_link
```

따라서 로봇의 연결 위치는 Joint가 결정하고, Link 내부의 외형 위치는 Link 안에 있는 `visual`의 `origin`으로 조정합니다.

---

#### 기본 도형을 이용한 Link 시각화

Link를 화면에 표시하려면 `<visual>` 태그를 사용해야 합니다.

다음은 `base_link`를 박스 형태로 표시하는 예제입니다.

```xml
<link name="base_link">

    <visual>
        <geometry>
            <box size="0.1 0.1 0.05"/>
        </geometry>
    </visual>

</link>
```

각 태그의 역할은 다음과 같습니다.

- `<visual>`: 화면에 표시할 외형 정보
- `<geometry>`: 외형의 형상 정보
- `<box>`: 박스 형태의 기본 도형
- `size`: X, Y, Z 방향의 크기

URDF에서 길이의 기본 단위는 미터입니다.

```xml
<box size="0.1 0.1 0.05"/>
```

위 박스의 실제 크기는 다음과 같습니다.

- X축 길이: `0.1m` = `100mm`
- Y축 길이: `0.1m` = `100mm`
- Z축 길이: `0.05m` = `50mm`

URDF에서는 박스 외에도 다음과 같은 기본 도형을 사용할 수 있습니다.

```xml
<box size="0.1 0.1 0.05"/>
```

```xml
<cylinder radius="0.03" length="0.1"/>
```

```xml
<sphere radius="0.05"/>
```

기본 도형은 구조가 단순하고 계산량이 적기 때문에 초기 테스트나 충돌 영역을 정의할 때 유용합니다.

---

#### 3D Mesh를 이용한 Link 시각화

실제 로봇은 단순한 박스나 원통만으로 표현하기 어렵기 때문에 CAD 프로그램에서 만든 3D 모델을 사용합니다.

```xml
<link name="wrist_link">

    <visual>
        <geometry>
            <mesh filename="package://so_arm101_description/meshes/wrist.stl"/>
        </geometry>
    </visual>

</link>
```

`<mesh>` 태그의 `filename` 속성에는 3D 모델 파일의 경로를 작성합니다.

```xml
package://패키지이름/패키지_내부_경로
```

다음 경로가 지정되었다면

```xml
package://so_arm101_description/meshes/wrist.stl
```

ROS2는 `so_arm101_description` 패키지를 찾은 뒤, 그 안의 `meshes/wrist.stl` 파일을 불러옵니다.

URDF에서 주로 사용하는 3D 모델 형식은 다음과 같습니다.

- STL
- DAE
- OBJ

STL은 구조가 단순하고 다양한 3D CAD 프로그램에서 지원하기 때문에 로봇 모델링에서 많이 사용됩니다. 다만 색상이나 재질 정보가 필요한 경우에는 DAE와 같은 형식을 사용할 수 있습니다.

---

#### Link의 주요 구성 요소

Link는 필요에 따라 다음과 같은 정보를 포함할 수 있습니다.

| 구성 요소 | 역할 |
| --- | --- |
| `name` | Link를 식별하는 고유한 이름 |
| `visual` | 화면에 표시되는 외형 |
| `geometry` | 박스, 원통, 구 또는 Mesh 형상 |
| `collision` | 충돌 검사에 사용하는 형상 |
| `inertial` | 질량, 무게중심 및 관성 정보 |
| `origin` | Link Frame을 기준으로 한 외형 또는 물리 정보의 위치 |

RViz2에서 외형만 확인하는 단계에서는 `visual` 정보만 있어도 로봇을 표시할 수 있습니다. 하지만 Gazebo에서 물리 시뮬레이션을 수행하려면 `collision`과 `inertial` 정보도 정확하게 정의해야 합니다.

---

#### 정리

Link는 URDF에서 로봇의 물리적인 몸체를 표현하는 기본 단위입니다.

핵심 내용을 정리하면 다음과 같습니다.

- Link는 로봇을 구성하는 강체입니다.
- 고정되어 함께 움직이는 여러 부품을 하나의 Link로 표현할 수 있습니다.
- Link는 고유한 이름으로 식별합니다.
- Root Link는 로봇 트리 구조의 시작점입니다.
- Link 사이의 연결과 움직임은 Joint가 정의합니다.
- Child Link의 위치는 Joint의 `origin`에 의해 결정됩니다.
- Link에는 `visual`, `collision`, `inertial` 정보를 추가할 수 있습니다.
- 기본 도형이나 STL과 같은 3D Mesh로 외형을 표현할 수 있습니다.

Link는 로봇의 뼈대를 구성하는 가장 기본적인 요소입니다. 다음 장에서는 Link와 Link를 연결하고 실제 움직임을 정의하는 Joint에 대해 알아보겠습니다.
