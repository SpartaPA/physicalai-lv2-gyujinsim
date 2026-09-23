## 문제1
### 1.원격 수행 환경 구성
```
라즈베리파이 OS
PRETTY_NAME="Ubuntu 22.04.5 LTS"
NAME="Ubuntu"
VERSION_ID="22.04"
VERSION="22.04.5 LTS (Jammy Jellyfish)"
VERSION_CODENAME=jammy
ID=ubuntu
ID_LIKE=debian

SSH 접속
echo $SSH_CLIENT
10.2.16.204 46122 22

포트
echo "$PORT"
/dev/ttyACM0
```

### 2.OpenCR 펌웨어 업로드
```
opencr_ld ver 1.0.4
opencr_ld_main 
>>
file name : /home/pa09/pa-opencr-build/output/opencr_position_p.ino.bin 
file size : 102 KB
Open port OK
Clear Buffer Start
Clear Buffer End
Board Name : OpenCR R1.0
Board Ver  : 0x17020800
Board Rev  : 0x00000000
>>
flash_erase : 0 : 0.919000 sec
flash_write : 0 : 1.182000 sec 
CRC OK 9B7D94 9B7D94 0.003000 sec
[OK] Download 
jump_to_fw 
jump finished

crw-rw-rw- 1 root dialout 166, 0 Sep 23 11:25 /dev/ttyACM0
--- Miniterm on /dev/ttyACM0  115200,8,N,1 ---
--- Quit: Ctrl+] | Menu: Ctrl+T | Help: Ctrl+T followed by Ctrl+H ---
```

### 3.목표 입력과 응답 기록
```
s 1 5 10

target_deg:10.000       position_deg:0.088      error_deg:9.912 p_deg_s:9.912   i_deg_s:0.000   d_deg_s:0.000   pid_deg_s:9.912 speed_deg_s:0.000       u_deg_s:4.122   v_limit_deg_s:5.000    dt_ms:10.000     kp:1.0000       ki:0.0000       kd:0.0000       t_s:2.000
target_deg:10.000       position_deg:0.176      error_deg:9.824 p_deg_s:9.824   i_deg_s:0.000   d_deg_s:0.000   pid_deg_s:9.824 speed_deg_s:0.000       u_deg_s:4.122   v_limit_deg_s:5.000    dt_ms:10.001     kp:1.0000       ki:0.0000       kd:0.0000       t_s:2.100
target_deg:10.000       position_deg:0.615      error_deg:9.385 p_deg_s:9.385   i_deg_s:0.000   d_deg_s:0.000   pid_deg_s:9.385 speed_deg_s:4.122       u_deg_s:4.122   v_limit_deg_s:5.000    dt_ms:10.002     kp:1.0000       ki:0.0000       kd:0.0000       t_s:2.200
target_deg:10.000       position_deg:0.967      error_deg:9.033 p_deg_s:9.033   i_deg_s:0.000   d_deg_s:0.000   pid_deg_s:9.033 speed_deg_s:5.496       u_deg_s:4.122   v_limit_deg_s:5.000    dt_ms:10.001     kp:1.0000       ki:0.0000       kd:0.0000       t_s:2.300
target_deg:10.000       position_deg:1.318      error_deg:8.682 p_deg_s:8.682   i_deg_s:0.000   d_deg_s:0.000   pid_deg_s:8.682 speed_deg_s:1.374       u_deg_s:4.122   v_limit_deg_s:5.000    dt_ms:10.002     kp:1.0000       ki:0.0000       kd:0.0000       t_s:2.400
target_deg:10.000       position_deg:1.846      error_deg:8.154 p_deg_s:8.154   i_deg_s:0.000   d_deg_s:0.000   pid_deg_s:8.154 speed_deg_s:4.122       u_deg_s:4.122   v_limit_deg_s:5.000    dt_ms:10.000     kp:1.0000       ki:0.0000       kd:0.0000       t_s:2.500
target_deg:10.000       position_deg:2.197      error_deg:7.803 p_deg_s:7.803   i_deg_s:0.000   d_deg_s:0.000   pid_deg_s:7.803 speed_deg_s:1.374       u_deg_s:4.122   v_limit_deg_s:5.000    dt_ms:10.000     kp:1.0000       ki:0.0000       kd:0.0000       t_s:2.600
target_deg:10.000       position_deg:2.637      error_deg:7.363 p_deg_s:7.363   i_deg_s:0.000   d_deg_s:0.000   pid_deg_s:7.363 speed_deg_s:5.496       u_deg_s:4.122   v_limit_deg_s:5.000    dt_ms:10.002     kp:1.0000       ki:0.0000       kd:0.0000       t_s:2.700
target_deg:10.000       position_deg:2.988      error_deg:7.012 p_deg_s:7.012   i_deg_s:0.000   d_deg_s:0.000   pid_deg_s:7.012 speed_deg_s:1.374       u_deg_s:4.122   v_limit_deg_s:5.000    dt_ms:10.002     kp:1.0000       ki:0.0000       kd:0.0000       t_s:2.800
target_deg:10.000       position_deg:3.340      error_deg:6.660 p_deg_s:6.660   i_deg_s:0.000   d_deg_s:0.000   pid_deg_s:6.660 speed_deg_s:2.748       u_deg_s:4.122   v_limit_deg_s:5.000    dt_ms:10.000     kp:1.0000       ki:0.0000       kd:0.0000       t_s:2.900
target_deg:10.000       position_deg:3.955      error_deg:6.045 p_deg_s:6.045   i_deg_s:0.000   d_deg_s:0.000   pid_deg_s:6.045 speed_deg_s:4.122       u_deg_s:4.122   v_limit_deg_s:5.000    dt_ms:10.001     kp:1.0000       ki:0.0000       kd:0.0000       t_s:3.000
target_deg:10.000       position_deg:4.219      error_deg:5.781 p_deg_s:5.781   i_deg_s:0.000   d_deg_s:0.000   pid_deg_s:5.781 speed_deg_s:2.748       u_deg_s:4.122   v_limit_deg_s:5.000    dt_ms:10.002     kp:1.0000       ki:0.0000       kd:0.0000       t_s:3.100
target_deg:10.000       position_deg:4.746      error_deg:5.254 p_deg_s:5.254   i_deg_s:0.000   d_deg_s:0.000   pid_deg_s:5.254 speed_deg_s:4.122       u_deg_s:4.122   v_limit_deg_s:5.000    dt_ms:10.001     kp:1.0000       ki:0.0000       kd:0.0000       t_s:3.200

```

