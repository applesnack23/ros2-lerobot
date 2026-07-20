# 전체 학습 흐름

Robot 개발에는 다양한 기술이 사용됩니다.

Linux, Python, ROS2, URDF, TF, Gazebo, MoveIt, Digital Twin 등 수 많은 기술들이 함께 사용되며, 각각은 서로 밀접하게 연결되어 있습니다.

문제는 이러한 기술들이 개별적으로 존재하는 것이 아니라는 점입니다.

Robot의 구조를 이해하지 못하면 Robot을 제어할 수 없고, Robot을 제어하지 못하면 Simulation이나 Motion Planning을 이해하기 어렵습니다.

반대로 Motion Planning을 이해하지 못하면 실제 산업용 Robot이 어떻게 동작하는지 이해하기 어렵습니다.

따라서 Robot 개발은 기술을 개별적으로 학습하기보다 전체 흐름 안에서 이해하는 것이 중요합니다.

이 교재에서는 실제 Robot 개발 과정과 유사한 순서로 학습을 진행합니다.

---

#### Robot Basic

가장 먼저 Robot의 구조와 동작 원리를 학습합니다.

Robot은 Link와 Joint의 조합으로 구성되며, 각 관절의 움직임에 따라 Robot의 자세와 위치가 결정됩니다.

이 과정에서는 Coordinate System, Forward Kinematics, URDF, TF 등을 학습하며 Robot 내부 구조를 이해합니다.

이러한 개념은 이후 진행되는 모든 Robot 제어의 기반이 됩니다.

---

#### ROS2 Control

Robot의 구조를 이해했다면 실제 Robot을 제어합니다.

ROS2를 이용하여 Robot과 통신하고 Joint를 제어하며, Publisher, Subscriber, Service, Action과 같은 핵심 개념을 학습합니다.

또한 Pick & Place와 같은 기본 작업을 구현하며 Robot 제어 프로그램의 구조를 이해합니다.

---

**Digital Twin**

실제 Robot을 제어할 수 있게 되면 현실 세계와 가상 세계를 연결합니다.

Digital Twin은 실제 Robot과 가상 Robot이 동일한 상태를 유지하도록 연결하는 기술입니다.

실제 Robot의 상태를 가상 환경에 반영하고, 가상 환경에서 검증된 결과를 다시 실제 Robot에 적용할 수 있습니다.

현재 스마트팩토리와 AI Robot 분야에서 가장 중요하게 활용되고 있는 기술 중 하나입니다.

---

**Simulation**

실제 Robot을 이용한 개발은 비용과 위험 부담이 존재합니다.

따라서 대부분의 Robot 시스템은 Simulation 환경에서 충분한 검증 과정을 거친 후 실제 장비에 적용됩니다.

이 교재에서는 Gazebo를 이용하여 Robot을 가상 환경에서 제어하고, 센서 데이터 생성, 충돌 검출, 물리 시뮬레이션 등을 수행합니다.

Simulation 환경은 실제 장비 없이도 다양한 알고리즘과 제어 방식을 실험할 수 있는 매우 강력한 개발 도구입니다.

---

**MoveIt**

Robot을 윈하는 방향으로 움직이는 것과 원하는 위치까지 안전하게 이동시키는 것은 서로 다른 문제입니다.

MoveIt은 Robot Motion Planning Framework로, 목표 위치까지의 경로를 계산하고 충돌을 회피하며 최적의 움직임을 생성합니다.

Forward Kinematics와 Inverse Kinematics를 기반으로 Pose Goal 제어와 Pick & Place 작업을 구현하며, 실제 산업용 Robot에서 사용되는 Motion Panning 개념을 경험할 수 있습니다.

---

이러한 과정을 통해 독자는 단순히 Robot을 움직이는 수준을 넘어,

- Robot 구조 이해
- 실제 Robot wㅔ어
- Digital Twin 구축
- Simulation 검증
- Motion Planning 구현

까지 포함하는 Robot 개발의 전체 흐름을 경험하게 됩니다.

이 책의 목표는 특정 기술 하나를 깊게 다루는 것이 아닙니다.

Robot 개발 과정 전체를 경험하고 각 기술들이 서로 어떻게 연결 되는지를 이해하는 것,

그리고 이후 어떤 Robot 플랫폼이나 개발 환경을 만나더라도 빠르게 적응할 수 있는 기반을 만드는 것이 이 책의 목표입니다.
