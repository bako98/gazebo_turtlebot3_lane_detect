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

**터미널2 realsense 노드 실행**
```bash
ros2 launch turtlebot3_autorace_camera intrinsic_camera_calibration.launch.py
```

---

## 3. extrinsic camera calibration

lane detect용 -z 방향으로 카메라 projection


**터미널3 get_keyword 노드 실행**
```bash
ros2 launch turtlebot3_autorace_camera extrinsic_camera_calibration.launch.py
```
---

## 4. detect_lane

DetectLane 노드는 카메라 영상에서 HSV 색상 필터링을 통해 흰색과 노란색 차선을 실시간으로 검출합니다. 

2차 다항식 곡선 피팅과 이동평균을 활용해 부드럽게 차선을 추적하며, 차선 상태와 중심 좌표를 퍼블리시하여 로봇이나 차량의 주행 제어에 활용할 수 있도록 돕는 ROS2 노드입니다.

주요 기능
1. 파라미터 선언 및 초기화

HSV 색공간에서 흰색과 노란색 차선을 구분하기 위한 색상 범위(hue, saturation, value)를 파라미터로 선언

범위 유효성 검사 (Hue: 0179, Saturation/Value: 0255) 수행

실행 중 파라미터 값을 동적으로 변경 가능 (캘리브레이션 모드에서 실시간 조정 지원)

2. 이미지 구독 및 발행

원본 카메라 영상 구독 (raw 이미지 또는 압축 이미지 선택 가능)

차선 검출 후 결과 영상 및 차선 중심 좌표 등 다양한 결과를 토픽으로 발행

캘리브레이션 모드에서는 흰색/노란색 차선 마스크 영상도 별도 발행

3. 차선 마스크 생성

입력된 BGR 이미지를 HSV로 변환 후, 흰색과 노란색 차선에 해당하는 픽셀만 마스크 생성 (cv2.inRange 사용)

마스크 내 픽셀 수(fraction)를 계산해 차선 검출 정도 판단

명도(Value) 값을 자동으로 조절하여 과다 혹은 과소 검출 시 동적으로 보정

차선 신뢰도(길이 등)를 계산해 유효성 확인

4. 차선 곡선 추정

차선 마스크 이미지에서 픽셀 좌표 추출 후, 2차 다항식(포물선) 곡선 피팅(np.polyfit)

피팅 실패 시에는 느리지만 확실한 sliding window 방식으로 다시 추정

최근 프레임들의 차선 계수를 이동평균하여 부드러운 추적 유지

5. 차선 중심 및 상태 계산

좌측(노란색)과 우측(흰색) 차선 곡선을 기반으로 중앙선 계산

차선 검출 상태(양쪽 모두, 한쪽만, 없음)를 판단해 상태 값 발행

차선 위에 검출된 곡선을 영상에 시각적으로 표시 (빨강: 노란차선, 노랑: 흰차선, 녹색: 차선 내부 영역)

차선 중심 좌표를 실시간으로 퍼블리시하여 주행 제어에서 참조 가능

6. 퍼포먼스 및 프레임률 제어

전체 입력 프레임 중 약 3분의 1만 처리해 과부하 방지 (예: 10fps 입력 시 약 3.3fps 처리)

**터미널4 robot_control 노드 실행**
```bash
ros2 launch turtlebot3_autorace_detect detect_lane.launch.py
```

---
# 5. detect_traffic_light

detect_traffic_light 노드는 ROS2 환경에서 카메라 영상을 구독해 신호등의 빨강, 노랑, 초록 불빛을 실시간으로 검출하는 노드입니다. HSV 색상 필터링과 SIFT 특징점 매칭을 결합하여 신호등 색상을 정확하게 인식하고, 검출 결과 영상과 신호등 색상 상태를 퍼블리시합니다.

주요 기능
1. 이미지 구독 및 발행

원본 카메라 영상 구독 (raw 또는 compressed 이미지 선택 가능)

신호등 검출 결과 영상 퍼블리시 (raw 또는 compressed)

검출된 신호등 색상 상태(‘red’, ‘yellow’, ‘green’, ‘none’)를 /detect/traffic_light 토픽으로 퍼블리시

2. 신호등 색상 검출

ROI(관심 영역)를 지정하여 불필요한 영역 제외

HSV 색공간 기반 엄격한 색상 범위 필터링으로 빨강, 노랑, 초록색 영역 검출

