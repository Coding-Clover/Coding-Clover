# Coding Clover

Java 기반 온라인 코딩 교육 플랫폼. 강의 수강, 시험, 코딩 테스트, AI 챗봇, 결제 기능을 통합 제공합니다.

---

## 전체 시스템 아키텍처

```mermaid
graph TB
    subgraph CLIENT["🖥️ Client (Browser)"]
        Browser["Browser"]
    end

    subgraph FRONTEND["⚛️ Frontend — React 19 + Vite 5 (localhost:5173)"]
        direction TB
        Pages["Pages\nHome / Login / Register\nStudent / Instructor / Admin / Coding"]
        UILib["UI Libraries\nRadix UI · Tailwind CSS\nMonaco Editor · React Markdown\nEmbla Carousel · Lottie · Sonner"]
        Axios["Axios 1.13\nwithCredentials: true\nProxy → localhost:3333"]
        AuthClient["Auth (Client)\nCookie Session (JSESSIONID)\nRole Guard (ProtectedRoute)\nlocalStorage (loginId, users)"]
    end

    subgraph BACKEND["☕ Backend — Spring Boot 3.4.13 / Java 21 (localhost:3333)"]
        direction TB
        Security["Spring Security\nCORS: localhost:5173·5174\nBCrypt · Session 30min\nApiLoginFilter (Custom)\n/instructor/** → INSTRUCTOR\n/admin/** → ADMIN"]

        subgraph DOMAINS["Domain Modules (MVC Pattern)"]
            direction LR
            D1["Users\nCourse\nLecture\nEnrollment"]
            D2["Exam\nExamAttempt\nProblem\nSubmission"]
            D3["Payment\nUserWallet\nWalletHistory\nLectureProgress"]
            D4["Qna · QnaAnswer\nCommunityPost\nNotice · Notification\nStudentProfile · InstructorProfile"]
            D5["Image · Mail\nSearch · ScoreHistory\nChatBot · AiQuiz"]
        end

        CodeExec["Code Executor\nJavaNativeExecutor\n(코딩 테스트 채점 엔진)"]
        YoutubeService["YouTube Service\n영상 메타데이터\n자막 추출"]
        SpringAI["Spring AI 1.0.0-M5\nChatBot / AiQuiz"]
    end

    subgraph EXTERNAL["🌐 External Services"]
        direction TB
        OpenAI["OpenAI\ngpt-4o-mini\ntemp: 0.3"]
        TossPayments["Toss Payments\napi.tosspayments.com/v1\n결제 확인·승인"]
        Gmail["Gmail SMTP\nsmtp.gmail.com:587\nSTARTTLS"]
        OAuth2["OAuth2 Providers\nGoogle · Kakao · Naver"]
        YouTubeAPI["YouTube Data API\n영상 정보·자막"]
    end

    subgraph INFRA["☁️ Infrastructure (AWS)"]
        RDS["AWS RDS MySQL 8\nap-northeast-2\ncoding_clover DB\nDDL: update"]
        S3["AWS S3\ncoding-clover-images\nap-northeast-2\n최대 10MB\nThumbnailator 썸네일"]
    end

    Browser -->|HTTP/HTTPS| FRONTEND
    Pages --> UILib
    Pages --> Axios
    Pages --> AuthClient
    Axios -->|REST API / Session Cookie| Security
    Security --> DOMAINS
    DOMAINS --> CodeExec
    DOMAINS --> YoutubeService
    DOMAINS --> SpringAI
    SpringAI -->|API Key| OpenAI
    DOMAINS -->|결제 확인| TossPayments
    DOMAINS -->|인증 메일| Gmail
    Security -->|OAuth2 Redirect| OAuth2
    YoutubeService -->|API Key| YouTubeAPI
    DOMAINS -->|JPA / Hibernate| RDS
    DOMAINS -->|이미지 업로드| S3
```

---

## 인증 흐름

