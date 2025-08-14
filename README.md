# HosungFC — 축구팀 관리자 시스템
JSP + Java(서블릿/DAO) + Oracle 기반의 **팀 운영/전술 추천 관리자 페이지**  
포메이션 **드래그&드롭**, **Kakao Map** 라이벌 분석, **🤗 Hugging Face + 규칙 엔진** 전술 코멘트

---

## #0. 프로젝트 개요
**프로젝트 이름**: HosungFC (축구팀 관리 & 전술 추천 시스템)

**프로젝트 목적 및 개요**  
아마추어 감독/코치를 위해 **선수·포메이션·라이벌 분석·전술 추천**을 한 곳에서 제공하는 관리자 사이트를 구축.  
선수 데이터와 상대 전술 정보를 바탕으로 **규칙 엔진**에서 1차 추천안을 만들고, **Hugging Face Inference API**로 전술 설명을 **요약·보완**하여 의사결정을 돕는다.  
드래그&드롭 포메이션 편집, Kakao Map 기반 라이벌 정보 시각화로 **운영 효율**과 **가독성**을 동시에 확보한다.

**프로젝트 기간**: 2025.05.30 ~ 2025.07.09

---

## 기술 스택

### 프론트엔드 (View / JSP 기반)
- **뷰엔진**: JSP (JavaServer Pages), HTML5  
- **템플릿 태그**: JSTL, EL  
- **스타일링**: Bootstrap, CSS3  
- **JavaScript**: Vanilla JS (필드 드래그&드롭), jQuery  
- **지도**: Kakao Map JavaScript API

