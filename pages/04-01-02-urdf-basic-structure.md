# URDF 의 기본 구조

앞 장에서는 URDF가 무엇인지, 그리고 로봇을 제어하거나 시뮬레이션할 때 URDF가 왜 필요한지 살펴보았습니다.

이번 장에서는 URDF 파일이 실제로 어떤 구조로 작성되는지 알아보겠습니다.

URDF는 XML 형식으로 작성되며, 태그(Tag)와 속성(Attribute)을 이용하여 로봇의 구조를 정의합니다. XML은 HTML과 비슷한 형태지만 화면을 구성하는 것이 아니라 데이터를 계층적으로 표현하기 위해 사용합니다.

URDF 파일의 가장 기본적인 구조는 다음과 같습니다.

```xml
<?xml version="1.0"?>

<robot name="so_arm101">

    <link name="base_link"/>

    <link name="shoulder_link"/>

    <joint name="shoulder_pan" type="revolute">
        <parent link="base_link"/>
        <child link="shoulder_link"/>
    </joint>

</robot>
```

이 예제는 매우 단순하지만 URDF의 핵심 요소인 Robot, Link, Joint를 모두 포함하고 있습니다.

---

#### XML 태그와 속성

XML에서는 태그를 이용하여 데이터의 시작과 끝을 구분합니다.

**시작 태그와 종료 태그**

태그 안에 다른 내용이나 하위 태그가 포함될 때는 시작 태그와 종료 태그를 함께 사용합니다.

```xml
<tag_name>
    태그 내용
</tag_name>
```

URDF의 `<robot>`과 `<joint>`가 대표적인 예입니다.

```xml
<robot name="so_arm101">
    ...
</robot>
```

**자체 종료 태그**

태그 안에 별도의 내용이 없을 때는 `/`를 사용하여 바로 닫을 수 있습니다.

```xml
<tag_name/>
```

URDF의 `<link>`, `<parent>`, `<child>`와 같은 태그에서 자주 사용됩니다.

```xml
<link name="base_link"/>
<parent link="base_link"/>
```

**속성**

태그 안에 작성되는 `name`, `type`, `link`와 같은 항목을 속성이라고 합니다.

```xml
<joint name="shoulder_pan" type="revolute">
```

위 코드에는 다음과 같은 두 개의 속성이 있습니다.

- `name`: Joint의 이름
- `type`: Joint의 종류

XML과 URDF의 태그 이름은 대소문자를 구분합니다. 따라서 `<link>`와 `<Link>`는 서로 다른 태그로 인식됩니다. URDF에서 정의한 태그 이름은 반드시 소문자로 작성해야 합니다.

---

#### Robot 태그

URDF의 가장 바깥쪽에는 반드시 `<robot>` 태그가 있어야 합니다.

```xml
<robot name="so_arm101">
</robot>
```

`<robot>` 태그는 하나의 로봇 전체를 나타내는 최상위 요소입니다. `name` 속성에는 로봇의 이름을 작성합니다.

```xml
<robot name="my_robot">
</robot>
```

위와 같이 작성하면 로봇의 이름은 `my_robot`이 됩니다.

하나의 URDF 파일에는 일반적으로 하나의 `<robot>` 태그만 존재하며, 모든 Link와 Joint는 이 태그 안에 작성해야 합니다.

```
robot
├── link
├── link
├── joint
├── link
└── joint
```

`<robot>` 태그는 로봇의 모든 구성 요소를 담는 가장 상위 컨테이너라고 이해할 수 있습니다.

---

#### Link 태그

Link는 로봇을 구성하는 물리적인 몸체를 정의합니다.

```xml
<link name="base_link"/>
```

`name` 속성은 Link의 이름을 나타냅니다. 하나의 로봇 안에서 Link의 이름은 서로 중복되지 않아야 합니다.

SO-ARM101은 다음과 같은 Link로 구성할 수 있습니다.

- `base_link`
- `shoulder_link`
- `upper_arm_link`
- `forearm_link`
- `wrist_link`
- `gripper_link`

각 Link는 로봇을 구성하는 독립적인 구조물을 의미합니다.

예를 들면 다음과 같습니다.

- 로봇의 바닥과 몸체
- 금속 프레임과 브래킷
- 상완과 전완 구조물
- 손목 구조물
- 그리퍼 몸체

Link에는 이후 다음과 같은 세부 정보를 추가할 수 있습니다.

- `visual`: 화면에 표시할 외형
- `collision`: 충돌 검사에 사용할 형상
- `inertial`: 질량, 무게중심 및 관성 정보

Link가 다른 Link에 대해 어떻게 움직이는지는 Link 자체가 아니라 두 Link 사이에 정의된 Joint에 의해 결정됩니다.

