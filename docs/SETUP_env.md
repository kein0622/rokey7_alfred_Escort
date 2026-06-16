# 실행 환경 설정 (Environment Setup)

이 프로젝트를 **새 PC / 다른 계정**에서 돌릴 때 필요한 ROS2 환경 설정 정리.
개인 `~/.bashrc`에 의존하지 않도록, 이식 가능한 핵심만 모았다.

---

## 1. 필수 환경 변수

```bash
export ROS_DOMAIN_ID=2                          # 로봇과 반드시 동일 (팀 공통값)
export RMW_IMPLEMENTATION=rmw_cyclonedds_cpp     # DDS 구현 통일
```

> ⚠️ `ROS_DOMAIN_ID`가 로봇과 다르면 토픽이 보이지 않는다. 서로 다른 값을 두 번
> export 하지 말 것 (마지막 값이 적용됨).

## 2. 소싱 순서 (base → overlay)

```bash
source /opt/ros/humble/setup.bash                # 1) ROS2 base
source ~/alfred_ws/install/setup.bash            # 2) 이 워크스페이스 overlay (colcon build 후)
source /etc/turtlebot4_discovery/setup.bash      # 3) TurtleBot4 디스커버리 서버
```

> ⭐ **디스커버리 서버 설정이 로봇과 일치해야 한다.** 불일치 시 노드는 정상 기동하지만
> 데이터가 **조용히 0건** 들어오는 함정이 있다 (에러 없이 빈 화면). monitor_server에
> 데이터가 안 들어오면 가장 먼저 이 설정(`ROS_DISCOVERY_SERVER`)부터 확인할 것.

## 3. 빌드

```bash
cd ~/alfred_ws
colcon build --symlink-install
source install/setup.bash
```

astra_camera(카메라 드라이버) 빌드용 시스템 의존성:

```bash
sudo apt install ros-humble-camera-info-manager ros-humble-image-publisher libuvc-dev
# 카메라를 안 쓰면 제외 가능:
colcon build --symlink-install --packages-skip astra_camera
```

## 4. 관제 서버 (monitor_server)

```bash
cd ~/alfred_ws/monitor_server
FMS_BACKEND=sqlite python3 main.py        # 로컬 SQLite + 대시보드  http://localhost:5000
#   기본 로그인: admin / admin1234  (.env 의 FMS_ADMIN_* 로 변경)
FMS_BACKEND=supabase python3 main.py      # 외부(Vercel)용 write-only 펌프 (.env 의 SUPABASE_* 필요)
```

로봇 없이 데모 데이터 주입:

```bash
python3 tools/demo_patrol_lost_item.py
```

## 5. 자주 쓰는 alias (참고)

```bash
alias robot-nav='ros2 launch turtlebot4_navigation nav2.launch.py namespace:=/robot2'
alias robot-view='ros2 launch turtlebot4_viz view_robot.launch.py namespace:=/robot2'
alias ros-restart='ros2 daemon stop; ros2 daemon start'
```

---

> 개인 `~/.bashrc`에는 위 설정 외에 PC 전용 경로(CUDA, Isaac Sim 등)와 다른 워크스페이스
> 소싱이 섞여 있으므로, 그대로 복사해 쓰지 말고 위 항목만 옮길 것.
