# 2025ESWContest_free_1091

## 🔖 Intro

WINEQUEEN은 **버튼 한 번**으로 와인병을 자동 밀봉·진공 보관하는 디바이스입니다.  
YOLO 비전 + 로봇팔 + 기압 센서 피드백 제어로, 마지막 한 방울까지 신선하게 보관합니다.

## 💡 Inspiration

개봉 후 와인은 공기 접촉으로 빠르게 산화됩니다. 수동 펌핑·스토퍼는 번거롭고 진공 품질이 일정하지 않아 신선도 유지에 한계가 있죠.  
“**더 간편하고, 더 완벽하게 보관할 수 없을까?**”라는 질문에서 WINEQUEEN은 시작되었습니다.

## 📸 Overview

<img src="images/overview.png" width="700" height="400">

<br>

1. 사용자가 와인병을 거치
2. 카메라로 병·뚜껑 위치 인식 (YOLO)
3. 로봇팔이 뚜껑 픽업 → 병 입구 정렬
4. Z축 하강으로 뚜껑 삽입
5. 진공 노즐 위치 정렬
6. 진공 펌프 ON → 기압 센서로 목표 압력 도달 확인
7. 펌프 OFF 및 초기 위치 복귀
8. UI에 진행 상태·완료 표시

## 👀 Main feature

- ### 1️⃣ AI 모델로 병·뚜껑 정렬

  온디바이스 실시간 동작을 위해 320×320 입력으로 경량 YOLOv8을 사용합니다.  
  외부 이벤트(버튼/센서) 기반으로만 추론을 수행해 불필요한 연산을 줄이고, **뚜껑 중심점/병 입구**를 검출해 X축 미세 이동으로 정렬합니다.

- ### 2️⃣ 디바이스 동작 (State Machine)

  #### 1. 뚜껑 픽업 & 정렬

  <img src="images/state_machine_pick_align.png" width="700" height="400">

  <br>
  <details>
    <summary>state 상세설명 ⏬</summary>

  - **PICK_CAP**  
    전자석 ON → 뚜껑 보관소로 이동 → Z 하강 픽업 → 상승.
  - **FIND_BOTTLE**  
    병 예상 위치로 이동, 카메라 시야 내 대상 확보.
  - **ALIGN_CENTER**  
    YOLO 중심 오차를 계산하여 X축 미세 이동 반복 → 중앙 정렬 완료(ACK).
  - **INSERT_CAP**  
   Z 하강으로 뚜껑 삽입 → 전자석 OFF → Z 상승.
  </details>

  #### 2. 진공 시퀀스

  <img src="images/state_machine_vacuum.png" width="700" height="400">

  <br>
  <details>
    <summary>state 상세설명 ⏬</summary>

  - **MOVE_TO_NOZZLE**  
    진공 노즐 위치 정렬 → Z 하강 밀착.
  - **VACUUM_ON**  
    펌프 ON, 기압 센서 스트리밍으로 목표치 도달 감시.
  - **HOLD/STOP**  
    목표 압력 도달 시 펌프 OFF → 안정화 대기(소정 시간/히스테리시스).
  - **RETURN_HOME**  
   Z 상승, 로봇팔 홈 복귀 → 완료 신호 전송.
  </details>

  #### 3. 개봉(Release) 시퀀스

  <img src="images/state_machine_release.png" width="700" height="400">

  <br>
  <details>
    <summary>state 상세설명 ⏬</summary>

  - **ALIGN_CENTER**  
    병 입구 정렬(동일 방식).
  - **EQUALIZE_PRESSURE**  
    릴리즈 밸브 개방으로 내부 압력 정상화 확인.
  - **REMOVE_CAP**  
   전자석으로 뚜껑 픽업 → 보관소로 이동해 해제.
  </details>

- ### 3️⃣ 서버 및 디스플레이

  #### 1. 서버(FastAPI)

  <img src="images/server.png" width="700" height="400">

  <br>
  <details>
    <summary>module 상세설명 ⏬</summary>

  - **API(Controller)**: REST/WebSocket 엔드포인트, 상태 조회·명령 수신.
  - **YOLO Inference**: 프레임 캡처, 추론, 중심 오차 계산.
  - **Control Logic**: 정렬/시퀀스 상태 머신, 타이밍·안전 인터록.
  - **Serial Bridge**: Arduino(UART)와 명령/ACK 교환, 재시도·타임아웃.
  - **Pressure Monitor**: 목표 압력 도달, 히스테리시스, 이동평균 필터.
  </details>

  #### 2. Device UI(React)

  <img src="images/ui.png" width="700" height="400">

  <br>
  WebSocket/HTTP로 **현재 단계, 압력, 추론 결과**를 실시간 표시합니다.  
  사용자는 버튼(밀봉/개봉/정지)만으로 전체 공정을 조작할 수 있습니다.

## Environment

### Embedded

<img src="https://img.shields.io/badge/Raspberry%20Pi-A22846?style=for-the-badge&logo=raspberrypi&logoColor=white">
<img src="https://img.shields.io/badge/Arduino-008184?style=for-the-badge&logo=arduino&logoColor=white">
<img src="https://img.shields.io/badge/C/C++-00599C?style=for-the-badge&logo=cplusplus&logoColor=white">

### Backend

<img src="https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white">
<img src="https://img.shields.io/badge/FastAPI-009688?style=for-the-badge&logo=fastapi&logoColor=white">
<img src="https://img.shields.io/badge/OpenCV-5C3EE8?style=for-the-badge&logo=opencv&logoColor=white">
<img src="https://img.shields.io/badge/Ultralytics%20YOLOv8-FF4C00?style=for-the-badge&logoColor=white">

