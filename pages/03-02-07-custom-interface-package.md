# 커스텀 인터페이스 패키지 만들기

앞 절까지는 `Twist`와 `Pose`처럼 ROS2에서 제공하는 메시지 타입을 사용했습니다.

`turtlesim`이 이미 정해진 메시지 타입으로 Topic을 발행하고 구독하므로 기존 타입에 맞춰 데이터를 사용하면 됐습니다.

하지만 프로젝트에서 필요한 데이터 구조가 기존 인터페이스에 없다면 직접 정의해야 합니다. 이를 커스텀 인터페이스(Custom Interface)라고 합니다.

ROS2 인터페이스 파일은 세 종류로 구분됩니다.

| 확장자 | 사용 목적 | 데이터 구성 |
| --- | --- | --- |
| `.msg` | Topic | 메시지 |
| `.srv` | Service | Request, Response |
| `.action` | Action | Goal, Result, Feedback |

Service와 Action 서버를 직접 만든다고 해서 반드시 커스텀 인터페이스가 필요한 것은 아닙니다. 기존 인터페이스로 필요한 데이터를 표현할 수 있다면 그대로 사용할 수 있습니다.

이번 절에서는 인터페이스의 정의와 빌드 과정을 익히기 위해 `.msg`, `.srv`, `.action` 파일을 직접 만들겠습니다.

---

#### 인터페이스 패키지를 분리하는 이유

커스텀 인터페이스는 일반적으로 노드 코드와 분리된 별도 패키지에서 관리합니다.

인터페이스 패키지를 분리하면 여러 서버와 클라이언트 패키지가 같은 데이터 타입을 공유할 수 있습니다. 노드 패키지는 인터페이스 패키지만 의존성으로 추가하면 됩니다.

또한 인터페이스 생성에 사용하는 `rosidl_generate_interfaces()`는 CMake 기반으로 동작합니다. 따라서 지금까지 사용한 `ament_python` 패키지가 아닌 `ament_cmake` 패키지로 생성합니다.

인터페이스 패키지는 일반적으로 다음과 같이 `_interfaces`를 붙여 이름을 정합니다.

```
first_interfaces
```

#### 인터페이스 패키지 생성

워크스페이스의 `src` 폴더에서 패키지를 생성합니다.

```wasm
cd ~/project/ros2_ws/src
ros2 pkg create first_interfaces--build-type ament_cmake
```

생성된 기본 구조는 다음과 같습니다.

```
first_interfaces/
├── CMakeLists.txt
├── package.xml
├── include/
└── src/
```

`include`와 `src`는 C++ 코드를 작성할 때 사용하는 폴더입니다. 이번 실습에서는 인터페이스 파일만 정의하므로 사용하지 않습니다.

---

#### 인터페이스 폴더 생성

패키지 안에 인터페이스 종류별 폴더를 만듭니다.

```bash
cd ~/project/ros2_ws/src/first_interfacesmkdir-p msg srv action
```

구조는 다음과 같습니다.

```
first_interfaces/
├── msg/
├── srv/
├── action/
├── CMakeLists.txt
└── package.xml
```

인터페이스 파일은 일반적으로 다음 규칙에 따라 작성합니다.

- 파일 이름은 대문자로 시작하는 PascalCase를 사용합니다.
- 필드 이름은 소문자와 언더스코어로 구성된 snake_case를 사용합니다.
- Topic 인터페이스는 `msg` 폴더에 저장합니다.
- Service 인터페이스는 `srv` 폴더에 저장합니다.
- Action 인터페이스는 `action` 폴더에 저장합니다.

---

#### Topic 메시지 정의

`msg` 폴더에 `TopicTest.msg` 파일을 만들고 다음 내용을 작성합니다.

```python
string topic_string
float32 topic_float
```

`.msg` 파일은 한 줄에 하나씩 자료형과 필드 이름을 작성합니다.

```
자료형 필드_이름
```

`TopicTest` 메시지는 문자열과 실수 값을 하나씩 전달할 수 있습니다.

---

#### Service 인터페이스 정의

`srv` 폴더에 `ServiceTest.srv` 파일을 만들고 다음 내용을 작성합니다.

```bash
string request_string
float32 request_float
---
bool response_bool
string response_string
```

Service 인터페이스는 `---`를 기준으로 두 부분으로 나뉩니다.

```
Request
---
Response
```

클라이언트는 `request_string`과 `request_float`을 서버에 전달합니다. 서버는 처리 결과로 `response_bool`과 `response_string`을 반환합니다.

---

#### Action 인터페이스 정의

`action` 폴더에 `ActionTest.action` 파일을 만들고 다음 내용을 작성합니다.

```bash
string goal_string
float32 goal_float
---
int32 result_int
---
bool feedback_bool
string feedback_message
```

Action 인터페이스는 두 개의 `---`를 기준으로 세 부분으로 나뉩니다.

```
Goal
---
Result
---
Feedback
```

각 부분의 역할은 다음과 같습니다.

| 구분 | 역할 |
| --- | --- |
| Goal | 클라이언트가 서버에 전달하는 목표 |
| Result | 작업 완료 후 서버가 반환하는 최종 결과 |
| Feedback | 작업 중 서버가 전달하는 진행 상태 |

#### CMakeLists.txt 수정

인터페이스 파일을 빌드하려면 `CMakeLists.txt`에 `rosidl_default_generators`와 인터페이스 파일을 등록해야 합니다.

#### 전체 소스 코드

