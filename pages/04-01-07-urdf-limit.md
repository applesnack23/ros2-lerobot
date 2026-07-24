# Limit 이해

#### Limit 이해

앞 장에서는 Joint가 움직이는 방향을 정의하는 `Axis`에 대해 알아보았습니다.

하지만 움직이는 축만 정의해서는 로봇을 안전하게 제어할 수 없습니다. 실제 로봇의 관절은 기구적인 구조와 모터 성능에 따라 움직일 수 있는 각도, 힘, 속도가 제한되어 있기 때문입니다.

이러한 관절의 동작 범위와 성능 한계를 정의하는 요소가 `Limit`입니다.

> **Axis는 움직이는 방향을 정의하고, Limit는 움직일 수 있는 범위와 최대 성능을 정의합니다.**
> 

Limit 정보는 다음과 같은 작업에 활용됩니다.

- 관절의 동작 범위 제한
- Gazebo 물리 시뮬레이션
- MoveIt 경로 계획
- 잘못된 관절 명령 방지
- 로봇의 기구적 충돌 방지

다만 URDF의 Limit는 로봇의 제한값을 설명하는 정보입니다. 실제 로봇을 확실하게 보호하려면 모터 드라이버와 제어 프로그램에서도 별도의 제한 및 안전 기능을 적용해야 합니다.

---

#### Limit 기본 구조

Limit의 기본 구조는 다음과 같습니다.

```xml
<limit
    lower="-1.57"
    upper="1.57"
    effort="10.0"
    velocity="1.0"/>
```

Limit에는 네 가지 주요 속성이 있습니다.

| 속성 | 의미 | 회전 관절 단위 | 직선 관절 단위 |
| --- | --- | --- | --- |
| `lower` | 최소 위치 | rad | m |
| `upper` | 최대 위치 | rad | m |
| `effort` | 최대 힘 또는 토크 | N·m | N |
| `velocity` | 최대 속도 | rad/s | m/s |

정리하면 다음과 같습니다.

```
Limit = 동작 범위 + 최대 힘 + 최대 속도
```

---

#### Lower와 Upper

`lower`와 `upper`는 Joint가 움직일 수 있는 최소 위치와 최대 위치를 정의합니다.

```xml
<limit lower="-1.57" upper="1.57" effort="10.0" velocity="1.0"/>
```

회전 관절에서는 라디안 단위를 사용하므로 위 설정은 약 `-90°`부터 `+90°`까지 회전할 수 있다는 의미입니다.

주요 각도를 라디안으로 나타내면 다음과 같습니다.

| 각도 | 라디안 |
| --- | --- |
| 30° | 약 0.52 rad |
| 45° | 약 0.79 rad |
| 90° | 약 1.57 rad |
| 180° | 약 3.14 rad |
| 360° | 약 6.28 rad |

한 방향으로만 움직이는 관절은 다음과 같이 작성할 수 있습니다.

```xml
<limit lower="0.0" upper="3.14" effort="10.0" velocity="1.0"/>
```

이는 관절이 `0°`부터 약 `180°`까지 움직일 수 있다는 의미입니다.

---

#### Limit가 중요한 이유

관절의 실제 동작 범위보다 지나치게 큰 Limit를 설정하면 다음과 같은 문제가 발생할 수 있습니다.

- 로봇 부품끼리 충돌
- 케이블 꼬임 또는 단선
- 모터와 감속기 손상
- Gazebo에서 비정상적인 동작 발생
- MoveIt이 실행할 수 없는 경로 생성
- 실제 로봇과 시뮬레이션의 동작 불일치

따라서 URDF의 Limit는 실제 로봇의 기구적 범위와 모터 사양을 기준으로 작성해야 합니다.

---

#### Effort

`effort`는 Joint가 사용할 수 있는 최대 힘 또는 토크를 의미합니다.

```xml
<limit lower="-1.57" upper="1.57" effort="10.0" velocity="1.0"/>
```

Joint 종류에 따라 단위가 달라집니다.

- `revolute`, `continuous`: N·m
- `prismatic`: N

예를 들어 회전 관절에서 `effort="10.0"`은 최대 관절 토크를 `10 N·m`로 정의한 것입니다.

이 값은 Gazebo와 `ros2_control` 같은 제어 환경에서 관절의 힘을 제한할 때 활용될 수 있습니다. 실제 모터가 정확히 이 힘을 내도록 보장하는 값은 아니므로 하드웨어 사양과 제어기 설정도 함께 확인해야 합니다.

---

#### Velocity

`velocity`는 Joint의 최대 속도를 의미합니다.

```xml
<limit lower="-1.57" upper="1.57" effort="10.0" velocity="1.5"/>
```

Joint 종류에 따른 단위는 다음과 같습니다.

- `revolute`, `continuous`: rad/s
- `prismatic`: m/s

예를 들어 다음 값은 초당 약 90°의 속도를 나타냅니다.

```
1.57 rad/s ≈ 90°/s
```

Velocity를 실제 모터의 속도보다 지나치게 크게 설정하면 시뮬레이션과 실제 로봇의 움직임에 차이가 발생할 수 있습니다.

---

#### SO-ARM101 Joint Limit 예제