색상별 픽셀 비율 임계값을 초과할 경우 해당 색상을 후보로 판단

3. 특징점 기반 SIFT 매칭

각 신호등 색상별 템플릿 이미지에서 SIFT 특징점 미리 추출 및 저장

검출 후보 영역에서 SIFT 특징점 추출 후 템플릿과 FLANN 매칭 수행

일정 개수 이상의 좋은 매칭과 MSE 기반 유사도 검사로 검출 신뢰도 평가

4. 결과 영상 처리

신호등 색상 검출 시 템플릿과 매칭 결과를 그려 결과 영상 생성

색상 미검출 시 ROI 영역 영상 그대로 발행

5. 퍼포먼스 제어

프레임 처리 속도 제한(3프레임마다 1회 처리)으로 연산 부하 완화

---

# 6. detect_intersection

이 노드는 Realsense RGB 및 Depth 영상을 기반으로 의료 도구(예: 메스)를 인식하고, 해당 객체의 3D 위치 및 기울기(theta)를 계산하여 `robot_control` 노드에 제공합니다. 또한, 객체 내부의 검은 선(예: 테이프)을 검출하여 정밀한 위치/방향 추정을 수행합니다.


## 주요 기능

- YOLO 모델을 통한 객체 탐지
- 검출된 객체의 중심점 또는 검은선 중심점을 기준으로 3D 위치 추출
- `robot_control` 노드에 `/get_3d_position` 서비스로 위치와 각도 제공
- 검은 선 기울기 추정 (`theta`)
- `Detection2DArray` 메시지로 인식 결과 퍼블리시



## 사용 토픽 및 서비스

| 유형 | 이름 | 메시지 타입 | 설명 |
|-----|----|----|------|
| 구독 | /camera/camera/color/image_raw | sensor_msgs/Image | RGB 이미지 |
| 구독 | /camera/camera/ aligned_depth_to_color/image_raw | sensor_msgs/Image | 깊이 이미지 |
| 구독 | /camera/camera/color/camera_info | sensor_msgs/CameraInfo | 카메라 내부 파라미터 |
| 서비스서버 | /get_3d_position | hospital_interfaces/ srv/DepthAnglePos | 객체 3D 위치 및 각도 요청 처리 |
| 퍼블리시 | /detection_result | vision_msgs/Detection2DArray | 객체 바운딩 박스 및 클래스 퍼블리시 |




## 주요 로직 요약

### 1. 객체 인식 및 바운딩 박스 추출
- `YOLOv8n` 모델을 통해 지정된 클래스를 탐지
- 신뢰도가 가장 높은 바운딩 박스를 사용

### 2. 검은 선 검출 (중심점 + 기울기)
- 바운딩 박스 내부에서 `cv2.threshold()`로 마스킹
- `cv2.findContours()`로 윤곽선 추출
- `cv2.minAreaRect()`로 중심점 및 각도 계산

### 3. 3D 위치 추출
- 중심점 또는 검은 선 중심점의 픽셀 좌표로부터 깊이(z) 획득
- 카메라 내부 파라미터를 활용한 카메라 좌표계 변환

### 4. 서비스 응답
- 위치: `(x, y, z)`
- 각도: `theta` (검은선 기울기)
- 실패 시 기본값 반환



**터미널6 detection 노드 실행**
```bash
cd robokrates_ws/
ros2 run hospital object_detection
```


## 참고
검은 선이 감지되지 않을 경우 중심점 기준으로 위치 및 기울기 0으로 반환
대상 클래스가 scalpel일 경우 영상 밝기 보정 적용

---
# 7. tracking_detection 노드

이 노드는 RealSense 카메라로부터 RGB/Depth 이미지를 받아 의료 도구(`scalpel_tip`)를 실시간으로 탐지하고, **DeepSORT** 알고리즘으로 객체를 추적하여 **3D 위치 좌표**를 계산 및 발행합니다. 감지된 객체의 바운딩 박스와 클래스 정보도 detection_manager 노드 등으로 퍼블리시됩니다.



## 주요 기능

