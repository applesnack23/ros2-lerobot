# Python 함수

함수(Function)는 특정 작업을 수행하는 코드를 하나의 이름으로 묶어 놓은 것입니다.

Python에서는 `def`를 사용하여 함수를 정의합니다.

```python
def print_robot_name():
    print("SO-ARM101")
```

정의한 함수를 사용하려면 함수 이름을 호출합니다.

```python
print_robot_name()
```

실행 결과:

```
SO-ARM101
```

#### Parameter와 return

Parameter를 사용하면 함수에 필요한 값을 전달할 수 있습니다.

```python
def tick_to_radian(tick):
    radian = (tick - 2048) * 0.00153398

    return radian
```

`return`은 함수에서 계산한 결과를 함수 외부로 반환합니다.

```python
joint_position = tick_to_radian(2148)

print(joint_position)
```

실행 결과:

```
0.153398
```

함수를 사용하면 반복되는 계산이나 동작을 한 번만 작성하고 필요할 때마다 다시 사용할 수 있습니다.

Class 내부에 정의된 함수는 Method라고 합니다. 이후 ROS2 Node를 작성할 때 다음과 같은 Method를 사용하게 됩니다.

```python
self.create_publisher()
self.create_subscription()
```

이 단계에서는 다음 내용만 이해하면 충분합니다.

- `def`로 함수를 정의합니다.
- Parameter로 함수에 값을 전달합니다.
- `return`으로 실행 결과를 반환합니다.
- 함수는 반복되는 코드를 줄이고 프로그램을 기능별로 나누는 데 사용합니다.