![HTML5](https://img.shields.io/badge/HTML5-E34F26?style=for-the-badge&logo=html5&logoColor=white)
![CSS3](https://img.shields.io/badge/CSS3-1572B6?style=for-the-badge&logo=css3&logoColor=white)
![JavaScript](https://img.shields.io/badge/JavaScript-ES6+-F7DF1E?style=for-the-badge&logo=javascript&logoColor=white)
![Bootstrap](https://img.shields.io/badge/Bootstrap-563D7C?style=for-the-badge&logo=bootstrap&logoColor=white)
![KakaoMap](https://img.shields.io/badge/Kakao%20Map-API-FFCD00?style=for-the-badge)

### 백엔드
- **아키텍처**: Java Servlet + DAO(또는 Service/DAO 계층), JSP MVC2  
- **언어**: Java  
- **ORM/매퍼**: (선택) MyBatis 또는 JDBC Template  
- **AI 연동**: 🤗 Hugging Face Inference API (GPT-2 계열 요약/코멘트)  
- **API 스타일**: JSP + Controller(Servlet) + Ajax

![Java](https://img.shields.io/badge/Java-007396?style=for-the-badge&logo=java&logoColor=white)
![JSP](https://img.shields.io/badge/JSP-MVC2-2D4470?style=for-the-badge)
![HuggingFace](https://img.shields.io/badge/Hugging%20Face-Inference%20API-FFCC4D?style=for-the-badge)

### 데이터베이스
- **DB**: Oracle (또는 MySQL)  
- **주요 테이블**:  
  `player`, `formation`, `formation_position`, `rival_club`, `region`, `player_analysis`, `player_tactic_recommendation`  
- 포지션 좌표, 상대 전술, 예상 승률 등 **의사결정용 필드** 보유

![Oracle](https://img.shields.io/badge/Oracle_DB-F80000?style=for-the-badge&logo=oracle&logoColor=white)

### 빌드 도구 / 개발 환경
- **IDE**: Eclipse 또는 VS Code  
- **빌드**: Maven  
- **서버**: Apache Tomcat  
- **버전 관리**: Git / GitHub  
- **패키징**: WAR

![Maven](https://img.shields.io/badge/Maven-C71A36?style=for-the-badge&logo=apachemaven&logoColor=white)
![Tomcat](https://img.shields.io/badge/Apache_Tomcat-F8DC75?style=for-the-badge&logo=apachetomcat&logoColor=black)
![GitHub](https://img.shields.io/badge/GitHub-181717?style=for-the-badge&logo=github&logoColor=white)

---

## #1. 설명

### 시스템 아키텍처 (Mermaid)
```mermaid
graph LR
  %% 스타일
  classDef ui  fill:#f5f5f5,stroke:#888,color:#222
  classDef svc fill:#e8f4ff,stroke:#4a90e2,color:#0b3a66
  classDef db  fill:#eef9f0,stroke:#3fbf6f,color:#0d5132
  classDef ext fill:#fffbe6,stroke:#f2c744,color:#333

  %% 클라이언트
  subgraph Client["클라이언트 (브라우저)"]
    UI[JSP · HTML/CSS/JS<br/>jQuery/Vanilla · Drag&Drop]:::ui
    MAP[Kakao Map JS SDK<br/>(지도·마커·모달)]:::ext
  end

  %% 앱 서버
  subgraph App["웹 애플리케이션 (Tomcat / JSP·Servlet)"]
    F[Filter/Interceptor<br/>세션·권한검사]:::svc
    CTRL[Controller/Servlet<br/>/login · /players · /formations ·<br/>/rivals · /ai/recommend]:::svc
    SRV[Service<br/>검증·트랜잭션·오케스트레이션]:::svc
    RULES[규칙 엔진<br/>포지션/스타일/상대전술 매칭]:::svc
    DAO[DAO / MyBatis Mapper]:::svc
  end

  %% 리소스
  DB[(Oracle DB)<br/>player · formation · formation_position ·<br/>rival_club · region · player_analysis ·<br/>player_tactic_recommendation]:::db
  HF[(🤗 Hugging Face Inference API<br/>(GPT-2 계열, 코멘트/요약))]:::ext
  LOG[(로깅/모니터링)]:::ext

  %% 연결
  UI -- "AJAX(JSON) CRUD/조회/추천" --> F --> CTRL
  UI -. "프론트에서 직접 사용" .- MAP
  CTRL --> SRV --> DAO --> DB
  SRV --> RULES
  SRV -. "요약·보완(옵션)" .-> HF
  App -. "애플리케이션 로그/에러" .- LOG
```

### 시스템 흐름도 (AI 전술 추천)
```mermaid
sequenceDiagram
  autonumber
  participant U as 사용자
  participant B as 브라우저(JSP/JS)
  participant F as Filter/세션
  participant C as Controller(/ai/recommend)
  participant S as Service
  participant R as 규칙 엔진
  participant D as Oracle DB
  participant H as 🤗 HF API

  U->>B: 선수·상대 전술 선택 후 "전술 추천" 클릭
  B->>F: 요청 전송(세션/권한검사)
  F-->>B: 통과(미통과 시 로그인)
  B->>C: POST /ai/recommend {players, opponentTactics}
  C->>S: 파라미터 검증/오케스트레이션
  S->>D: 선수/분석/라이벌 데이터 조회
  D-->>S: 조회 결과
  S->>R: 규칙 매칭(포지션·스타일·상대전술)
  R-->>S: 1차 추천안(전술/포메이션 포인트)
  alt HF 사용
    S->>H: 프롬프트 전송(요약/코멘트 생성)
    H-->>S: 자연어 코멘트/요약
    S-->>C: 규칙 결과 + HF 코멘트 병합
  else HF 실패/미사용
    S-->>C: 규칙 결과만 반환(페일세이프)
  end
  C-->>B: JSON {추천 전술, 근거, 코멘트}
  B->>U: 모달/패널에 결과 렌더링
```

### 포메이션 저장 흐름도 (Drag&Drop → 좌표 저장)
```mermaid
flowchart TD
  A[필드에서 드래그&드롭으로<br/>선수 배치/수정] --> B[저장 클릭]
  B --> C[JS로 좌표/포지션 수집<br/>(JSON payload)]
  C --> D[POST /formations/{id}/positions]
  D --> E[Controller: 인증/검증]
  E --> F[Service: 트랜잭션 시작]
  F --> G[DAO: 기존 포지션 삭제/업데이트]
  G --> H[(Oracle DB)<br/>formation_position 커밋]
  H --> I[Service: 성공 응답 생성]
  I --> J[브라우저: 토스트/재렌더]
  E -->|검증 오류| X[400/422 JSON 에러<br/>UI에 메시지 표시]
  F -->|DB 오류| Y[500 JSON 에러<br/>재시도/로그 확인]
```

**AI 추천 흐름(요약)**  
1) DB에서 선수 스타일/강점/약점 + 상대 전술 조회  
2) **규칙 엔진**으로 1차 추천(포지션/스타일/상대 전술 매칭)  
3) **🤗 HF Inference API (GPT-2)** 호출 → 코멘트/설명 생성  
4) (2)+(3) 결과 병합하여 화면 노출 *(API 실패 시 2번만 노출)*

---

## #2. 요구 사항 분석
- **로그인/권한(선택)**: 관리자만 접근 가능하도록 보호(추가 예정 시)  
- **선수 관리**: 등록/수정/삭제, 이름/포지션 검색, 페이징 목록  
- **포메이션 관리**: 필드 배경 위 **드래그&드롭 배치**, 좌표/라벨 저장, 수정/삭제  
- **라이벌 분석**: Kakao Map 마커/지역 핀, 모달로 전적/전술/예상 승률/강점선수 등 상세 표시  
- **AI 전술 추천**: 규칙 엔진 결과를 기본으로, HF 요약/코멘트로 **보완**  
- **성능/안정성**: Ajax 표준 응답(JSON), 예외 메시지 규격화, 필드 검증

---

## #3. 워크플로우 영상
- 선수/포메이션: 드래그&드롭 배치/저장, 선수 검색/페이징  
- 라이벌 분석: 마커·지역 필터, 모달에서 전적/전술/예상 승률 확인  
- AI 전술 추천: 규칙 결과 + 🤗HF 요약 설명 병합 출력  
> (영상 업로드 시 `<video src=".../raw/main/assets/xxx.mp4" controls>` 형식 권장)

---

## 📄 데이터베이스 설계 (ERD · Mermaid)
```mermaid
erDiagram
  PLAYER ||--o{ FORMATION_POSITION : "배치"
  FORMATION ||--o{ FORMATION_POSITION : "좌표/라벨"
  RIVAL_CLUB }o--|| REGION : "소속"
  PLAYER ||--|| PLAYER_ANALYSIS : "분석(1:1)"
  PLAYER ||--o{ PLAYER_TACTIC_RECOMMENDATION : "전술추천(1:N)"
  RIVAL_CLUB }o--o{ PLAYER_TACTIC_RECOMMENDATION : "상대기반(옵션)"

  PLAYER {
    int       player_id PK
    varchar   name
    varchar   position
    date      birth
    varchar   foot
  }

  FORMATION {
    int       formation_id PK
    varchar   name
    datetime  created_at
  }

  FORMATION_POSITION {
    int       position_id PK
    int       formation_id FK
    int       player_id FK
    int       x
    int       y
    varchar   role_label
  }

  RIVAL_CLUB {
    int       rival_id PK
    varchar   name
    varchar   logo_url
    varchar   main_tactic
    float     winrate_pred
    int       region_id FK
  }

  REGION {
    int       region_id PK
    varchar   name
  }

  PLAYER_ANALYSIS {
    int       player_id PK,FK
    varchar   style
    varchar   strengths
    varchar   weaknesses
    varchar   recommended_tactics
  }

  PLAYER_TACTIC_RECOMMENDATION {
    int       rec_id PK
    int       player_id FK
    int       rival_id  FK
    varchar   tactic
    varchar   rationale
    datetime  created_at
  }
```

---

## 🔌 환경 변수 / 설정
`.env` 또는 서버 환경변수로 관리

```bash
# Kakao Map
KAKAO_MAP_APPKEY=여기에_키

# Hugging Face Inference API
HF_API_URL=https://api-inference.huggingface.co/models/gpt2
HF_API_TOKEN=hf_xxxxxxxxxxxxxxxxxxxxx
```