- RealSense 카메라의 RGB, Depth, Camera Info 토픽 구독
- `YOLO` 모델을 통해 `scalpel_tip` 객체 탐지
- `DeepSORT`로 `scalpel_tip` 지속적 추적
- 객체의 **3D 위치 ([x, y, z])** 계산 및 발행
- `/scalpel_result` 토픽으로 추적된 scalpel 결과 발행
- `/general_result` 토픽으로 일반 객체 탐지 결과 발행
- `/tracked_objects_3d` 토픽으로 추적된 scalpel_tip의 3D 좌표 발행


## 토픽 및 메시지

| 구분 | 토픽명 | 메시지 타입 | 설명 |
|------|-----|----|------|
| 구독 | `/camera/camera/color/image_raw` | `sensor_msgs/Image` | RGB 영상 입력 |
| 구독 | `/camera/camera/ aligned_depth_to_color/image_raw` | `sensor_msgs/Image` | Depth 영상 입력 |
| 구독 | `/camera/camera/color/camera_info` | `sensor_msgs/CameraInfo` | 카메라 내부 파라미터 |
| 발행 | `/scalpel_result` | `vision_msgs/Detection2DArray` | 추적된 scalpel_tip 바운딩 박스 결과 |
| 발행 | `/general_result` | `vision_msgs/Detection2DArray` | 일반 YOLO 탐지 결과 |
| 발행 | `/tracked_objects_3d` | `std_msgs/Float32MultiArray` | 추적된 scalpel_tip의 [track_id, class_id, x, y, z] 좌표 목록 |


## 동작 방식 요약

### 1. 초기화
- `YOLO` 모델 (scalpel 전용 + 일반) 로딩
- RealSense 이미지/깊이 노드 초기화
- DeepSORT 추적기 구성

### 2. 객체 감지 및 추적
- `YOLOv8n`로 `scalpel_tip` 감지
- 신뢰도 0.6 이상만 필터링
- DeepSORT를 통해 트랙 유지 및 ID 할당

### 3. 3D 위치 계산
- 바운딩 박스 중심점 픽셀 좌표에서 깊이 추출
- 카메라 내파라미터를 이용해 실세계 3D 좌표 변환
- `/tracked_objects_3d` 토픽으로 전송

### 4. 시각화 결과 발행
- `/scalpel_result`: 추적된 scalpel_tip 객체 정보
- `/general_result`: 일반 YOLO 탐지 결과

---

**터미널7 tracking_detection 노드 실행**
```bash
cd robokrates_ws/
ros2 run hospital tracking_detection
```

# 8. detection_manager 노드

이 노드는 RealSense RGB 영상과 객체 탐지 결과(`/scalpel_result`, `/general_result`)를 받아 OpenCV 창 및 웹 클라이언트에 실시간으로 시각화하고, 선택된 객체 정보를 처리하는 인터페이스 역할을 수행합니다.



## 주요 기능

- RealSense RGB 이미지 토픽 구독 (`/camera/camera/color/image_raw`)
- `tracking_detection` 노드의 YOLO 탐지 결과 수신
  - `/scalpel_result`: 의료 도구 `scalpel_tip` 추적 결과
  - `/general_result`: 일반 객체 탐지 결과
- 객체 바운딩 박스 시각화 (OpenCV)
- SocketIO를 통해 웹에 이미지 및 탐지 결과 전송
- 웹에서 객체 선택 클릭 이벤트 수신 (`pick_object`)
- 선택된 객체에 대한 강조 시각화 처리



## 사용 토픽 및 이벤트

### 구독 토픽

| 토픽명 | 메시지 타입 | 설명 |
|------|------|------|
| `/camera/camera/color/image_raw` | `sensor_msgs/Image` | RealSense RGB 이미지 |
| `/scalpel_result` | `vision_msgs/Detection2DArray` | Scalpel 객체 탐지 결과 |
| `/general_result` | `vision_msgs/Detection2DArray` | 일반 객체 탐지 결과 |

### 웹 Emit 이벤트

| 이벤트명 | 설명 |
|-------|------|
| `binary_frame` | 웹으로 전송되는 JPEG 이미지 프레임 (Base64 인코딩) |
| `detection_list` | 현재 추적된 객체 리스트 (label, class_id, 신뢰도 포함) |

### 웹 수신 이벤트

| 이벤트명 | 설명 |
|-------|------|
| `pick_object` | 웹 UI에서 선택된 객체 전달 (label + raw_id) |


## 시각화 예시

- `scalpel_tip`: 하늘색 박스 (`/scalpel_result`)
- 일반 객체: 초록색 박스 (`/general_result`)
- 선택된 객체: 파란색 강조