SO-ARM101의 `shoulder_lift` Joint를 단순화하면 다음과 같이 작성할 수 있습니다.

```xml
<joint name="shoulder_lift" type="revolute">
    <parent link="shoulder_link"/>
    <child link="upper_arm_link"/>

    <origin xyz="0 0 0.1" rpy="0 0 0"/>
    <axis xyz="0 1 0"/>

    <limit
        lower="-1.57"
        upper="1.57"
        effort="8.0"
        velocity="1.5"/>
</joint>
```

각 설정의 의미는 다음과 같습니다.

- `axis="0 1 0"`: Y축을 기준으로 회전
- `lower="-1.57"`: 최소 각도 약 -90°
- `upper="1.57"`: 최대 각도 약 +90°
- `effort="8.0"`: 최대 관절 토크 8 N·m
- `velocity="1.5"`: 최대 관절 속도 1.5 rad/s

이 값들은 구조를 설명하기 위한 예시입니다. 실제 URDF를 작성할 때는 SO-ARM101의 기구적 범위와 모터 사양을 확인해 정확한 값을 사용해야 합니다.

---

#### Joint 종류에 따른 Limit

Joint 종류에 따라 Limit의 사용 방법이 달라집니다.

| Joint 종류 | `lower`, `upper` | `effort`, `velocity` |
| --- | --- | --- |
| `revolute` | 사용 | 사용 |
| `continuous` | 사용하지 않음 | 사용 |
| `prismatic` | 사용 | 사용 |
| `fixed` | 사용하지 않음 | 사용하지 않음 |

**Revolute Joint**

정해진 범위 안에서 회전하므로 위치 범위와 최대 성능을 모두 정의합니다.

```xml
<joint name="elbow_joint" type="revolute">
    <limit lower="-1.57" upper="1.57" effort="8.0" velocity="1.5"/>
</joint>
```

**Continuous Joint**

제한 없이 계속 회전할 수 있으므로 `lower`와 `upper`를 작성하지 않습니다. 다만 최대 힘과 속도는 설정할 수 있습니다.

```xml
<joint name="wheel_joint" type="continuous">
    <axis xyz="0 1 0"/>
    <limit effort="5.0" velocity="3.0"/>
</joint>
```

**Prismatic Joint**

직선으로 이동하므로 `lower`와 `upper`의 단위가 미터입니다.

```xml
<joint name="slider_joint" type="prismatic">
    <axis xyz="0 0 1"/>
    <limit lower="0.0" upper="0.2" effort="100.0" velocity="0.1"/>
</joint>
```

위 Joint는 Z축 방향으로 `0 m`부터 `0.2 m`까지 이동할 수 있습니다.

**Fixed Joint**

움직이지 않는 고정 관절이므로 Limit를 사용하지 않습니다.

```xml
<joint name="camera_mount" type="fixed">
    <parent link="base_link"/>
    <child link="camera_link"/>
</joint>
```

---

#### Limit와 MoveIt

MoveIt은 URDF에 정의된 Joint Limit를 참고하여 실행 가능한 경로를 생성합니다.

예를 들어 목표 자세가 Joint의 최대 각도를 벗어나면 MoveIt은 해당 자세에 도달할 수 없다고 판단할 수 있습니다.

Limit가 잘못 설정되면 다음과 같은 문제가 발생합니다.

- 도달할 수 있는 자세인데 경로 계획 실패
- 실제로 도달할 수 없는 자세까지 계획
- 지나치게 빠른 관절 속도 생성
- 실제 로봇과 계획된 동작의 차이 발생

MoveIt 설정에는 URDF 외에도 별도의 Joint Limit 설정 파일이 사용될 수 있으므로 두 곳의 값이 일치하는지 확인해야 합니다.

---

#### Limit 확인 시 주의사항

Limit를 작성하거나 검토할 때는 다음 항목을 확인해야 합니다.

- 회전 관절의 위치 단위가 라디안인지 확인
- 직선 관절의 위치 단위가 미터인지 확인
- `lower`가 `upper`보다 작은지 확인
- 실제 기구적 동작 범위를 넘지 않는지 확인
- 모터가 지원하는 최대 속도와 토크를 반영했는지 확인
- MoveIt과 `ros2_control`의 제한값이 일치하는지 확인

---

#### 정리

Limit는 Joint의 안전한 동작 범위와 성능 한계를 정의하는 요소입니다.

- `lower`: 최소 위치
- `upper`: 최대 위치
- `effort`: 최대 힘 또는 토크
- `velocity`: 최대 속도
- `revolute`: 각도 범위가 있는 회전 관절
- `continuous`: 위치 범위 없이 계속 회전하는 관절
- `prismatic`: 정해진 거리 안에서 직선으로 이동하는 관절
- `fixed`: 움직이지 않으므로 Limit를 사용하지 않는 관절

가장 중요한 관계는 다음과 같습니다.

> **Origin은 Joint의 위치, Axis는 움직이는 방향, Limit는 움직일 수 있는 범위와 성능을 정의합니다.**
> 

다음 장에서는 Link가 RViz2와 시뮬레이터 화면에 어떻게 표시되는지를 결정하는 `Visual`에 대해 알아보겠습니다.
