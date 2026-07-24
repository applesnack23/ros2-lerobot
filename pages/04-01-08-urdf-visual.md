# Visual 이해

앞 장에서는 Joint의 움직임 범위와 최대 성능을 정의하는 `Limit`에 대해 알아보았습니다.

지금까지 살펴본 요소를 이용하면 로봇의 기본 구조와 움직임을 정의할 수 있습니다.

- `Link`: 로봇을 구성하는 몸체
- `Joint`: Link 사이의 연결과 움직임
- `Origin`: 연결 위치와 방향
- `Axis`: Joint가 움직이는 축
- `Limit`: Joint의 동작 범위와 성능 한계

하지만 Link를 다음과 같이 이름만 정의하면 RViz2나 Gazebo 화면에서 로봇의 외형을 확인할 수 없습니다.

```xml
<link name="arm_link"/>
```

이 코드는 `arm_link`라는 Link가 존재한다는 것만 알려줄 뿐, Link가 어떤 모양과 크기를 가지는지는 정의하지 않았기 때문입니다.

Link를 화면에 표시하려면 외형 정보를 추가해야 합니다. 이 역할을 하는 요소가 `Visual`입니다.

> **Visual은 Link가 화면에 어떻게 보일지를 정의합니다.**
> 

핵심 관계는 다음과 같습니다.

```
Link   = 로봇의 몸체
Visual = 몸체가 화면에 보이는 모습
```

---

#### Visual 기본 구조

Visual의 기본 구조는 다음과 같습니다.

```xml
<link name="base_link">
    <visual>
        <geometry>
            <box size="0.1 0.1 0.05"/>
        </geometry>
    </visual>
</link>
```

Visual은 일반적으로 다음과 같은 구조로 구성됩니다.

```
visual
├── origin
├── geometry
│   └── shape
└── material
```

각 요소의 역할은 다음과 같습니다.

| 요소 | 역할 |
| --- | --- |
| `<visual>` | Link의 외형 정보 |
| `<origin>` | 외형의 위치와 방향 |
| `<geometry>` | 외형의 형상 |
| `<material>` | 외형의 색상과 투명도 |

`origin`과 `material`은 필요한 경우에만 작성할 수 있지만, 실제 형상을 지정하는 `geometry`는 반드시 필요합니다.

---

#### Geometry

`geometry`는 Link의 외형이 어떤 모양인지 정의합니다.

```xml
<geometry>
    <box size="0.1 0.1 0.05"/>
</geometry>
```

위 코드는 다음 크기의 박스를 의미합니다.

- X축 방향 크기: 0.1 m
- Y축 방향 크기: 0.1 m
- Z축 방향 크기: 0.05 m

센티미터 단위로 변환하면 다음과 같습니다.

```
10 cm × 10 cm × 5 cm
```

URDF에서 길이의 기본 단위는 미터입니다.

```
0.01 m = 1 cm
0.1 m  = 10 cm
1.0 m  = 100 cm
```

단위를 잘못 입력하면 로봇이 지나치게 크거나 작게 표시될 수 있으므로 주의해야 합니다.

---

#### 기본 Shape

URDF는 다음과 같은 기본 형상을 지원합니다.

- Box
- Cylinder
- Sphere
- Mesh

**Box**

Box는 직육면체 형태를 정의합니다.

```xml
<geometry>
    <box size="0.2 0.1 0.05"/>
</geometry>
```

`size`는 X, Y, Z축 방향의 전체 크기를 의미합니다.

```
X = 0.2 m
Y = 0.1 m
Z = 0.05 m
```

Box는 다음과 같은 구조를 단순하게 표현할 때 사용합니다.

- 로봇의 Base
- 제어반
- 브래킷
- 테스트용 로봇 몸체
- 작업물

**Cylinder**

Cylinder는 원통 형태를 정의합니다.

```xml
<geometry>
    <cylinder radius="0.03" length="0.2"/>
</geometry>
```

각 속성의 의미는 다음과 같습니다.