### 4.결과 설명
```
모터 모델, ID : XM430-W350(모델1030), ID: 1
게인: 1
목표각: 90
속도 상한: 30

목표값(target_deg): 모터가 도달해야 하는 목표 각도,(도)deg
측정값(position_deg): 엔코더로 측정한 현재 모터 각도, (도)deg
제어출력(u_deg_s): 모터에 명령하는 목표 회전 속도, deg/s

목표값 90°에 대해 측정값이 0°에서 61.084°, 80.684°, 89.385°까지 증가하여 실제 모터가 목표 위치 방향으로 움직이며 목표값에 가까워졌다.
또한 오차는 90°에서 약 0.615°까지 감소하고, 목표에 가까워질수록 제어 출력도 28.854°/s에서 0°/s로 감소하므로 P 제어가 정상적으로 목표에 수렴하고 있음을 확인할 수 있다
```

## 문제2
|시점|목표각|현재각|오차|
| --- | --- | --- | --- |
|초기, 2.000 s|+90|+0.000|+90|
|중간, 5.401 s|+90|80.684|+9.316|
|마지막, 13.602 s|+90+|+89.385|0.615|

```
통신경로: Dynamixel 내부 엔코더 → Dynamixel 제어기 → Dynamixel 통신 버스 → OpenCR DXL 포트(Serial3) → OpenCR 제어 프로그램

보정방향: 목표 +30, 현재 +35 인 경우 오차는 -5도이므로 음수 오차가 된다. 

Kp가 양수일 때 P 제어 출력은 오차와 같은 부호를 가지므로 제어 출력도 음수가 된다. 따라서 모터는 현재각을 감소시키는 음의 방향으로 회전하여 +35°에서 목표인 +30° 방향으로 보정된다.
```

## 문제3
### 1. 실행 조건

| 구분 | 실행 A | 실행 B |
|---|---:|---:|
| Kp | 1.0 | 10.0 |
| Ki | 0.0 | 0.0 |
| Kd | 0.0 | 0.0 |
| 목표각 | +90° | +90° |
| 속도 상한 | 30°/s | 30°/s |
| 측정 주기 | 약 10 ms | 약 10 ms |
| 시작 위치 | 0° 기준 | 0° 기준 |

실행 A는 `s 1 30 90`, 실행 B는 `s 10 30 90`으로 수행하였다.

두 실행 모두 목표각은 +90°, 속도 상한은 30°/s로 동일하게 설정하였고, Ki와 Kd는 0으로 유지하여 Kp만 변경하였다.

부하와 기구의 이동 여유는 동일한 실험 환경을 유지하여 비교하였다.
---
### 2. 같은 경과 시간에서 현재각 비교

| 경과 시간 | 실행 A 현재각 (Kp=1) | 실행 B 현재각 (Kp=10) | 목표각 | A 목표 초과 | B 목표 초과 |
|---|---:|---:|---:|---|---|
| 약 4.0 s | 52.207° | 52.471° | 90° | 없음 | 없음 |
| 약 5.0 s | 75.850° | 81.738° | 90° | 없음 | 없음 |
| 약 5.4 s | 80.596° | 91.494° | 90° | 없음 | 있음 (+1.494°) |

4.0초에서는 실행 A가 52.207°, 실행 B가 52.471°로 두 실행의 현재각이 거의 비슷하였다.

5.0초에서는 실행 A가 75.850°인 반면 실행 B는 81.738°까지 이동하여 Kp가 큰 실행 B가 목표각에 더 빠르게 접근하였다.

