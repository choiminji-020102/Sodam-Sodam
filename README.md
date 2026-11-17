# 소담소담

AI 젖소 관리 앱

2025.04 ~ 2025.07 · 3인 팀 · 농림축산식품 공공데이터 활용 창업경진대회 **우수상**

농림축산식품 공공데이터 활용 창업경진대회 출품을 위해 만든 프로젝트입니다.

## 저장소 구성

이 저장소는 프로젝트 소개용입니다. 코드는 아래 세 저장소에 있으며, 각각 팀 원본 저장소를 브랜치와 커밋 기록까지 그대로 포크한 것입니다.

| 폴더 | 내용 | 포크 | 원본 |
|---|---|---|---|
| `ai-server/` | AI 예측 모델 서버 | [sodam-ai-server](https://github.com/choiminji-020102/sodam-ai-server) | [BlackCows-Team/blackcows-ai-server](https://github.com/BlackCows-Team/blackcows-ai-server) |
| `server/` | 백엔드 + RAG 챗봇 '소담이' | [sodam-server](https://github.com/choiminji-020102/sodam-server) | [BlackCows-Team/blackcows-server](https://github.com/BlackCows-Team/blackcows-server) |
| `app/` | Flutter 앱 | [sodam-app](https://github.com/choiminji-020102/sodam-app) | [BlackCows-Team/BlackCows](https://github.com/BlackCows-Team/BlackCows) |

## 담당 — AI 파트 전체

데이터 수집 · 전처리부터 모델 학습 · 비교, 챗봇 라우팅 설계, 서비스 연동까지 맡았습니다.
AI 서버의 모델은 직접 학습시켰으며, 서버 저장소 커밋은 팀원이 올렸습니다.

**RAG 챗봇 '소담이'** (`server/services/chatbot_langgraph`)
- Intent Classifier + LangGraph 조건부 라우팅 4분기 (RAG 응답, 개체 정보 조회, 일반 응답, 범위 밖 질문)
- 개체 번호가 없으면 처리되지 않던 문제를 서브그래프 5개 노드로 분리해 해결
- 배포 환경에서 Chroma 생성 실패와 캐시 미작동 문제 해결

**예측 모델**
- 럼피스킨 진단: YOLOv8 분류 모델 학습 (앱에는 연결하지 않고 목업 화면으로 표시)
- 유방염 예측: RandomForest — 체세포수 값이 없는 기록이 대부분이라, 값이 있으면 규칙으로 판정하고 없으면 ML로 추정하는 이중 로직으로 처리
- 착유량 예측: RandomForestRegressor
- 예측 6종 중 2종은 공공데이터로 직접 학습, 4종은 같은 파이프라인으로 입력 변수까지 설계

## 기술 스택

AI: Python, LangGraph, ChromaDB, OpenAI API, YOLOv8, scikit-learn, XGBoost, pandas
Backend: FastAPI, Firebase Firestore
App: Flutter
