# 🚤 Boating Life Platform (Backend)
> **해양 레저 관광 활성화를 위한 AI 기반 보팅 라이프 플랫폼** > 본 저장소는 서비스의 핵심 로직과 데이터 처리를 담당하는 백엔드 서버 프로젝트입니다.

<br>

## 🔗 Interactive Documents
시각적으로 최적화된 상세 기획안과 설계 문서를 확인하실 수 있습니다.

* **[📄 서버 상세 기획안 (HTML) 다운로드](./html/server_plan.html)**
* **[📊 ERD 상세 설계 문서](./ERD.md)**
* **[⚙️ 기술 상세 사양서](./Server_Design.md)**

---

## 🛠 Tech Stack

* **Language & Framework**: Java 17, Spring Boot
* **Database**: postgreSQL, postGIS (AWS RDS),기반 데이터 모델링 및 쿼리 최적화
* **Infrastructure**: AWS EC2 기반 배포
* **Media**: Firebase Storage (고해상도 이미지 데이터 관리)
* **Cache**: Redis cache
* **Authorization**: Clerk OAuth, JWT, RBAC
---

## 🌟 Key Features
작성된 서버 기획안에 기반한 핵심 기능 구현 목록입니다.

### 1. 인증 및 보안 (Auth & Security)
* **OAuth 2.0**: Clerk로 Google 및 Kakao 소셜 로그인을 통한 간편 인증 지원
* **JWT**: JSON Web Token을 활용한 무상태(Stateless) API 보안 검증
* **보안 검증**: 서버 측 권한 체크 및 민감 정보 암호화 관리

### 2. 커뮤니티 및 SNS (Social Feed)
* **피드 관리**: 이미지 업로드(Firebase 연동) 및 게시글 CRUD 처리
* **인터랙션**: 게시글 좋아요 토글 및 댓글 시스템 구현

### 3. 위치 기반 서비스 (LBS)
* **스팟 검색**: Google Maps API 연동을 통한 주변 레저 스팟(보트/요트) 검색
* **트래킹**: 사용자의 현재 위치 기록 및 항해 경로 데이터 저장

### 4. 예약 및 결제 (Booking & Payment)
* **결제 시스템**: 토스페이먼츠(Toss Payments) API 연동
* **이중 검증**: 결제 위변조 방지를 위한 서버 측 금액 검증 로직 적용

---

## Architecture
이 프로젝트는 높은 확장성과 독립성을 위해 다음과 같은 구조로 설계되었습니다.

```
graph TD
    A[Mobile App - Flutter] -->|HTTPS| B[Spring Boot Server]
    B --> C[(PostgreSql)]
    B --> D[Firebase Storage]
    B --> E[Toss Payments]
    B --> F[Social Auth - Clerk + Google/Kakao]

"(상세한 데이터 흐름은 서버 기획안의 Flow Diagram 섹션에서 확인 가능합니다.)"
```