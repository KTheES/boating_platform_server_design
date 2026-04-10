# ERD 

## ERD 도식화 (html)




# 🗂️ Boating Life Platform — ERD 설계 문서

> **문서 유형**: 데이터베이스 설계 (ERD)
> **연동 문서**: [서버 설계 문서](./SERVER_DESIGN.md) | [ERD HTML download](./html/erd_table_reference.html)
> **상태**: 🚧 초안 (프론트엔드 화면 확정 전 — 변경 가능)

---

## 목차

1. [테이블 목록](#1-테이블-목록)
2. [ERD 다이어그램](#2-erd-다이어그램)
3. [테이블 상세 명세](#3-테이블-상세-명세)
4. [관계 정의 (Relations)](#4-관계-정의-relations)
5. [주요 제약 조건](#5-주요-제약-조건)
6. [변경 이력](#6-변경-이력)

---

## 1. 테이블 목록

| # | 테이블명 | 설명 | 상태 |
|---|----------|------|------|
| 1 | `users` | 유저 정보 (Clerk 연동) | ✅ 확정 |
| 2 | `spots` | 해양 레저 스팟 | ✅ 확정 |
| 3 | `posts` | SNS 게시글 | ✅ 확정 |
| 4 | `post_images` | 게시글 이미지 | ✅ 확정 |
| 5 | `comments` | 댓글 | ✅ 확정 |
| 6 | `likes` | 게시글 좋아요 | ✅ 확정 |
| 7 | `reservations` | 스팟 예약 | ✅ 확정 |
| 8 | `payments` | 결제 내역 (Toss) | ✅ 확정 |
| 9 | `location_logs` | 위치 추적 이력 | 🚧 검토 중 |
| 10 | `reviews` | 스팟 리뷰/평점 | ❓ 미정 |

---

## 2. ERD 다이어그램

```
┌─────────────┐         ┌──────────────────┐
│    users    │         │      spots       │
├─────────────┤         ├──────────────────┤
│ id (PK)     │         │ id (PK)          │
│ clerk_id    │         │ name             │
│ email       │         │ type             │
│ nickname    │         │ description      │
│ profile_url │         │ latitude         │
│ provider    │         │ longitude        │
│ created_at  │         │ address          │
│ updated_at  │         │ created_at       │
└──────┬──────┘         └────────┬─────────┘
       │ 1                       │ 1
       │                         │
       │ N    ┌──────────────┐   │ N
       └──────► reservations ◄───┘
              ├──────────────┤
              │ id (PK)      │
              │ user_id (FK) │
              │ spot_id (FK) │
              │ visit_date   │
              │ status       │──────────────────────┐
              │ created_at   │                      │ 1
              └──────────────┘                      │
                                                    │ N
       ┌────────────────────────────────────────────▼──────┐
       │                    payments                        │
       ├────────────────────────────────────────────────────┤
       │ id (PK)                                            │
       │ reservation_id (FK)                                │
       │ toss_payment_key                                   │
       │ order_id                                           │
       │ amount                                             │
       │ status (PENDING / PAID / CANCELLED)                │
       │ paid_at                                            │
       │ created_at                                         │
       └────────────────────────────────────────────────────┘


┌─────────────┐  1      N  ┌──────────────────┐
│    users    ├────────────► posts             │
└─────────────┘            ├──────────────────┤
                           │ id (PK)           │
                           │ user_id (FK)      │
                           │ spot_id (FK) ──┐  │
                           │ content        │  │
                           │ latitude       │  │
                           │ longitude      │  │
                           │ created_at     │  │
                           │ updated_at     │  │
                           └───────┬────────┘  │
                                   │           │ N
                                   │ 1         ▼
                          ┌────────┴──────┐  spots
                          │               │
              ┌───────────▼──┐  ┌─────────▼─────┐  ┌──────────────┐
              │ post_images  │  │   comments    │  │    likes     │
              ├──────────────┤  ├───────────────┤  ├──────────────┤
              │ id (PK)      │  │ id (PK)       │  │ id (PK)      │
              │ post_id (FK) │  │ post_id (FK)  │  │ post_id (FK) │
              │ image_url    │  │ user_id (FK)  │  │ user_id (FK) │
              │ order_num    │  │ content       │  │ created_at   │
              │ created_at   │  │ created_at    │  └──────────────┘
              └──────────────┘  │ updated_at    │  (UNIQUE: post_id
                                └───────────────┘   + user_id)


┌─────────────┐  1      N  ┌─────────────────────┐
│    users    ├────────────► location_logs        │
└─────────────┘            ├─────────────────────┤
                           │ id (PK)              │
                           │ user_id (FK)         │
                           │ latitude             │
                           │ longitude            │
                           │ recorded_at          │
                           └─────────────────────┘
```

---

## 3. 테이블 상세 명세

---

### `users` — 유저 정보

> Clerk OAuth 인증과 연동되는 핵심 테이블.  
> `clerk_id`는 Clerk에서 발급하는 고유 식별자로, 소셜 로그인 시 자동 생성됩니다.

| 컬럼명 | 타입 | 제약 조건 | 설명 |
|--------|------|-----------|------|
| `id` | `BIGINT` | `PK`, `AUTO_INCREMENT` | 내부 식별자 |
| `clerk_id` | `VARCHAR(255)` | `UNIQUE`, `NOT NULL` | Clerk 고유 ID |
| `email` | `VARCHAR(255)` | `UNIQUE`, `NOT NULL` | 이메일 |
| `nickname` | `VARCHAR(100)` | `NOT NULL` | 닉네임 |
| `profile_img_url` | `TEXT` | `NULLABLE` | 프로필 이미지 URL (Firebase) |
| `provider` | `VARCHAR(20)` | `NOT NULL` | 소셜 로그인 종류 (`google` / `kakao`) |
| `created_at` | `TIMESTAMP` | `NOT NULL`, `DEFAULT NOW()` | 가입일 |
| `updated_at` | `TIMESTAMP` | `NOT NULL`, `DEFAULT NOW()` | 수정일 |

---

### `spots` — 해양 레저 스팟

> 보트, 페리, 요트 등 해양 레저 장소 정보.  
> 관리자 권한으로만 등록/수정 가능합니다.

| 컬럼명 | 타입 | 제약 조건 | 설명 |
|--------|------|-----------|------|
| `id` | `BIGINT` | `PK`, `AUTO_INCREMENT` | 스팟 식별자 |
| `name` | `VARCHAR(255)` | `NOT NULL` | 스팟 이름 |
| `type` | `VARCHAR(20)` | `NOT NULL` | 종류 (`BOAT` / `FERRY` / `YACHT`) |
| `description` | `TEXT` | `NULLABLE` | 스팟 설명 |
| `address` | `VARCHAR(500)` | `NULLABLE` | 주소 |
| `latitude` | `DECIMAL(10,7)` | `NOT NULL` | 위도 |
| `longitude` | `DECIMAL(10,7)` | `NOT NULL` | 경도 |
| `thumbnail_url` | `TEXT` | `NULLABLE` | 대표 이미지 URL |
| `created_at` | `TIMESTAMP` | `NOT NULL`, `DEFAULT NOW()` | 등록일 |
| `updated_at` | `TIMESTAMP` | `NOT NULL`, `DEFAULT NOW()` | 수정일 |

---

### `posts` — SNS 게시글

> 유저가 방문한 스팟에 대해 작성하는 SNS 피드.  
> **예약/방문 확인 후에만 작성 가능** (reservation_id 연결 필요).

| 컬럼명 | 타입 | 제약 조건 | 설명 |
|--------|------|-----------|------|
| `id` | `BIGINT` | `PK`, `AUTO_INCREMENT` | 게시글 식별자 |
| `user_id` | `BIGINT` | `FK → users.id`, `NOT NULL` | 작성자 |
| `spot_id` | `BIGINT` | `FK → spots.id`, `NULLABLE` | 연결된 스팟 |
| `reservation_id` | `BIGINT` | `FK → reservations.id`, `NULLABLE` | 연결된 예약 (방문 인증) |
| `content` | `TEXT` | `NOT NULL` | 본문 내용 |
| `latitude` | `DECIMAL(10,7)` | `NULLABLE` | 작성 위치 위도 |
| `longitude` | `DECIMAL(10,7)` | `NULLABLE` | 작성 위치 경도 |
| `created_at` | `TIMESTAMP` | `NOT NULL`, `DEFAULT NOW()` | 작성일 |
| `updated_at` | `TIMESTAMP` | `NOT NULL`, `DEFAULT NOW()` | 수정일 |

---

### `post_images` — 게시글 이미지

> Firebase Storage에 업로드된 이미지의 URL을 저장.  
> 한 게시글에 여러 이미지 첨부 가능, `order_num`으로 순서 관리.

| 컬럼명 | 타입 | 제약 조건 | 설명 |
|--------|------|-----------|------|
| `id` | `BIGINT` | `PK`, `AUTO_INCREMENT` | 이미지 식별자 |
| `post_id` | `BIGINT` | `FK → posts.id`, `NOT NULL` | 게시글 |
| `image_url` | `TEXT` | `NOT NULL` | Firebase Storage URL |
| `order_num` | `INT` | `NOT NULL`, `DEFAULT 0` | 이미지 표시 순서 |
| `created_at` | `TIMESTAMP` | `NOT NULL`, `DEFAULT NOW()` | 업로드일 |

---

### `comments` — 댓글

| 컬럼명 | 타입 | 제약 조건 | 설명 |
|--------|------|-----------|------|
| `id` | `BIGINT` | `PK`, `AUTO_INCREMENT` | 댓글 식별자 |
| `post_id` | `BIGINT` | `FK → posts.id`, `NOT NULL` | 게시글 |
| `user_id` | `BIGINT` | `FK → users.id`, `NOT NULL` | 작성자 |
| `content` | `TEXT` | `NOT NULL` | 댓글 내용 |
| `created_at` | `TIMESTAMP` | `NOT NULL`, `DEFAULT NOW()` | 작성일 |
| `updated_at` | `TIMESTAMP` | `NOT NULL`, `DEFAULT NOW()` | 수정일 |

---

### `likes` — 좋아요

> 한 유저는 같은 게시글에 **1회만** 좋아요 가능 (토글).  
> `(post_id, user_id)` 복합 UNIQUE 제약으로 중복 방지.

| 컬럼명 | 타입 | 제약 조건 | 설명 |
|--------|------|-----------|------|
| `id` | `BIGINT` | `PK`, `AUTO_INCREMENT` | 좋아요 식별자 |
| `post_id` | `BIGINT` | `FK → posts.id`, `NOT NULL` | 게시글 |
| `user_id` | `BIGINT` | `FK → users.id`, `NOT NULL` | 유저 |
| `created_at` | `TIMESTAMP` | `NOT NULL`, `DEFAULT NOW()` | 좋아요 일시 |

**제약 조건**

```sql
UNIQUE (post_id, user_id)
```

---

### `reservations` — 예약

> 유저가 스팟을 예약하는 테이블.  
> 게시글 작성 시 방문 인증 용도로도 연결됩니다.

| 컬럼명 | 타입 | 제약 조건 | 설명 |
|--------|------|-----------|------|
| `id` | `BIGINT` | `PK`, `AUTO_INCREMENT` | 예약 식별자 |
| `user_id` | `BIGINT` | `FK → users.id`, `NOT NULL` | 예약자 |
| `spot_id` | `BIGINT` | `FK → spots.id`, `NOT NULL` | 예약 스팟 |
| `visit_date` | `DATE` | `NOT NULL` | 방문 예정일 |
| `headcount` | `INT` | `NOT NULL`, `DEFAULT 1` | 인원 수 |
| `status` | `VARCHAR(20)` | `NOT NULL`, `DEFAULT 'PENDING'` | 상태 |
| `created_at` | `TIMESTAMP` | `NOT NULL`, `DEFAULT NOW()` | 예약일 |
| `updated_at` | `TIMESTAMP` | `NOT NULL`, `DEFAULT NOW()` | 수정일 |

**`status` Enum**

| 값 | 설명 |
|----|------|
| `PENDING` | 결제 대기 중 |
| `CONFIRMED` | 예약 확정 |
| `VISITED` | 방문 완료 |
| `CANCELLED` | 취소됨 |

---

### `payments` — 결제 내역

> Toss Payments 연동 결제 테이블.  
> `reservation_id`와 1:1로 연결됩니다.

| 컬럼명 | 타입 | 제약 조건 | 설명 |
|--------|------|-----------|------|
| `id` | `BIGINT` | `PK`, `AUTO_INCREMENT` | 결제 식별자 |
| `reservation_id` | `BIGINT` | `FK → reservations.id`, `UNIQUE`, `NOT NULL` | 예약 |
| `toss_payment_key` | `VARCHAR(500)` | `UNIQUE`, `NULLABLE` | Toss 결제 키 |
| `order_id` | `VARCHAR(255)` | `UNIQUE`, `NOT NULL` | 주문 ID |
| `amount` | `INT` | `NOT NULL` | 결제 금액 (원) |
| `status` | `VARCHAR(20)` | `NOT NULL`, `DEFAULT 'PENDING'` | 결제 상태 |
| `paid_at` | `TIMESTAMP` | `NULLABLE` | 결제 완료 시각 |
| `created_at` | `TIMESTAMP` | `NOT NULL`, `DEFAULT NOW()` | 생성일 |

**`status` Enum**

| 값 | 설명 |
|----|------|
| `PENDING` | 결제 대기 |
| `PAID` | 결제 완료 |
| `CANCELLED` | 결제 취소 |
| `FAILED` | 결제 실패 |

---

### `location_logs` — 위치 추적 이력

> 🚧 **검토 중** — 실시간 위치 추적 필요 여부 확인 후 확정 예정

| 컬럼명 | 타입 | 제약 조건 | 설명 |
|--------|------|-----------|------|
| `id` | `BIGINT` | `PK`, `AUTO_INCREMENT` | 로그 식별자 |
| `user_id` | `BIGINT` | `FK → users.id`, `NOT NULL` | 유저 |
| `latitude` | `DECIMAL(10,7)` | `NOT NULL` | 위도 |
| `longitude` | `DECIMAL(10,7)` | `NOT NULL` | 경도 |
| `recorded_at` | `TIMESTAMP` | `NOT NULL`, `DEFAULT NOW()` | 기록 시각 |

---

### `reviews` — 스팟 리뷰/평점

> ❓ **미정** — 게시글로 대체 가능한지 논의 필요

| 컬럼명 | 타입 | 제약 조건 | 설명 |
|--------|------|-----------|------|
| `id` | `BIGINT` | `PK`, `AUTO_INCREMENT` | 리뷰 식별자 |
| `spot_id` | `BIGINT` | `FK → spots.id`, `NOT NULL` | 스팟 |
| `user_id` | `BIGINT` | `FK → users.id`, `NOT NULL` | 작성자 |
| `reservation_id` | `BIGINT` | `FK → reservations.id`, `NULLABLE` | 방문 예약 연결 |
| `rating` | `SMALLINT` | `NOT NULL`, `CHECK(1~5)` | 별점 (1~5) |
| `content` | `TEXT` | `NULLABLE` | 리뷰 내용 |
| `created_at` | `TIMESTAMP` | `NOT NULL`, `DEFAULT NOW()` | 작성일 |

---

## 4. 관계 정의 (Relations)

| 관계 | 설명 |
|------|------|
| `users` 1 : N `posts` | 한 유저는 여러 게시글을 작성할 수 있다 |
| `users` 1 : N `comments` | 한 유저는 여러 댓글을 작성할 수 있다 |
| `users` 1 : N `likes` | 한 유저는 여러 게시글에 좋아요를 누를 수 있다 |
| `users` 1 : N `reservations` | 한 유저는 여러 예약을 할 수 있다 |
| `users` 1 : N `location_logs` | 한 유저는 여러 위치 기록을 남길 수 있다 |
| `spots` 1 : N `posts` | 한 스팟에 여러 게시글이 연결될 수 있다 |
| `spots` 1 : N `reservations` | 한 스팟은 여러 예약을 받을 수 있다 |
| `posts` 1 : N `post_images` | 한 게시글에 여러 이미지가 첨부될 수 있다 |
| `posts` 1 : N `comments` | 한 게시글에 여러 댓글이 달릴 수 있다 |
| `posts` 1 : N `likes` | 한 게시글에 여러 좋아요가 달릴 수 있다 |
| `reservations` 1 : 1 `payments` | 한 예약에 하나의 결제가 연결된다 |
| `reservations` 1 : N `posts` | 한 예약(방문)에 여러 게시글을 작성할 수 있다 |

---

## 5. 주요 제약 조건

### 중복 방지

```sql
-- 좋아요: 같은 유저가 같은 게시글에 중복 불가
ALTER TABLE likes ADD CONSTRAINT uq_likes_post_user UNIQUE (post_id, user_id);

-- 결제: 예약 1건에 결제 1건만
ALTER TABLE payments ADD CONSTRAINT uq_payments_reservation UNIQUE (reservation_id);

-- 유저: clerk_id 중복 불가
ALTER TABLE users ADD CONSTRAINT uq_users_clerk_id UNIQUE (clerk_id);
```

### 인덱스 (성능 최적화)

```sql
-- 피드 조회: 최신순 정렬 빈번
CREATE INDEX idx_posts_created_at ON posts (created_at DESC);

-- 스팟 기준 게시글 조회
CREATE INDEX idx_posts_spot_id ON posts (spot_id);

-- 유저 기준 게시글 조회
CREATE INDEX idx_posts_user_id ON posts (user_id);

-- 위치 기반 검색 (PostGIS 사용 시 대체)
CREATE INDEX idx_spots_location ON spots (latitude, longitude);

-- 댓글 조회
CREATE INDEX idx_comments_post_id ON comments (post_id);

-- 결제 상태 조회
CREATE INDEX idx_payments_status ON payments (status);
```

### Cascade 삭제 정책

| 부모 삭제 시 | 자식 처리 |
|-------------|-----------|
| `users` 삭제 | `posts`, `comments`, `likes`, `reservations`, `location_logs` → CASCADE |
| `posts` 삭제 | `post_images`, `comments`, `likes` → CASCADE |
| `reservations` 삭제 | `payments` → RESTRICT (결제 기록 보존) |
| `spots` 삭제 | `posts.spot_id` → SET NULL |

---

## 6. 변경 이력

| 버전 | 날짜 | 내용 |
|------|------|------|
| v0.1 | 2025-01 | 초안 작성 (구두 기획 기반) |
| v0.2 | — | 프론트엔드 화면 확정 후 업데이트 예정 |

---

> ⚠️ **주의**: 이 문서는 초안입니다. 프론트엔드 화면 확정 후 테이블 구조가 변경될 수 있습니다.  
> 변경 시 반드시 이 문서와 [서버 설계 문서](./SERVER_DESIGN.md) 업데이트 예정입니다.

_최종 수정: 2025 | 담당: Backend Engineer_


## postgreSQL을 사용한 프로토타입 쿼리 