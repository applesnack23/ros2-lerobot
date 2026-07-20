# 개발환경 소개

#### Ubuntu

Robot 분야를 공부하다 보면 가장 먼저 접하게 되는 운영체제가 Ubuntu 입니다.

ROS2는 Windows나 macOS에서도 일부 기능을 사용할 수 있지만, 대부분의 공식 문서와 예제, 산업용 Robot 환경은 Ubuntu를 기준으로 개발되고 있습니다.

따라서 ROS2를 학습하기 위해서는 Ubuntu에 익숙해지는 것이 사실상 필수라고 할 수 있습니다.

Ubuntu는 출시 연도와 월을 기준으로 버전을 관리합니다.

예를 들어:

- Ubuntu 24.04 → 2024년 4월 출시
- Ubuntu 26.04 → 2026년 4월 출시

일반적으로 Robot 개발에서는 장기간 지원되는 LTS(Long Term Support) 버전을 사용합니다. LTS 버전은 5년간 지원되며, 26.06 버전은 2031년까지 지원됩니다.

본 교재에서는 Ubuntu 26.04 LTS를 기준으로 진행합니다.

다만, 24.04 에서도 충분히 실습 가능한 내용으로만 구성되어 있습니다.

---

#### ROS2 Lyrical Luth

ROS2는 운영체제 위에서 동작하는 Robot Middleware입니다.

Node, Topic, Service, Action과 같은 통신 구조를 제공하며, Robot Application 개발을 위한 다양한 기능을 제공합니다.

ROS 역시 Ubuntu LTS 주기에 맞춰 발전해 왔습니다.

**ROS1**

- ROS1 Kinetic Kame
- ROS1 Melodic Morenia
- ROS1 Noetic Ninjemys

**ROS2**

- ROS2 Humble Hawksbill
- ROS2 Jazzy Jalisco
- ROS2 Lyrical Luth

본 교재에서는 최신 LTS 환경인 ROS2 Lyrical Luth를 사용합니다.

---

#### Python

ROS2는 Python(rclpy)과 C++(rclcpp)를 모두 지원합니다.

하지만 학습 단계에서는 Python이 문법이 간결하고 결과를 빠르게 확인할 수 있기 때문에 입문에 적합합니다.

또한 Python은 현재 AI, 데이터 분석, 자동화 분야에서 가장 많이 사용되는 언어 중 하나이며, ROS2 생태계에서도 매우 중요한 역할을 담당하고 있습니다.

따라서 본 교재에서는 Python을 준심으로 설명하며, 필요에 따라 C++ 코드도 함께 살펴봅니다.

---

**VS Code**

Visual Sudio Code는 Microsoft에서 제공하는 무료 코드 편집기입니다.

Python 개발, ROS2 패키지 개발, 디버깅, Git 관리 등을 하나의 환경에서 수행할 수 있기 때문에 현재 가장 많이 사용되는 개발 도구 중 하나입니다.

본 교재에서는 모든 Python 코드와 ROS2 패키지를 VS Code를 사용하여 개발합니다.

---

**GitHub**

Robot 개발에서는 오픈소스 활용이 매우 중요합니다.

실제로 대부분의 ROS2 패키지와 Robot 프로젝트는 GitHub를 통해 공개되고 관리됩니다.

또한 Robot 생태계는 매우 빠르게 변화하기 때문에 공식 문서와 GitHHub 저장소를 참고하는 습관이 중요합니다.

본 교재에서 사용되는 예제 코드와 업데이트 내용 역시 GitHub를 통해 제공할 예정입니다.
