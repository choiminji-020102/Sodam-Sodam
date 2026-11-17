# 소담소담

센서 장비 없이 앱과 AI로 소규모 낙농가를 돕는 서비스<br>
**3인 팀 · AI 파트 단독** · 2025.04 ~ 2025.07 · 농림축산식품 공공데이터 활용 창업경진대회 **우수상**

> 이 저장소는 팀 원본 저장소(BlackCows-Team)를 포크해 하위 모듈로 묶은 포트폴리오용 저장소입니다.
> 원본의 협업 기록은 그대로 보존했고, 아래에는 제가 맡은 **AI 파트**를 중심으로 정리했습니다.

<br>

## 배경

소규모 젖소농가의 폐업이 빨라지고 있습니다. 팀원의 외삼촌이 운영하던 20두 농장도 문을 닫았습니다.

| 원인 | 해결 방향 |
|---|---|
| 스마트팜 도입이 어려운 1위 원인: 설치비용 확보 | 센서 없이, 사진과 기록만으로 진단·예측하는 AI |
| 후계농 정착이 어려운 1위 원인: 영농 기술 습득 부족 | 낙농 지식을 바로 묻고 답할 수 있는 챗봇 |

<br>

## 내 역할: AI 파트 단독

데이터 수집·전처리부터 모델 학습·비교, 챗봇 라우팅 설계, API·앱 연동까지 AI 기능 전반을 맡았습니다.

| 기능 | 내용 |
|---|---|
| **챗봇 '소담이'** | 낙농 전문 문서를 근거로 답하는 RAG 챗봇, 질문을 4유형으로 나눠 LangGraph로 분기 |
| **럼피스킨병 진단** | 피부 병변 사진으로 정상 / 럼피스킨 의심을 분류하는 YOLOv8 모델 학습 (앱에는 연결하지 않고 목업 화면으로 표시) |
| **착유량·유방염 예측** | 착유 기록으로 유방염 위험도 분류, 착유량 회귀 예측 (Random Forest) |

<!-- 앱 화면 스크린샷 추가 위치
<p align="center">
  <img src="docs/images/chatbot.png" width="30%">
  <img src="docs/images/diagnosis.png" width="30%">
  <img src="docs/images/prediction.png" width="30%">
</p>
-->

<br>

## 챗봇 '소담이'

### 왜 RAG인가

일반 챗GPT를 그대로 쓰면 근거 없는 답(환각)이 나오고, 최신 자료가 반영되지 않으며, 전문 용어를 일상 용어와 혼동합니다.
그래서 **낙농 전문 문서를 검색해, 근거와 출처를 붙여 답하도록** 만들었습니다.

### 동작 흐름

```
질문 → 의도 분류(classifier)
        ├─ 낙농 전문 지식 → rag_response        : 문서 검색 → 답변 + 출처
        ├─ 농장 기록 조회 → cow_info_graph      : 정규식 + Firestore 조회
        ├─ 일반 대화     → general_response    : 이전 대화 맥락을 이어 응답
        └─ 무관 질문     → irrelevant_response : 고정 문구로 안내
```

### 설계

- **RAG 파이프라인**
  - 문서: 낙농 전문 문서(질병도감·백과 등), PDF는 PyMuPDF로 텍스트 추출
  - 분할: `RecursiveCharacterTextSplitter` (300자, overlap 50)
  - 임베딩: OpenAI `text-embedding-ada-002`
  - 저장: Chroma (로컬 sqlite, 전역 변수로 캐시) — pip 설치만으로 로컬에서 바로 쓰고 LangChain과 호환되어 소규모 RAG에 적합
  - 검색: `similarity_search_with_score`, 가까운 문서 3개
- **노드마다 규칙을 다르게** (모델: gpt-4o-mini)

  | 노드 | temperature | 규칙 |
  |---|---|---|
  | 질문 분류 | 0 | 4개 값 중 하나만 출력, 그 외 출력은 irrelevant로 처리 |
  | 문서 기반 답변 | 0.3 | 참고 자료를 우선 쓰고, 없으면 일반 지식으로 답하되 답변 아래 출처 표기 |
  | 일반 대화 | 0.5 | 40~70대 사용자도 이해하도록 쉬운 단어, 질문 언어에 맞춰 응답 |

- **출처를 붙여 신뢰도 확보**: 코드가 검색된 문서 조각마다 `[출처: 파일명 | 유사도]`를 붙여 넘기고, LLM은 답변 아래에 출처를 적고, 사용자는 화면에서 근거 문서를 확인

<br>

## 예측 모델

| 모델 | 방식 | 비고 |
|---|---|---|
| 유방염 위험도 | Random Forest 분류, 5개 피처 | 체세포수 유무에 따라 규칙 · ML 이중 로직 |
| 착유량 | Random Forest 회귀, 8개 피처 | |
| 럼피스킨병 진단 | YOLOv8 분류 | 모델 학습까지 진행, 앱에는 목업 화면으로 표시 |

- **유방염**: 착유 기록 대부분에 체세포수 값이 없었습니다. 그래서 값이 있으면 규칙으로 판정하고, 없으면 ML로 추정하는 이중 로직을 만들었습니다.
- **피처 줄이기**: Accuracy는 높은데 일부 등급의 Recall이 무너지는 문제를 발견했습니다. 등급을 4단계에서 3단계로 다시 설계하고, Ablation Study로 피처를 줄였습니다.

<br>

## 기술 스택

| 구분 | 기술 |
|---|---|
| LLM · RAG | LangChain, LangGraph, OpenAI (gpt-4o-mini, Embedding), ChromaDB |
| ML · CV | scikit-learn, YOLOv8, pandas |
| Server | FastAPI, Firebase Firestore |
| 팀 전체 | Flutter |

<br>

## 저장소 구성

| 모듈 | 내용 |
|---|---|
| [`ai-server`](./ai-server) | FastAPI 예측 모델 서버 (유방염 · 착유량) |
| [`server`](./server) | FastAPI 백엔드 + 챗봇 '소담이' (`services/chatbot_langgraph`) |
| [`app`](./app) | Flutter 앱 |

```bash
# 하위 모듈까지 함께 받기
git clone --recurse-submodules https://github.com/choiminji-020102/Sodam-Sodam.git
```