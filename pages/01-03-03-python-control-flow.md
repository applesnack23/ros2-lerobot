# Python 제어

파이썬 예제들은 텍스트 에디터를 통해 파일을 만듭니다.

```python
gnome-text-editor test.py
```

실행을 위해 파일을 저장하고 Terminal 에서 python3 명령을 통해 실행 결과를 확인합니다.

```python
python3 test.py
```

#### 제어문

프로그램은 단순히 위에서 아래로 한 줄씩 실행되는 것만으로는 부족합니다.

특정 조건에 따른 다른 동작을 수행하거나, 같은 작업을 여러 번 반복해야 하는 경우가 많습니다.

이러한 기능을 제어문(Contol Statement)이라고 합니다.

---

**if**

`if` 문은 조건에 따라 다른 독장을 수행할 때 사용합니다.

```python
speed = 50

if speed > 0:
	print("Robot Move")
```

위 코드는 `speed` 값이 0보다 클 경우에만 `“Robot Move”` 를 출력합니다.

---

**for**

`for` 문은 같은 작업을 여러 번 반복할 때 사용합니다.

```python
for i in range(5):
	print(i)
```

실행 결과:

```python
0
1
2
3
4
```

ROS2에서는 여러 개의 센서 값이나 관절 데이터를 반복적으로 처리할 때 자주 사용됩니다.

---

**while**

`while` 문은 특정 조건이 만족하는 동안 계속 반복 실행됩니다.

```python
count = 0

while count < 5:
	print(count)
	count += 1
```

실형 결과:

```python
0
1
2
3
4
```

ROS2에서는 프로그램이 종료될 때까지 계속 실행되어야 하는 경우가 많기 때문에 `while` 문을 자주 사용합니다.

예를 들어 ROS2에서는 다음과 같이 횟수가 정해져 있지 않은 형태의 반복문을 자주 보게 됩니다.

```python
while rclpy.ok():
	pass
```

---
