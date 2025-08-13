# HosungFC — 축구팀 관리자 시스템
JSP + Java(서블릿/DAO) + Oracle 기반의 **팀 운영/전술 추천 관리자 페이지**  
포메이션 **드래그&드롭**, **Kakao Map** 라이벌 분석, **🤗 Hugging Face + 규칙 엔진** 전술 코멘트

---

## #0. 프로젝트 개요
**프로젝트 이름**: HosungFC (축구팀 관리 & 전술 추천 시스템)

**프로젝트 목적 및 개요**:  
아마추어 감독/코치를 위해 **선수·포메이션·라이벌 분석·전술 추천**을 한 곳에서 제공하는 관리자 사이트를 구축.  
선수 데이터와 상대 전술 정보를 바탕으로 **규칙 엔진**에서 1차 추천안을 만들고, **Hugging Face Inference API**로 전술 설명을 **요약·보완**하여 의사결정을 돕는다.  
드래그&드롭 포메이션 편집, Kakao Map 기반 라이벌 정보 시각화로 **운영 효율**과 **가독성**을 동시에 확보한다.

**프로젝트 기간**: 전체 기간: YYYY.MM.DD ~ YYYY.MM.DD (채워주세요)

---

## 기술 스택

### 프론트엔드 (View / JSP 기반)
- **뷰엔진**: JSP (JavaServer Pages), HTML5  
- **템플릿 태그**: JSTL, EL  
- **스타일링**: Bootstrap, CSS3  
- **JavaScript**: Vanilla JS (필드 드래그&드롭), jQuery 일부  
- **지도**: Kakao Map JavaScript API

![HTML5](https://img.shields.io/badge/HTML5-E34F26?style=for-the-badge&logo=html5&logoColor=white)
![CSS3](https://img.shields.io/badge/CSS3-1572B6?style=for-the-badge&logo=css3&logoColor=white)
![JavaScript](https://img.shields.io/badge/JavaScript-ES6+-F7DF1E?style=for-the-badge&logo=javascript&logoColor=white)
![Bootstrap](https://img.shields.io/badge/Bootstrap-563D7C?style=for-the-badge&logo=bootstrap&logoColor=white)
![KakaoMap](https://img.shields.io/badge/Kakao%20Map-API-FFCD00?style=for-the-badge)

---

### 백엔드
- **아키텍처**: Java Servlet + DAO(또는 Service/DAO 계층), JSP MVC2  
- **언어**: Java  
- **ORM/매퍼**: (선택) MyBatis 또는 JDBC Template  
- **AI 연동**: Hugging Face Inference API (GPT-2 계열 호출로 요약/코멘트 생성)  
- **API 스타일**: JSP + Controller(Servlet) + Ajax

![Java](https://img.shields.io/badge/Java-007396?style=for-the-badge&logo=java&logoColor=white)
![JSP](https://img.shields.io/badge/JSP-MVC2-2D4470?style=for-the-badge)
![HuggingFace](https://img.shields.io/badge/Hugging%20Face-Inference%20API-FFCC4D?style=for-the-badge)

---

### 데이터베이스
- **DB**: Oracle (또는 MySQL)  
- **주요 테이블**:  
  `player`, `formation`, `formation_position`, `rival_club`, `region`, `player_analysis`, `player_tactic_recommendation`  
- 포지션 좌표, 상대 전술, 예상 승률 등 **의사결정용 필드** 보유

![Oracle](https://img.shields.io/badge/Oracle_DB-F80000?style=for-the-badge&logo=oracle&logoColor=white)

---

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

### 시스템 아키텍처
- **클라이언트 (브라우저)**: 선수/포메이션 편집, 지도 기반 라이벌 분석, AI 추천 코멘트 열람  
- **웹 서버 (Tomcat)**: JSP/Servlet 라우팅, Ajax 처리  
- **비즈니스 로직**: 규칙 엔진(내장) + HF Inference API 호출/결합  
- **데이터베이스**: 선수/포메이션/라이벌/분석 테이블 저장

### 시스템 다이어그램
(이미지 파일을 레포 `assets/`에 넣고 아래 경로를 교체하세요)
![시스템 아키텍처](assets/architecture.png)
![시스템 흐름도](assets/sequence.png)

**AI 추천 흐름(요약)**  
1) DB에서 선수 스타일/강점/약점 + 상대 전술 조회  
2) **규칙 엔진**으로 1차 추천(포지션/스타일/상대 전술 매칭)  
3) **🤗 Hugging Face Inference API (GPT-2)** 호출 → 코멘트/설명 생성  
4) (2)+(3) 결과를 병합하여 화면에 노출 *(API 실패 시 2번만 노출하는 페일세이프)*

---

## #2. 요구 사항 분석
- **로그인/권한(선택)**: 관리자만 접근 가능하도록 보호(추가 예정 시)  
- **선수 관리**: 등록/수정/삭제, 이름/포지션 검색, 페이징 목록  
- **포메이션 관리**: 필드 배경 위 **드래그&드롭 배치**, 좌표/라벨 저장, 수정/삭제  
- **라이벌 분석**: Kakao Map 마커/지역 핀, 모달로 전적/전술/예상 승률/강점선수 등 상세 표시  
- **AI 전술 추천**:  
  - 기본: **규칙 엔진** (포지션/스타일/상대 전술 일치 기반)  
  - 보강: **Hugging Face Inference API(GPT-2)**로 코멘트/전술 설명 **요약·보완**  
- **성능/안정성**: Ajax 표준 응답(JSON), 예외 메시지 규격화, 필드 검증

---

## #3. 워크플로우 영상
각 기능별 워크플로우 영상은 아래에 첨부합니다. (URL만 교체)

### 1. 선수/포메이션
드래그&드롭 포메이션 배치, 저장/수정, 선수 검색/페이징  
<a href="영상_URL_여기에">선수·포메이션 영상</a>

### 2. 라이벌 분석 (Kakao Map)
마커/지역 필터, 모달에서 전적·전술·예상 승률 확인  
<a href="영상_URL_여기에">라이벌 분석 영상</a>

### 3. AI 전술 추천
규칙 엔진 결과 + 🤗HF 요약 설명 병합 출력  
<a href="영상_URL_여기에">AI 전술 추천 영상</a>

---

## 📄 데이터베이스 설계 (ERD)
(ERD 이미지를 `assets/erd.png`로 추가하거나 아래 링크를 교체)
![ERD](assets/erd.png)

---

## 🔌 환경 변수 / 설정
`.env` 또는 서버 환경변수로 관리하세요.

```bash
# Kakao Map
KAKAO_MAP_APPKEY=여기에_키

# Hugging Face Inference API
HF_API_URL=https://api-inference.huggingface.co/models/gpt2
HF_API_TOKEN=hf_xxxxxxxxxxxxxxxxxxxxxxxxx
