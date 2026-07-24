# Collision 이해

앞 장에서는 Link의 외형을 정의하는 `Visual`에 대해 알아보았습니다.

Visual을 사용하면 로봇의 모습을 RViz2나 Gazebo 화면에 표시할 수 있습니다. 하지만 로봇을 화면에 보여주는 것만으로는 물리 시뮬레이션이나 경로 계획을 수행할 수 없습니다.

로봇을 움직이려면 컴퓨터가 다음과 같은 상황을 판단할 수 있어야 합니다.

- 로봇이 벽이나 바닥과 부딪히는가?
- 로봇의 Link끼리 서로 충돌하는가?
- 로봇이 작업물과 충돌하는가?
- 그리퍼가 물체에 접촉했는가?
- 계획한 경로를 안전하게 실행할 수 있는가?

이러한 충돌을 계산하기 위해 사용하는 요소가 `Collision`입니다.

> **Collision은 Link의 충돌 판정에 사용할 형상을 정의합니다.**
> 

Visual과 Collision의 차이는 다음과 같습니다.

```
Visual    = 화면에 보여주기 위한 외형
Collision = 충돌 계산에 사용하기 위한 형상
```

두 요소는 같은 Link 안에 정의되지만 사용 목적은 다릅니다.

---

#### Collision이 필요한 이유

컴퓨터는 화면에 보이는 3D 모델만 보고 자동으로 충돌을 판단하지 않습니다. 충돌 판정에 사용할 형상을 명확하게 정의해야 합니다.

예를 들어 로봇 팔이 벽 가까이 이동한다고 가정해 보겠습니다. Collision이 정의되어 있다면 시뮬레이터나 경로 계획 프로그램은 로봇과 벽 사이의 충돌 가능성을 계산할 수 있습니다.

Collision이 없거나 잘못 정의되어 있으면 다음과 같은 문제가 발생할 수 있습니다.

- 로봇이 바닥이나 벽을 통과함
- 로봇 Link끼리 서로 겹침
- 그리퍼가 작업물을 통과함
- 실제로는 충돌하는 경로가 생성됨
- 충돌하지 않는 자세를 충돌로 잘못 판단함
- MoveIt 경로 계획이 불필요하게 실패함
- Gazebo 물리 시뮬레이션이 비정상적으로 동작함

따라서 Collision은 Gazebo와 MoveIt을 사용할 때 매우 중요한 정보입니다.

---

#### Collision 기본 구조

Collision의 기본 구조는 다음과 같습니다.

```xml
<link name="base_link">
    <collision>
        <geometry>
            <box size="0.1 0.1 0.05"/>
        </geometry>
    </collision>
</link>
```

Collision은 일반적으로 다음과 같은 구조를 가집니다.

```
collision
├── origin
└── geometry
    └── shape
```

각 요소의 역할은 다음과 같습니다.

| 요소 | 역할 |
| --- | --- |
| `<collision>` | 충돌 형상 정보 |
| `<origin>` | 충돌 형상의 위치와 방향 |
| `<geometry>` | 충돌 계산에 사용할 형상 |
| Shape | Box, Cylinder, Sphere 또는 Mesh |

Collision은 Visual과 달리 색상이나 재질을 표시하는 요소가 아닙니다. 따라서 일반적으로 `<material>`을 사용하지 않습니다.

---

#### Collision에서 사용할 수 있는 Shape

Collision에서도 Visual과 마찬가지로 기본 형상과 Mesh를 사용할 수 있습니다.

- Box
- Cylinder
- Sphere
- Mesh

같은 Geometry를 사용할 수 있지만, Collision에서는 정확한 외형보다 계산 효율이 더 중요합니다.

#### Box

Box는 직육면체 형태의 충돌 영역을 정의합니다.

```xml
<collision>
    <geometry>
        <box size="0.2 0.1 0.05"/>
    </geometry>
</collision>
```

`size` 값은 X, Y, Z축 방향의 전체 크기를 의미합니다.

