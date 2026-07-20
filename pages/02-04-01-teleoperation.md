# Teleoperation

Teleoperation은 사용자가 한쪽 Robot을 직접 조작하면 다른 Robot이 그 움직임을 따라가도록 만드는 원격조작 방식입니다.

SO-ARM101에서는 사용자가 Leader Arm을 손으로 움직이고, Follower Arm이 Leader Arm의 자세를 따라 움직이도록 구성할 수 있습니다.

```python
사용자가 Leader Arm 조작
→ Leader Motor 위치 읽기
→ Follower Motor 위치로 변환
→ Follower Arm 이동
```

Leader Arm과 Follower Arm은 각각 6개의 Motor로 구성됩니다.

| Motor | Joint | 역할 |
| --- | --- | --- |
| `m1` | `shoulder_pan` | Base 좌우 회전 |
| `m2` | `shoulder_lift` | Shoulder 상하 이동 |
| `m3` | `elbow_flex` | Elbow 굽힘과 펼침 |
| `m4` | `wrist_flex` | Wrist 상하 이동 |
| `m5` | `wrist_roll` | Wrist 회전 |
| `m6` | `gripper` | Gripper Open과 Close |

두 Arm은 Motor의 기어비와 실제 위치값 범위가 다를 수 있습니다. 따라서 Leader Arm의 위치값을 그대로 Follower Arm에 전달하지 않고, 각 Arm의 최솟값과 최댓값을 기준으로 위치를 변환하여 전달합니다.

이번 실습은 다음 순서로 진행합니다.

```
Leader Arm 위치 범위 확인
→ Follower Arm 위치 범위 확인
→ 위치 범위를 CSV 파일로 저장
→ Leader 위치 읽기
→ Follower 목표 위치로 변환
→ Follower Arm 이동
```

---
