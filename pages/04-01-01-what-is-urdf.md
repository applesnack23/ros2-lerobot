# URDF 란?

로봇을 제어하거나 시뮬레이션하려면 먼저 로봇의 구조를 컴퓨터가 이해할 수 있는 형태로 정의해야 합니다.

사람은 로봇을 보면서 몸체와 관절의 위치, 움직이는 방향, 각 구조물의 크기 등을 자연스럽게 파악할 수 있습니다. 하지만 컴퓨터는 이러한 정보를 스스로 판단할 수 없으므로 일정한 규칙에 따라 로봇의 구조를 기술해야 합니다.

이때 사용하는 것이 **URDF(Unified Robot Description Format)**입니다.

URDF는 로봇의 구조를 정의하기 위한 XML 기반의 표준 형식입니다. 쉽게 말하면 로봇의 기계적인 구조를 컴퓨터가 읽을 수 있도록 작성한 디지털 설계도라고 할 수 있습니다.

---

#### URDF의 역할

URDF에는 로봇을 구성하는 몸체와 관절뿐만 아니라 각 부품의 위치, 방향, 크기, 질량, 충돌 영역과 같은 정보가 포함됩니다.

ROS2에서는 URDF를 기반으로 다음과 같은 작업을 수행할 수 있습니다.

- RViz2에서 로봇의 외형과 관절 움직임 시각화
- JointState를 이용한 관절 상태 반영
- TF를 이용한 링크 사이의 좌표 관계 계산
- Gazebo를 이용한 물리 시뮬레이션
- MoveIt을 이용한 경로 계획과 충돌 회피
- 실제 로봇과 시뮬레이션을 연결한 디지털트윈 구성

따라서 URDF는 단순히 로봇의 외형을 표현하는 파일이 아니라 로봇의 구조를 이해하고 제어하기 위한 가장 기본적인 데이터입니다.

---

#### URDF의 핵심 구성 요소

URDF는 크게 **Link**와 **Joint**라는 두 가지 요소로 구성됩니다.

**Link**

Link는 로봇을 구성하는 물리적인 몸체를 의미합니다.

SO-ARM101을 예로 들면 다음과 같은 구조물이 각각 하나의 Link가 될 수 있습니다.

- Base
- Shoulder
- Upper Arm
- Forearm
- Wrist
- Gripper

Link에는 필요에 따라 다음과 같은 정보를 정의할 수 있습니다.

- `visual`: 화면에 표시할 외형
- `collision`: 충돌을 검사할 영역
- `inertial`: 질량, 무게중심 및 관성 정보

**Joint**

Joint는 두 Link를 연결하고 움직임의 규칙을 정의합니다.

Joint에는 다음과 같은 정보가 포함됩니다.

- 어떤 Link와 어떤 Link를 연결하는지
- 회전 또는 직선 이동이 가능한지
- 어느 축을 기준으로 움직이는지
- 움직일 수 있는 범위가 어디까지인지
- 최대 속도와 힘이 얼마인지

SO-ARM101의 구조를 간단하게 표현하면 다음과 같습니다.

```
Base → Shoulder → Upper Arm → Forearm → Wrist → Gripper
```

각 구조물은 Link가 되고 Link 사이를 연결하는 관절은 Joint가 됩니다. 이러한 연결 관계가 쌓여 하나의 계층적인 로봇 구조를 형성합니다.

---

#### URDF의 기본 구조

URDF는 XML 형식으로 작성합니다. 다음은 두 개의 Link와 하나의 Joint를 연결한 간단한 예입니다.

```xml
<?xml version="1.0"?>
<robot name="so_arm101">

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

</robot>
```

이 예제에는 다음과 같은 내용이 정의되어 있습니다.

- 로봇 이름: `so_arm101`
- 부모 Link: `base_link`
- 자식 Link: `shoulder_link`
- Joint 이름: `shoulder_pan`
- Joint 형식: `revolute`
- 회전축: Z축
- 관절의 최소·최대 회전 범위
- 관절의 최대 힘과 속도

`parent`와 `child`는 두 Link의 연결 관계를 정의합니다. `origin`은 부모 Link를 기준으로 자식 Joint가 위치하는 좌표와 방향을 나타냅니다. `axis`는 관절이 움직이는 축을 지정하며, `limit`는 관절의 움직임 범위를 제한합니다.

---

#### URDF와 Xacro

로봇의 구조가 복잡해질수록 URDF 파일은 길어지고 반복되는 코드도 많아집니다. 또한 링크의 길이, 질량, 관절 제한값 등을 변경할 때 여러 위치를 직접 수정해야 하는 불편함이 있습니다.

이러한 문제를 해결하기 위해 사용하는 것이 **Xacro(XML Macros)**입니다.

Xacro는 URDF를 더욱 효율적으로 작성하기 위한 확장 문법으로 다음과 같은 기능을 제공합니다.

- 변수 선언 및 재사용
- 수식을 이용한 값 계산
- 반복되는 구조의 매크로화
- 다른 Xacro 파일 불러오기
- 조건에 따른 구성 변경
- 실제 로봇과 시뮬레이션 설정 분리

Xacro로 작성된 파일은 실행 과정에서 일반 URDF 형식으로 변환되어 사용됩니다.

---

#### 마무리

URDF는 로봇의 Link, Joint, 위치, 방향 및 물리적 특성을 정의하는 로봇 구조의 기본 설계도입니다. RViz2, TF, Gazebo, ros2_control, MoveIt과 같은 ROS2의 주요 기능도 URDF에 정의된 로봇 구조를 바탕으로 동작합니다.

앞으로는 SO-ARM101의 실제 URDF와 Xacro 파일을 분석하면서 Link와 Joint의 연결 구조를 이해하고, 이를 기반으로 RViz2 시각화, TF 좌표 확인, Gazebo 시뮬레이션 및 MoveIt 경로 계획까지 단계적으로 확장합니다.