- `radius`: 원통의 반지름
- `length`: 원통의 전체 길이

위 코드는 반지름이 3 cm이고 길이가 20 cm인 원통을 의미합니다.

URDF의 기본 Cylinder는 길이 방향이 Z축을 향합니다. 다른 방향으로 배치하려면 Visual Origin의 `rpy`를 사용해 회전해야 합니다.

Cylinder는 다음과 같은 구조를 표현할 때 사용합니다.

- 회전축
- 파이프
- 롤러
- 원통형 링크
- 모터 외형

**Sphere**

Sphere는 구 형태를 정의합니다.

```xml
<geometry>
    <sphere radius="0.05"/>
</geometry>
```

위 코드는 반지름이 5 cm인 구를 의미합니다.

Sphere는 다음과 같은 용도로 사용할 수 있습니다.

- 센서 위치 표시
- 관절 중심 표시
- 간단한 충돌 모델
- 로봇 끝단 위치 표시

**Mesh**

Mesh는 외부에서 제작한 3D 모델 파일을 불러올 때 사용합니다.

```xml
<geometry>
    <mesh filename="package://so_arm101_description/meshes/wrist.stl"/>
</geometry>
```

`filename`에는 Mesh 파일의 위치를 작성합니다.

```
package://패키지_이름/패키지_내부_경로/파일_이름
```

예를 들어 다음 경로는 `so_arm101_description` 패키지 안의 `meshes/wrist.stl` 파일을 의미합니다.

```
package://so_arm101_description/meshes/wrist.stl
```

ROS2는 패키지가 빌드되고 환경에 등록되면 `package://` 경로를 이용해 실제 파일 위치를 찾습니다.

---

#### Mesh 파일 형식

로봇 모델에서 주로 사용하는 Mesh 파일 형식은 다음과 같습니다.

| 형식 | 특징 |
| --- | --- |
| STL | 형상 정보 중심, 로봇 모델에서 많이 사용 |
| DAE | 형상뿐 아니라 색상과 재질 정보를 포함할 수 있음 |
| OBJ | 형상과 재질 파일을 함께 사용할 수 있음 |

SO-ARM101과 같은 실제 로봇의 외형은 기본 도형만으로 정확하게 표현하기 어렵기 때문에 일반적으로 STL과 같은 Mesh 파일을 사용합니다.

```xml
<geometry>
    <mesh filename="package://so_arm101_description/meshes/upper_arm.stl"/>
</geometry>
```

Mesh 파일은 Fusion 360, SolidWorks, Onshape와 같은 3D CAD 프로그램에서 제작하거나 내보낼 수 있습니다.

---

#### Mesh 크기 조정

Mesh의 크기가 URDF에서 사용하는 미터 단위와 맞지 않을 경우 `scale`을 사용하여 크기를 조정할 수 있습니다.

```xml
<geometry>
    <mesh
        filename="package://so_arm101_description/meshes/wrist.stl"
        scale="0.001 0.001 0.001"/>
</geometry>
```

예를 들어 STL 파일이 밀리미터 단위로 제작되었다면 `0.001`을 적용해 미터 단위로 변환할 수 있습니다.

```
1 mm × 0.001 = 0.001 m
```

다만 Mesh 내보내기 설정에 따라 이미 미터 단위가 적용되어 있을 수도 있습니다. 따라서 무조건 `0.001`을 사용하지 말고 RViz2에서 실제 크기를 확인해야 합니다.

---

#### Visual Origin

Visual 안에서도 Origin을 사용할 수 있습니다.

```xml
<visual>
    <origin xyz="0 0 0.05" rpy="0 0 0"/>

    <geometry>
        <box size="0.1 0.1 0.1"/>
    </geometry>
</visual>
```

Visual Origin은 Link Frame을 기준으로 외형의 위치와 방향을 조정합니다.

위 코드는 박스의 중심을 Link Frame보다 Z축 방향으로 0.05 m 이동시킵니다.

```
Link Frame
    ↑ 0.05 m
Visual 중심
```