```
X = 0.2 m
Y = 0.1 m
Z = 0.05 m
```

Box의 장점은 다음과 같습니다.

- 충돌 계산이 빠름
- 구조가 단순함
- Base나 브래킷 표현에 적합함
- 시뮬레이션이 안정적임

단점은 곡면이 많은 부품을 정확하게 표현하기 어렵다는 것입니다.

#### Cylinder

Cylinder는 원통 형태의 충돌 영역을 정의합니다.

```xml
<collision>
    <geometry>
        <cylinder radius="0.03" length="0.2"/>
    </geometry>
</collision>
```

각 속성의 의미는 다음과 같습니다.

- `radius`: 원통의 반지름
- `length`: 원통의 전체 길이

Cylinder는 다음과 같은 부품의 충돌 형상을 표현하기에 적합합니다.

- 로봇 팔 Link
- 회전축
- 모터 Body
- 롤러
- 파이프

Cylinder의 기본 길이 방향은 Z축입니다. 실제 Link의 방향과 다르면 Collision Origin의 `rpy`를 사용하여 회전해야 합니다.

#### Sphere

Sphere는 구 형태의 충돌 영역을 정의합니다.

```xml
<collision>
    <geometry>
        <sphere radius="0.05"/>
    </geometry>
</collision>
```

Sphere는 충돌 계산이 단순하고 방향에 영향을 받지 않는다는 장점이 있습니다.

다음과 같은 부분에 사용할 수 있습니다.

- 관절 주변
- 둥근 센서
- 로봇 끝단
- 단순화된 그리퍼
- 구형 작업물

단순한 충돌 검사를 위해 여러 복잡한 구조를 하나의 Sphere로 근사할 수도 있습니다.

#### Mesh

Mesh를 사용하면 실제 3D 모델과 비슷한 충돌 형상을 정의할 수 있습니다.

```xml
<collision>
    <geometry>
        <mesh
            filename="package://so_arm101_description/meshes/arm_collision.stl"
            scale="0.001 0.001 0.001"/>
    </geometry>
</collision>
```

Mesh는 다음과 같은 장점이 있습니다.

- 실제 로봇과 유사한 충돌 영역 표현
- 복잡한 형상의 정확한 근사
- 정밀한 경로 계획에 활용 가능

하지만 다음과 같은 단점도 있습니다.

- 충돌 계산량 증가
- 시뮬레이션 성능 저하
- 복잡한 Mesh에서 물리 계산이 불안정할 수 있음
- 경로 계획 시간이 증가할 수 있음

따라서 Visual에 사용하는 정교한 Mesh를 Collision에도 그대로 사용하는 것은 신중해야 합니다.

특히 오목한 형태를 가진 복잡한 Mesh는 시뮬레이터의 물리 엔진에서 계산하기 어려울 수 있습니다. 필요한 경우 단순한 여러 개의 볼록 형상으로 분해하여 사용합니다.

---

#### Collision Origin

Collision에도 Origin을 사용할 수 있습니다.

```xml
<collision>
    <origin xyz="0 0 0.1" rpy="0 0 0"/>

    <geometry>
        <box size="0.1 0.1 0.2"/>
    </geometry>
</collision>
```

Collision Origin은 Link Frame을 기준으로 충돌 형상의 위치와 방향을 조정합니다.

위 코드는 높이가 0.2 m인 Box의 중심을 Z축 방향으로 0.1 m 이동합니다. 따라서 Box의 아래쪽 면이 Link Frame의 원점에 위치합니다.

Collision Origin과 Visual Origin은 서로 독립적입니다.

```xml
<link name="arm_link">
    <visual>
        <origin xyz="0 0 0.1" rpy="0 0 0"/>
        <geometry>
            <cylinder radius="0.03" length="0.2"/>
        </geometry>
    </visual>

    <collision>
        <origin xyz="0 0 0.1" rpy="0 0 0"/>
        <geometry>
            <cylinder radius="0.035" length="0.2"/>
        </geometry>
    </collision>
</link>
```