## 동작 흐름

1. RealSense RGB 이미지 수신 → OpenCV 이미지 변환
2. YOLO 결과 (`/scalpel_result`, `/general_result`)를 받아 시각화
3. 바운딩 박스와 클래스 정보를 이미지에 그리기
4. 웹으로:
   - 실시간 영상 프레임 전송
   - 탐지된 객체 리스트 전송
5. 웹에서 선택된 객체(raw_id)를 다시 수신해 강조 시각화


**터미널8 detection_manager 노드 실행**
```bash
cd robokrates_ws/
ros2 run hospital detection_manager
```

---

# 9. Flask Server 

이 Flask 서버는 DICOM 의료 영상의 시각화와 실시간 객체 탐지 정보의 SocketIO 통신을 동시에 제공하는 통합 웹 서버입니다. 수술 현장의 데이터 시각화, 음성 출력, 명령 수신, 객체 선택 등을 브라우저와 연동하여 처리할 수 있습니다.

<img src="https://github.com/user-attachments/assets/b0ee48bb-22ae-474f-a91e-ff185bab4a2e" width="1280"/>

## 주요 기능

### DICOM 뷰어
- `dicom_output/` 폴더에서 `.dcm` 파일 자동 로드
- DICOM → PNG로 변환하여 웹 브라우저에 표시
- 환자 정보(PatientName, 성별, 나이, 종 등) 메타데이터 표시

### SocketIO 실시간 통신
- 웹 → 서버: 명령어 전송 (`keyword_text`, `pick_object`)
- 서버 → 웹: 객체 선택 반영 (`selection_confirmed`), 탐지 결과 공유 (`detection_list`)
- 웹 ↔ Python Client: `binary_frame` 및 객체 리스트 상호 전송

### gTTS 음성 출력

- 환자 메타데이터를 텍스트로 구성해 `gTTS`로 음성 출력
- `info` 이벤트로 트리거됨
- 웹 클라이언트에 텍스트 결과 전송



**터미널9 flask 서버 실헹**
```bash
cd robokrates_ws/flask_hospital/
python3 flask_server_fin.py
```

## 의존 라이브러리
```bash
pip install flask flask_socketio gtts playsound pydicom pillow langchain sounddevice openwakeword
```

### SocketIO 이벤트 정리
### 웹에서 서버로
|이벤트명	|설명|
|--------|--------|
|keyword_text	|명령어(JSON) 수신: object, target, commands|
|info	|현재 DICOM 파일 정보 → 음성 출력 요청|
|pick_object	|객체 선택 (label, raw_id)|
|binary_frame	|탐지 이미지 프레임 전달|
|detection_list	|탐지된 객체 리스트 전달|

### 서버에서 웹으로
|이벤트명	|설명|
|--------|--------|
|spoken_text	|음성으로 출력된 텍스트 반환|
|selection_confirmed	|선택된 객체 ID 및 label 확인|
|binary_frame	|실시간 이미지 스트림|
|detection_list	|객체 리스트 공유|
|pick_object	|Python 클라이언트에서 전파된 선택|

### 동작 예시 흐름
- 웹에서 DICOM 뷰어 접근 → .dcm 이미지 변환 후 표시
- /info 요청 → 환자 이름/성별/나이를 음성으로 출력 (gTTS)
- 웹에서 객체 선택 → pick_object SocketIO 이벤트 발생
- Python ROS 클라이언트로 선택 객체 전달
- 실시간 탐지 프레임 → binary_frame으로 웹 스트리밍


---

## 전체 노드 실행 순서

<img src="https://github.com/user-attachments/assets/71b2445a-b805-4df6-bea2-24032527cc84" width="1280"/>

---
## 욜로 학습 곡선 및 탐지 결과

<img src="https://github.com/user-attachments/assets/1bfd6365-c0da-46fa-8160-fdf5a114a38c" width="1280"/>

<img src="https://github.com/user-attachments/assets/a3f041d5-10d0-4f37-858e-a2bce9ad9c58" width="1280"/>



---
## 팀 소개

**TEAM C-4조 - ROBOKRATES**  
- 이희우  
- 정석환  
- 김동건  
- 김재훈

---

## 라이선스

본 프로젝트는 연구 및 학습 목적으로 개발되었으며, 상업적 사용을 금합니다.

---