여기서 중요한 점은 기본 도형의 중심이 일반적으로 Link Frame의 원점에 배치된다는 것입니다.

예를 들어 높이가 0.1 m인 박스를 Link Frame 위에 올려놓으려면 박스 높이의 절반인 0.05 m만큼 이동해야 합니다.

```xml
<origin xyz="0 0 0.05" rpy="0 0 0"/>
```

Visual Origin의 역할은 이전에 살펴본 Joint Origin과 다릅니다.

| 구분 | 역할 |
| --- | --- |
| Joint Origin | Parent Link를 기준으로 Child Link를 배치 |
| Visual Origin | Link Frame을 기준으로 외형을 배치 |

즉, Joint Origin은 Link 사이의 연결 위치를 결정하고, Visual Origin은 Link 안에서 모델이 표시되는 위치를 결정합니다.

---

#### Visual 방향 조정

Mesh나 기본 도형의 방향이 맞지 않을 때는 `rpy`를 사용하여 회전할 수 있습니다.

```xml
<visual>
    <origin xyz="0 0 0" rpy="0 1.5708 0"/>

    <geometry>
        <cylinder radius="0.03" length="0.2"/>
    </geometry>
</visual>
```

위 코드는 기본적으로 Z축 방향을 향하는 Cylinder를 Y축을 기준으로 약 90° 회전시킵니다.

주요 회전값은 다음과 같습니다.

| 각도 | 라디안 |
| --- | --- |
| 45° | 약 0.7854 |
| 90° | 약 1.5708 |
| 180° | 약 3.1416 |

Mesh가 옆으로 눕거나 반대 방향을 향한다면 Visual Origin의 `rpy`를 조정하여 방향을 맞출 수 있습니다.

---

#### Material

Visual에는 Material을 추가하여 색상과 투명도를 지정할 수 있습니다.

```xml
<material name="blue">
    <color rgba="0 0 1 1"/>
</material>
```

색상은 `rgba` 형식으로 작성합니다.

```
R = Red
G = Green
B = Blue
A = Alpha
```

각 값은 일반적으로 `0.0`부터 `1.0` 사이의 값을 사용합니다.

| RGBA 값 | 색상 |
| --- | --- |
| `1 0 0 1` | 빨간색 |
| `0 1 0 1` | 초록색 |
| `0 0 1 1` | 파란색 |
| `1 1 0 1` | 노란색 |
| `1 1 1 1` | 흰색 |
| `0 0 0 1` | 검은색 |

마지막 값인 Alpha는 투명도를 나타냅니다.

```
1.0 = 완전 불투명
0.5 = 반투명
0.0 = 완전 투명
```

예를 들어 반투명한 파란색은 다음과 같이 표현할 수 있습니다.

```xml
<material name="transparent_blue">
    <color rgba="0 0 1 0.5"/>
</material>
```

---

#### Material 재사용

로봇 전체에서 동일한 색상을 여러 번 사용한다면 `<robot>` 태그 바로 아래에 Material을 정의하고 이름으로 재사용할 수 있습니다.

```xml
<robot name="so_arm101">

    <material name="blue">
        <color rgba="0 0 1 1"/>
    </material>

    <link name="base_link">
        <visual>
            <geometry>
                <box size="0.1 0.1 0.05"/>
            </geometry>

            <material name="blue"/>
        </visual>
    </link>

</robot>
```

이 방식은 여러 Link의 색상을 한 번에 관리할 때 유용합니다.

---

#### Visual 전체 예제

다음은 파란색 원통 형태의 `arm_link`를 정의한 예제입니다.

```xml
<link name="arm_link">
    <visual>
        <origin xyz="0 0 0.1" rpy="0 0 0"/>

        <geometry>
            <cylinder radius="0.03" length="0.2"/>
        </geometry>

        <material name="blue">
            <color rgba="0 0 1 1"/>
        </material>
    </visual>
</link>
```

이 Visual은 다음과 같은 외형을 가집니다.

