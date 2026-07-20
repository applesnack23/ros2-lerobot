# 모터 설정

개발환경 구축이 완료되었다면 각 Motor에 고유한 ID를 설정해야 합니다.

SO-ARM101은 하나의 Servo Bus에 6개의 Motor가 연결됩니다. Motor Controller가 각각의 Motor를 구분하려면 Motor마다 서로 다른 ID가 필요합니다.

새로운 Motor는 일반적으로 기본 ID가 `1`로 설정되어 있습니다. 따라서 여러 Motor를 한 번에 연결하기 전에 Motor를 하나씩 연결하여 ID와 Baudrate를 설정해야 합니다.

Motor ID 설정은 처음 조립할 때 한번 수행하며, 설정값은 Motor 내부에 저장됩니다.

---

#### Follower Arm 모터 설정

Follower Arm의 Motor Controller에 USB Cable과 전원을 연결합니다.

다음 명령을 실행합니다.

```bash
lerobot-setup-motors \
  --robot.type=so101_follower \
  --robot.port=/dev/ttyACM0
```

/dev/ttyACM0은 예시이므로 앞에서 확인한 Follower Arm의 실제 Port를 입력해야 합니다.

Program이 실행되면 Terminal에 연결해야 할 Motor가 순서대로 표시됩니다.

```bash
Connect the gripper motor only and press Enter.
```

안내된 Motor 하나만 Controller에 연결하고 Enter를 누릅니다. Program이 Motor를 검색하고 해당 Joint에 필요한 ID와 Baudrate를 설정합니다.

모터에는 두개의 통신 연결 커넥터가 있는데 어느것에 연결해도 좋습니다.

설정이 완료되면 다음 Motor를 연결하라는 안내가 표시됩니다.

```bash
Connect the wrist_roll motor only and press Enter.
```

이 과정을 반복하여 6개의 Motor를 모두 설정합니다.

최종 Motor ID는 다음과 같이 구성됩니다.

| Joint | Assigned ID |
| --- | --- |
| shoulder_pan | 1 |
| shoulder_lift | 2 |
| elbow_flex | 3 |
| wrist_flex | 4 |
| wrist_roll | 5 |
| gripper | 6 |

Motor 설정 중에는 안내된 Motor 하나만 Controller에 연결해야 합니다. 여러 Motor를 동시에 연결하면 기본 ID가 중복되어 올바르게 설정되지 않을 수 있습니다.

---

#### Leader Arm 모터 설정

Leader Arm도 동일한 방법으로 Motor ID와 Baudrate를 설정합니다.

```bash
lerobot-setup-motors \
    --teleop.type=so101_leader \
    --teleop.port=/dev/ttyACM1
```

`/dev/ttyACM1`은 예시이므로 앞에서 확인한 Leader Arm의 실제 Port를 입력해야 합니다.

Terminal의 안내에 따라 Motor를 하나씩 연결하고 Enter를 누릅니다.

Leader Arm의 Motor ID도 다음과 같이 구성됩니다.

| Joint | Assigned ID |
| --- | --- |
| shoulder_pan | 1 |
| shoulder_lift | 2 |
| elbow_flex | 3 |
| wrist_flex | 4 |
| wrist_roll | 5 |
| gripper | 6 |

---
