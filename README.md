# lane detect and sift in gazebo

본 프로젝트는 Gazebo 시뮬레이터에서 동작하는 TurtleBot3 모델을 기반으로, ROS 2 (Humble) 환경에서 자율주행 기능을 구현한 시뮬레이션 프로젝트입니다.

노란선/흰선 차선 인식, 사람 인식 및 정지, 신호등/표지판 인식, IMU 기반 경사로 속도 제어 등의 기능을 포함하며, 사용자는 PyQt5 기반 GUI를 통해 카메라 영상 확인 및 제어가 가능합니다.

---

## 동작 영상

[![Video Label](http://img.youtube.com/vi/gTjKGNT-14w/0.jpg)](https://youtu.be/gTjKGNT-14w)

---

## 💻 사용 환경 및 개발 도구

| 장비 | 사양 |
|------|------|
| **PC** | RTX3050 탑재 ASUS 노트북 |
| **로봇 모델** |	Gazebo TurtleBot3 (burger_cam) 모델 |
| **운영체제** |	Ubuntu 22.04 LTS |
| **ROS 버전** |	ROS 2 Humble |
| **시뮬레이터** | Gazebo (TurtleBot3 World - 차선, 신호등, 표지판 포함) |

---
## 🧠 주요 기술 스택

**ROS 2 멀티노드 구조**

**컴퓨터 비전 처리** - OpenCV 활용
- 차선 인식: 색상 마스크 및 외곽선 검출
- 사람/표지판/신호등 인식: HSV 기반 색상 필터링, 윤곽선 기반 객체 감지
- SIFT (Scale-Invariant Feature Transform): 신호등 및 표지판 인식 정확도 향상에 활용

**IMU 기반 제어**

Gazebo 내 제공되는 IMU 센서 데이터를 활용하여 pitch 값 기반 경사로 인식

- 경사 구간에서 속도 동적 조절 수행

## 🖥️ 사용자 인터페이스 (UI)

PyQt5 기반 GUI 시스템

- 실시간 카메라 영상 표시
- 주요 상태 표시 및 미니맵 제어 버튼 제공
- 카메라 영상은 ROS 이미지 토픽을 받아 cv_bridge로 OpenCV 이미지 변환 후 표시

---

## 주요 기능 시나리오

1. 터틀봇이 신호등상태(빨간불, 노란불, 초록불)에 따라 정지, 서행, 정상 속도 주행 
2. 교차로 시작 표지판, 우회전 표지판, 교차로 탈출 표지판을 sift로 검출하여 교차로 주행 
3. 차선안에 사람이 있을경우 차량 정지
4. 차량의 pitch 기울기값에 따라 오르막길 고속주행, 내리막길 서행

<img width="1860" height="1188" alt="image" src="https://github.com/user-attachments/assets/b79a269c-be40-4e16-bdaf-e51696632a75" />

---

## 프로젝트 트리 구조

```
turtlebot3_ws/src/
├── [aruco_yolo]
├── [dynamic_obstacle_plugin]
├── DynamixelSDK
├── fsd_pkg
├── image_pipeline
├── ld08_driver
├── [pyqt_robot]
├── sample_pkg
├── turtlebot3
│   ├── turtlebot3
│   ├── turtlebot3_bringup
│   ├── turtlebot3_cartographer
│   ├── turtlebot3_description
│   ├── turtlebot3_example
│   ├── turtlebot3_navigation2
│   ├── turtlebot3_node
│   └── turtlebot3_teleop
├── turtlebot3_autorace
│   ├── turtlebot3_autorace
│   ├── [turtlebot3_autorace_camera]
│   ├── [turtlebot3_autorace_detect]
│   └── [turtlebot3_autorace_mission]
├── turtlebot3_manipulation
│   ├── turtlebot3_manipulation
│   ├── turtlebot3_manipulation_bringup
│   ├── turtlebot3_manipulation_description
│   ├── turtlebot3_manipulation_hardware
│   ├── turtlebot3_manipulation_moveit_config
│   └── turtlebot3_manipulation_teleop
├── turtlebot3_msgs
├── turtlebot3_simulations
│   ├── turtlebot3_fake_node
│   ├── [turtlebot3_gazebo]
│   ├── turtlebot3_manipulation_gazebo
│   └── turtlebot3_simulations
├── turtlebot_cosmo_interface
├── turtlebot_moveit
│   ├── manipulator_moveit
│   └── turtlebot_moveit
└── vision_opencv
    ├── cv_bridge
    ├── image_geometry
    ├── opencv_tests
    └── vision_opencv
```

**prerequirement**

[robotis turtlebot3 emanual](https://emanual.robotis.com/docs/en/platform/turtlebot3/quick-start/)

3.Quick start guide, 6. Simulation, 8. Autonomous driving 참고하면서 가제보월드(turtlebot3_autorace_2020)상에서 자율주행까지 진행

**설치방법**

project_turtlebot3_autorace_simulation.zip 을 다운받고 위 트리구조에서 [package]처럼 대괄호 처리된 폴더 추가 및 교체하고 빌드

```bash
cd turtlebot3_ws/
colcon build
```
---

## ROS2 노드 구성 (요약)

- **robot_state_publisher(gazebo world)**
- **intrinsic camera calibration**
- **extrinsic camera calibration**
- **detect_lane**
- **detect_trafficlight**
- **detect_intersection**
- **detect_person**
- **control_lane**
- **pyqt_robot**

<img width="1294" height="869" alt="image" src="https://github.com/user-attachments/assets/26914b54-c3f3-45c9-b1a6-8d113a1bd4b3" />


---
## 1. gazebo world

**터미널1 gazebo world 노드 실행**
```bash
ros2 launch turtlebot3_gazebo turtlebot3_autorace_2020.launch.py
```
---
## 2. intrinsic camera calibration

이미지 전처리 노드

<img width="320" alt="image" src="https://github.com/user-attachments/assets/1d7b2ca0-2e56-4eee-b2fa-e5d981a2a23b" />

**터미널2 realsense 노드 실행**
```bash
ros2 launch turtlebot3_autorace_camera intrinsic_camera_calibration.launch.py
```

---

## 3. extrinsic camera calibration

lane detect용 -z 방향으로 카메라 projection

<img width="320" alt="image" src="https://github.com/user-attachments/assets/a0ba9e92-861a-42fa-b366-0279c98e8d3a" />


**터미널3 get_keyword 노드 실행**
```bash
ros2 launch turtlebot3_autorace_camera extrinsic_camera_calibration.launch.py
```
---

## 4. detect_lane

<img width="600" alt="image" src="https://github.com/user-attachments/assets/bc16b96a-5c52-41dd-9e2d-b4c85a1b1247" />


DetectLane 노드는 카메라 영상에서 HSV 색상 필터링을 통해 흰색과 노란색 차선을 실시간으로 검출합니다. 2차 다항식 곡선 피팅과 이동평균을 활용해 부드럽게 차선을 추적하며, 차선 상태와 중심 좌표를 퍼블리시하여 로봇이나 차량의 주행 제어에 활용할 수 있도록 돕는 ROS2 노드입니다.

주요 기능
1. 파라미터 선언 및 초기화
- HSV 색공간에서 흰색과 노란색 차선을 구분하기 위한 색상 범위(hue, saturation, value)를 파라미터로 선언
- 범위 유효성 검사 (Hue: 0179, Saturation/Value: 0255) 수행
- 실행 중 파라미터 값을 동적으로 변경 가능 (캘리브레이션 모드에서 실시간 조정 지원)

2. 이미지 구독 및 발행
- 원본 카메라 영상 구독 (raw 이미지 또는 압축 이미지 선택 가능)
- 차선 검출 후 결과 영상 및 차선 중심 좌표 등 다양한 결과를 토픽으로 발행
- 캘리브레이션 모드에서는 흰색/노란색 차선 마스크 영상도 별도 발행

3. 차선 마스크 생성
- 입력된 BGR 이미지를 HSV로 변환 후, 흰색과 노란색 차선에 해당하는 픽셀만 마스크 생성 (cv2.inRange 사용)
- 마스크 내 픽셀 수(fraction)를 계산해 차선 검출 정도 판단
- 명도(Value) 값을 자동으로 조절하여 과다 혹은 과소 검출 시 동적으로 보정
- 차선 신뢰도(길이 등)를 계산해 유효성 확인

4. 차선 곡선 추정
- 차선 마스크 이미지에서 픽셀 좌표 추출 후, 2차 다항식(포물선) 곡선 피팅(np.polyfit)
- 피팅 실패 시에는 느리지만 확실한 sliding window 방식으로 다시 추정
- 최근 프레임들의 차선 계수를 이동평균하여 부드러운 추적 유지

5. 차선 중심 및 상태 계산
- 좌측(노란색)과 우측(흰색) 차선 곡선을 기반으로 중앙선 계산
- 차선 검출 상태(양쪽 모두, 한쪽만, 없음)를 판단해 상태 값 발행
- 차선 위에 검출된 곡선을 영상에 시각적으로 표시 (빨강: 노란차선, 노랑: 흰차선, 녹색: 차선 내부 영역)
- 차선 중심 좌표를 실시간으로 퍼블리시하여 주행 제어에서 참조 가능

6. 퍼포먼스 및 프레임률 제어
- 전체 입력 프레임 중 약 3분의 1만 처리해 과부하 방지 (예: 10fps 입력 시 약 3.3fps 처리)

**터미널4 robot_control 노드 실행**
```bash
ros2 launch turtlebot3_autorace_detect detect_lane.launch.py
```

---
# 5. detect_traffic_light

<img width="600" alt="image" src="https://github.com/user-attachments/assets/d9b9bba6-c510-436f-adbf-bc326ab3764d" />


detect_traffic_light 노드는 ROS2 환경에서 카메라 영상을 구독해 신호등의 빨강, 노랑, 초록 불빛을 실시간으로 검출하는 노드입니다. HSV 색상 필터링과 SIFT 특징점 매칭을 결합하여 신호등 색상을 정확하게 인식하고, 검출 결과 영상과 신호등 색상 상태를 퍼블리시합니다.

주요 기능
1. 이미지 구독 및 발행
- 원본 카메라 영상 구독 (raw 또는 compressed 이미지 선택 가능)
- 신호등 검출 결과 영상 퍼블리시 (raw 또는 compressed)
- 검출된 신호등 색상 상태(‘red’, ‘yellow’, ‘green’, ‘none’)를 /detect/traffic_light 토픽으로 퍼블리시

2. 신호등 색상 검출
- ROI(관심 영역)를 지정하여 불필요한 영역 제외
- HSV 색공간 기반 엄격한 색상 범위 필터링으로 빨강, 노랑, 초록색 영역 검출
- 색상별 픽셀 비율 임계값을 초과할 경우 해당 색상을 후보로 판단

3. 특징점 기반 SIFT 매칭
- 각 신호등 색상별 템플릿 이미지에서 SIFT 특징점 미리 추출 및 저장
- 검출 후보 영역에서 SIFT 특징점 추출 후 템플릿과 FLANN 매칭 수행
- 일정 개수 이상의 좋은 매칭과 MSE 기반 유사도 검사로 검출 신뢰도 평가

4. 결과 영상 처리
- 신호등 색상 검출 시 템플릿과 매칭 결과를 그려 결과 영상 생성
- 색상 미검출 시 ROI 영역 영상 그대로 발행

5. 퍼포먼스 제어
- 프레임 처리 속도 제한(3프레임마다 1회 처리)으로 연산 부하 완화

---

# 6. detect_intersection

<img width="600" alt="image" src="https://github.com/user-attachments/assets/408be2f0-dcae-4a37-9dd1-d0a05b669cbd" />


detect_intersection 노드는 ROS2 환경에서 카메라 영상을 구독해 교차로, 교차로 탈출, 우회전 표지판을 실시간으로 검출하는 노드입니다. SIFT 특징점 검출과 FLANN 매칭을 활용하여 표지판의 위치와 상태를 인식하고, 검출 결과 영상과 표지판 종류를 퍼블리시합니다.

주요 기능
1. 이미지 구독 및 발행
- 원본 카메라 영상 구독 (raw 또는 compressed 이미지 선택 가능)
- 검출 결과 영상 퍼블리시 (raw 또는 compressed 이미지 선택 가능)
- 검출된 표지판 종류("intersection", "intersection_escape", "right")를 /detect/traffic_sign 토픽으로 문자열(String) 형태로 퍼블리시

2. 표지판 이미지 전처리 및 특징점 추출
- 교차로, 교차로 탈출, 우회전 표지판 템플릿 이미지를 불러와 그레이스케일 변환 및 비율 유지 리사이즈 수행
- 각 템플릿 이미지에 대해 SIFT 특징점과 디스크립터를 미리 추출하여 저장

3. 실시간 영상 내 특징점 검출 및 매칭
- 영상에서 하단 25% 영역을 제외한 관심 영역(상단 75%)에서 SIFT 특징점 검출
- 템플릿별로 FLANN 매칭 수행, 매칭 품질(거리비교)로 좋은 매칭 후보 선정
- 매칭된 후보들의 위치와 크기를 기반으로 표지판 존재 여부 판단

4. 검출 신뢰도 평가 및 필터링
- 매칭 점수가 일정 기준 이상인지 확인 (MIN_MATCH_COUNT)
- 매칭된 영역 크기(면적)로 너무 작거나 큰 영역 필터링
- 표지판 위치 조건 (예: 교차로 표지판은 화면 오른쪽 1/3 영역 내) 만족 여부 체크
- 호모그래피 기반 위치 정합 및 MSE 계산으로 검출 신뢰도 평가

5. 검출 결과 퍼블리시 및 시각화
- 검출 성공 시 표지판 종류 문자열 퍼블리시
- 검출 결과와 매칭점이 표시된 영상 퍼블리시
- 검출 실패 시 원본 영상 그대로 퍼블리시

6. 연산량 절감
- 프레임 스킵 처리로 3프레임 중 1프레임만 처리하여 실시간 성능 유지


---
# 7. detect_person

<img width="600" alt="image" src="https://github.com/user-attachments/assets/ab686f9a-24f4-4228-b8da-831df71e6d4e" />


detect_person 노드는 ROS2 환경에서 카메라 영상을 구독해 움직이는 영역 내 사람을 실시간으로 검출하는 노드입니다. 배경 차분(Background Subtraction)과 색상 필터링, HOG 기반 사람 검출 알고리즘을 활용하여 사람 존재 여부를 판단하고, 검출 결과 영상과 함께 사람 검출 신호를 퍼블리시합니다.

주요 기능
1. 이미지 구독 및 발행
- 원본 카메라 영상 구독 (raw 또는 compressed 이미지 선택 가능)
- 사람 검출 결과 영상 퍼블리시 (raw 또는 compressed 이미지 선택 가능)
- 사람 검출 유무를 UInt8 타입으로 /camera/person_detected 토픽에 퍼블리시 (1: 감지, 0: 미감지)
2. 사람 검출 전처리 및 알고리즘
- 배경 차분 알고리즘(MOG2)으로 움직이는 영역 추출
- HSV 색공간에서 특정 보라색 영역 필터링 및 컨투어(외곽선) 분석으로 후보 영역 선정
- HOGDescriptor를 활용한 사람 검출 모델 초기화 (현재는 선언만, 실제 HOG 검출 함수는 미사용)
3. 차선 정보 연동
- /detect/lane, /detect/lane_state, /lane/x_bounds 토픽을 구독하여 현재 차선 위치 및 상태 정보를 수신
- 사람 위치가 차선 범위 내에 존재하는지 필터링 적용
4. 사람 위치 및 크기 조건 검사
- 검출된 후보 컨투어의 면적이 임계값 이상인지 확인
- 차선 범위를 벗어나는 후보 영역 무시
- 차선 상태가 정상일 때만 사람 감지 결과 인정
- 사람 중심 좌표가 적절한 중앙 영역(70~280 픽셀)에 위치해야 검출로 판단
5. 결과 퍼블리시 및 시각화
- 사람 감지 유무를 한 번만 퍼블리시하여 연속 신호 발행 방지
- 감지 시 검출 박스 표시한 영상 퍼블리시
- 미감지 시 원본 영상 그대로 퍼블리시
6. 연산 효율화
- 프레임 스킵 처리로 3프레임 중 1프레임만 처리하여 실시간 연산 부담 완화

**터미널5 detect_sign.launch.py 런치 파일 실행**
```bash
ros2 launch turtlebot3_autorace_detect detect_sign.launch.py 
```
---
# 8. control_lane

control_lane 노드는 ROS2 환경에서 다양한 센서와 인식 결과를 구독하여 로봇의 주행 상태를 판단하고, 차선 추종 및 신호등/사람 감지에 따른 주행 제어를 수행하는 주행 제어 노드입니다.

주요 기능
1. 토픽 구독 (센서 및 상태 입력)
- 차선 상태(/detect/lane_state), 차선 중심 위치(/control/lane), 흰색 차선 중심(/detect/white_center)
- 교통 표지판(/detect/traffic_sign), 신호등 상태(/detect/traffic_light 및 /traffic_light_state)
- 사람 감지(/camera/person_detected)
- 최대 속도 설정(/control/max_vel)
- 로봇 위치 및 자세 정보: 오도메트리(/odom), IMU 센서(/imu)
2. 토픽 발행 (제어 신호 및 디버그 정보)
- 속도 제어 명령: /control/cmd_vel (Twist 메시지), /control/linear_vel (선형 속도)
- 디버깅 정보: /debug/control_state (Vector3 메시지, PID 제어 파라미터 및 IMU pitch 정보)
3. 주행 상태 관리 및 이벤트 처리
- 로봇 상태 RobotState Enum 으로 정의 (정상 주행, 사람 감지 정지, 빨간불 정지, 노란불 감속)
- 사람 감지 시 2초간 정지 상태 유지
- 빨간불 감지 시 즉시 정지, 노란불 감지 시 감속 주행
- 신호등 감지 후 2초 이내 유지 기능 (끊기면 상태 자동 해제)
- 교차로 진입, 우회전, 교차로 탈출 등 복합 이벤트 처리 및 주행 모드 전환
4. 주행 제어 루프 (0.1초 주기)
- 현재 상태에 따라 정지, 감속, 정상 주행 등 동작 분기
- 차선 중심 오차를 PID 제어로 보정하여 각속도와 선속도 계산
- IMU pitch를 이용한 속도 보정 (오르막/내리막 감지)
- 교차로 탈출 시 흰색 차선 추종 기능 포함
- 회전 명령 처리 및 시간 경과에 따른 회전 종료 처리
5. IMU 데이터 처리 및 저역통과 필터 적용
- IMU 쿼터니언 → Euler 각 변환 후 pitch 값 추출
- 초기 10 프레임 필터 안정화 및 이후 필터 적용
- pitch 변화에 따른 속도 오프셋 보정 및 로그 출력

6. 기타 기능
- 오도메트리로부터 yaw(방향) 추출
- 안전 속도 제한 및 속도 변동 최소화
- 종료 시 cmd_vel 0 발행으로 안전 정지


**터미널6 control_lane 노드 실행**
```bash
ros2 launch turtlebot3_autorace_mission control_lane.launch.py 
```

---

# 9. pyqt_robot

- ROS2 토픽에서 로봇 위치(오도메트리/TF), 카메라 영상, 속도, 제어 상태, 신호등 상태, 사람 감지 정보를 구독
- PyQt5를 활용해 2D 맵 위에 로봇 위치 및 방향 표시
- 카메라 영상 실시간 표시
- 속도 및 제어 상태(angular_z, error, pitch) 실시간 그래프 표시
- 신호등 상태 및 사람 감지 메시지 UI 표시
- 지도와 로봇 아이콘의 위치를 두 가지 모드(지도 고정 / 로봇 고정)로 보여줌

<img src="https://github.com/user-attachments/assets/b0ee48bb-22ae-474f-a91e-ff185bab4a2e" width="1280"/>

주요 기능
1. ROS2 토픽 구독 및 데이터 수신
- 로봇 위치 및 자세: /tf (TFMessage) → Odometry 변환 후 좌표(x, y)와 yaw 추출
- 카메라 원본 영상: /camera/image_raw (sensor_msgs/Image)
- 로봇 선형 속도: /control/linear_vel (Float64)
- 제어 상태 (각속도, 오차, pitch): /debug/control_state (Vector3)
- 신호등 상태: /detect/traffic_light (String)
- 사람 감지: /camera/person_detected (UInt8)
2. PyQt5 기반 GUI 구성
- 2D 지도(맵) 위에 리사이즈 및 회전된 맵 이미지 표시
- 로봇 아이콘 위치 및 방향 실시간 업데이트 (맵 고정 모드 / 로봇 고정 모드 토글 가능)
- 카메라 영상 실시간 표시 (QLabel 위젯)
- 속도, 각속도, 오차, pitch 값 실시간 그래프 표시 (pyqtgraph)
3. 신호등 상태 및 사람 감지 상태 텍스트 알림 라벨
- 맵 좌표계 및 변환 처리
 - 실제 미터 좌표 → 픽셀 좌표 변환
- 맵 중심 고정 / 로봇 중심 고정 모드 지원
- 로봇 방향에 맞춘 맵 회전 처리
4. UI 및 사용자 인터랙션
- 모드 토글 버튼 (맵 고정 ↔ 로봇 고정)
- 윈도우 크기 조절 시 UI 컴포넌트 재배치 및 뷰 리셋
- 타이머 기반 ROS spin 및 UI 업데이트
5. 실행 및 초기화
- 맵 이미지 4m x 4m 크기로 리사이즈 및 180도 회전 후 저장
- ROS2 초기화 및 노드 생성
- PyQt 앱 실행 및 메인 윈도우 표시
6. 기타
- 데이터 수신 시 PyQt 시그널을 통해 안전하게 GUI 스레드에서 처리
- 오류 발생 시 로그 출력

**터미널7 pyqt robot 노드 실행**
```bash
ros2 run pyqt_robot pyqt_robot_2880x1620 
# 해상도에 따라 pyqt_robot_1920x1080 실행
```


## 의존 라이브러리
```bash
pip install cv_bridge PyQt5 pyqtgraph opencv-python Pillow ament_index_python
```

---
## 팀 소개

**TEAM C-4조**  
- 이희우  
- 정석환  
- 김동건  
- 김재훈

---

## 라이선스

본 프로젝트는 연구 및 학습 목적으로 개발되었으며, 상업적 사용을 금합니다.

---

