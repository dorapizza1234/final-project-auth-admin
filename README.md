# 🛒 중고거래 플랫폼 — 담당 파트

> **팀 프로젝트** 중 내가 담당한 파트만 분리한 레포지토리입니다.
> 인증/보안(JWT), 관리자 페이지, 광고 시스템, 마이페이지, 알림함을 구현했습니다.

[![Java](https://img.shields.io/badge/Java-17-007396?style=flat-square&logo=openjdk&logoColor=white)](https://openjdk.org/)
[![Spring Boot](https://img.shields.io/badge/Spring%20Boot-3.x-6DB33F?style=flat-square&logo=springboot&logoColor=white)](https://spring.io/projects/spring-boot)
[![Spring Security](https://img.shields.io/badge/Spring%20Security-JJWT-6DB33F?style=flat-square&logo=springsecurity&logoColor=white)](https://spring.io/projects/spring-security)
[![MyBatis](https://img.shields.io/badge/MyBatis-ORM-DC382D?style=flat-square)](https://mybatis.org/)
[![Oracle](https://img.shields.io/badge/Oracle%20DB-F80000?style=flat-square&logo=oracle&logoColor=white)](https://www.oracle.com/)

---

## 담당 기능

| 모듈 | 설명 |
|------|------|
| `security / jwt` | Spring Security + JWT 인증·인가 구현 |
| `admin` | 관리자 대시보드 (회원·상품·거래·신고·문의·광고·통계) |
| `ad` | 광고 신청 및 관리자 승인 처리 |
| `noti` | 실시간 알림함 (읽음·전체읽음·미읽음 카운트) |
| `mypage` | 마이페이지 쪽지함 |

---

## 기술 스택

| 분류 | 기술 |
|------|------|
| Language | Java 17 |
| Framework | Spring Boot 3.x |
| Security | Spring Security + JJWT (JSON Web Token) |
| ORM | MyBatis |
| Database | Oracle XE |
| View | Thymeleaf |
| Service Discovery | Spring Cloud Eureka Client |
| 스케줄링 | Spring Scheduler |

---

## 주요 구현 내용

### 🔐 JWT 인증 / Spring Security

- **Access Token (1시간) + Refresh Token (7일)** 이중 토큰 구조
- `JwtTokenProvider` — HMAC-SHA 알고리즘으로 토큰 생성·서명·검증
- `JwtAuthenticationFilter` — 모든 요청에서 토큰 추출 후 `SecurityContextHolder` 등록
- `JwtAccessDeniedHandler` / `JwtAuthenticationEntryPoint` — 인증·인가 실패 처리
- `TokenController` — Access Token 재발급 엔드포인트

```
요청
 ↓
JwtAuthenticationFilter
  ├── Authorization 헤더 "Bearer {token}" 추출
  └── 검증 성공 시 SecurityContext 등록
 ↓
SecurityConfig 경로 권한 검사
  ├── 공개: 메인, 상품 목록 등
  ├── ADMIN: /admin/**
  └── USER: 마이페이지, 알림, 채팅 등
```

---

### 🛠 관리자 대시보드 (`/admin`)

관리자 전용 통합 대시보드로, 플랫폼 전체 현황을 모니터링하고 처리합니다.

**대시보드 메인**
- 오늘 신규 상품 수, 일별 거래 금액 통계
- 인기 검색 키워드 실시간 집계
- 미처리 신고·문의·광고 건수 요약
- 신규 가입자 통계 (주간 / 월간 차트)
- 탈퇴 사유 통계 차트 (Highcharts)

**회원 관리**
- 회원 목록 조회 (상태·키워드 필터, 페이징)
- 회원 상세 정보 조회 (계좌번호 AES-256 복호화 표시)
- 회원 정지 / 정지 해제 / 탈퇴 처리
- **정지 예약 스케줄러** — 매일 새벽 2:30 자동 실행

```java
@Scheduled(cron = "0 30 2 * * ?")
public void processScheduledSuspensions() { ... }
```

**상품·거래·신고·리뷰 관리**
- 상품 목록 조회 및 강제 삭제
- 거래 내역 전체 조회
- 신고 접수·처리 (신고자/피신고자 정보 포함)
- 리뷰 관리 및 삭제

**문의 관리**
- 1:1 문의 목록 및 답변 처리

**광고 관리**
- 광고 신청 목록 조회
- 광고 승인 / 반려 처리

---

### 📢 광고 신청 (`/ad`)

일반 회원이 광고를 신청하고, 관리자가 승인·반려하는 플로우입니다.

- 광고 신청 폼 (이미지 첨부 포함)
- 신청 시 로그인 회원 정보 자동 연동
- 관리자 대시보드에서 승인/반려 처리

---

### 🔔 알림함 (`/noti`)

서비스 내 발생하는 이벤트(거래 요청, 채팅, 신고 처리 등)를 알림으로 전달합니다.

| 기능 | 엔드포인트 |
|------|-----------|
| 알림 목록 페이지 | `GET /noti/list` |
| 마이페이지 미리보기 (JSON) | `GET /noti/preview` |
| 미읽음 카운트 (헤더 배지) | `GET /noti/count` |
| 단건 읽음 처리 | `POST /noti/read/{notiNo}` |
| 전체 읽음 처리 | `POST /noti/readAll` |

---

### 👤 마이페이지 (`/mypage`)

- 쪽지함 UI (`mypage_message.html`)

---

## 프로젝트 구조

```
src/main/java/com/spring/app/
├── ad/
│   └── controller/AdController.java
├── admin/
│   ├── aop/AdminProductAspect.java       ← AOP 로깅
│   ├── controller/AdminController.java
│   ├── domain/                            ← AdDTO, SearchDTO, StatDTO 등
│   ├── model/AdminDAO, AdminDAO_imple
│   ├── scheduler/SuspendScheduleRunner    ← 정지 예약 배치
│   └── service/AdminService, AdminService_imple
├── noti/
│   ├── controller/NotiController.java
│   ├── domain/NotiDTO.java
│   ├── model/NotiDAO, NotiDAO_imple
│   └── service/NotiService, NotiService_imple
├── mypage/
│   ├── controller/MyPageController.java
│   ├── domain/                            ← AccountDTO, NotificationDTO 등
│   ├── model/MyPageDAO, MyPageDAO_imple
│   └── service/MyPageService, MyPageService_imple
├── security/
│   ├── controller/                        ← MemberController, TokenController
│   ├── domain/                            ← MemberDTO, JwtToken, CustomUserDetails
│   ├── jwt/                               ← JwtTokenProvider, JwtAuthenticationFilter
│   ├── loginfail/                         ← 로그인 실패 핸들러
│   ├── loginsuccess/                      ← 로그인 성공 핸들러
│   ├── model/MemberDAO, MemberDAO_imple
│   └── scheduler/MemberIdleScheduler      ← 휴면 회원 처리
├── config/
│   ├── SecurityConfig.java
│   ├── AES256Config.java
│   ├── FirebaseConfig.java
│   └── WebSocketConfig.java
└── common/
    ├── AES256.java                        ← 계좌번호 암호화/복호화
    ├── FileManager.java
    └── MyUtil.java
```

---

## 전체 시스템 아키텍처

이 레포는 팀 프로젝트의 일부입니다. 전체 구성은 아래와 같습니다.

```
[클라이언트]
     │
[API Gateway]
     ├── [메인 서버 (Port: 9080)]  ← 이 레포 담당 파트 포함
     └── [Board Service (Port: 8002)]  ← 공지사항·FAQ·1:1문의
          │
     [Eureka Server]  ← 서비스 디스커버리
          │
     [Oracle DB]
```