약 5.4초에서는 실행 A가 80.596°로 아직 목표각 90°에 도달하지 않은 반면, 실행 B는 91.494°까지 이동하여 목표각을 약 1.494° 초과하였다.

### 3. 결과 해석

오차가 큰 초반에는 두 실행 모두 계산된 P 제어 출력이 속도 상한인 30°/s에 의해 제한되기 때문에 Kp가 달라도 실제 모터의 움직임 차이는 크지 않았다.

그러나 목표각에 가까워지면서 실행 A는 작은 Kp로 인해 제어 출력이 일찍 감소한 반면, 실행 B는 Kp가 10으로 크기 때문에 같은 위치 오차에 대해 더 큰 제어 출력을 유지하여 목표에 더 빠르게 접근하였다.

실행 B에서는 약 5.4초에 현재각이 91.494°가 되어 목표각 90°를 1.494° 초과하였다.

이때 오차는 다음과 같다.

`오차 = 목표각 - 현재각`

`오차 = 90° - 91.494° = -1.494°`

오차가 음수가 되면서 P 제어 출력도 음수 방향으로 바뀌었고, 로그에서 `u_deg_s = -15.114°/s`가 나타났다.

따라서 모터가 반대 방향으로 보정되면서 이후 현재각은 약 90.352°, 90.088°, 90.000°로 다시 목표각에 가까워졌다.

반면 실행 A는 같은 시점에 현재각이 80.596°였기 때문에 실행 B보다 목표 접근 속도는 느렸지만 해당 시점에서는 목표각을 초과하지 않았다.

---

### 4. 변경한 Kp의 위치

이번 실험에서 변경한 Kp는 **Dynamixel 내부의 Position P Gain이 아니라 OpenCR 프로그램에서 계산하는 위치 제어 게인**이다.

OpenCR에서는 다음과 같이 위치 오차를 이용하여 목표 속도를 계산한다.

`위치 오차 = 목표 위치 - 현재 위치`

`P 제어 출력 = Kp × 위치 오차`

따라서 전체적인 제어 흐름은 다음과 같다.

`목표 위치 → 위치 오차 계산 → OpenCR의 P 제어 → 목표 속도 → Dynamixel → 모터`

이번 실험에서 변경한 Kp는 위 과정 중 **OpenCR의 P 제어 계산에 사용되는 Kp**이다.

Dynamixel 내부의 위치 PID 게인을 변경한 것은 아니며, Dynamixel은 OpenCR로부터 전달받은 목표 속도를 이용해 모터를 구동한다.

---

### 5. I항과 D항의 역할

- **I항(Integral)**: 위치 오차를 시간에 따라 누적하여 P 제어만으로 남을 수 있는 지속적인 정상상태 오차를 줄이는 역할을 한다.
- **D항(Derivative)**: 오차 또는 움직임의 변화 속도에 반응하여 급격한 움직임, 진동 및 오버슈트를 억제하는 제동 역할을 한다.

이번 실험에서는

- `Ki = 0`
- `Kd = 0`

으로 설정되어 있으므로 I항과 D항은 사용하지 않고 P항만을 이용하여 제어하였다.

---

## 문제 4. 제어와 통신의 역할 해석

### 1. 전체 구조와 데이터 흐름

#### 목표값 전달 방향

```
PC ROS2 노드
   │
   │ /motor/target
   │ 목표각 30.0°
   ▼
micro-ROS Agent
   │
   ▼
OpenCR
   │
   │ 10 ms마다 위치 제어 계산
   ▼
Dynamixel 모터
측정값 전달 방향

Dynamixel 내부 엔코더
   │
   │ 현재 위치 측정
   ▼
OpenCR
   │
   │ /motor/state
   │ 100 ms마다 현재각 발행
   ▼
micro-ROS Agent
   │
   ▼
PC ROS2 노드
```
---
송수신 역할
- 측정: Dynamixel 내부 엔코더

- 처리 및 송신: OpenCR

- 중계: micro-ROS Agent

- 수신: PC ROS2 노드

- 값의 의미: 현재각

- 단위: degree(°)

---

OpenCR 제어 계산주기 : 10ms
상태 발행 주기: 100ms
상태 한번 발행하는 동안 수행되는 제어 계산 횟수: 100/10 = 10회
제어 계산 : 10 ms마다 = 100 Hz
상태 발행 : 100 ms마다 = 10 Hz

---
통신이 끊겼는데 마지막으로 받은 목표 명령을 계속 사용하는 것은 위험할 수 있으므로, OpenCR에서는 일정 시간 동안 새로운 명령을 받지 못하면 기존 명령을 계속 실행하지 않는 통신 타임아웃(Watchdog) 정책을 사용할 수 있다. 이러한 정책이 필요한 이유는 PC와 OpenCR 사이의 통신이 끊긴 상태에서 마지막으로 수신한 이동 명령을 계속 수행하면 모터가 사용자의 의도와 관계없이 움직여 충돌이나 장비 손상이 발생할 수 있기 때문이다.
