# 🤖 ALFRED — 지하철 자율주행 안내·에스코트 로봇 시스템

> ROS2 기반 **다중 로봇 릴레이 에스코트** + **클라우드 관제** 통합 워크스페이스
>
> 지하철역에서 승객을 목적지까지 안내하는 다중 로봇 시스템. 음성/터치 키오스크,
> 보행 속도 추종, 화재·부상자·분실물 감지 자동 알림, 외부 실시간 관제를 지원합니다.

---

## 📌 프로젝트 개요

| 항목 | 내용 |
|------|------|
| **종류** | 팀 프로젝트 (ROKEY 7기 지능1 팀 Alfred) |
| **원본 저장소** | [`sungyu-sung/rokey_alfred`](https://github.com/sungyu-sung/rokey_alfred) |
| **이 저장소** | 팀 코드(main)와 본인 작업(관제 서버)을 **통합한 버전** · 개인 백업/포트폴리오용 |
| **기간** | <!-- 예: 2026.05 ~ 2026.06 --> |
| **로봇 플랫폼** | TurtleBot4 ×2 (robot2 / robot4), ROS2 Humble |

> ℹ️ 이 저장소는 팀 결과물을 포함합니다. 모든 원저자의 커밋 이력·작성자 정보는 그대로 보존되어 있습니다.

---

## 🙋 본인 기여 (My Contribution)

> 작성자: **kim (kein0622)**

- **외부 관제 서버 `monitor_server` 설계·구현**
  - ROS2 토픽 수신 → 상태/이벤트 저장 → 대시보드 API 제공
  - **백엔드 추상화**: 로컬 `SQLite` ↔ 클라우드 `Supabase` 를 환경변수(`FMS_BACKEND`) 한 줄로 전환
  - **Vercel 정적 대시보드** + Supabase 미러로 **외부에서 실시간 관제** (모바일 대응, 3D 시각화)
  - 로봇 상태/순찰 경로/이상감지(화재·분실물 등) 이벤트 시각화 및 조치완료 동기화
- **메인 코드 통합(merge)**: 팀 `main`(주행/비전/인터랙션/카메라 드라이버)과 관제 브랜치를 충돌 해결하며 병합, 빌드·데모 검증까지 수행
- WebRTC 로봇 카메라 영상 송신 노드 연동

---

## 📷 데모 & 스크린샷

> 아래 자리에 실제 캡처/영상을 채워 넣으세요. 이미지는 `docs/` 에 넣고 파일명을 맞추면 됩니다.

### 🎬 데모 영상
<!-- YouTube면 썸네일 클릭 시 재생 -->
[![데모 영상](docs/demo_thumbnail.png)](https://youtu.be/VIDEO_ID)

<!-- 또는 GIF: docs/demo.gif 저장 후 아래 주석 해제 -->
<!-- ![데모](docs/demo.gif) -->

### 🖥️ 관제 대시보드 (monitor_server)
| 실시간 로봇 상태 | 순찰 경로 · 이상감지 3D |
|:---:|:---:|
| ![대시보드](docs/screenshot_dashboard.png) | ![3D 시각화](docs/screenshot_viz.png) |

### 🤖 로봇 동작 / 키오스크
| 에스코트 시나리오 | 키오스크 UI |
|:---:|:---:|
| ![에스코트](docs/screenshot_escort.png) | ![키오스크](docs/screenshot_kiosk.png) |

---

## 🗺️ 시스템 구성

```
┌───────────────────────────────────────────┬──────────────────────────┐
│              관리자 PC 🖥️                  │  키오스크 노트북 💻 (층별) │
│   (ROS2 허브 + 관제)                        │     (터치/음성 UI)        │
│   alfred_bridge / alfred_driving            │   alfred_interaction      │
│   alfred_vision / monitor_server / fms_server│   (Node.js · React)      │
└───────────────────────────────────────────┴──────────────────────────┘
        │  ROS2 DDS (토픽/액션)                    │ WebSocket (rosbridge/state_ws)
        └──────────────► 클라우드 관제 (Supabase · Vercel) ◄── 외부 모니터링
```

| 로봇 | 역할 | 층 |
|------|------|----|
| **robot2** 🟦 | 1층 순찰 + 에스코트 | 1F |
| **robot4** 🟩 | 2층 목적지 안내 | 2F |

**에스코트 시나리오**: 1F→1F(robot2 단독) · 2F→2F(robot4 단독) · **1F→2F(릴레이: robot2→robot4)**

---

## 📦 주요 구성 요소

| 디렉토리 | 역할 |
|----------|------|
| `fms_server/` | 다중 로봇 릴레이 에스코트 **FMS 서버** (미션·상태 조정) |
| `monitor_server/` | **외부 관제 서버** (대시보드 API · SQLite/Supabase · Vercel) ⭐ |
| `src/alfred_driving/` | 주행·순찰·Nav2·아루코 추종 |
| `src/alfred_vision/` + `astra_camera/` | 비전 탐지(화재/부상자/분실물) + 카메라 드라이버 |
| `src/alfred_interaction/` | 키오스크 UI (음성/터치, React) |
| `src/alfred_bridge/` · `alfred_bringup/` · `alfred_interfaces/` | 상태 브릿지 · 기동 · 커스텀 메시지 |
| `viz_3d/` | 3D 플릿 시각화 |

> 패키지별 상세 구조·인터페이스 정의는 [`README_team.md`](README_team.md)(팀 원본 문서) 참고.

---

## 🚀 빠른 시작

> 📄 상세 환경 설정(ROS_DOMAIN_ID·RMW·디스커버리 서버·빌드 의존성)은 [`docs/SETUP_env.md`](docs/SETUP_env.md) 참고.

```bash
# 1) ROS2 워크스페이스 빌드 (ROS2 Humble)
colcon build --symlink-install
source install/setup.bash

# 2) (관제) 로컬 SQLite 대시보드
cd monitor_server
FMS_BACKEND=sqlite python3 main.py          # http://localhost:5000  (admin / admin1234)

# 3) (관제) 로봇 없이 데모 데이터 주입
python3 tools/demo_patrol_lost_item.py
```

> astra_camera 빌드에는 시스템 의존성 필요:
> `sudo apt install ros-humble-camera-info-manager ros-humble-image-publisher libuvc-dev`
> 카메라를 안 쓰면 `colcon build --packages-skip astra_camera` 로 제외 가능.

---

## 🛠️ 기술 스택

`ROS2 Humble` · `Python` · `Nav2` · `OpenCV/YOLO` · `Flask` · `SQLite` · `Supabase` · `Vercel` · `React/TypeScript` · `WebRTC` · `rosbridge`

---

## 📄 라이선스 · 출처

- 본 저장소는 팀 프로젝트 [`sungyu-sung/rokey_alfred`](https://github.com/sungyu-sung/rokey_alfred) 에서 파생되었습니다.
- 라이선스는 [`LICENSE`](LICENSE) 파일을 따릅니다.
- 공개 시 팀원 동의 및 출처 표기를 권장합니다.
