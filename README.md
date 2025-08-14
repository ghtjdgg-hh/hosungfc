# HosungFC — 축구팀 관리자 시스템
JSP + Java(서블릿/DAO) + Oracle 기반의 **팀 운영/전술 추천 관리자 페이지**  
포메이션 **드래그&드롭**, **Kakao Map** 라이벌 분석, **Hugging Face + 규칙 엔진** 전술 코멘트

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
- **AI 연동**: Hugging Face Inference API (GPT-2 계열 요약/코멘트)  
- **API 스타일**: JSP + Controller(Servlet) + Ajax

![Java](https://img.shields.io/badge/Java-007396?style=for-the-badge&logo=java&logoColor=white)
![JSP](https://img.shields.io/badge/JSP-MVC2-2D4470?style=for-the-badge)
![HuggingFace](https://img.shields.io/badge/Hugging%20Face-Inference%20API-FFCC4D?style=for-the-badge)

### 데이터베이스
- **DB**: Oracle (또는 MySQL)  
- **주요 테이블**:  
  `player`, `formation`, `formation_position`, `rival_club`, `region`, `player_analysis`, `player_tactic_recommendation`  
- 포지션 좌표, 상대 전술, 예상 승률 등 의사결정용 필드 보유

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
    UI[UI: JSP / HTML / CSS / JS, Drag&Drop]:::ui
    MAP[Kakao Map JS SDK]:::ext
  end

  %% 앱 서버
  subgraph App["웹 애플리케이션 (Tomcat / JSP-Servlet)"]
    F[Filter / Interceptor: 세션, 권한]:::svc
    CTRL[Controller / Servlet: /login, /players, /formations, /rivals, /ai/recommend]:::svc
    SRV[Service: 검증, 트랜잭션, 오케스트레이션]:::svc
    RULES[규칙 엔진: 포지션/스타일/상대전술 매칭]:::svc
    DAO[DAO / MyBatis Mapper]:::svc
  end

  %% 리소스
  DB[(Oracle DB: player, formation, formation_position, rival_club, region, player_analysis, player_tactic_recommendation)]:::db
  HF[(Hugging Face Inference API: GPT-2 계열 요약/코멘트)]:::ext
  LOG[(로깅 / 모니터링)]:::ext

  %% 연결
  UI -->|AJAX JSON| F --> CTRL
  UI -.-> MAP
  CTRL --> SRV --> DAO --> DB
  SRV --> RULES
  SRV -.-> HF
  App -.-> LOG