보통 Visual과 Collision이 같은 위치에 있어야 하므로 `origin` 값을 비슷하게 설정합니다.

Collision Origin이 잘못되면 화면상으로는 로봇이 물체와 떨어져 있는데 충돌이 발생하거나, 로봇이 겹쳐 있는데도 충돌하지 않는 문제가 생길 수 있습니다.

---

#### Visual과 Collision 비교

Visual과 Collision의 차이를 정리하면 다음과 같습니다.

| 구분 | Visual | Collision |
| --- | --- | --- |
| 목적 | 화면 표시 | 충돌 계산 |
| 주요 사용자 | 사람 | 시뮬레이터와 경로 계획 프로그램 |
| 형상 | 정교한 Mesh 사용 가능 | 단순한 형상 권장 |
| 색상 | Material 사용 가능 | 일반적으로 사용하지 않음 |
| 계산 비용 | 비교적 영향이 작음 | 성능에 직접적인 영향 |
| RViz2 | 기본적으로 표시 | 설정을 통해 확인 가능 |
| Gazebo | 로봇의 외형 표시 | 접촉 및 충돌 계산 |
| MoveIt | 로봇 외형 표시 | 충돌 회피와 경로 계획 |

핵심 개념은 다음과 같습니다.

> **Visual은 사람이 보는 모델이고, Collision은 컴퓨터가 계산하는 모델입니다.**
> 

---

#### Collision을 단순하게 만드는 이유

Collision은 시뮬레이션과 경로 계획이 실행되는 동안 반복해서 계산됩니다.

로봇이 움직일 때마다 시뮬레이터는 다음과 같은 계산을 수행할 수 있습니다.

- 각 Link와 바닥의 충돌
- 각 Link와 주변 물체의 충돌
- 로봇 Link 사이의 충돌
- 작업물과 그리퍼의 접촉
- 계획 경로상의 충돌 가능성

Visual과 동일한 정교한 Mesh를 Collision에 사용하면 계산해야 하는 삼각형의 수가 크게 증가합니다.

따라서 일반적으로 다음과 같이 구성합니다.

```
Visual    = 정교한 실제 외형 Mesh
Collision = 단순한 Box, Cylinder 또는 Sphere
```

예를 들어 복잡한 로봇 팔 Link를 하나의 Cylinder로 근사할 수 있습니다.

```xml
<visual>
    <geometry>
        <mesh filename="package://so_arm101_description/meshes/upper_arm.stl"/>
    </geometry>
</visual>

<collision>
    <geometry>
        <cylinder radius="0.035" length="0.2"/>
    </geometry>
</collision>
```

Collision을 너무 작게 만들면 실제 충돌을 감지하지 못할 수 있습니다. 반대로 너무 크게 만들면 충돌하지 않는 자세도 충돌로 판단할 수 있습니다.

따라서 단순하게 만들되 실제 외형을 충분히 포함하도록 크기를 설정해야 합니다.

---

#### Visual과 Collision 크기 비교

Collision은 Visual보다 약간 크게 정의할 수도 있습니다.

```xml
<visual>
    <geometry>
        <box size="0.1 0.1 0.1"/>
    </geometry>
</visual>

<collision>
    <geometry>
        <box size="0.11 0.11 0.11"/>
    </geometry>
</collision>
```

이와 같이 Collision을 약간 크게 만들면 로봇이 실제 물체와 접촉하기 전에 충돌을 감지할 수 있습니다.

하지만 Collision이 지나치게 크면 다음과 같은 문제가 생길 수 있습니다.

- 경로 계획 공간 감소
- 도달할 수 있는 자세 감소
- 불필요한 우회 경로 생성
- 정상적인 자세를 충돌로 판단

따라서 안전 여유를 어느 정도 적용할지는 로봇의 구조와 사용 목적에 따라 결정해야 합니다.

---

#### Link 전체 예제

다음은 Visual과 Collision을 모두 정의한 `arm_link` 예제입니다.