> GitHub Link: [https://github.com/applesnack23/ros2-lerobot-code/blob/main/chapter3/CMakeLists.txt](https://github.com/applesnack23/ros2-lerobot-code/blob/main/chapter3/CMakeLists.txt)
> 

```makefile
cmake_minimum_required(VERSION 3.20)
project(first_interfaces)

if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic)
endif()

find_package(ament_cmake REQUIRED)
find_package(rosidl_default_generators REQUIRED)

rosidl_generate_interfaces(${PROJECT_NAME}
  "msg/TopicTest.msg"
  "srv/ServiceTest.srv"
  "action/ActionTest.action"
)

if(BUILD_TESTING)
  find_package(ament_lint_auto REQUIRED)

  set(ament_cmake_copyright_FOUND TRUE)
  set(ament_cmake_cpplint_FOUND TRUE)

  ament_lint_auto_find_test_dependencies()
endif()

ament_package()
```

핵심 설정은 다음 부분입니다.

```makefile
find_package(rosidl_default_generators REQUIRED)
```

`rosidl_default_generators`는 인터페이스 파일을 Python과 C++에서 사용할 수 있는 코드로 변환합니다.

```makefile
rosidl_generate_interfaces(${PROJECT_NAME}
  "msg/TopicTest.msg"
  "srv/ServiceTest.srv"
  "action/ActionTest.action"
)
```

`rosidl_generate_interfaces()`에는 빌드할 인터페이스 파일을 등록합니다. 새로운 인터페이스를 추가하면 이 목록에도 파일 경로를 추가해야 합니다.

#### package.xml 수정

`package.xml`에 인터페이스 생성과 실행에 필요한 의존성을 추가합니다.

#### 전체 소스 코드

> GitHub Link: [https://github.com/applesnack23/ros2-lerobot-code/blob/main/chapter3/package.xml](https://github.com/applesnack23/ros2-lerobot-code/blob/main/chapter3/package.xml)
> 

```xml
<?xml version="1.0"?>
<?xml-model
  href="http://download.ros.org/schema/package_format3.xsd"
  schematypens="http://www.w3.org/2001/XMLSchema"?>

<package format="3">
  <name>first_interfaces</name>
  <version>0.0.0</version>

  <description>Custom interfaces for ROS 2 practice</description>

  <maintainer email="twiniex@todo.todo">
    twiniex
  </maintainer>

  <license>Apache-2.0</license>

  <buildtool_depend>ament_cmake</buildtool_depend>
  <buildtool_depend>rosidl_default_generators</buildtool_depend>

  <exec_depend>rosidl_default_runtime</exec_depend>

  <test_depend>ament_lint_auto</test_depend>
  <test_depend>ament_lint_common</test_depend>

  <member_of_group>
    rosidl_interface_packages
  </member_of_group>

  <export>
    <build_type>ament_cmake</build_type>
  </export>
</package>
```

주요 의존성의 역할은 다음과 같습니다.

| 설정 | 역할 |
| --- | --- |
| `ament_cmake` | CMake 기반 패키지 빌드 |
| `rosidl_default_generators` | 인터페이스 코드 생성 |
| `rosidl_default_runtime` | 생성된 인터페이스를 실행할 때 사용 |
| `rosidl_interface_packages` | 인터페이스 패키지 그룹 등록 |

#### 인터페이스 패키지 빌드

워크스페이스로 이동하여 `first_interfaces` 패키지를 빌드합니다.

```bash
cd ~/project/ros2_ws
colcon build--packages-select first_interfaces
```

빌드가 완료되면 새로운 인터페이스를 현재 터미널에서 사용할 수 있도록 환경을 갱신합니다.

```bash
pkg_enable
```

또는 다음 명령을 직접 실행합니다.

```bash
source ~/project/ros2_ws/install/setup.bash
```

#### 인터페이스 등록 확인

다음 명령으로 `first_interfaces` 패키지의 인터페이스가 등록되었는지 확인합니다.

```bash
ros2 interface list |grep first_interfaces
```

정상적으로 빌드되었다면 다음 세 가지 인터페이스가 출력됩니다.

```bash
first_interfaces/msg/TopicTest
first_interfaces/srv/ServiceTest
first_interfaces/action/ActionTest
```

각 인터페이스의 내부 구조도 확인할 수 있습니다.

```bash
ros2 interface show first_interfaces/msg/TopicTest
```

```bash
ros2 interface show first_interfaces/srv/ServiceTest
```

```bash
ros2 interface show first_interfaces/action/ActionTest
```

작성한 필드와 구분선이 그대로 출력되면 인터페이스가 정상적으로 생성된 것입니다.

#### Python에서 사용하는 방법

빌드된 커스텀 인터페이스는 Python 노드에서 다음과 같이 불러올 수 있습니다.

```python
from first_interfaces.msg import TopicTest
from first_interfaces.srv import ServiceTest
from first_interfaces.action import ActionTest
```

노드에서 실제로 사용하려면 노드가 들어 있는 패키지의 `package.xml`에도 다음 의존성을 추가해야 합니다.

```xml
<depend>first_interfaces</depend>
```

#### 마무리

이번 절에서는 `ament_cmake` 기반의 커스텀 인터페이스 패키지를 만들고 다음 파일을 정의했습니다.

- `TopicTest.msg`
- `ServiceTest.srv`
- `ActionTest.action`

커스텀 인터페이스 제작의 핵심 과정은 다음과 같습니다.

1. `ament_cmake` 패키지를 생성합니다.
2. `msg`, `srv`, `action` 폴더를 만듭니다.
3. 인터페이스 파일을 작성합니다.
4. `CMakeLists.txt`에 파일을 등록합니다.
5. `package.xml`에 의존성을 추가합니다.
6. 패키지를 빌드하고 인터페이스 구조를 확인합니다.

다음 절에서는 이번에 만든 Service 인터페이스를 이용하여 Service Server와 Service Client 노드를 작성해보겠습니다.