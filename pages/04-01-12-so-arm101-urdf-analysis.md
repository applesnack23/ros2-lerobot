# SO ARM 101 URDF/ 구조 분석

앞 장까지 URDF를 구성하는 핵심 요소들을 하나씩 살펴보았습니다.

- Link
- Joint
- Origin
- Axis
- Limit
- Visual
- Collision
- Inertial
- TF Tree

이제부터는 각 요소가 실제 로봇의 URDF에서 어떻게 사용되는지 분석해 보겠습니다.

이번 장에서는 SO-ARM101의 URDF를 기준으로 로봇의 구조, 움직임, 외형, 충돌 영역, 물리 특성을 확인합니다.

실제 프로젝트에서는 URDF를 처음부터 작성하는 경우도 있지만, 이미 만들어진 URDF나 Xacro 파일을 읽고 수정해야 하는 경우가 더 많습니다.

따라서 URDF의 각 값을 외우는 것보다 다음 내용을 파악할 수 있어야 합니다.

- 로봇이 몇 개의 Link로 구성되어 있는가?
- Link들이 어떤 Joint로 연결되어 있는가?
- 각 Joint는 어느 축을 기준으로 움직이는가?
- 각 Joint의 움직임 범위는 얼마인가?
- 3D 모델은 어느 경로에서 불러오는가?
- 충돌 형상은 실제 외형과 일치하는가?
- Link의 질량과 무게중심은 어떻게 정의되어 있는가?
- 전체 구조가 올바른 TF Tree를 만드는가?

---

#### SO-ARM101 URDF 원문

#### 전체 소스 코드