- 형상: 원통
- 반지름: 0.03 m
- 길이: 0.2 m
- 위치: Z축 방향으로 0.1 m 이동
- 색상: 파란색

Cylinder의 길이가 0.2 m이고 중심이 Z축 방향으로 0.1 m 이동했으므로, 원통의 아래쪽 끝이 Link Frame의 원점에 위치하게 됩니다.

---

#### Mesh를 사용한 Link 예제

실제 SO-ARM101과 같은 로봇에서는 다음과 같이 Mesh를 사용할 수 있습니다.

```xml
<link name="wrist_link">
    <visual>
        <origin xyz="0 0 0" rpy="0 0 0"/>

        <geometry>
            <mesh
                filename="package://so_arm101_description/meshes/wrist.stl"
                scale="0.001 0.001 0.001"/>
        </geometry>

        <material name="gray">
            <color rgba="0.6 0.6 0.6 1"/>
        </material>
    </visual>
</link>
```

이 코드는 `wrist.stl` 파일을 불러와 `wrist_link`의 외형으로 사용합니다.

---

#### Visual과 Collision의 차이

Visual은 화면에 보이는 외형만 정의합니다.

```
Visual = 화면에 보이는 모델
```

Visual을 정의했다고 해서 Gazebo가 자동으로 충돌을 계산하는 것은 아닙니다. 충돌 계산에는 별도의 `Collision` 요소가 필요합니다.

```
Visual   = 시각적 표현
Collision = 물리적 충돌 계산
```

실제 로봇과 비슷한 정교한 Mesh는 Visual에 사용하고, 충돌 계산에는 Box나 Cylinder처럼 단순한 형상을 사용하는 경우가 많습니다.

복잡한 Mesh를 Collision에 그대로 사용하면 물리 계산량이 증가해 시뮬레이션 성능이 저하될 수 있기 때문입니다.

---

#### Visual 확인 방법

작성한 Visual은 RViz2에서 확인할 수 있습니다.

확인 과정은 일반적으로 다음과 같습니다.

1. URDF 또는 Xacro 파일 작성
2. `robot_state_publisher` 실행
3. RViz2 실행
4. `RobotModel` Display 추가
5. Fixed Frame 설정
6. Link의 크기, 위치, 방향 확인

외형이 정상적으로 표시되지 않는다면 다음 항목을 확인해야 합니다.

| 문제 | 확인할 항목 |
| --- | --- |
| Link가 보이지 않음 | `<visual>`과 `<geometry>` 존재 여부 |
| Mesh가 보이지 않음 | 패키지 이름과 파일 경로 |
| 모델이 너무 큼 | Mesh 단위와 `scale` |
| 모델이 너무 작음 | Mesh 단위와 `scale` |
| 모델 위치가 어긋남 | Visual Origin의 `xyz` |
| 모델 방향이 틀어짐 | Visual Origin의 `rpy` |
| 색상이 적용되지 않음 | `rgba` 철자와 Material 설정 |
| 일부 Link만 보이지 않음 | Joint 및 TF 연결 관계 |

---

#### 정리

Visual은 Link가 RViz2나 Gazebo에서 어떻게 보일지를 정의하는 요소입니다.

- `<visual>`: 외형 정보의 시작
- `<geometry>`: 외형의 형상
- `<box>`: 직육면체
- `<cylinder>`: 원통
- `<sphere>`: 구
- `<mesh>`: 외부 3D 모델
- `<origin>`: 외형의 위치와 방향
- `<material>`: 색상과 투명도
- `scale`: Mesh의 크기 조정

가장 중요한 관계는 다음과 같습니다.

> **Link는 몸체, Joint는 연결과 움직임, Visual은 화면에 보이는 외형을 정의합니다.**
> 

Visual은 로봇의 외형을 담당하지만 물리적인 충돌은 처리하지 않습니다. 다음 장에서는 Gazebo와 MoveIt에서 충돌 판정에 사용되는 `Collision`에 대해 알아보겠습니다.
