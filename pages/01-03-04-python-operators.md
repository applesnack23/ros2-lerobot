# Python 연산자

연산자(Operator)는 값을 계산하거나 비교할 때 사용하는 기호입니다.

#### 산술 연산자

| 연산자 | 의미 |
| --- | --- |
| `+` | 덧셈 |
| `-` | 뺄셈 |
| `*` | 곱셈 |
| `/` | 나눗셈 |
| `%` | 나머지 |
| `**` | 거듭제곱 |

Robot 제어에서는 목표 위치와 현재 위치의 차이를 계산할 때 연산자를 사용합니다.

```python
target_position = 1.2
current_position = 0.8

position_error = target_position - current_position

print(position_error)
```

실행 결과:

```
0.4
```

#### 비교 연산자

비교 연산자는 두 값을 비교하며 결과는 `True` 또는 `False`입니다.

| 연산자 | 의미 |
| --- | --- |
| `==` | 같다 |
| `!=` | 다르다 |
| `>` | 크다 |
| `<` | 작다 |
| `>=` | 크거나 같다 |
| `<=` | 작거나 같다 |

```python
temperature = 65

print(temperature >= 60)
```

실행 결과:

```
True
```

`=`는 값을 저장하고 `==`는 두 값이 같은지 비교하므로 주의해야 합니다.

#### 논리 연산자

논리 연산자는 여러 조건을 함께 확인할 때 사용합니다.

| 연산자 | 의미 |
| --- | --- |
| `and` | 모든 조건이 참 |
| `or` | 하나 이상의 조건이 참 |
| `not` | 조건을 반대로 변경 |

```python
power_on = True
emergency_stop = False

if power_on and not emergency_stop:
    print("Robot을 이동할 수 있습니다.")
```

연산자는 관절 위치를 계산하거나 Robot의 상태와 안전 조건을 판단하는 데 사용됩니다.