> GitHub Link: [https://github.com/applesnack23/ros2-lerobot-code/blob/main/chapter4/so_arm101.urdf](https://github.com/applesnack23/ros2-lerobot-code/blob/main/chapter4/so_arm101.urdf)
> 

```xml
<?xml version="1.0" encoding="utf-8"?>
<!-- Generated using onshape-to-robot -->
<!-- Onshape https://cad.onshape.com/documents/7715cc284bb430fe6dab4ffd/w/4fd0791b683777b02f8d975a/e/826c553ede3b7592eb9ca800 -->
<robot name="so101_new_calib">

  <!-- Materials -->
  <material name="3d_printed">
    <color rgba="1.0 0.82 0.12 1.0"/>
  </material>
  <material name="sts3215">
    <color rgba="0.1 0.1 0.1 1.0"/>
  </material>

  <!-- Link base -->
  <link name="base_link">
    <inertial>
      <origin xyz="0.0137179 -5.19711e-05 0.0334843" rpy="0 0 0"/>
      <mass value="0.147"/>
      <inertia ixx="0.000114686" ixy="-4.59787e-07" ixz="4.97151e-06"
               iyy="0.000136117" iyz="9.75275e-08" izz="0.000130364"/>
    </inertial>

    <!-- Part base_motor_holder_so101_v1 -->
    <visual>
      <origin xyz="-0.00636471 -9.94414e-05 -0.0024"
              rpy="1.5708 -1.67685e-15 1.5708"/>
      <geometry>
        <mesh filename="assets/base_motor_holder_so101_v1.stl"/>
      </geometry>
      <material name="3d_printed"/>
    </visual>
    <collision>
      <origin xyz="-0.00636471 -9.94414e-05 -0.0024"
              rpy="1.5708 -1.67685e-15 1.5708"/>
      <geometry>
        <mesh filename="assets/base_motor_holder_so101_v1.stl"/>
      </geometry>
    </collision>

    <!-- Part base_so101_v2 -->
    <visual>
      <origin xyz="-0.00636471 -8.97657e-09 -0.0024"
              rpy="1.5708 -2.78073e-29 1.5708"/>
      <geometry>
        <mesh filename="assets/base_so101_v2.stl"/>
      </geometry>
      <material name="3d_printed"/>
    </visual>
    <collision>
      <origin xyz="-0.00636471 -8.97657e-09 -0.0024"
              rpy="1.5708 -2.78073e-29 1.5708"/>
      <geometry>
        <mesh filename="assets/base_so101_v2.stl"/>
      </geometry>
    </collision>

    <!-- Part sts3215_03a_v1 -->
    <visual>
      <origin xyz="0.0263353 -8.97657e-09 0.0437"
              rpy="-8.21148e-16 7.84513e-18 1.249e-15"/>
      <geometry>
        <mesh filename="assets/sts3215_03a_v1.stl"/>
      </geometry>
      <material name="sts3215"/>
    </visual>
    <collision>
      <origin xyz="0.0263353 -8.97657e-09 0.0437"
              rpy="-8.21148e-16 7.84513e-18 1.249e-15"/>
      <geometry>
        <mesh filename="assets/sts3215_03a_v1.stl"/>
      </geometry>
    </collision>

    <!-- Part waveshare_mounting_plate_so101_v2 -->
    <visual>
      <origin xyz="-0.0309827 -0.000199441 0.0474"
              rpy="1.5708 -1.35493e-14 1.5708"/>
      <geometry>
        <mesh filename="assets/waveshare_mounting_plate_so101_v2.stl"/>
      </geometry>
      <material name="3d_printed"/>
    </visual>
    <collision>
      <origin xyz="-0.0309827 -0.000199441 0.0474"
              rpy="1.5708 -1.35493e-14 1.5708"/>
      <geometry>
        <mesh filename="assets/waveshare_mounting_plate_so101_v2.stl"/>
      </geometry>
    </collision>
  </link>

  <!-- Link shoulder -->
  <link name="shoulder_link">
    <inertial>
      <origin xyz="-0.0307604 -1.66727e-05 -0.0252713" rpy="0 0 0"/>
      <mass value="0.100006"/>
      <inertia ixx="8.3759e-05" ixy="7.55525e-08" ixz="-1.16342e-06"
               iyy="8.10403e-05" iyz="1.54663e-07" izz="2.39783e-05"/>
    </inertial>

    <!-- Part sts3215_03a_v1_2 -->
    <visual>
      <origin xyz="-0.0303992 0.000422241 -0.0417"
              rpy="1.5708 1.5708 0"/>
      <geometry>
        <mesh filename="assets/sts3215_03a_v1.stl"/>
      </geometry>
      <material name="sts3215"/>
    </visual>
    <collision>
      <origin xyz="-0.0303992 0.000422241 -0.0417"
              rpy="1.5708 1.5708 0"/>
      <geometry>
        <mesh filename="assets/sts3215_03a_v1.stl"/>
      </geometry>
    </collision>

    <!-- Part motor_holder_so101_base_v1 -->
    <visual>
      <origin xyz="-0.0675992 -0.000177759 0.0158499"
              rpy="1.5708 -1.5708 0"/>
      <geometry>
        <mesh filename="assets/motor_holder_so101_base_v1.stl"/>
      </geometry>
      <material name="3d_printed"/>
    </visual>
    <collision>
      <origin xyz="-0.0675992 -0.000177759 0.0158499"
              rpy="1.5708 -1.5708 0"/>
      <geometry>
        <mesh filename="assets/motor_holder_so101_base_v1.stl"/>
      </geometry>
    </collision>

    <!-- Part rotation_pitch_so101_v1 -->
    <visual>
      <origin xyz="0.0122008 2.22413e-05 0.0464"
              rpy="-1.5708 2.35221e-33 0"/>
      <geometry>
        <mesh filename="assets/rotation_pitch_so101_v1.stl"/>
      </geometry>
      <material name="3d_printed"/>
    </visual>
    <collision>
      <origin xyz="0.0122008 2.22413e-05 0.0464"
              rpy="-1.5708 2.35221e-33 0"/>
      <geometry>
        <mesh filename="assets/rotation_pitch_so101_v1.stl"/>
      </geometry>
    </collision>
  </link>

  <!-- Link upper_arm -->
  <link name="upper_arm_link">
    <inertial>
      <origin xyz="-0.0898471 -0.00838224 0.0184089" rpy="0 0 0"/>
      <mass value="0.103"/>
      <inertia ixx="4.08002e-05" ixy="-1.97819e-05" ixz="-4.03016e-08"
               iyy="0.000147318" iyz="8.97326e-09" izz="0.000142487"/>
    </inertial>

    <!-- Part sts3215_03a_v1_3 -->
    <visual>
      <origin xyz="-0.11257 -0.0155 0.0187"
              rpy="-3.14159 -5.27356e-16 -1.5708"/>
      <geometry>
        <mesh filename="assets/sts3215_03a_v1.stl"/>
      </geometry>
      <material name="sts3215"/>
    </visual>
    <collision>
      <origin xyz="-0.11257 -0.0155 0.0187"
              rpy="-3.14159 -5.27356e-16 -1.5708"/>
      <geometry>
        <mesh filename="assets/sts3215_03a_v1.stl"/>
      </geometry>
    </collision>

    <!-- Part upper_arm_so101_v1 -->
    <visual>
      <origin xyz="-0.065085 0.012 0.0182"
              rpy="3.14159 -0 -1.30911e-30"/>
      <geometry>
        <mesh filename="assets/upper_arm_so101_v1.stl"/>
      </geometry>
      <material name="3d_printed"/>
    </visual>
    <collision>
      <origin xyz="-0.065085 0.012 0.0182"
              rpy="3.14159 -0 -1.30911e-30"/>
      <geometry>
        <mesh filename="assets/upper_arm_so101_v1.stl"/>
      </geometry>
    </collision>
  </link>

  <!-- Link lower_arm -->
  <link name="lower_arm_link">
    <inertial>
      <origin xyz="-0.0980701 0.00324376 0.0182831" rpy="0 0 0"/>
      <mass value="0.104"/>
      <inertia ixx="2.87438e-05" ixy="7.41152e-06" ixz="1.26409e-06"
               iyy="0.000159844" iyz="-4.90188e-08" izz="0.00014529"/>
    </inertial>

    <!-- Part under_arm_so101_v1 -->
    <visual>
      <origin xyz="-0.0648499 -0.032 0.0182"
              rpy="3.14159 -0 6.67202e-31"/>
      <geometry>
        <mesh filename="assets/under_arm_so101_v1.stl"/>
      </geometry>
      <material name="3d_printed"/>
    </visual>
    <collision>
      <origin xyz="-0.0648499 -0.032 0.0182"
              rpy="3.14159 -0 6.67202e-31"/>
      <geometry>
        <mesh filename="assets/under_arm_so101_v1.stl"/>
      </geometry>
    </collision>

    <!-- Part motor_holder_so101_wrist_v1 -->
    <visual>
      <origin xyz="-0.0648499 -0.032 0.018"
              rpy="-3.14159 -2.55351e-15 -1.83387e-30"/>
      <geometry>
        <mesh filename="assets/motor_holder_so101_wrist_v1.stl"/>
      </geometry>
      <material name="3d_printed"/>
    </visual>
    <collision>
      <origin xyz="-0.0648499 -0.032 0.018"
              rpy="-3.14159 -2.55351e-15 -1.83387e-30"/>
      <geometry>
        <mesh filename="assets/motor_holder_so101_wrist_v1.stl"/>
      </geometry>
    </collision>

    <!-- Part sts3215_03a_v1_4 -->
    <visual>
      <origin xyz="-0.1224 0.0052 0.0187"
              rpy="-3.14159 -7.88861e-31 -3.14159"/>
      <geometry>
        <mesh filename="assets/sts3215_03a_v1.stl"/>
      </geometry>
      <material name="sts3215"/>
    </visual>
    <collision>
      <origin xyz="-0.1224 0.0052 0.0187"
              rpy="-3.14159 -7.88861e-31 -3.14159"/>
      <geometry>
        <mesh filename="assets/sts3215_03a_v1.stl"/>
      </geometry>
    </collision>
  </link>

  <!-- Link wrist -->
  <link name="wrist_link">
    <inertial>
      <origin xyz="-0.000103312 -0.0386143 0.0281156" rpy="0 0 0"/>
      <mass value="0.079"/>
      <inertia ixx="3.68263e-05" ixy="1.7893e-08" ixz="-5.28128e-08"
               iyy="2.5391e-05" iyz="3.6412e-06" izz="2.1e-05"/>
    </inertial>

    <!-- Part sts3215_03a_no_horn_v1 -->
    <visual>
      <origin xyz="8.32667e-17 -0.0424 0.0306"
              rpy="1.5708 1.5708 0"/>
      <geometry>
        <mesh filename="assets/sts3215_03a_no_horn_v1.stl"/>
      </geometry>
      <material name="sts3215"/>
    </visual>
    <collision>
      <origin xyz="8.32667e-17 -0.0424 0.0306"
              rpy="1.5708 1.5708 0"/>
      <geometry>
        <mesh filename="assets/sts3215_03a_no_horn_v1.stl"/>
      </geometry>
    </collision>

    <!-- Part wrist_roll_pitch_so101_v2 -->
    <visual>
      <origin xyz="0 -0.028 0.0181"
              rpy="-1.5708 -1.5708 0"/>
      <geometry>
        <mesh filename="assets/wrist_roll_pitch_so101_v2.stl"/>
      </geometry>
      <material name="3d_printed"/>
    </visual>
    <collision>
      <origin xyz="0 -0.028 0.0181"
              rpy="-1.5708 -1.5708 0"/>
      <geometry>
        <mesh filename="assets/wrist_roll_pitch_so101_v2.stl"/>
      </geometry>
    </collision>
  </link>

  <!-- Link gripper -->
  <link name="gripper_link">
    <inertial>
      <origin xyz="0.000213627 0.000245138 -0.025187" rpy="0 0 0"/>
      <mass value="0.087"/>
      <inertia ixx="2.75087e-05" ixy="-3.35241e-07" ixz="-5.7352e-06"
               iyy="4.33657e-05" iyz="-5.17847e-08" izz="3.45059e-05"/>
    </inertial>

    <!-- Part sts3215_03a_v1_5 -->
    <visual>
      <origin xyz="0.0077 0.0001 -0.0234"
              rpy="-1.5708 -5.19179e-17 -1.66533e-16"/>
      <geometry>
        <mesh filename="assets/sts3215_03a_v1.stl"/>
      </geometry>
      <material name="sts3215"/>
    </visual>
    <collision>
      <origin xyz="0.0077 0.0001 -0.0234"
              rpy="-1.5708 -5.19179e-17 -1.66533e-16"/>
      <geometry>
        <mesh filename="assets/sts3215_03a_v1.stl"/>
      </geometry>
    </collision>

    <!-- Part wrist_roll_follower_so101_v1 -->
    <visual>
      <origin xyz="8.32667e-17 -0.000218214 0.000949706"
              rpy="-3.14159 -5.55112e-17 0"/>
      <geometry>
        <mesh filename="assets/wrist_roll_follower_so101_v1.stl"/>
      </geometry>
      <material name="3d_printed"/>
    </visual>
    <collision>
      <origin xyz="8.32667e-17 -0.000218214 0.000949706"
              rpy="-3.14159 -5.55112e-17 0"/>
      <geometry>
        <mesh filename="assets/wrist_roll_follower_so101_v1.stl"/>
      </geometry>
    </collision>
  </link>

  <!-- Gripper frame (dummy link + fixed joint) -->
  <link name="gripper_frame_link">
    <origin xyz="0 0 0" rpy="0 -0 0"/>
    <inertial>
      <origin xyz="0 0 0" rpy="0 0 0"/>
      <mass value="1e-9"/>
      <inertia ixx="0" ixy="0" ixz="0"
               iyy="0" iyz="0" izz="0"/>
    </inertial>
  </link>

  <joint name="gripper_frame_joint" type="fixed">
    <origin xyz="-0.0079 -0.000218121 -0.0981274"
            rpy="0 3.14159 0"/>
    <parent link="gripper_link"/>
    <child link="gripper_frame_link"/>
    <axis xyz="0 0 0"/>
  </joint>

  <!-- Link moving_jaw_so101_v1 -->
  <link name="moving_jaw_so101_v1_link">
    <inertial>
      <origin xyz="-0.00157495 -0.0300244 0.0192755" rpy="0 0 0"/>
      <mass value="0.012"/>
      <inertia ixx="6.61427e-06" ixy="-3.19807e-07" ixz="-5.90717e-09"
               iyy="1.89032e-06" iyz="-1.09945e-07" izz="5.28738e-06"/>
    </inertial>

    <!-- Part moving_jaw_so101_v1 -->
    <visual>
      <origin xyz="-5.55112e-17 -5.55112e-17 0.0189"
              rpy="9.53145e-17 6.93889e-18 1.24077e-24"/>
      <geometry>
        <mesh filename="assets/moving_jaw_so101_v1.stl"/>
      </geometry>
      <material name="3d_printed"/>
    </visual>
    <collision>
      <origin xyz="-5.55112e-17 -5.55112e-17 0.0189"
              rpy="9.53145e-17 6.93889e-18 1.24077e-24"/>
      <geometry>
        <mesh filename="assets/moving_jaw_so101_v1.stl"/>
      </geometry>
    </collision>
  </link>

  <!-- Joint from gripper to moving_jaw_so101_v1 -->
  <joint name="gripper" type="revolute">
    <origin xyz="0.0202 0.0188 -0.0234"
            rpy="1.5708 -5.24284e-08 -1.41553e-15"/>
    <parent link="gripper_link"/>
    <child link="moving_jaw_so101_v1_link"/>
    <axis xyz="0 0 1"/>
    <limit effort="10" velocity="10"
           lower="-0.174533" upper="1.74533"/>
  </joint>

  <transmission name="gripper_trans">
    <type>transmission_interface/SimpleTransmission</type>
    <joint name="gripper">
      <hardwareInterface>
        hardware_interface/PositionJointInterface
      </hardwareInterface>
    </joint>
    <actuator name="motor6">
      <hardwareInterface>
        hardware_interface/PositionJointInterface
      </hardwareInterface>
      <mechanicalReduction>1</mechanicalReduction>
    </actuator>
  </transmission>

  <!-- Joint from wrist to gripper -->
  <joint name="wrist_roll" type="revolute">
    <origin xyz="5.55112e-17 -0.0611 0.0181"
            rpy="1.5708 0.0486795 3.14159"/>
    <parent link="wrist_link"/>
    <child link="gripper_link"/>
    <axis xyz="0 0 1"/>
    <limit effort="10" velocity="10"
           lower="-2.74385" upper="2.84121"/>
  </joint>

  <transmission name="wrist_roll_trans">
    <type>transmission_interface/SimpleTransmission</type>
    <joint name="wrist_roll">
      <hardwareInterface>
        hardware_interface/PositionJointInterface
      </hardwareInterface>
    </joint>
    <actuator name="motor5">
      <hardwareInterface>
        hardware_interface/PositionJointInterface
      </hardwareInterface>
      <mechanicalReduction>1</mechanicalReduction>
    </actuator>
  </transmission>

  <!-- Joint from lower_arm to wrist -->
  <joint name="wrist_flex" type="revolute">
    <origin xyz="-0.1349 0.0052 3.62355e-17"
            rpy="4.02456e-15 8.67362e-16 -1.5708"/>
    <parent link="lower_arm_link"/>
    <child link="wrist_link"/>
    <axis xyz="0 0 1"/>
    <limit effort="10" velocity="10"
           lower="-1.65806" upper="1.65806"/>
  </joint>

  <transmission name="wrist_flex_trans">
    <type>transmission_interface/SimpleTransmission</type>
    <joint name="wrist_flex">
      <hardwareInterface>
        hardware_interface/PositionJointInterface
      </hardwareInterface>
    </joint>
    <actuator name="motor4">
      <hardwareInterface>
        hardware_interface/PositionJointInterface
      </hardwareInterface>
      <mechanicalReduction>1</mechanicalReduction>
    </actuator>
  </transmission>

  <!-- Joint from upper_arm to lower_arm -->
  <!-- Note: 5-degree calibration offset applied to joint limits -->
  <joint name="elbow_flex" type="revolute">
    <origin xyz="-0.11257 -0.028 1.73763e-16"
            rpy="-3.63608e-16 8.74301e-16 1.5708"/>
    <parent link="upper_arm_link"/>
    <child link="lower_arm_link"/>
    <axis xyz="0 0 1"/>
    <limit effort="10" velocity="10"
           lower="-1.69" upper="1.69"/>
  </joint>

  <transmission name="elbow_flex_trans">
    <type>transmission_interface/SimpleTransmission</type>
    <joint name="elbow_flex">
      <hardwareInterface>
        hardware_interface/PositionJointInterface
      </hardwareInterface>
    </joint>
    <actuator name="motor3">
      <hardwareInterface>
        hardware_interface/PositionJointInterface
      </hardwareInterface>
      <mechanicalReduction>1</mechanicalReduction>
    </actuator>
  </transmission>

  <!-- Joint from shoulder to upper_arm -->
  <joint name="shoulder_lift" type="revolute">
    <origin xyz="-0.0303992 -0.0182778 -0.0542"
            rpy="-1.5708 -1.5708 0"/>
    <parent link="shoulder_link"/>
    <child link="upper_arm_link"/>
    <axis xyz="0 0 1"/>
    <limit effort="10" velocity="10"
           lower="-1.74533" upper="1.74533"/>
  </joint>

  <transmission name="shoulder_lift_trans">
    <type>transmission_interface/SimpleTransmission</type>
    <joint name="shoulder_lift">
      <hardwareInterface>
        hardware_interface/PositionJointInterface
      </hardwareInterface>
    </joint>
    <actuator name="motor2">
      <hardwareInterface>
        hardware_interface/PositionJointInterface
      </hardwareInterface>
      <mechanicalReduction>1</mechanicalReduction>
    </actuator>
  </transmission>

  <!-- Joint from base to shoulder -->
  <joint name="shoulder_pan" type="revolute">
    <origin xyz="0.0388353 -8.97657e-09 0.0624"
            rpy="3.14159 4.18253e-17 -3.14159"/>
    <parent link="base_link"/>
    <child link="shoulder_link"/>
    <axis xyz="0 0 1"/>
    <limit effort="10" velocity="10"
           lower="-1.91986" upper="1.91986"/>
  </joint>

  <transmission name="shoulder_pan_trans">
    <type>transmission_interface/SimpleTransmission</type>
    <joint name="shoulder_pan">
      <hardwareInterface>
        hardware_interface/PositionJointInterface
      </hardwareInterface>
    </joint>
    <actuator name="motor1">
      <hardwareInterface>
        hardware_interface/PositionJointInterface
      </hardwareInterface>
      <mechanicalReduction>1</mechanicalReduction>
    </actuator>
  </transmission>

</robot>
```

---

#### URDF 분석 순서

복잡한 URDF를 처음부터 한 줄씩 읽으면 전체 구조를 파악하기 어렵습니다.

다음 순서로 분석하면 비교적 쉽게 이해할 수 있습니다.

1. 전체 Link 목록을 확인합니다.
2. 전체 Joint 목록을 확인합니다.
3. Parent와 Child 관계를 따라 TF Tree를 구성합니다.
4. 각 Joint의 Origin을 확인합니다.
5. 각 Joint의 Axis와 Limit를 확인합니다.
6. Link의 Visual과 Mesh 경로를 확인합니다.
7. Collision 형상의 위치와 크기를 확인합니다.
8. Inertial의 질량과 무게중심을 확인합니다.
9. RViz2에서 외형과 TF를 검증합니다.
10. Gazebo에서 물리 동작을 검증합니다.

이 순서를 사용하면 먼저 로봇의 전체 뼈대를 이해한 다음 세부적인 외형과 물리 정보를 확인할 수 있습니다.

---

#### 전체 구조 확인

SO-ARM101의 구조를 단순화하면 다음과 같이 표현할 수 있습니다.

```
base_link
└── shoulder_pan_joint
    └── shoulder_link
        └── shoulder_lift_joint
            └── upper_arm_link
                └── elbow_joint
                    └── forearm_link
                        └── wrist_flex_joint
                            └── wrist_link
                                └── wrist_roll_joint
                                    └── tool_link
                                        └── gripper_joint
                                            └── gripper_link
```

Link와 Joint만 구분해서 보면 다음 구조가 반복됩니다.

```
Link
└── Joint
    └── Link
        └── Joint
            └── Link
```

SO-ARM101은 여러 개의 회전 관절이 직렬로 연결된 다관절 로봇 팔입니다.

전체적인 동작 흐름은 다음과 같습니다.

- Base를 기준으로 Shoulder Pan 회전
- Shoulder Lift를 이용한 팔의 상승과 하강
- Elbow를 이용한 팔꿈치 굽힘
- Wrist Flex를 이용한 손목 굽힘
- Wrist Roll을 이용한 Tool 회전
- Gripper를 이용한 물체 잡기와 놓기

상위 Joint가 움직이면 그 아래에 연결된 모든 Link의 위치와 방향이 함께 변경됩니다.

예를 들어 `shoulder_pan_joint`가 움직이면 다음 Link들이 모두 영향을 받습니다.

- `shoulder_link`
- `upper_arm_link`
- `forearm_link`
- `wrist_link`
- `tool_link`
- `gripper_link`

---

#### Link 구성 분석

Link는 SO-ARM101을 구성하는 각각의 강체를 의미합니다.

구조를 단순화하면 다음과 같은 Link가 존재합니다.

```xml
<link name="base_link"/>
<link name="shoulder_link"/>
<link name="upper_arm_link"/>
<link name="forearm_link"/>
<link name="wrist_link"/>
<link name="tool_link"/>
<link name="gripper_link"/>
```

각 Link의 역할은 다음과 같이 정리할 수 있습니다.

| Link | 역할 |
| --- | --- |
| `base_link` | 전체 로봇의 기준이 되는 Base |
| `shoulder_link` | Shoulder Pan 이후의 몸체 |
| `upper_arm_link` | Shoulder와 Elbow 사이의 상부 Arm |
| `forearm_link` | Elbow와 Wrist 사이의 하부 Arm |
| `wrist_link` | 손목 굽힘을 담당하는 몸체 |
| `tool_link` | 손목 회전 이후 Tool 또는 Gripper 연결부 |
| `gripper_link` | 물체를 잡고 놓는 Gripper 몸체 |

`tool_link`는 Wrist와 Gripper 사이의 회전 및 연결을 담당하는 Link로 볼 수 있습니다.

```
wrist_link
    ↓
wrist_roll_joint
    ↓
tool_link
    ↓
gripper_joint
    ↓
gripper_link
```

Link 이름을 역할이 드러나도록 작성하면 URDF 코드만 보고도 로봇의 전체 구조를 어느 정도 추측할 수 있습니다.

---

#### Root Link 확인

위 구조에서 Parent Joint를 가지지 않는 `base_link`가 URDF 모델의 Root Link입니다.

```
base_link
└── 나머지 로봇 구조
```

모든 Link의 위치와 방향은 최종적으로 `base_link`를 기준으로 계산할 수 있습니다.

다만 `base_link`가 Root Link라는 것은 URDF 내부 구조의 시작점이라는 뜻입니다. 자동으로 Gazebo의 바닥이나 `world` Frame에 고정되는 것은 아닙니다.

로봇을 Gazebo 바닥에 고정하려면 필요에 따라 다음과 같은 관계를 추가할 수 있습니다.

```
world
└── base_link
```

예를 들어 고정 Joint를 사용할 수 있습니다.

```xml
<link name="world"/>

<joint name="world_to_base" type="fixed">
    <parent link="world"/>
    <child link="base_link"/>
    <origin xyz="0 0 0" rpy="0 0 0"/>
</joint>
```

실제 로봇의 URDF와 Gazebo용 URDF를 분리하여 관리한다면 이 Joint는 Gazebo용 Xacro 파일에만 추가할 수도 있습니다.

---

#### Joint 구성 분석

Joint는 Link와 Link를 연결하고 로봇의 움직임을 정의합니다.

SO-ARM101의 주요 Joint를 단순화하면 다음과 같습니다.

```xml
<joint name="shoulder_pan_joint" type="revolute">
    ...
</joint>

<joint name="shoulder_lift_joint" type="revolute">
    ...
</joint>

<joint name="elbow_joint" type="revolute">
    ...
</joint>

<joint name="wrist_flex_joint" type="revolute">
    ...
</joint>

<joint name="wrist_roll_joint" type="revolute">
    ...
</joint>

<joint name="gripper_joint" type="revolute">
    ...
</joint>
```

이 모델에서는 주요 Joint가 `revolute` 타입으로 구성되어 있습니다.

`revolute` Joint는 정해진 최소 각도와 최대 각도 안에서 회전합니다.

서보 모터를 사용하는 로봇 팔은 일반적으로 다음과 같은 이유로 Revolute Joint를 사용합니다.

- 서보 모터의 출력이 회전 운동임
- 기구적으로 회전 가능한 범위가 정해져 있음
- 각도 단위로 위치를 제어함
- Joint Limit로 안전 범위를 설정할 수 있음

각 Joint의 기능은 다음과 같습니다.

| Joint | 주요 기능 |
| --- | --- |
| `shoulder_pan_joint` | Base 위에서 Arm 전체 회전 |
| `shoulder_lift_joint` | Arm 상승 및 하강 |
| `elbow_joint` | 팔꿈치 굽힘 |
| `wrist_flex_joint` | 손목 굽힘 |
| `wrist_roll_joint` | Tool 또는 Gripper 회전 |
| `gripper_joint` | Gripper 열기 및 닫기 |

---

#### Parent와 Child 분석

각 Joint를 분석할 때는 가장 먼저 Parent Link와 Child Link를 확인해야 합니다.

예를 들어 다음 Joint가 있다고 가정해 보겠습니다.

```xml
<joint name="elbow_joint" type="revolute">
    <parent link="upper_arm_link"/>
    <child link="forearm_link"/>
</joint>
```

이 Joint의 연결 관계는 다음과 같습니다.

```
upper_arm_link
      ↓
 elbow_joint
      ↓
forearm_link
```

즉, `forearm_link`는 `upper_arm_link`를 기준으로 움직입니다.

주요 Parent와 Child 관계를 표로 정리하면 다음과 같습니다.

| Joint | Parent Link | Child Link |
| --- | --- | --- |
| `shoulder_pan_joint` | `base_link` | `shoulder_link` |
| `shoulder_lift_joint` | `shoulder_link` | `upper_arm_link` |
| `elbow_joint` | `upper_arm_link` | `forearm_link` |
| `wrist_flex_joint` | `forearm_link` | `wrist_link` |
| `wrist_roll_joint` | `wrist_link` | `tool_link` |
| `gripper_joint` | `tool_link` | `gripper_link` |

실제 URDF의 이름이나 연결 관계가 다르다면 실제 파일을 기준으로 표를 수정해야 합니다.

---

#### Origin 분석

Origin은 Link, Visual, Collision, Inertial의 위치와 방향을 정의합니다.

SO-ARM101의 `base_link`에는 Visual, Collision, Inertial에 각각 다른 Origin이 사용될 수 있습니다.

Origin을 분석할 때는 어떤 요소 안에 들어 있는 Origin인지 먼저 확인해야 합니다.

**Visual Origin**

`base_link`의 Visual Origin이 다음과 같이 정의되어 있다고 가정해 보겠습니다.

```xml
<origin
    xyz="-0.00636471 -9.94414e-05 -0.0024"
    rpy="1.5708 -1.67685e-15 1.5708"/>
```

`xyz` 값은 Link Frame을 기준으로 Visual Mesh를 이동시키는 값입니다.

| 축 | 값 | 약식 변환 |
| --- | --- | --- |
| X | -0.00636471 m | 약 -6.36 mm |
| Y | -0.0000994414 m | 약 -0.10 mm |
| Z | -0.0024 m | 약 -2.4 mm |

`rpy` 값은 Visual Mesh의 방향을 조정합니다.

```
Roll  = 1.5708 rad ≈ 90°
Pitch = 거의 0°
Yaw   = 1.5708 rad ≈ 90°
```

STL 모델을 제작한 CAD 좌표계와 URDF의 Link Frame 방향이 다르기 때문에 이러한 회전 보정이 적용된 것입니다.

Visual Origin은 Link 자체를 움직이는 값이 아닙니다.

> **Visual Origin은 Link Frame을 기준으로 화면에 표시할 Mesh의 위치와 방향을 조정합니다.**
> 

**Collision Origin**

Collision Origin이 다음과 같이 Visual Origin과 동일하게 정의되어 있을 수 있습니다.

```xml
<origin
    xyz="-0.00636471 -9.94414e-05 -0.0024"
    rpy="1.5708 -1.67685e-15 1.5708"/>
```

Visual과 Collision이 동일한 Mesh를 사용하고 Origin도 같다면 화면에 보이는 외형과 충돌 계산에 사용하는 형상이 일치합니다.

```
Visual 위치    = Collision 위치
Visual 방향    = Collision 방향
```

다만 Collision에 단순화된 Box나 Cylinder를 사용한다면 Visual과 다른 Origin이 필요할 수도 있습니다.

**Inertial Origin**

Inertial Origin이 다음과 같이 정의되어 있다고 가정해 보겠습니다.

```xml
<origin
    xyz="0.0137179 -5.19711e-05 0.0334843"
    rpy="0 0 0"/>
```

이 값은 Link Frame을 기준으로 질량 중심의 위치를 나타냅니다.

| 축 | 값 | 약식 변환 |
| --- | --- | --- |
| X | 0.0137179 m | 약 +13.72 mm |
| Y | -0.0000519711 m | 약 -0.052 mm |
| Z | 0.0334843 m | 약 +33.48 mm |

따라서 질량 중심은 Link Frame의 원점과 정확히 일치하지 않습니다.

방향을 해석할 때는 해당 Link Frame의 축을 기준으로 판단해야 합니다. 일반적인 ROS2 좌표계가 적용되어 있다면 다음과 같이 볼 수 있습니다.

```
+X = 앞쪽
+Y = 왼쪽
+Z = 위쪽
```

이 기준에서는 음수 Y가 오른쪽 방향입니다. 다만 실제 방향은 RViz2에서 해당 Link Frame의 축을 확인한 후 판단하는 것이 정확합니다.

---

#### Origin 비교

같은 Link 안에서도 Origin의 목적은 서로 다릅니다.

| Origin | 역할 |
| --- | --- |
| Joint Origin | Parent Link를 기준으로 Child Link 배치 |
| Visual Origin | Link Frame을 기준으로 Mesh 배치 |
| Collision Origin | Link Frame을 기준으로 충돌 형상 배치 |
| Inertial Origin | Link Frame을 기준으로 질량 중심 배치 |

Origin을 분석할 때 단순히 숫자만 보는 것이 아니라, 어느 요소의 Origin인지 반드시 함께 확인해야 합니다.

---

#### Axis 분석

Axis는 Joint의 회전축을 정의합니다.

예를 들어 다음 Axis는 Z축을 기준으로 회전한다는 의미입니다.

```xml
<axis xyz="0 0 1"/>
```

주요 Axis의 의미는 다음과 같습니다.

| Axis | 움직임 |
| --- | --- |
| `1 0 0` | X축 기준 회전 |
| `0 1 0` | Y축 기준 회전 |
| `0 0 1` | Z축 기준 회전 |
| `-1 0 0` | -X축 기준 회전 |
| `0 -1 0` | -Y축 기준 회전 |
| `0 0 -1` | -Z축 기준 회전 |

SO-ARM101의 축 구조는 개념적으로 다음과 같이 구성될 수 있습니다.

| Joint | 예상 동작 |
| --- | --- |
| `shoulder_pan_joint` | 수평 방향 회전 |
| `shoulder_lift_joint` | 어깨 상승 및 하강 |
| `elbow_joint` | 팔꿈치 굽힘 |
| `wrist_flex_joint` | 손목 굽힘 |
| `wrist_roll_joint` | 손목 또는 Tool 회전 |
| `gripper_joint` | Gripper 열기 및 닫기 |

Axis를 단순히 로봇 전체의 X, Y, Z축으로 해석하면 안 됩니다. Joint Axis는 Joint Origin의 `rpy`가 적용된 Joint Frame을 기준으로 해석해야 합니다.

따라서 실제 축 방향은 다음 두 요소를 함께 확인해야 합니다.

- Joint Origin의 `rpy`
- Joint Axis의 `xyz`

Axis가 잘못 정의되면 다음과 같은 문제가 발생합니다.

- Joint가 예상과 다른 방향으로 회전함
- 양수와 음수 방향이 반대로 움직임
- Link가 비정상적인 방향으로 꺾임
- RViz2와 실제 로봇의 움직임이 다름
- MoveIt의 경로 계획 결과가 잘못됨

---

#### Limit 분석

Limit는 Joint의 움직임 범위와 최대 성능을 정의합니다.

SO-ARM101의 Shoulder Pan Joint가 다음과 같이 정의되어 있다고 가정해 보겠습니다.

```xml
<limit
    effort="10"
    velocity="10"
    lower="-1.91986"
    upper="1.91986"/>
```

각 속성의 의미는 다음과 같습니다.

| 속성 | 값 | 의미 |
| --- | --- | --- |
| `lower` | -1.91986 rad | 최소 Joint 위치 |
| `upper` | 1.91986 rad | 최대 Joint 위치 |
| `effort` | 10 | 최대 관절 토크 |
| `velocity` | 10 rad/s | 최대 관절 속도 |

라디안을 각도로 변환하는 공식은 다음과 같습니다.

$$
degree = radian \times \frac{180}{\pi}
$$

계산하면 다음과 같습니다.

$$
1.91986 \times \frac{180}{\pi} \approx 110^\circ
$$

따라서 Joint의 동작 범위는 다음과 같습니다.

```
약 -110° ~ +110°
```

여기서 양수와 음수 회전 방향은 Axis와 오른손 법칙에 따라 결정됩니다. 따라서 단순히 양수를 오른쪽, 음수를 왼쪽으로 고정해서 해석하면 안 됩니다.

RViz2에서 Joint를 움직이며 실제 방향을 확인하는 것이 가장 정확합니다.

Limit를 분석할 때는 다음 내용을 확인해야 합니다.

- 실제 기구의 회전 제한과 일치하는가?
- 서보 모터의 동작 범위와 일치하는가?
- 케이블이 꼬이지 않는 범위인가?
- 다른 Link와 충돌하지 않는 범위인가?
- 실제 Driver가 사용하는 범위와 일치하는가?
- MoveIt의 Joint Limit 설정과 일치하는가?

URDF의 Limit가 실제 하드웨어를 직접 보호해 주는 것은 아닙니다. 모터 드라이버와 제어 노드에서도 별도의 제한 처리가 필요합니다.

---

#### Visual 분석

Visual은 RViz2나 Gazebo 화면에 표시되는 외형을 정의합니다.

예를 들어 다음과 같이 STL 파일을 불러올 수 있습니다.

```xml
<visual>
    <origin xyz="0 0 0" rpy="0 0 0"/>

    <geometry>
        <mesh
            filename="assets/upper_arm.stl"/>
    </geometry>
</visual>
```

Mesh 경로는 다음 구조를 가집니다.

```
assets/파일_이름
```

위 경로는 `so_arm101_description` 패키지 안의 다음 파일을 의미합니다.

```
meshes/upper_arm.stl
```

Visual을 분석할 때는 다음 항목을 확인합니다.

- Mesh 파일이 실제로 존재하는가?
- 패키지 이름이 올바른가?
- Mesh 파일명이 Link 역할과 일치하는가?
- Mesh의 크기가 올바른가?
- Visual Origin의 위치가 올바른가?
- Visual Origin의 방향이 올바른가?
- RViz2에서 Link끼리 자연스럽게 연결되는가?

Mesh가 보이지 않으면 경로뿐 아니라 `setup.py` 또는 `CMakeLists.txt`에서 Mesh 파일이 설치 대상으로 등록되어 있는지도 확인해야 합니다.

---

#### Collision 분석

Collision은 Gazebo와 MoveIt에서 충돌 판정에 사용하는 형상입니다.

단순한 원통으로 정의한다면 다음과 같이 작성할 수 있습니다.

```xml
<collision>
    <origin xyz="0 0 0.075" rpy="0 0 0"/>

    <geometry>
        <cylinder radius="0.03" length="0.15"/>
    </geometry>
</collision>
```

또는 Visual과 같은 Mesh를 사용할 수도 있습니다.

```xml
<collision>
    <origin xyz="0 0 0" rpy="0 0 0"/>

    <geometry>
        <mesh
            filename="assets/upper_arm.stl"/>
    </geometry>
</collision>
```

Collision을 분석할 때는 다음 내용을 확인합니다.

- Collision이 Visual을 충분히 포함하는가?
- Collision이 Visual보다 지나치게 크지 않은가?
- Collision Origin과 Visual Origin이 적절히 정렬되었는가?
- Collision Mesh가 지나치게 복잡하지 않은가?
- 인접 Link끼리 기본 자세에서 과도하게 겹치지 않는가?
- MoveIt에서 불필요한 자기 충돌이 발생하지 않는가?

정교한 Mesh를 Collision에 사용하면 실제 형상과 잘 일치하지만 계산량이 증가할 수 있습니다. 시뮬레이션 성능이 중요하다면 단순한 Box, Cylinder 또는 단순화된 Collision Mesh를 사용하는 것이 좋습니다.

---

#### Inertial 분석

Inertial은 Gazebo에서 Link의 물리적 움직임을 계산하는 데 사용됩니다.

예를 들어 다음과 같이 질량이 정의되어 있다고 가정해 보겠습니다.

```xml
<mass value="0.147"/>
```

URDF의 질량 단위는 kg이므로 다음과 같이 변환할 수 있습니다.

0.147 kg=147 g0.147\,kg = 147\,g

0.147kg=147g

Inertial의 전체 구조는 다음과 같습니다.

```xml
<inertial>
    <origin
        xyz="0.0137179 -5.19711e-05 0.0334843"
        rpy="0 0 0"/>

    <mass value="0.147"/>

    <inertia
        ixx="..."
        ixy="..."
        ixz="..."
        iyy="..."
        iyz="..."
        izz="..."/>
</inertial>
```

Inertial을 분석할 때는 다음 항목을 확인합니다.

- 질량이 실제 Link 무게와 비슷한가?
- kg 단위가 올바르게 적용되었는가?
- 질량 중심 위치가 실제 구조와 비슷한가?
- 관성값의 단위가 kg·m²인가?
- 관성값이 0이거나 지나치게 작지 않은가?
- CAD에서 계산한 관성축과 URDF 좌표축이 일치하는가?
- 실제 재질의 밀도가 CAD 모델에 적용되어 있는가?

Link의 질량만 정확하고 질량 중심이나 관성 모멘트가 잘못되면 Gazebo에서 로봇이 흔들리거나 비정상적으로 움직일 수 있습니다.

---

#### TF Tree 분석

Link와 Joint 연결 관계를 이용하면 SO-ARM101의 TF Tree가 만들어집니다.

```
base_link
└── shoulder_link
    └── upper_arm_link
        └── forearm_link
            └── wrist_link
                └── tool_link
                    └── gripper_link
```

URDF와 `/joint_states`를 `robot_state_publisher`에 전달하면 각 Link의 Transform이 `/tf`와 `/tf_static`으로 발행됩니다.

```
URDF
  + /joint_states
          ↓
robot_state_publisher
          ↓
/tf, /tf_static
```

TF Tree에서는 다음 내용을 확인해야 합니다.

- 모든 Link가 하나의 Tree로 연결되어 있는가?
- Root Frame이 올바른가?
- Parent와 Child 관계가 올바른가?
- 움직이는 Joint가 `/joint_states`에 포함되어 있는가?
- Frame 위치와 축 방향이 실제 로봇과 일치하는가?
- `tool_link` 또는 End-Effector Frame이 올바른 위치에 있는가?

전체 구조는 다음 명령으로 확인할 수 있습니다.

```bash
ros2 run tf2_tools view_frames
```

두 Frame 사이의 좌표 관계는 다음과 같이 확인할 수 있습니다.

```bash
ros2 run tf2_ros tf2_echo base_link tool_link
```

---

#### End-Effector 확인

SO-ARM101에서 실제 작업 기준점은 `tool_link` 또는 Gripper 끝단에 별도로 정의한 Tool Frame이 될 수 있습니다.

```
wrist_link
└── tool_link
    └── gripper_link
        └── end_effector_frame
```

`tool_link`가 단순히 손목 회전 후의 연결부라면 물체를 잡는 정확한 위치를 나타내기 위해 별도의 End-Effector Frame을 추가할 수 있습니다.

```xml
<link name="end_effector_frame"/>

<joint name="end_effector_joint" type="fixed">
    <parent link="gripper_link"/>
    <child link="end_effector_frame"/>
    <origin xyz="0.1 0 0" rpy="0 0 0"/>
</joint>
```

MoveIt을 이용해 Pose 목표를 지정할 때는 일반적으로 이 End-Effector Frame을 기준으로 사용합니다.

따라서 다음 내용을 확인해야 합니다.

- End-Effector Frame이 실제 작업점에 있는가?
- Gripper 중심과 일치하는가?
- X, Y, Z축 방향이 작업 방향과 일치하는가?
- Pick & Place에서 사용할 접근 방향이 올바른가?

---

#### RViz2를 이용한 검증

URDF를 분석한 후에는 RViz2에서 실제 구조를 확인해야 합니다.

RViz2에서는 다음 내용을 확인할 수 있습니다.

- 로봇 외형
- Link 연결 상태
- Joint 움직임
- TF Frame 위치
- X, Y, Z축 방향
- Visual과 Collision 형상
- End-Effector Frame 위치

검증할 때는 다음 순서가 좋습니다.

1. `robot_state_publisher`를 실행합니다.
2. `joint_state_publisher_gui`를 실행합니다.
3. RViz2를 실행합니다.
4. `RobotModel` Display를 추가합니다.
5. `TF` Display를 추가합니다.
6. Fixed Frame을 `base_link`로 설정합니다.
7. 각 Joint를 하나씩 움직입니다.
8. 회전축과 방향을 확인합니다.
9. Visual과 Collision을 각각 확인합니다.
10. End-Effector Frame의 위치를 확인합니다.

Joint를 한 번에 여러 개 움직이면 어떤 설정이 잘못되었는지 파악하기 어렵습니다. 따라서 한 번에 하나의 Joint만 움직이면서 검증하는 것이 좋습니다.

---

#### Gazebo를 이용한 검증

RViz2에서 구조와 외형이 정상적으로 보인다면 Gazebo에서 물리 동작을 확인합니다.

Gazebo에서는 다음 내용을 검증해야 합니다.

- 로봇이 바닥에 안정적으로 고정되는가?
- Link의 질량이 현실적으로 적용되는가?
- 중력에 의해 Joint가 비정상적으로 내려가지 않는가?
- Collision이 Visual과 적절히 일치하는가?
- 관절이 Limit 범위 안에서 움직이는가?
- 로봇이 떨리거나 튀지 않는가?
- Controller 명령에 안정적으로 반응하는가?

RViz2에서 정상적으로 보이더라도 Gazebo에서 문제가 발생할 수 있습니다.

```
RViz2 검증
→ 구조, 외형, TF 확인

Gazebo 검증
→ Collision, Inertial, 중력, 물리 동작 확인
```

---

#### 실제 로봇과 비교

URDF 분석의 마지막 단계는 실제 SO-ARM101과 비교하는 것입니다.

다음 항목을 비교해야 합니다.

| 항목 | URDF | 실제 로봇 |
| --- | --- | --- |
| Joint 순서 | Parent·Child 연결 | 모터 배치 순서 |
| Axis | Joint 회전축 | 실제 모터 회전 방향 |
| Limit | 최소·최대 각도 | 기구적 동작 범위 |
| Visual | STL 외형 | 실제 프레임과 브래킷 |
| Mass | Link 질량 | 실제 부품 무게 |
| COM | Inertial Origin | 실제 무게중심 |
| End-Effector | Tool Frame | 실제 작업점 |

URDF와 실제 로봇이 다르면 다음과 같은 문제가 발생할 수 있습니다.

- RViz2와 실제 로봇의 자세가 다름
- Joint 명령 방향이 반대로 적용됨
- MoveIt이 잘못된 경로를 계획함
- Gazebo와 실제 로봇의 움직임이 다름
- Pick & Place 목표 위치에 도달하지 못함

---

#### SO-ARM101 URDF 분석표

SO-ARM101 URDF를 분석한 내용을 다음과 같이 정리할 수 있습니다.

| 분석 요소 | 확인 내용 |
| --- | --- |
| Link | 로봇을 구성하는 각 몸체 |
| Joint | Link 연결과 움직임 |
| Parent·Child | TF Tree 연결 구조 |
| Origin | 연결 위치와 방향 |
| Axis | Joint 회전 방향 |
| Limit | 최소·최대 위치, 최대 힘과 속도 |
| Visual | RViz2와 Gazebo에 표시할 외형 |
| Collision | 충돌 계산용 형상 |
| Inertial | 질량, 질량 중심, 관성 모멘트 |
| TF Tree | 전체 좌표계 연결 구조 |
| End-Effector | 실제 작업 기준점 |

---

#### 정리

SO-ARM101 URDF 분석은 지금까지 배운 내용을 실제 로봇 모델에 연결하는 과정입니다.

- Link를 보면 로봇의 몸체 구조를 이해할 수 있습니다.
- Joint를 보면 로봇의 움직임 구조를 이해할 수 있습니다.
- Parent와 Child를 보면 TF Tree를 구성할 수 있습니다.
- Origin을 보면 Link와 모델의 배치 위치를 이해할 수 있습니다.
- Axis를 보면 Joint의 회전 방향을 이해할 수 있습니다.
- Limit를 보면 Joint의 동작 범위와 성능 제한을 확인할 수 있습니다.
- Visual을 보면 화면에 표시되는 로봇 외형을 확인할 수 있습니다.
- Collision을 보면 충돌 판정에 사용하는 형상을 확인할 수 있습니다.
- Inertial을 보면 로봇의 질량과 물리 특성을 확인할 수 있습니다.
- TF Tree를 보면 로봇 전체의 좌표계 관계를 이해할 수 있습니다.
- End-Effector Frame을 보면 로봇의 실제 작업 기준점을 확인할 수 있습니다.

가장 중요한 것은 각각의 태그를 따로 보는 것이 아니라 서로 연결해서 이해하는 것입니다.

> **URDF는 Link와 Joint를 이용해 로봇의 구조를 정의하고, Visual·Collision·Inertial을 통해 외형과 물리 특성을 추가하며, 최종적으로 TF Tree를 구성합니다.**
> 

URDF 분석이 완료되면 RViz2에서 구조와 TF를 검증하고, Gazebo에서 Collision과 Inertial을 확인한 뒤 실제 SO-ARM101의 움직임과 비교해야 합니다. 이 과정을 거쳐야 MoveIt 경로 계획과 실제 로봇 제어에 사용할 수 있는 신뢰도 높은 로봇 모델을 만들 수 있습니다.
