# 실습 장비 소개

#### SO ARM101

![SO101_Leader.webp](%EC%8B%A4%EC%8A%B5%20%EC%9E%A5%EB%B9%84%20%EC%86%8C%EA%B0%9C/SO101_Leader.webp)

SO ARM101 Leader Arm

![SO101_Follower.webp](%EC%8B%A4%EC%8A%B5%20%EC%9E%A5%EB%B9%84%20%EC%86%8C%EA%B0%9C/SO101_Follower.webp)

SO ARM101 Follower Arm

본 교재에서는 실습용 Robot Arm으로 SO ARM101을 사용합니다.

SO ARM101은 Hugging Face의 LeRobot 프로젝트를 위해 개발된 오픈소스 교육용 Robot Arm으로, 비교적 저렴한 비용으로 실제 Robot 제어와 AI Robot 기술을 학습할 수 있도록 설계되었습니다.

기존 SO ARM100의 후속 모델로 배선 구조와 조립성이 개선 되었으며, 모방학습(Imitation Learning)과 원격조작 (Teleoperation)을 고려하여 설계되었습니다.

SO ARM101은 산업용 Robot과 비교하면 크기와 성능은 제한적이지만, Robot 제어에 필요한 대부분의 핵심 개념을 학습 하기에는 충분한 기능을 제공합니다.

본 교재에서는 SO ARM101을 이용하여 다음과 같은 내용을 학습합니다.

- Joint 제어
- End Effector 제어
- Pick & Place
- ROS2 기반 Robot 제어
- Gazebo Simulation
- MoveIt Motion Planning
- Digital Twin
- Sim2Real

또한 SO ARM101은 실제 Robot과 가상 Robot을 함께 다루기 위한 교육용 플랫폼으로 매우 적합하며, 향후 AI Robot 및 모방학습 분야까지 확장할 수 있다는 장점이 있습니다.

이 책에서는 단순히 Robot을 움직이는 것에 그치지 않고, 실제 Robot과 Simulation 환경을 연결하고 비교하면서 Robot 시스템 전체를 이해하는 것을 목표로 합니다.

---

### PC 사양

Robot 개발 환경은 일반적인 프로그래밍 환경보다 높은 성능을 요구합니다.

특히 Gazebo Simulation, RViz2, MoveIt, Digital Twin 환경을 동시에 실행하게 되면 CPU와 메모리 사용량이 크게 증가하며, 3D 렌더링을 위해 GPU 성능 역시 중요합니다.

다행히 ROS2 자체는 비교적 가벼운 시스템이기 때문에 학습 단계에서는 고사양 장비가 반드시 필요한 것은 아닙니다.

**최소 사양 (학습용 권장 사양)**

학습 목적으로 ROS2와 SO ARM101을 사용하는 경우 아래 정도의 사양이면 충분합니다.

- CPU : Intel i5 또는 Ryzen 5 이상
- RAM : 16GB 이상
- GPU : NVIDIA GTX1660 또는 동급 이상
- SSD : 256GB 이상

**권장 사양 (실무용 권장 사양)**

Gazebo, RViz2, MoveIt, Digital Twin 환경을 보다 쾌적하게 사용하려면 아래 사양을 권장합니다.

- CPU : Intel i7 또는 Ryzen 7 이상
- RAM : 16GB 이상
- GPU : NVIDIA RTX3060 이상
- SSD : 512GB 이상