```xml
<link name="arm_link">
    <visual>
        <origin xyz="0 0 0.1" rpy="0 0 0"/>

        <geometry>
            <mesh
                filename="package://so_arm101_description/meshes/arm.stl"
                scale="0.001 0.001 0.001"/>
        </geometry>

        <material name="gray">
            <color rgba="0.6 0.6 0.6 1"/>
        </material>
    </visual>

    <collision>
        <origin xyz="0 0 0.1" rpy="0 0 0"/>

        <geometry>
            <cylinder radius="0.035" length="0.2"/>
        </geometry>
    </collision>
</link>
```

위 Link는 다음과 같이 구성됩니다.

- Visual: 실제 로봇과 유사한 정교한 Mesh
- Collision: 계산이 빠른 단순 원통
- Visual과 Collision의 중심 위치: Z축 방향 0.1 m
- Visual 색상: 회색

이와 같이 화면에는 실제 로봇과 비슷한 외형을 표시하면서, 충돌 계산에는 단순한 형상을 사용하는 것이 일반적입니다.

---

#### 여러 형상으로 Collision 구성하기

로봇 Link가 하나의 Box나 Cylinder로 표현하기 어려운 경우에는 여러 개의 단순한 형상으로 나누어 근사할 수 있습니다.

예를 들어 L자 형태의 Link라면 다음과 같이 단순화할 수 있습니다.

```
L자 형태 Link
├── Box 1
└── Box 2
```

다만 여러 개의 Collision 요소 사용 여부와 처리 방식은 사용하는 URDF 파서와 시뮬레이션 환경에 따라 확인해야 합니다. 호환성이 중요한 경우에는 하나의 단순 Mesh로 결합하거나 시뮬레이터에서 지원하는 형식을 사용하는 것이 안전합니다.

---

#### Gazebo에서 Collision의 역할

Gazebo는 Collision 정보를 사용하여 물리적인 접촉을 계산합니다.

대표적인 계산 항목은 다음과 같습니다.

- 바닥과의 접촉
- 로봇과 작업물의 접촉
- Link 사이의 충돌
- 그리퍼와 물체의 접촉
- 마찰과 반발
- 물체 밀기와 잡기

Collision이 없다면 Link의 Visual이 화면에 보이더라도 물리적으로는 존재하지 않는 것처럼 처리될 수 있습니다.

예를 들어 로봇의 Base에 Collision이 없다면 상황에 따라 바닥을 통과하거나 다른 물체가 Base를 통과할 수 있습니다.

다만 Gazebo에서 안정적인 물리 시뮬레이션을 수행하려면 Collision뿐 아니라 다음 장에서 다룰 `Inertial` 정보도 필요합니다.

- 질량
- 무게중심
- 관성 모멘트

#### MoveIt에서 Collision의 역할

MoveIt은 로봇의 Collision 형상을 이용하여 경로상의 충돌 가능성을 확인합니다.

대표적으로 다음과 같은 충돌을 검사합니다.

**Self Collision**

로봇의 Link끼리 서로 충돌하는지 검사합니다.

예를 들어 손목이 로봇의 Base 또는 Upper Arm과 부딪히는지 확인할 수 있습니다.

**Environment Collision**

로봇이 주변 환경과 충돌하는지 검사합니다.

예를 들면 다음과 같습니다.

- 작업대
- 벽
- 안전 펜스
- 컨베이어
- 작업물
- 주변 설비

**Path Collision**

로봇이 시작 자세에서 목표 자세까지 이동하는 전체 경로에서 충돌이 발생하는지 확인합니다.

시작점과 목표점이 모두 안전하더라도 이동 중간에 장애물과 부딪힐 수 있기 때문에 경로 전체를 검사해야 합니다.

MoveIt의 자기 충돌 설정에는 URDF 외에도 SRDF의 Self-Collision Matrix가 사용될 수 있습니다. 서로 인접하여 항상 접촉하거나 충돌 검사가 불필요한 Link 쌍을 제외하면 경로 계획 성능을 높일 수 있습니다.