---

#### Joint 태그

Joint는 두 Link를 연결하고 움직임의 규칙을 정의합니다.

```xml
<joint name="shoulder_pan" type="revolute">
    <parent link="base_link"/>
    <child link="shoulder_link"/>
</joint>
```

위 코드의 연결 관계는 다음과 같습니다.

- Joint 이름: `shoulder_pan`
- Joint 종류: `revolute`
- Parent Link: `base_link`
- Child Link: `shoulder_link`

구조적으로 표현하면 다음과 같습니다.

```
base_link
    ↓
shoulder_pan
    ↓
shoulder_link
```

Joint는 반드시 하나의 Parent Link와 하나의 Child Link를 가져야 합니다.

- `parent`: 움직임의 기준이 되는 부모 Link
- `child`: Joint에 의해 연결되고 움직이는 자식 Link

이러한 Parent와 Child의 연결 관계가 쌓이면서 로봇 전체의 구조가 만들어집니다.

---

#### Link와 Joint의 관계

URDF의 로봇 구조는 Link와 Joint가 번갈아 연결되는 형태를 가집니다.

```
Link → Joint → Link → Joint → Link
```

SO-ARM101을 단순하게 표현하면 다음과 같습니다.

```
base_link
    ↓ shoulder_pan
shoulder_link
    ↓ shoulder_lift
upper_arm_link
    ↓ elbow_flex
forearm_link
    ↓ wrist_flex
wrist_link
    ↓ gripper_joint
gripper_link
```

이처럼 Link와 Joint가 연속적으로 연결된 구조를 **Kinematic Chain**이라고 합니다.

로봇은 이 연결 구조와 각 Joint의 상태를 이용하여 각 Link의 위치와 방향을 계산합니다.

URDF 파일에서 Link와 Joint가 작성된 순서가 로봇의 연결 순서를 결정하는 것은 아닙니다. 실제 연결 구조는 Joint 안의 `<parent>`와 `<child>` 태그를 기준으로 결정됩니다.

---

#### Root Link

모든 URDF 로봇에는 구조의 시작점이 되는 Link가 하나 있어야 합니다. 이를 **Root Link**라고 합니다.

대부분의 로봇에서는 다음과 같이 `base_link`를 Root Link로 사용합니다.

```xml
<link name="base_link"/>
```

Root Link는 다른 Joint의 Child로 연결되지 않는 Link입니다. 즉, 부모 Link를 가지지 않는 로봇 구조의 최상위 Link입니다.

```
base_link
    └── shoulder_pan
            └── shoulder_link
```

Root Link는 로봇 내부 좌표계의 기준이 됩니다. 다만 `base_link`가 자동으로 시뮬레이션의 `world` 좌표계에 고정되는 것은 아닙니다. 필요한 경우 별도의 Fixed Joint나 시뮬레이션 설정을 이용하여 `world`와 연결해야 합니다.

---

#### Tree 구조

URDF는 하나의 Root Link에서 여러 Child Link가 분기되는 트리(Tree) 구조를 사용합니다.

```
base_link
└── shoulder_pan
    └── shoulder_link
        └── shoulder_lift
            └── upper_arm_link
                └── elbow_flex
                    └── forearm_link
```

트리 구조에서는 Parent Link의 좌표를 기준으로 Child Link의 위치와 방향을 순서대로 계산할 수 있습니다.

이러한 계층 관계는 이후 학습할 TF Tree와 Forward Kinematics에서 매우 중요하게 사용됩니다.

일반적인 URDF는 트리 구조를 전제로 하므로 하나의 Child Link가 여러 Parent Link에 동시에 연결되거나 연결 구조가 원형으로 이어지는 폐쇄형 구조를 직접 표현하기 어렵습니다.

---

#### 기본 구조 정리

URDF의 기본 구조는 다음 세 가지 핵심 요소로 구성됩니다.

- `<robot>`: 로봇 전체를 정의하는 최상위 태그
- `<link>`: 로봇을 구성하는 물리적인 몸체
- `<joint>`: 두 Link의 연결 관계와 움직임을 정의하는 관절

이들의 관계를 정리하면 다음과 같습니다.

```
Robot
└── Root Link
    └── Joint
        └── Child Link
            └── Joint
                └── Child Link
```

URDF에서 가장 중요한 것은 Link와 Joint의 Parent–Child 관계입니다. 이 구조가 로봇의 기본 뼈대가 되며, 이후 Visual, Collision, Inertial, Origin, Axis, Limit 등의 정보를 추가하여 완전한 로봇 모델을 구성하게 됩니다.