```mermaid
sequenceDiagram
    participant B as Browser
    participant F as Frontend (React)
    participant S as Spring Security
    participant OAuth as OAuth2 Provider
    participant DB as MySQL (RDS)

    Note over B,DB: 일반 로그인
    B->>F: 이메일/비밀번호 입력
    F->>S: POST /auth/login
    S->>DB: UsersSecurityService.loadUserByUsername()
    DB-->>S: Users Entity (BCrypt 검증)
    S-->>F: Set-Cookie: JSESSIONID (HttpOnly)
    F-->>B: 로그인 성공 → Role 기반 라우팅

    Note over B,DB: 소셜 로그인 (Google / Kakao / Naver)
    B->>F: 소셜 로그인 버튼 클릭
    F->>S: GET /oauth2/authorization/{provider}
    S->>OAuth: OAuth2 Redirect
    OAuth-->>S: Authorization Code
    S->>OAuth: Access Token 교환
    OAuth-->>S: 사용자 정보
    S->>DB: SocialLoginService (신규 가입 or 기존 계정 연동)
    S-->>F: Set-Cookie: JSESSIONID
    F-->>B: 로그인 성공
```

---

## 결제 흐름

```mermaid
sequenceDiagram
    participant S as Student
    participant F as Frontend
    participant B as Backend
    participant T as Toss Payments API
    participant DB as MySQL (RDS)

    S->>F: 강의 구매 / 포인트 충전 요청
    F->>T: Toss SDK 결제창 호출
    T-->>S: 결제창 표시
    S->>T: 결제 정보 입력
    T-->>F: paymentKey, orderId, amount 반환
    F->>B: POST /payment/confirm
    B->>T: POST api.tosspayments.com/v1/payments/confirm\n(Base64 Secret Key 인증)
    T-->>B: 결제 승인 결과
    B->>DB: Payment 저장 (SUCCESS/FAILED)
    B->>DB: UserWallet 포인트 업데이트
    B->>DB: WalletHistory 거래 기록
    B-->>F: 결제 결과 응답
    F-->>S: 완료 화면
```

---

## AI 기능 흐름

```mermaid
sequenceDiagram
    participant U as User
    participant F as Frontend
    participant B as Backend (Spring AI)
    participant O as OpenAI (gpt-4o-mini)

    Note over U,O: ChatBot — 강의 질문 답변
    U->>F: ChatBot.jsx에서 질문 입력
    F->>B: POST /ask
    B->>O: ChatClient.prompt()\nSystem Prompt + 사용자 질문\ntemperature: 0.3
    O-->>B: AI 응답
    B-->>F: ChatDto 응답
    F-->>U: 마크다운 렌더링 출력

    Note over U,O: AiQuiz — AI 문제 자동 생성
    U->>F: 시험 생성 요청
    F->>B: POST /instructor/exam/ai-generate
    B->>O: 강의 내용 기반 문제 생성 프롬프트
    O-->>B: JSON 형식 문제 목록
    B-->>F: AiQuizResponseDto
    F-->>U: 문제 편집 화면
```

---

## 코딩 테스트 채점 흐름

```mermaid
flowchart TD
    A["학생: 코드 제출"] --> B["POST /submission"]
    B --> C["SubmissionController"]
    C --> D["SubmissionService"]
    D --> E{"CodeExecutor Interface"}
    E --> F["JavaNativeExecutor\n(Java 코드 실행)"]
    F --> G["임시 파일 생성\n컴파일 javac\n실행 java"]
    G --> H{"실행 결과"}
    H -->|성공| I["GradingResult: PASS\n실행 시간·출력값 비교"]
    H -->|실패| J["GradingResult: FAIL\n오류 메시지 반환"]
    I --> K["Submission Entity 저장 (MySQL)"]
    J --> K
    K --> L["ExecutionResponse → Frontend"]
    L --> M["결과 화면 출력"]
```

---

## 강의 업로드 워크플로우

