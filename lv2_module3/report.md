# 모듈 3 과제 보고서 — 설정값에 따른 Mask와 검출 결과 변화

작성자: 심규진 · 실험일: 2026-10-12  
과제 기준: [노션 「임시」 페이지](https://app.notion.com/p/3f7070e42ae0805eb596f03e620f8c77)  
기존 코드와 실험 장면: [Lv2_noon_Assignment, `48be67d`](https://github.com/visualkhh/Lv2_noon_Assignment/tree/48be67d5b73536649dc062264d1fa7021a53c866)

## 1. 실행 환경과 입력

| 항목 | 기록 |
| --- | --- |
| 사용 코드 | 기존 C++ `realsense::PerceptionNode`와 `fake_camera_bringup`을 수정 없이 사용. [인지 노드](https://github.com/visualkhh/Lv2_noon_Assignment/blob/48be67d5b73536649dc062264d1fa7021a53c866/lv2_module5/ros2_ws/src/realsense/src/perception_node.cpp), [검출기](https://github.com/visualkhh/Lv2_noon_Assignment/blob/48be67d5b73536649dc062264d1fa7021a53c866/lv2_module5/ros2_ws/src/realsense/src/detector.cpp), [설정 파일](https://github.com/visualkhh/Lv2_noon_Assignment/blob/48be67d5b73536649dc062264d1fa7021a53c866/lv2_module5/ros2_ws/src/realsense/config/realsense.yaml) |
| 실행 환경 | Docker `lv2-lyrical:testbed`, Ubuntu 26.04.1 LTS, ROS 2 Lyrical, Python 3.14.4, OpenCV 4.10.0. [환경 확인 로그](results/environment.txt) |
| 입력 방식·출처 | 카메라나 rosbag 대신 저장소의 [조명 아래 파란 기둥 사진 `lit_pillar_20261002.png`](https://github.com/visualkhh/Lv2_noon_Assignment/blob/48be67d5b73536649dc062264d1fa7021a53c866/lv2_module5/ros2_ws/src/realsense/test/data/lit_pillar_20261002.png)을 `fake_camera`가 약 30fps로 반복 발행. 원본 SHA-256: `3a4b56695023ab3f4f227e998e270292f13342a484701339e335a08a2e72f15c` |
| 입력 조건 | 고정된 640×480 컬러 프레임 한 장. 오른쪽 아래 파란 사각 기둥이 목표이며, 왼쪽의 하늘색 반투명 판과 실내 배경이 함께 보임. A/B 모두 같은 파일을 사용함. |
| 실행 명령 | `ros2 run fake_camera_bringup fake_camera --ros-args -r __ns:=/camera -r __node:=camera -p live_image:=/ws/exp/input.png -p current_image:=/ws/exp/current.png`; `ros2 launch realsense realsense.launch.py use_camera:=false`; `ros2 run fake_camera_bringup monitor_manager --ros-args -p debug_dir:=/ws/exp/monitor -p raw_images:=true`. 각 명령은 Docker에서 `/opt/ros/lyrical/setup.bash`와 빌드한 `/ws/install/setup.bash`를 불러온 뒤 실행함. |
| 관찰 방식 | `monitor_manager`가 토픽을 직접 구독해 저장한 이미지·메시지와 `ros2 topic echo --once /target --qos-reliability best_effort`를 기록. 각 A/B의 원본 이미지 파일 SHA-256은 동일함. |

호스트의 `lv2_module5/ros2_ws/src`를 Docker의 `/ws/src`에 읽기 전용으로 연결하고, 실험 작업 디렉터리를 `/ws/exp`에 연결했다. 컨테이너에서 `colcon build --packages-up-to fake_camera_bringup` 후 두 setup 파일을 불러왔고, 위 원본 사진을 `/ws/exp/input.png`에 복사해 발행했다. 실험에서는 인지 노드와 가짜 카메라, 구독 모니터만 실행했다. 모터 제어 노드는 실행하지 않았다. 기존 저장소의 [조명·HSV 선행 기록](https://github.com/visualkhh/Lv2_noon_Assignment/blob/48be67d5b73536649dc062264d1fa7021a53c866/lv2_module5/results/perception_env_record.md)은 장면과 기본값을 고르는 데 참고했으며, 아래 수치는 이번 ROS 토픽 실험에서 다시 측정한 값이다.

### 과제 1. 기존 비전 노드 실행과 토픽 구조 확인

| 자료 | 실제 토픽 이름 | 메시지 타입 | 발행 노드 | 실제 수신 도구 |
| --- | --- | --- | --- | --- |
| Raw Image | `/camera/camera/color/image_raw` | `sensor_msgs/msg/Image` | `/camera/camera` | `/perception_node`, `/monitor_manager` |
| Mask | `/perception_node/mask/compressed` | `sensor_msgs/msg/CompressedImage` | `/perception_node` | `/monitor_manager` |
| 검출 수치 | `/target` | `geometry_msgs/msg/PointStamped` | `/perception_node` | `/monitor_manager`, `ros2 topic echo` |
| 검출 Overlay | `/perception_node/debug_image/compressed` | `sensor_msgs/msg/CompressedImage` | `/perception_node` | `/monitor_manager` |

[실행 중 `ros2 topic info --verbose` 결과](results/topics.txt)에 발행·구독 노드와 QoS가 있다. Raw·Mask·Overlay는 reliable로 발행되고, `/target`은 best effort로 발행된다. 모니터의 수신 결과는 아래 이미지와 메시지 파일에 보관했다.

## 2. 토픽 구성

### 과제 2. Raw Image 토픽 받아보기

| 항목 | 기록 |
| --- | --- |
| 선택한 Raw Image 토픽 | `/camera/camera/color/image_raw` |
| 실제 수신 영상 | [Raw Image](results/exp1_A/raw_image.jpg), [수신 메시지 메타데이터](results/exp1_A/raw_message.json) |
| 해상도·encoding | `width=640`, `height=480`, `encoding=rgb8`, `step=1920` |
| 입력 프레임·구간 | 위 고정 사진을 반복 발행한 구간. A/B 사이 원본 JPEG 파일 해시가 같음. |

화면 오른쪽 아래에 파란 사각 기둥이 있고 왼쪽에는 더 큰 하늘색 반투명 판이 있다. 책상, 모니터, 카메라 등이 배경에 보이며, 목표의 밝은 면은 조명 때문에 채도가 낮아졌다. 물리 카메라의 연속 장면을 비교한 결과가 아니라 같은 저장 이미지를 반복 입력한 결과다.

### 과제 3. Mask 토픽 받아보기

| 항목 | 기록 |
| --- | --- |
| Mask 토픽 | `/perception_node/mask/compressed` |
| 실제 수신 Mask | [실험 1 A Mask](results/exp1_A/mask.jpg), [수신 메시지](results/exp1_A/mask_message.json) |
| 원본에서 남은 영역 | A에서는 오른쪽 파란 기둥의 대부분과 일부 작은 밝은 픽셀이 흰색으로 남고, 하늘색 판과 대부분의 배경은 검정임. |
| 전경·배경 값 | `make_mask`의 `cv::inRange`가 조건을 통과한 픽셀을 255, 나머지를 0으로 만듦. 발행 파일은 JPEG 압축이므로 경계에는 중간 회색값이 생길 수 있음. |
| 목표 외 영역·누락 | A에서 작은 주변 흰 점이 남지만 목표 후보는 기둥 한 개. B에서는 기둥의 채도 낮은 면이 빠지고 작은 조각만 남음. |
| 생성·발행 코드 | [`detector.cpp`의 `make_mask`](https://github.com/visualkhh/Lv2_noon_Assignment/blob/48be67d5b73536649dc062264d1fa7021a53c866/lv2_module5/ros2_ws/src/realsense/src/detector.cpp#L253-L275), [`perception_node.cpp`의 `onImage`·`publishDebug`](https://github.com/visualkhh/Lv2_noon_Assignment/blob/48be67d5b73536649dc062264d1fa7021a53c866/lv2_module5/ros2_ws/src/realsense/src/perception_node.cpp#L220-L298) |
| HSV 설정·적용 위치 | [기본 YAML](https://github.com/visualkhh/Lv2_noon_Assignment/blob/48be67d5b73536649dc062264d1fa7021a53c866/lv2_module5/ros2_ws/src/realsense/config/realsense.yaml#L23-L32)의 범위 1 `[103,220,20]`~`[130,255,255]`, 범위 2 `[103,150,130]`~`[130,255,255]`. 두 범위를 OR로 합치고 3px open, 5px close를 적용함. 실행 중 범위 2의 S 하한만 ROS 파라미터로 변경함. |

같은 입력에서 A Mask의 밝은 픽셀은 JPEG를 127 기준으로 읽었을 때 **6,396px**, B는 **968px**이었다. 이는 컨투어 면적과 다른 측정값이다. 원본 영상은 같고 S 하한 변경 후 기둥의 밝은 면이 Mask에서 사라진 것을 [A](results/exp1_A/mask.jpg)와 [B](results/exp1_B/mask.jpg)로 비교할 수 있다.

### 과제 4. 검출 결과 토픽 받아보기

| 항목 | 기록 |
| --- | --- |
| 검출 수치 토픽·타입 | `/target` · `geometry_msgs/msg/PointStamped` |
| 실제 수신 메시지 | [실험 1 A `/target` 출력](results/exp1_A/target_echo.txt), [실험 1 B 출력](results/exp1_B/target_echo.txt) |
| 필드·의미 | `header.stamp`는 입력 프레임 시각, `frame_id=camera_color_optical_frame`. `point.x=(cx-W/2)/(W/2)`, `point.y=(cy-H/2)/(H/2)`는 화면 중심 기준 정규화 오차로 단위가 없다. `point.z=contour_area/(W×H)`는 면적비이며 거리값이 아니다. 미검출이면 `x=y=z=0`. |
| 실제 값·Overlay | A: `x=0.38358463698787554`, `y=0.5376745388726081`, `z=0.018883463541666666`. [Overlay](results/exp1_A/detection_overlay.jpg)의 기둥 박스는 `(412,312) 60×115`, 중심은 오른쪽 아래여서 x·y가 양수인 메시지와 일치한다. B: `0,0,0`; [Overlay](results/exp1_B/detection_overlay.jpg)에 `NOT DETECTED (z=0)`. |
| 생성·발행 코드 | [`detect`의 면적·중심 계산](https://github.com/visualkhh/Lv2_noon_Assignment/blob/48be67d5b73536649dc062264d1fa7021a53c866/lv2_module5/ros2_ws/src/realsense/src/detector.cpp#L285-L402), [`onImage`의 `/target` 발행](https://github.com/visualkhh/Lv2_noon_Assignment/blob/48be67d5b73536649dc062264d1fa7021a53c866/lv2_module5/ros2_ws/src/realsense/src/perception_node.cpp#L220-L239) |

`/camera/camera/color/image_raw` 수신 → BGR 변환 → HSV 두 범위와 형태학 연산으로 Mask 생성 → 컨투어 면적·모양 필터와 후보 선택 → `/target` 수치 및 Mask·Overlay 발행. 박스 크기는 Overlay가 제공하며, `PointStamped` 자체에는 박스·검출 개수 필드가 없다.

## 3. 실험 설계

### 과제 5. 설정값을 바꿔 변화 검증하기 — 변경 전 예상

| 실험 | 변경할 값 | 관찰할 변화 | 선택 이유를 설명할 근거 |
| --- | --- | --- | --- |
| 1 Mask 생성 | 밝은 물체용 HSV 범위의 `hsv_bright_lower[1]`(S 하한)만 150 → 220 | 조명으로 채도가 낮아진 기둥 영역이 Mask에서 줄어드는지, `/target`이 미검출로 바뀌는지 | 기존 코드의 `inRange`·OR 조건과 선행 기록의 조명 장면 측정 |
| 2 검출 후보 필터 | `min_area_ratio`만 0.0006 → 0.02 | Mask는 유지되고 컨투어 후보만 `small`로 탈락하는지 | 기준 검출의 컨투어 면적비가 약 0.0189여서 0.02보다 작음 |

| 항목 | 실험 1 | 실험 2 |
| --- | --- | --- |
| 실제 변경 변수·의미 | `hsv_bright_lower[1]`, 밝은 HSV 범위의 S 하한. OpenCV S 범위 0~255 | `min_area_ratio`, 컨투어 면적/전체 영상 면적의 최소값. 무차원 비율 |
| A값·B값 | A `[103,150,130]` / B `[103,220,130]` | A `0.0006`(640×480에서 약 184px) / B `0.02`(6,144px) |
| 변경 전 예상 | 기둥 중심 S가 약 198이므로 B에서는 밝은 면이 빠져 Mask가 줄고 검출이 실패할 것 | A의 컨투어 면적이 약 5,801px·면적비 0.01888이므로 B에서 `small`로 탈락할 것. Mask 자체는 같을 것 |
| 동일하게 유지한 입력 | [고정 사진](https://github.com/visualkhh/Lv2_noon_Assignment/blob/48be67d5b73536649dc062264d1fa7021a53c866/lv2_module5/ros2_ws/src/realsense/test/data/lit_pillar_20261002.png), 640×480, 반복 발행 | 실험 1과 동일한 사진·해상도 |
| 동일하게 유지한 나머지 설정 | `hsv_lower=[103,220,20]`, `min_area_ratio=0.0006`, `open_kernel=3`, `close_kernel=5`, `select_rule=shape` 등 | 두 HSV 하한 `[103,220,20]`, `[103,150,130]`, `open_kernel=3`, `close_kernel=5`, `select_rule=shape` 등 |
| 설정 적용·확인 | `ros2 param set /perception_node hsv_bright_lower "[103,220,130]"`; [A 설정](results/exp1_A/settings.txt), [B 설정](results/exp1_B/settings.txt) | HSV를 A값으로 복원한 뒤 `ros2 param set /perception_node min_area_ratio 0.02`; [A 설정](results/exp2_A/settings.txt), [B 설정](results/exp2_B/settings.txt) |

각 실험의 A·B를 촬영하는 동안 입력 PNG는 바꾸지 않았다. 다른 주요 파라미터가 그대로인지 각 묶음의 `settings.txt`로 재확인했다. 실험 2 A/B의 Raw와 Mask JPEG는 각각 SHA-256 해시가 같아 동일한 정적 입력과 Mask가 들어왔음을 추가 확인했다.

## 4. 실험 결과와 분석

### 실험 2개의 A/B 결과 비교

| 실험 | 변경 변수 | A값 | B값 | 예상 변화 | 실제 Mask·검출 변화 | 선택값과 이유 |
| --- | --- | --- | --- | --- | --- | --- |
| 1 | `hsv_bright_lower[1]` | 150 | 220 | 밝은 기둥 Mask 축소, 미검출 | Mask 밝은 픽셀 6,396 → 968px(압축 영상 기준); `/target` 면적비 0.018883 → 0; B Overlay `NOT DETECTED` | **150**. 이 조명 장면에서 목표를 유지하고 하늘색 판은 Mask에서 대부분 제외됨. |
| 2 | `min_area_ratio` | 0.0006 | 0.02 | Mask 불변, 후보 탈락 | Raw·Mask 각각 파일 해시 동일. A `/target` 면적비 0.018883, B는 0; B Overlay `NOT DETECTED` | **0.0006**. 이 프레임의 목표 컨투어 5,801px가 통과함. |

### A/B 수신 자료 — 총 4묶음

각 링크는 실제 ROS 구독 노드 `monitor_manager`가 저장한 프레임 또는 CLI가 수신한 메시지다. GUI 이미지 뷰어 스크린샷 대신 토픽별 수신 이미지를 그대로 보관했다. Raw·Mask·Overlay의 발행 주기가 달라 메시지 시각은 완전히 같지 않지만, 카메라는 같은 정적 PNG 한 장만 반복 발행했다.

| 묶음 | Raw Image `/camera/camera/color/image_raw` | Mask `/perception_node/mask/compressed` | 검출 결과 `/target` 및 Overlay | 실제 적용값 |
| --- | --- | --- | --- | --- |
| 실험 1 A | [Raw](results/exp1_A/raw_image.jpg) | [Mask](results/exp1_A/mask.jpg) | [수치](results/exp1_A/target_echo.txt) · [Overlay](results/exp1_A/detection_overlay.jpg) | [S 150·면적비 0.0006](results/exp1_A/settings.txt) |
| 실험 1 B | [Raw](results/exp1_B/raw_image.jpg) | [Mask](results/exp1_B/mask.jpg) | [수치](results/exp1_B/target_echo.txt) · [Overlay](results/exp1_B/detection_overlay.jpg) | [S 220·면적비 0.0006](results/exp1_B/settings.txt) |
| 실험 2 A | [Raw](results/exp2_A/raw_image.jpg) | [Mask](results/exp2_A/mask.jpg) | [수치](results/exp2_A/target_echo.txt) · [Overlay](results/exp2_A/detection_overlay.jpg) | [S 150·면적비 0.0006](results/exp2_A/settings.txt) |
| 실험 2 B | [Raw](results/exp2_B/raw_image.jpg) | [Mask](results/exp2_B/mask.jpg) | [수치](results/exp2_B/target_echo.txt) · [Overlay](results/exp2_B/detection_overlay.jpg) | [S 150·면적비 0.02](results/exp2_B/settings.txt) |

**실험 1 분석.** `make_mask`는 기본 범위와 밝은 범위를 OR로 합친다. 기둥 중심 HSV `[105,198,174]`는 기본 범위의 S 하한 220에는 못 미치지만 밝은 범위의 S 하한 150은 넘는다. 따라서 밝은 범위의 S 하한을 220으로 올리면 기둥 몸체가 빠진다. 남은 작은 조각으로는 유효 후보가 선택되지 않아 `/target`이 0이 됐다. A Overlay에서는 오른쪽 기둥 박스 한 개가 확인되며, 하늘색 판은 선택되지 않았다.

**실험 2 분석.** `min_area_ratio`는 Mask 생성 뒤 `detect`의 `area/frame_area < min_area_ratio` 조건에만 쓰인다. 그래서 A/B Mask는 파일 내용까지 같았다. A에서 선택된 컨투어는 5,801px, `5801/307200=0.018883`이며 0.02보다 작다. B에서 `small` 조건으로 탈락해 `/target`의 x·y·z가 모두 0이 됐다. 후보 탈락 이유는 코드 조건과 측정 면적에서 추론한 것이며 `/target` 메시지에 이유 문자열은 실리지 않는다.

두 실험 모두 변경 전 예상과 관찰이 일치했다. 수치 토픽에는 검출 개수 필드가 없으므로 “박스 한 개”는 Overlay를 눈으로 확인한 결과다. 이 프레임에서 관찰한 것은 **미검출**이며, 목표가 아닌 물체를 선택하는 **오검출**은 이번 A/B에서 발생하지 않았다.

## 5. 선택값과 결론

| 항목 | 결론 |
| --- | --- |
| 실험 1에서 선택한 값 | 밝은 HSV 범위의 S 하한 **150** (`hsv_bright_lower=[103,150,130]`). 220에서는 조명 받은 기둥 대부분이 Mask에서 빠져 미검출됨. |
| 실험 2에서 선택한 값 | `min_area_ratio=0.0006`. 0.02에서는 실제 기둥 컨투어의 면적비 0.018883이 하한보다 작아 탈락함. |
| 오검출과 미검출 | 미검출은 목표가 화면에 있는데 `/target.z=0`인 상태로, 두 B 조건에서 확인했다. 오검출은 목표 아닌 물체가 선택되는 경우인데 이번 고정 프레임에서는 확인되지 않았다. 선행 저장소 기록에는 하늘색 판이 목표 대신 선택된 사례가 있으나 이번 측정 결과로 세지 않는다. |
| 확인된 입력 조건 | 640×480, 저장된 조명 아래 파란 기둥 사진 한 장을 반복 발행한 ROS 토픽 경로. 선택값으로 기둥이 검출되고, 아래의 균일한 무목표 영상에서는 미검출됨. |
| 적용 한계 | 실제 카메라의 시간에 따른 노출·거리·가림·배경 변화, 다른 해상도, 다양한 무목표 장면에서의 오검출률은 이번 실험으로 확인하지 못했다. 이 값들이 전 조건에서 최적이라는 주장은 할 수 없다. |

### 목표가 없는 입력 확인

| 항목 | 기록 |
| --- | --- |
| 입력 출처·구간 | 직접 만든 640×480 균일 회색 `(R,G,B)=(180,180,180)` [무목표 입력](results/no_target/no_target_input.png)을 같은 `fake_camera`로 반복 발행. 실제 카메라의 빈 장면은 아님. |
| 수신 증거 | [Raw](results/no_target/raw_image.jpg) · [Mask](results/no_target/mask.jpg) · [`/target` 출력](results/no_target/target_echo.txt) · [Overlay](results/no_target/detection_overlay.jpg) · [설정 확인](results/no_target/settings.txt) |
| 미검출 표현 | Mask의 밝은 픽셀 0개(JPEG 127 기준), `/target.point=(0,0,0)`, Overlay `NOT DETECTED (z=0)` |
| 노드 동작 | 입력·Mask·Overlay가 계속 발행되고 새 시각의 `/target` 메시지도 수신됨. 토픽 연결이 끊겨서 0이 나온 상황은 아님. |

**최종 설명.** 조명으로 채도가 낮아진 기둥을 살리기 위해 밝은 HSV 범위의 S 하한을 비교했고, 150에서는 기둥이 Mask와 결과에 남았지만 220에서는 빠졌다. 후보 면적 하한을 비교할 때는 HSV를 고정했으며, 0.02에서는 같은 Mask의 0.018883 면적비 후보가 탈락했다. 이 고정 입력에 대해 S 하한 150과 `min_area_ratio=0.0006`을 선택한다.