---

#### RViz2에서 Collision 확인하기

RViz2에서는 RobotModel의 표시 설정을 변경하여 Collision 형상을 확인할 수 있습니다.

일반적인 확인 과정은 다음과 같습니다.

1. `robot_state_publisher` 실행
2. RViz2 실행
3. `RobotModel` Display 추가
4. Visual 표시 비활성화
5. Collision 표시 활성화
6. 각 Link의 Collision 위치와 크기 확인

RViz2 버전에 따라 설정 이름은 조금 다를 수 있지만 일반적으로 다음 항목을 확인합니다.

```
RobotModel
├── Visual Enabled
└── Collision Enabled
```

Collision을 확인할 때는 다음 내용을 살펴봐야 합니다.

- Collision 형상이 Visual을 충분히 포함하는가?
- Collision 위치가 Visual과 일치하는가?
- Collision 방향이 올바른가?
- Collision이 지나치게 크거나 작지 않은가?
- 인접한 Link가 기본 자세에서 과도하게 겹치지 않는가?

#### Collision 문제 확인

Collision 관련 문제가 발생하면 다음 항목을 확인합니다.

| 문제 | 확인할 항목 |
| --- | --- |
| 로봇이 바닥을 통과함 | Link에 Collision이 정의되어 있는지 확인 |
| 물체가 로봇을 통과함 | 로봇과 물체의 Collision 확인 |
| 충돌하지 않았는데 경로 계획 실패 | Collision의 크기와 위치 확인 |
| 실제로 겹치는데 충돌하지 않음 | Collision이 너무 작지 않은지 확인 |
| Collision 위치가 어긋남 | Collision Origin의 `xyz` 확인 |
| Collision 방향이 틀어짐 | Collision Origin의 `rpy` 확인 |
| 시뮬레이션이 느림 | 복잡한 Mesh를 사용하는지 확인 |
| Mesh 크기가 맞지 않음 | `scale`과 모델 단위 확인 |
| 자기 충돌이 계속 발생함 | Link 형상과 SRDF 충돌 설정 확인 |

---

#### Collision 작성 시 권장사항

Collision을 작성할 때는 다음 기준을 적용하는 것이 좋습니다.

- 가능하면 Box, Cylinder, Sphere를 사용합니다.
- 실제 외형을 충분히 포함하도록 크기를 설정합니다.
- 필요 이상으로 크게 만들지 않습니다.
- 복잡한 Mesh 사용은 최소화합니다.
- Visual Origin과 Collision Origin을 비교합니다.
- RViz2에서 Collision 형상을 직접 확인합니다.
- Gazebo와 MoveIt에서 각각 동작을 검증합니다.
- 실제 로봇과 시뮬레이션의 차이를 고려합니다.

---

#### 정리

Collision은 Link의 충돌 계산용 형상을 정의하는 요소입니다.

- `<collision>`: 충돌 정보의 시작
- `<geometry>`: 충돌 형상
- `<box>`: 직육면체 충돌 영역
- `<cylinder>`: 원통형 충돌 영역
- `<sphere>`: 구형 충돌 영역
- `<mesh>`: 외부 3D 모델을 이용한 충돌 영역
- `<origin>`: 충돌 형상의 위치와 방향
- `scale`: Mesh의 크기 조정

가장 중요한 차이는 다음과 같습니다.

> **Visual은 화면에 보이는 외형이고, Collision은 물리 및 경로 계획 계산에 사용하는 형상입니다.**
> 

일반적으로 Visual에는 정교한 Mesh를 사용하고 Collision에는 단순한 기본 형상을 사용합니다. 이를 통해 실제 로봇과 비슷한 외형을 유지하면서도 시뮬레이션과 경로 계획의 계산 성능을 높일 수 있습니다.

다음 장에서는 로봇의 질량, 무게중심, 관성 모멘트를 정의하는 `Inertial`에 대해 알아보겠습니다.
