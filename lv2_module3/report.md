# 모듈 3 과제 보고서 — rosbag 프레임의 Mask·검출 결과 비교

기존 코드: [Lv2_noon_Assignment 커밋 `48be67d`](https://github.com/visualkhh/Lv2_noon_Assignment/tree/48be67d5b73536649dc062264d1fa7021a53c866)

## 1. 실행 환경과 입력

| 항목 | 실제 사용 내용 |
| --- | --- |
| 원본 rosbag | `Lv2_noon_Assignment/lv2_module5/recordings/rosbag/0_20261008_123426_success_2026_10_08-12_34_29.mcap`. MCAP SHA-256은 [해시 기록](results/bag_reference/bag_sha256.txt)에 보관. 원본 bag은 이 개인 저장소에 복사하지 않았음. |
| bag 범위 | 2026-10-08 12:34:29~12:35:00 KST, 31.471초·3,624개 메시지. Raw Image 942개, Mask·Overlay 각 146개, `/target` 943개. [bag 정보](results/bag_reference/bag_info.txt) · [토픽별 요약](results/bag_reference/summary.json) |
| 실제 사용 프레임 | **A/B: Raw Image 926번**, bag 시작 후 30.936초, 원본 `header.stamp=1791430500188694336 ns`. **목표 없음: 721번**, 시작 후 24.098초, `1791430493350372314 ns`. 두 프레임 모두 원래 bag에 같은 시각의 Mask·Overlay·`/target`이 있음. [926번 메타데이터](results/bag_reference/target_frame/frame.json) · [721번 메타데이터](results/bag_reference/no_target_frame/frame.json) |
| 코드·환경 | 기존 C++ `realsense::PerceptionNode`와 `fake_camera_bringup`을 수정 없이 사용. Docker `lv2-lyrical:testbed`의 Ubuntu 26.04.1 LTS, ROS 2 Lyrical, Python 3.14.4, OpenCV 4.10.0. [버전 확인](results/environment.txt) |
| 입력 조건 | bag의 `rgb8` Raw Image에서 선택한 **424×240 프레임을 PNG로 무손실 추출**해, 같은 파일을 `fake_camera`가 약 30fps로 반복 발행. 926번에는 사람이 든 파란 사각 기둥과 실내 배경이 있고, 721번에는 목표가 보이지 않는 회색 표면이 있음. |
| 원본 검출 결과 | 926번 bag 기록은 `x=0.305094`, `y=0.326234`, `z=0.026361`. [원본 Mask](results/bag_reference/target_frame/recorded_mask.jpg) · [Overlay](results/bag_reference/target_frame/recorded_overlay.jpg) · [수치](results/bag_reference/target_frame/recorded_target.json). 721번은 `x=y=z=0`. |

bag에는 `/motor_cmd`와 `/tracking_status`도 저장돼 있지만 이 과제에서는 사용하거나 재생하지 않았다. [bag 장면 개요](results/bag_reference/overview.jpg)에는 목표가 가까워지고 사라졌다가 다시 등장하는 흐름이 보인다. A/B 측정에는 **bag 전체를 연속 재생한 것이 아니라 926번 Raw Image 한 장을 반복 입력**했다. 움직임·노출 변화가 결과에 섞이지 않도록 하기 위한 통제이며, 원본 bag에서의 장면 위치와 시각은 위에 기록했다. 원본 bag에서 `/target.z>0`인 메시지는 943개 중 463개였지만, 이동·가림·배경 변화가 포함되어 있어 이를 검출 정확도로 해석하지 않는다. [원본 `/target` 시계열](results/bag_reference/targets.csv)을 함께 보관했다.

호스트의 `lv2_module5/ros2_ws/src`를 Docker `/ws/src`에 읽기 전용으로 연결하고 `colcon build --packages-up-to fake_camera_bringup`로 빌드했다. `/opt/ros/lyrical/setup.bash`, `/ws/install/setup.bash`를 불러온 뒤 다음 명령으로 노드를 실행했다.

```bash
ros2 run fake_camera_bringup fake_camera --ros-args -r __ns:=/camera -r __node:=camera -p live_image:=/ws/exp/input.png -p current_image:=/ws/exp/current.png
ros2 launch realsense realsense.launch.py use_camera:=false
ros2 run fake_camera_bringup monitor_manager --ros-args -p debug_dir:=/ws/exp/monitor -p raw_images:=true
```

`/ws/exp/input.png`에는 bag에서 추출한 [926번 PNG](results/bag_reference/target_frame/raw_image.png)를 복사했다. 목표 없음 확인 때만 [721번 PNG](results/bag_reference/no_target_frame/raw_image.png)로 바꿨다. `monitor_manager`가 이미지 토픽을 직접 구독해 저장한 파일과 `ros2 topic echo --once /target --qos-reliability best_effort`로 수신한 메시지를 증거로 남겼다. GUI 이미지 뷰어의 화면 캡처는 사용하지 않았다.

### 과제 1. 기존 비전 노드 실행과 토픽 구조 확인

| 자료 | 실제 토픽 이름 | 메시지 타입 | 발행 노드 | 수신 도구 |
| --- | --- | --- | --- | --- |
| Raw Image | `/camera/camera/color/image_raw` | `sensor_msgs/msg/Image` | `/camera/camera` | `/perception_node`, `/monitor_manager` |
| Mask | `/perception_node/mask/compressed` | `sensor_msgs/msg/CompressedImage` | `/perception_node` | `/monitor_manager` |
| 검출 결과 수치 | `/target` | `geometry_msgs/msg/PointStamped` | `/perception_node` | `/monitor_manager`, `ros2 topic echo` |
| 검출 Overlay | `/perception_node/debug_image/compressed` | `sensor_msgs/msg/CompressedImage` | `/perception_node` | `/monitor_manager` |

[재실행 중 `ros2 topic info --verbose` 결과](results/topics.txt)에서 발행·구독 노드와 QoS를 확인했다. Raw·Mask·Overlay 발행은 reliable, `/target` 발행은 best effort다. [원본 bag 정보](results/bag_reference/bag_info.txt)에도 이 네 토픽의 타입과 메시지 개수가 있다.

## 2. 토픽 구성

### 과제 2. Raw Image 토픽 받아보기

| 항목 | 기록 |
| --- | --- |
| 선택 토픽 | `/camera/camera/color/image_raw` |
| 실제 수신 영상 | [bag 원본 926번 Raw](results/bag_reference/target_frame/raw_image.png) · [재실행 시 ROS 구독 Raw](results/bag_exp1_A/raw_image.jpg) · [수신 메시지](results/bag_exp1_A/raw_message.json) |
| 해상도·encoding | 424×240, `rgb8`, `step=1272`; `frame_id=camera_color_optical_frame` |
| 입력 프레임·시각 | bag Raw Image **926번**, 시작 후 30.936초. 재실행 A/B의 Raw JPEG 파일 해시는 모두 동일함. |

화면 오른쪽에서 사람이 파란 사각 기둥을 손으로 들고 있으며, 왼쪽에는 다른 사람과 책상·모니터가 보인다. 기둥의 밝은 면은 조명 아래 채도가 낮아져 기본 S 하한만으로는 일부가 제외될 수 있다. 이 장면 하나를 고정해 실험했다.

### 과제 3. Mask 토픽 받아보기

| 항목 | 기록 |
| --- | --- |
| Mask 토픽 | `/perception_node/mask/compressed` |
| 실제 수신 자료 | [bag에 기록된 926번 Mask](results/bag_reference/target_frame/recorded_mask.jpg) · [재실행 A Mask](results/bag_exp1_A/mask.jpg) · [재실행 B Mask](results/bag_exp1_B/mask.jpg) · [수신 메시지](results/bag_exp1_A/mask_message.json) |
| 원본과 비교 | A에서는 손에 든 오른쪽 파란 기둥이 주로 흰색으로 남고 사람·천장·책상은 대부분 검정임. B에서는 기둥 몸체가 거의 사라져 작은 흰 조각만 남음. |
| 전경·배경 값 | [`make_mask`](https://github.com/visualkhh/Lv2_noon_Assignment/blob/48be67d5b73536649dc062264d1fa7021a53c866/lv2_module5/ros2_ws/src/realsense/src/detector.cpp#L253-L275)의 `inRange`를 통과한 전경은 255, 배경은 0. Mask 토픽은 JPEG 압축이므로 경계 픽셀은 중간 값이 될 수 있음. |
| 목표 외 영역·누락 | A에서 주변 잡음이 일부 남지만 검출 목표는 기둥 한 개. B에서는 채도 낮은 기둥 면이 누락됨. JPEG를 밝기 127 기준으로 읽은 흰 픽셀은 A **2,795개**, B **52개**. 이 픽셀 수는 컨투어 면적과 구분함. |
| 코드·설정 위치 | [기본 YAML](https://github.com/visualkhh/Lv2_noon_Assignment/blob/48be67d5b73536649dc062264d1fa7021a53c866/lv2_module5/ros2_ws/src/realsense/config/realsense.yaml#L23-L44)의 HSV 범위 1 `[103,220,20]`~`[130,255,255]`와 밝은 범위 2 `[103,150,130]`~`[130,255,255]`를 OR로 합침. 3px open, 5px close 적용. [Mask 토픽 발행](https://github.com/visualkhh/Lv2_noon_Assignment/blob/48be67d5b73536649dc062264d1fa7021a53c866/lv2_module5/ros2_ws/src/realsense/src/perception_node.cpp#L220-L298) |

bag에 원래 기록된 926번 Mask와 재실행 A Mask는 JPEG 바이트 자체는 달라도, **127 기준 전경 위치는 모두 동일**하고 둘 다 2,795픽셀이다. 이를 통해 추출 프레임의 재실행 결과가 원본 bag의 기준 상태와 일치함을 확인했다.

### 과제 4. 검출 결과 토픽 받아보기

| 항목 | 기록 |
| --- | --- |
| 검출 결과 토픽·타입 | `/target` · `geometry_msgs/msg/PointStamped`; Overlay는 `/perception_node/debug_image/compressed` |
| 수신 값 | [bag 원본 926번](results/bag_reference/target_frame/recorded_target.json)과 [재실행 A](results/bag_exp1_A/target_echo.txt) 모두 `x=0.3050941637799151`, `y=0.32623382002692336`, `z=0.026361045597484277`. [재실행 B](results/bag_exp1_B/target_echo.txt)는 `0,0,0`. |
| 필드 의미·단위 | `header.stamp`는 입력 프레임 시각. `point.x=(cx-W/2)/(W/2)`, `point.y=(cy-H/2)/(H/2)`는 화면 중심 기준 정규화 오차로 무차원. `point.z=contour_area/(W×H)`는 면적비이며 거리값이 아님. 미검출 때는 모두 0. |
| Overlay와 원본 비교 | [bag 원본 Overlay](results/bag_reference/target_frame/recorded_overlay.jpg)와 [재실행 A Overlay](results/bag_exp1_A/detection_overlay.jpg)에 오른쪽 기둥의 박스 `(250,119) 55×80`이 보임. `/target`의 양수 x·y도 화면 중심보다 오른쪽 아래인 위치와 일치함. B는 [미검출 Overlay](results/bag_exp1_B/detection_overlay.jpg). |
| 생성·발행 코드 | [`detect`의 컨투어 필터·중심·면적비 계산](https://github.com/visualkhh/Lv2_noon_Assignment/blob/48be67d5b73536649dc062264d1fa7021a53c866/lv2_module5/ros2_ws/src/realsense/src/detector.cpp#L285-L402) → [`onImage`의 `/target` 발행](https://github.com/visualkhh/Lv2_noon_Assignment/blob/48be67d5b73536649dc062264d1fa7021a53c866/lv2_module5/ros2_ws/src/realsense/src/perception_node.cpp#L220-L239) |

`/camera/camera/color/image_raw` 수신 → BGR 변환 → HSV 두 범위와 형태학 연산으로 Mask 생성 → 컨투어 면적·모양 필터와 후보 선택 → `/target` 수치 및 Mask·Overlay 발행. `PointStamped`에는 Bounding Box·검출 개수 필드가 없어서 박스는 Overlay에서 확인했다.

## 3. 실험 설계

### 과제 5. 설정값을 바꿔 변화 검증하기 — 변경 전 예상

| 실험 | 변경할 값 | 관찰할 변화 | 선택 이유를 설명할 근거 |
| --- | --- | --- | --- |
| 1 Mask 생성 | 밝은 HSV 범위의 `hsv_bright_lower[1]`(S 하한)만 150 → 220 | 기둥의 밝은 면이 Mask에서 빠지고 목표 검출이 사라지는가 | 원본 Overlay의 기둥 중심 HSV `[105,205,153]`와 `inRange` 조건 |
| 2 검출 후보 필터 | `min_area_ratio`만 0.0006 → 0.03 | 동일한 Mask에서 면적비 0.026361인 기둥 후보가 탈락하는가 | [`area/frame_area < min_area_ratio` 코드 조건](https://github.com/visualkhh/Lv2_noon_Assignment/blob/48be67d5b73536649dc062264d1fa7021a53c866/lv2_module5/ros2_ws/src/realsense/src/detector.cpp#L304-L310) |

| 항목 | 실험 1 | 실험 2 |
| --- | --- | --- |
| 변수·의미 | `hsv_bright_lower[1]`, 밝은 범위의 S 하한. 유효 범위 0~255 | `min_area_ratio`, 컨투어 면적/전체 영상 면적의 하한. 무차원 비율 |
| A값·B값 | A `[103,150,130]` / B `[103,220,130]` | A `0.0006`(424×240에서 약 61px) / B `0.03`(약 3,053px) |
| 변경 전 예상 | 기둥 중심 S=205는 150을 통과하지만 220에는 못 미치므로 B Mask가 줄고 미검출될 것 | 원본 기둥 컨투어 면적 2,682.5px·면적비 0.026361이 0.03보다 작으므로 B에서 탈락할 것. Mask는 변하지 않을 것 |
| 같은 입력·해상도 | 두 실험 모두 bag Raw **926번 PNG**, 424×240을 반복 입력 | 실험 1과 동일 |
| 고정한 다른 설정 | `hsv_lower=[103,220,20]`, `min_area_ratio=0.0006`, `open_kernel=3`, `close_kernel=5`, `select_rule=shape` 등 | 두 HSV 하한 `[103,220,20]`, `[103,150,130]`, `open_kernel=3`, `close_kernel=5`, `select_rule=shape` 등 |
| 적용·확인 | `ros2 param set /perception_node hsv_bright_lower "[103,220,130]"`; [A](results/bag_exp1_A/settings.txt) · [B](results/bag_exp1_B/settings.txt) | HSV를 기본값으로 복원 후 `ros2 param set /perception_node min_area_ratio 0.03`; [A](results/bag_exp2_A/settings.txt) · [B](results/bag_exp2_B/settings.txt) |

각 A/B의 입력 PNG는 동일하다. `/target`·Mask 발행 시각은 재실행 중 새로 붙지만, 네 묶음의 **Raw JPEG SHA-256은 동일**하다. 실험 2 A/B는 **Mask JPEG SHA-256도 동일**해 면적 하한만 바뀌었음을 확인했다. 파일별 해시·픽셀 수·실제 설정·수치는 [실험 검증 기록](results/bag_reference/experiment_checks.txt)에 모았다.

## 4. 실험 결과와 분석

### 실험 2개의 A/B 결과 비교

| 실험 | 변경 변수 | A값 | B값 | 예상 변화 | 실제 Mask·검출 변화 | 선택값과 이유 |
| --- | --- | --- | --- | --- | --- | --- |
| 1 | `hsv_bright_lower[1]` | 150 | 220 | 기둥 Mask 축소·미검출 | Mask 전경 2,795 → 52px(JPEG 127 기준); `/target.z` 0.026361 → 0; B Overlay `NOT DETECTED` | **150**. 이 조명 장면에서 기둥을 유지함. |
| 2 | `min_area_ratio` | 0.0006 | 0.03 | Mask 불변·후보 탈락 | Raw·Mask 파일 해시 동일; `/target.z` 0.026361 → 0; B Overlay `NOT DETECTED` | **0.0006**. 2,682.5px 목표 컨투어를 통과시킴. |

### A/B 수신 자료 — 총 4묶음

각 파일은 926번 원본 PNG를 반복 입력할 때 `monitor_manager` 또는 CLI가 **실제로 구독**해 저장한 결과다. GUI 뷰어 스크린샷 대신 토픽별 수신 이미지와 메시지를 제출한다. Raw·Mask·Overlay 발행 주기가 달라 파일의 메시지 시각은 완전히 같지 않지만 입력 픽셀은 동일하다.

| 묶음 | Raw Image `/camera/camera/color/image_raw` | Mask `/perception_node/mask/compressed` | 검출 결과 `/target`·Overlay | 실제 적용값 |
| --- | --- | --- | --- | --- |
| 실험 1 A | [Raw](results/bag_exp1_A/raw_image.jpg) | [Mask](results/bag_exp1_A/mask.jpg) | [수치](results/bag_exp1_A/target_echo.txt) · [Overlay](results/bag_exp1_A/detection_overlay.jpg) | [S 150·면적비 0.0006](results/bag_exp1_A/settings.txt) |
| 실험 1 B | [Raw](results/bag_exp1_B/raw_image.jpg) | [Mask](results/bag_exp1_B/mask.jpg) | [수치](results/bag_exp1_B/target_echo.txt) · [Overlay](results/bag_exp1_B/detection_overlay.jpg) | [S 220·면적비 0.0006](results/bag_exp1_B/settings.txt) |
| 실험 2 A | [Raw](results/bag_exp2_A/raw_image.jpg) | [Mask](results/bag_exp2_A/mask.jpg) | [수치](results/bag_exp2_A/target_echo.txt) · [Overlay](results/bag_exp2_A/detection_overlay.jpg) | [S 150·면적비 0.0006](results/bag_exp2_A/settings.txt) |
| 실험 2 B | [Raw](results/bag_exp2_B/raw_image.jpg) | [Mask](results/bag_exp2_B/mask.jpg) | [수치](results/bag_exp2_B/target_echo.txt) · [Overlay](results/bag_exp2_B/detection_overlay.jpg) | [S 150·면적비 0.03](results/bag_exp2_B/settings.txt) |

**실험 1 분석.** 기본 HSV 범위의 S 하한은 220이고, 밝은 범위의 S 하한은 A에서 150이다. 원본 Overlay에 표시된 기둥 중심 S=205는 밝은 범위 A만 통과한다. B에서 밝은 범위 S 하한도 220으로 올리자 해당 면이 Mask에서 빠졌다. 남은 조각은 유효 후보가 되지 않아 `/target`이 0이 됐다. 주변 물체가 대신 선택되는 오검출은 이 프레임에서 관찰되지 않았다.

**실험 2 분석.** `min_area_ratio`는 Mask 생성 뒤 `detect`에서 후보의 `area/frame_area`를 비교한다. 이 프레임의 선택 컨투어 면적은 2,682.5px이고 영상 면적은 424×240=101,760px이므로 면적비는 **0.0263610456**이다. A의 0.0006은 통과하고 B의 0.03에는 못 미친다. 그래서 Mask는 그대로인데 후보가 `small` 조건으로 탈락하고 `/target`이 0이 됐다. `small`은 코드 조건과 면적에서 추론한 사유이며 `/target` 메시지에 사유 필드는 없다.

두 실험의 관찰은 변경 전 예상과 일치했다. 수치 토픽에는 검출 개수 필드가 없으므로 A의 “박스 한 개”는 Overlay에서 눈으로 확인한 값이다.

## 5. 선택값과 결론

| 항목 | 결론 |
| --- | --- |
| 실험 1 선택값 | `hsv_bright_lower=[103,150,130]`. S 하한 220에서는 조명 받은 기둥 몸체가 Mask에서 빠져 미검출됨. |
| 실험 2 선택값 | `min_area_ratio=0.0006`. 0.03에서는 이 프레임의 면적비 0.026361인 기둥이 탈락함. |
| 오검출과 미검출 | 두 B 설정은 목표가 보이는데 `/target.z=0`인 **미검출**이다. 목표가 아닌 물체를 선택하는 **오검출**은 이 고정 프레임에서 확인되지 않았다. 원본 bag 전체의 검출·미검출 횟수만으로 오검출률을 판단하지 않는다. |
| 확인된 범위 | 2026-10-08 bag의 424×240 기둥 프레임 926번을 반복 처리한 ROS 토픽 경로와 아래의 무목표 프레임 721번. |
| 한계 | 사람·기둥이 움직이는 bag 전체를 각 설정으로 재처리한 결과가 아니다. 다른 거리·조명·가림·해상도에서의 최적값이나 오검출률은 이번 A/B로 검증하지 못했다. |

### 목표가 없는 입력 확인

| 항목 | 기록 |
| --- | --- |
| 입력 출처·구간 | 같은 bag의 Raw Image **721번**, 시작 후 24.098초. [원본 PNG·프레임 정보](results/bag_reference/no_target_frame/frame.json) · [Raw](results/bag_reference/no_target_frame/raw_image.png). 화면은 목표가 보이지 않는 회색 표면임. |
| 원본 bag 수신 증거 | [Mask](results/bag_reference/no_target_frame/recorded_mask.jpg) · [Overlay](results/bag_reference/no_target_frame/recorded_overlay.jpg) · [`/target` 수치](results/bag_reference/no_target_frame/recorded_target.json): `x=y=z=0` |
| 선택값 재실행 증거 | [Raw](results/bag_no_target/raw_image.jpg) · [Mask](results/bag_no_target/mask.jpg) · [Overlay](results/bag_no_target/detection_overlay.jpg) · [`/target` 출력](results/bag_no_target/target_echo.txt) · [설정](results/bag_no_target/settings.txt) |
| 미검출·노드 동작 | Mask의 밝은 픽셀 0개(JPEG 127 기준), `/target.point=(0,0,0)`, Overlay `NOT DETECTED (z=0)`. Raw·Mask·Overlay와 새 시각의 `/target`이 계속 수신되어 노드 중단에 따른 0이 아님. |

**최종 설명.** 밝은 기둥을 Mask에 남기기 위해 S 하한을 비교했고, 150에서는 원본 bag과 같은 위치·면적비의 목표를 검출했지만 220에서는 기둥이 Mask에서 거의 사라졌다. 후보 면적 하한을 비교할 때 HSV를 고정했으며, 0.03에서는 같은 Mask의 면적비 0.026361 후보가 탈락했다. 따라서 이 bag 프레임에 대해서는 **밝은 HSV S 하한 150, `min_area_ratio=0.0006`**을 선택한다.