### Frontend

<img src="https://img.shields.io/badge/TypeScript-3178C6?style=for-the-badge&logo=typescript&logoColor=white">
<img src="https://img.shields.io/badge/React-61DAFB?style=for-the-badge&logo=react&logoColor=111827">
<img src="https://img.shields.io/badge/Vite-646CFF?style=for-the-badge&logo=vite&logoColor=white">
<img src="https://img.shields.io/badge/WebSocket-000000?style=for-the-badge&logo=socketdotio&logoColor=white">

## File Architecture

```
2025ESWContest_자유공모_1091_와인퀸_파일구조
.
├─ HW/                                   # 하드웨어(아두이노/메카/회로)
│  ├─ arduino/
│  │  ├─ src/
│  │  │  ├─ HW_Control.ino               # 메인 펌웨어 (시리얼 프로토콜 준수)
│  │  │  └─ modules/                     # 모터/전자석/압력센서 서브모듈
│  │  └─ include/
│  │     ├─ Constants/
│  │     ├─ CUP/
│  │     ├─ Queue/
│  │     ├─ StepperMulti/
│  │     └─ Waterpump/
│  ├─ mechanics/                         # 3D 모델/출력물
│  │  ├─ cad/                            # Fusion 360 등 원본
│  │  └─ prints/                         # STL, gcode
│  ├─ electronics/                       # 회로/배선
│  │  ├─ schematics/                     # 회로도
│  │  └─ bom.csv                         # 자재 목록(BOM)
│  └─ protocol/                          # HW<->Backend 공통 프로토콜(단일 소스)
│     ├─ messages.yaml                   # 명령/응답/상태 코드 정의 (단일 진실)
│     ├─ generate_protocol.py            # yaml → python(백엔드)/c(아두이노) 생성 스크립트
│     ├─ templates/
│     │  ├─ arduino_protocol.h.j2
│     │  └─ backend_protocol.py.j2
│     └─ README.md
│
└─ Display/                              # 사용자에 보이는 모든 소프트웨어
   ├─ Backend/                           # FastAPI (비전/제어/시리얼/WS)
   │  ├─ app/
   │  │  ├─ main.py                      # 엔트리 (uvicorn)
   │  │  ├─ api/                         # REST/WebSocket 엔드포인트
   │  │  │  ├─ routes.py
   │  │  │  └─ ws.py                     # 실시간 상태/로그 스트림
   │  │  ├─ control/                     # 상태머신/시퀀스 제어
   │  │  │  ├─ state_machine.py
   │  │  │  └─ alignment.py
   │  │  ├─ vision/                      # YOLO 추론/후처리
   │  │  │  ├─ infer.py
   │  │  │  └─ postprocess.py
   │  │  ├─ serial_bridge/               # Arduino UART 브릿지
   │  │  │  ├─ uart.py
   │  │  │  └─ protocol.py               # (HW/protocol에서 생성된 파일)
   │  │  ├─ schemas/                     # DTO/Pydantic
   │  │  │  └─ types.py
   │  │  ├─ services/                    # 비즈 로직(압력 모니터 등)
   │  │  ├─ utils/                       # 공통 유틸(로그/시간/필터)
   │  │  └─ config.py                    # 환경변수/포트/시리얼 경로 등
   │  ├─ models/
   │  │  └─ weights/                     # YOLO 가중치 (Git LFS 권장)
   │  │     ├─ best.pt
   │  │     ├─ best_wCrop.pt
   │  │     ├─ yolov8n.pt
   │  │     ├─ yolov8n_100.pt
   │  │     └─ yolov8n_200.pt
   │  ├─ requirements.txt
   │  ├─ README.md
   │  ├─ .gitignore
   │  └─ __pycache__/                    # 실행 시 생성
   │
   └─ Frontend/                          # React + TS (Vite)
      ├─ index.html
      ├─ src/
      │  ├─ assets/                      # 정적 리소스(SVG, 이미지)
      │  │  ├─ chevron.svg
      │  │  ├─ Icon.svg
      │  │  ├─ Wine_1.svg
      │  │  └─ Wine_2.svg
      │  ├─ constants/
      │  │  └─ constants.ts              # API/WS 엔드포인트, 테마 상수 등
      │  ├─ lib/
      │  │  └─ ws.ts                     # WebSocket 유틸/싱글턴
      │  ├─ pages/                       # 라우트 단위 화면
      │  │  ├─ Splash.tsx
      │  │  ├─ MainPage.tsx
      │  │  ├─ ConfirmSeal.tsx
      │  │  ├─ ConfirmOpen.tsx
      │  │  ├─ OpenWine.tsx
      │  │  ├─ CloseWine.tsx
      │  │  └─ Explation.tsx             # (필요 시 Explanation.tsx로 변경 권장)
      │  ├─ router/
      │  │  └─ Router.tsx
      │  ├─ styles/
      │  │  └─ index.css
      │  ├─ App.tsx                      # 앱 루트/레이아웃
      │  └─ main.tsx                     # React 엔트리
      └─ yarn.lock

```

## Video

- (시연 영상 링크 또는 썸네일 이미지)

## Team Member

<img src="assets/team.png" width="300" height="300">

<br>

| 팀원             | 역할                          |
| ---------------- | ----------------------------- |
| **이준형(팀장)** | 하드웨어 제어/시퀀스·통합     |
| **최준서**       | React UI/스트리밍·상태 시각화 |
| **최민혁**       | 하드웨어 설계/제작            |
| **김용진**       | FastAPI 서버/시리얼 브릿지    |