```mermaid
stateDiagram-v2
    [*] --> 강사_업로드: 강사가 강의 생성 요청
    강사_업로드 --> PENDING: LectureApprovalStatus.PENDING

    PENDING --> 관리자_검토: 관리자 승인 대기

    관리자_검토 --> APPROVED: 승인
    관리자_검토 --> REJECTED: 반려 (RejectRequest)

    APPROVED --> 학생_수강_가능: 수강 오픈
    REJECTED --> 강사_수정: 강사 재업로드

    강사_수정 --> PENDING

    note right of PENDING
        LectureUploadType
        IMMEDIATE: 즉시 업로드
        SCHEDULED: 예약 업로드
    end note
```

---

## DB 주요 엔티티 관계

```mermaid
erDiagram
    USERS {
        Long user_id PK
        String email
        String password
        UsersRole role
        UsersStatus status
    }
    COURSE {
        Long course_id PK
        Long creator_id FK
        String title
        CourseProposalStatus status
    }
    LECTURE {
        Long lecture_id PK
        Long course_id FK
        String title
        LectureApprovalStatus approval_status
        LectureUploadType upload_type
    }
    ENROLLMENT {
        Long enrollment_id PK
        Long user_id FK
        Long course_id FK
        EnrollmentStatus status
    }
    EXAM {
        Long exam_id PK
        Long course_id FK
        Integer time_limit
        Integer pass_score
    }
    EXAM_ATTEMPT {
        Long attempt_id PK
        Long exam_id FK
        Long user_id FK
        Integer score
    }
    PAYMENT {
        Long payment_id PK
        Long user_id FK
        PaymentStatus status
        PaymentType type
    }
    USER_WALLET {
        Long wallet_id PK
        Long user_id FK
        Integer balance
    }
    PROBLEM {
        Long problem_id PK
        ProblemDifficulty difficulty
        String base_code
    }
    SUBMISSION {
        Long submission_id PK
        Long problem_id FK
        Long user_id FK
        String code
    }
    LECTURE_PROGRESS {
        Long progress_id PK
        Long lecture_id FK
        Long user_id FK
        Boolean completed
    }

    USERS ||--o{ ENROLLMENT : "수강 신청"
    USERS ||--o{ PAYMENT : "결제"
    USERS ||--|| USER_WALLET : "보유"
    USERS ||--o{ SUBMISSION : "제출"
    USERS ||--o{ EXAM_ATTEMPT : "응시"
    COURSE ||--o{ LECTURE : "포함"
    COURSE ||--o{ ENROLLMENT : "등록"
    COURSE ||--o{ EXAM : "포함"
    LECTURE ||--o{ LECTURE_PROGRESS : "진도"
    EXAM ||--o{ EXAM_ATTEMPT : "응시"
    PROBLEM ||--o{ SUBMISSION : "제출"
```

---

## 기술 스택 요약

```mermaid
graph LR
    subgraph Frontend
        React["React 19.2.3"]
        Vite["Vite 5.4.11"]
        TailwindCSS["Tailwind CSS 3"]
        RadixUI["Radix UI"]
        Monaco["Monaco Editor"]
        Axios["Axios 1.13"]
        ReactRouter["React Router DOM 7"]
    end

    subgraph Backend
        SpringBoot["Spring Boot 3.4.13"]
        Java21["Java 21"]
        SpringSecurity["Spring Security\n+ OAuth2"]
        SpringAI["Spring AI 1.0.0-M5"]
        JPA["Spring Data JPA\n+ Hibernate"]
        WebFlux["Spring WebFlux"]
    end

    subgraph Database
        MySQL["MySQL 8\n(AWS RDS)"]
    end

    subgraph Storage
        S3["AWS S3"]
    end

    subgraph AI
        GPT["OpenAI\ngpt-4o-mini"]
    end

    subgraph Payment
        Toss["Toss Payments"]
    end

    subgraph Auth
        Google["Google OAuth2"]
        Kakao["Kakao OAuth2"]
        Naver["Naver OAuth2"]
    end

    subgraph Build
        Gradle["Gradle 9.2.1"]
        NodeJS["Node.js\n(Vite/npm)"]
    end

    subgraph Test
        JUnit5["JUnit 5\n(Jupiter)"]
    end

    subgraph Mail
        Gmail["Gmail SMTP"]
    end

    Frontend -->|REST API| Backend
    Backend --> Database
    Backend --> Storage
    Backend --> AI
    Backend --> Payment
    Backend --> Auth
    Backend --> Mail
    Frontend -.->|Build| Gradle
    Frontend -.->|Build| NodeJS
    Backend -.->|Test| JUnit5
```

