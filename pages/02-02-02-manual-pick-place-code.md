# 파이썬 코드 작성

#### Library 가져오기

```python
from lerobot.motors.feetech.feetech import FeetechMotorsBus
from lerobot.motors.motors_bus import Motor
```

`FeetechMotorBus`는 PC와 Feetech Servo Motor 사이의 통신을 관리하는 Class입니다.

이 Class를 이용하여 다음과 같은 작업을 수행할 수 있습니다.

- Motor 연결
- Torque 활성화
- 현재 위치 읽기
- 목표 위치 쓰기

`Motor`는 개별 Motor의 설정 정보를 정의하는 Class입니다.

주요 설정 정보는 다음과 같습니다.

- Motor ID
- Motor Model
- 동작 Mode

---

#### Motor 정의

먼저 Motor 한 개를 정의합니다

```python
motors = {
    "m1": Motor(
        id=4,
        model="sts3215",
        norm_mode="position"
    )
}
```

위 코드에서 `ID 4`인 STS3215 Servo Motor를 `wrist_flex`라는 이름으로 정의했습니다.

`norm_mode=”position”`은 Motor의 값을 위치 기준으로 정규화할 때 사용하는 설정입니다.

전체 Motor를 한 번에 제어하기 전에 Motor 하나의 연결과 동작을 먼저 확인합니다. Motor 하나를 정상적으로 제어할 수 있다면 같은 구조로 나머지 Motor도 추가할 수 있습니다.

---

#### Motor Bus 생성

```python
bus = FeetechMotorsBus(
	port="/dev/ttyACM0",
	motors=motors
)
```

Bus는 PC와 여러 Servo Motor가 데이터를 주고받는 통신 경로입니다.ㄴ

`port`에는 앞에서 확인한 Motor Controller의 USB Port를 입력합니다.

```bash
/dev/ttyACM0
```

USB 연결 환경에 따라 `/dev/ttyACM1`과 같이 다른 Port가 사용될 수 있으므로 자신의 환경에 맞게 수정해야 합니다.

---

#### Motor 연결

```python
bus.connect()
```

`connect()` 를 실행하여 Motor Controller와 연결합니다.

---

#### Servo On (=Torque On)

```python
bus.enable_torque()
```

Servo Motor를 제어하려면 Torque를 활성화해야 합니다.

Torque가 활성화되면 Motor는 목표 위치를 유지하기 위해 힘을 사용합니다. 실행 전에 Robot 주변에 장애물이 없는지 확인하고, 손이나 물체가 Joint 사이에 끼이지 않도록 주의해야 합니다.

---

#### 현재 위치값 읽기

```python
pos = bus.read("Present_Position", "m1", normalize=False)
print("pos:", pos)
```

`Present_Position`은 Motor의 현재 위치를 나타냅니다.

---

#### 목표 위치로 이동

```python
pos = bus.read("Present_Position", "m1", normalize=False)
print("pos:", pos)

target_pos = pos - 100

bus.write("Goal_Position", "m1", target_pos, normalize=False)
print("목표 위치:", target_pos)
```

`Goal_Position`은 Motor가 이동할 목표 위치를 나타냅니다.

위 코드에서는 현재 위치에서 Raw Position을 `100`만큼 감소시킨 값을 목표 위치로 사용했습니다.

Motor의 설치 방향에 따라 실제 회전 방향이 달라진 수 있으므로 처음 테스트할 때는 작은 변화량을 사용해야 합니다. Joint가 예상과 다른 방향으로 움직이거나 기구적으로 간섭하면 즉시 전원을 차단합니다.

---

#### 종료

```python
import time

time.sleep(1)

bus.disable_torque()
bus.disconnect()
```

`time.sleep(1)`은 Motor가 이동할 수 있도록 1초 동안 기다리는 코드입니다.

실습이 끝나면 `disable_torque()`로 Torque를 비활성화하고 `disconnect()`로 Motor Controller와의 연결을 종료합니다.

---

#### 전체 소스 코드

> GitHub Link: [https://github.com/applesnack23/ros2-lerobot-code/blob/main/chapter2/motor_test.py](https://github.com/applesnack23/ros2-lerobot-code/blob/main/chapter2/motor_test.py)
> 

```python
from lerobot.motors.feetech.feetech import FeetechMotorsBus
from lerobot.motors.motors_bus import Motor

# Motor 객체로 정의
motors = {
	"m1": Motor(
		id=4,
		model="sts3215",
		norm_mode="position"
	)
}

# 버스 생성
bus = FeetechMotorsBus(
	port="/dev/ttyACM0",
	motors=motors
)

# 연결
bus.connect()
bus.enable_torque()

# 현재 위치 읽기 (raw)
pos = bus.read("Present_Position", "m1", normalize=False)
print("pos:", pos)

# 조금 이동
bus.write("Goal_Position", "m1", pos - 100, normalize=False)
print("pos:", pos - 100)

# 종료
import time
time.sleep(1)

bus.disable_torque()
bus.disconnect()
```

#### 코드 실행

VS Code의 Terminal 에서 가상환경이 활성화되어 있는지 확인합니다.

```bash
(venv) username@lt:~/project/rosws/control$
```

가상환경이 활성화되어 있지 않다면 다음 명령을 실행합니다.

```bash
source ~/project/rosws/lerobot/venv/bin/activate
```

또는 .bashrc에 alias 등록이 되어 있다면

```bash
venv
```

를 입력해 가상환경에 진입할 수도 있습니다.

작성한 Python 파일을 실행합니다.

```bash
python motor_test.py
```

Motor의 현재 위치가 출력되고 `wrist_flex` Joint가 현재 위치에서 조금 이동하면 정상적으로 실행된 것입니다.