---

## 환경 구성

| 항목 | 개발 환경 | 비고 |
|------|-----------|------|
| Backend Port | `3333` | Spring Boot |
| Frontend Port | `5173` | Vite Dev Server |
| Vite Proxy | `/auth`, `/api`, `/course`, `/student`, `/instructor`, `/admin`, `/notice`, `/ask` | → localhost:3333 |
| DB | AWS RDS MySQL (ap-northeast-2) | `coding_clover` |
| Storage | AWS S3 `coding-clover-images` | ap-northeast-2 |
| AI | OpenAI gpt-4o-mini | temperature: 0.3 |
| 결제 | Toss Payments (테스트 키) | `test_sk_...` |
| 메일 | Gmail SMTP (:587, STARTTLS) | |
| 소셜 로그인 | Google / Kakao / Naver | OAuth2 |
| 파일 크기 제한 | 10MB | Multipart |
| 세션 만료 | 30분 | JSESSIONID (HttpOnly) |

### 환경변수 (.env)

```
OPENAI_API_KEY      # OpenAI API 인증
AWS_ACCESS_KEY      # AWS S3 접근 키
AWS_SECRET_KEY      # AWS S3 시크릿 키
AWS_S3_BUCKET       # coding-clover-images
```

---

## 프로젝트 구조

```
codingclover/
├── src/
│   ├── main/
│   │   ├── java/com/mysite/clover/
│   │   │   ├── Users/          # 사용자 인증·관리
│   │   │   ├── Course/         # 강의 과정 관리
│   │   │   ├── Lecture/        # 강의 영상 관리
│   │   │   ├── Enrollment/     # 수강 신청
│   │   │   ├── Exam/           # 시험 관리
│   │   │   ├── ExamAttempt/    # 시험 응시 기록
│   │   │   ├── Problem/        # 코딩 문제
│   │   │   ├── Submission/     # 코드 제출·채점
│   │   │   ├── Payment/        # 결제 (Toss)
│   │   │   ├── UserWallet/     # 포인트 지갑
│   │   │   ├── WalletHistory/  # 거래 내역
│   │   │   ├── LectureProgress/# 강의 진도
│   │   │   ├── Qna/            # Q&A
│   │   │   ├── QnaAnswer/      # Q&A 답변
│   │   │   ├── CommunityPost/  # 커뮤니티 게시판
│   │   │   ├── Notice/         # 공지사항
│   │   │   ├── Notification/   # 알림
│   │   │   ├── ScoreHistory/   # 성적 이력
│   │   │   ├── StudentProfile/ # 학생 프로필
│   │   │   ├── InstructorProfile/ # 강사 프로필
│   │   │   ├── Image/          # 이미지 업로드 (S3)
│   │   │   ├── Mail/           # 이메일 서비스
│   │   │   ├── ChatBot/        # AI 챗봇
│   │   │   ├── AiQuiz/         # AI 문제 생성
│   │   │   ├── Search/         # 통합 검색
│   │   │   └── SecurityConfig.java
│   │   └── resources/
│   │       └── application.properties
│   └── test/
│       └── java/com/mysite/clover/   # JUnit 5
├── frontend/
│   ├── src/
│   │   ├── pages/              # 페이지 컴포넌트 (101+ JSX)
│   │   │   ├── student/
│   │   │   ├── instructor/
│   │   │   ├── admin/
│   │   │   ├── coding/
│   │   │   └── public/
│   │   └── components/         # 공통 컴포넌트
│   │       └── ui/             # Radix UI 래퍼
│   ├── package.json
│   └── vite.config.js
├── build.gradle
├── settings.gradle
└── .env
```
