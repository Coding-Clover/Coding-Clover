# 📌 코딩 학습 LMS URL 구조 설계

## 🟦 공통 영역 (비로그인 / 로그인 공통)
/                       → 홈
/courses                → 강좌 목록
/courses/level/beginner → 입문 강좌
/courses/level/basic    → 초급 강좌
/courses/level/intermediate → 중급 강좌
/courses/level/advanced → 고급 강좌

/community              → 커뮤니티
/community/posts        → 게시글 목록
/community/posts/{id}   → 게시글 상세

/notices                → 공지사항
/notices/{id}           → 공지 상세

/auth/login             → 로그인
/auth/register          → 회원가입
/auth/oauth             → 소셜 로그인

## 🟩 수강생 영역

(로그인 + 수강생 권한 필요)

/student/dashboard      → 수강생 대시보드

/student/courses        → 내 강좌
/student/courses/active → 수강 중 강좌
/student/courses/completed → 수강 완료 강좌
/student/courses/planned → 수강 예정 강좌

/student/courses/{courseId} → 강좌 상세
/student/courses/{courseId}/lectures → 강의 목록
/student/lectures/{lectureId} → 강의 시청

### 📌 코딩 연습
/student/practice            → 실습 문제 목록
/student/practice/level/{level} → 레벨별 문제
/student/practice/{problemId} → 문제 상세
/student/practice/{problemId}/submit → 코드 제출

### 📌 시험 / 평가
/student/exams               → 시험 목록
/student/exams/levelup       → 레벨업 시험
/student/exams/{examId}      → 시험 응시
/student/exams/{examId}/result → 시험 결과

### 📌 출석 / 이력
/student/attendance          → 출석 기록
/student/history             → 학습 로그
/student/history/exams       → 시험 응시 이력

### 📌 결제 / 수강권
/student/payments            → 결제 내역
/student/payments/credits    → 크레딧 현황
/student/payments/purchase  → 수강권 구매
/student/payments/refunds   → 환불 내역

### 📌 Q/A
/student/qna                 → 내 질문 목록
/student/qna/new             → 질문 등록
/student/qna/{id}            → 질문 상세

### 📌 알림 / 설정
/student/notifications       → 알림 목록

/student/settings            → 설정
/student/settings/profile    → 회원 정보 수정
/student/settings/password   → 비밀번호 변경

## 🟨 강사 영역

(강사 승인 후 접근 가능)

/instructor/dashboard        → 강사 대시보드

/instructor/courses          → 강좌 관리
/instructor/courses/new      → 강좌 개설 요청
/instructor/courses/{id}     → 강좌 상세

### 📌 강의 / 과제 관리
/instructor/lectures         → 강의 관리
/instructor/lectures/upload  → 강의 업로드 요청

/instructor/assignments      → 과제 관리
/instructor/assignments/new  → 과제 등록
/instructor/assignments/{id} → 과제 상세

### 📌 Q/A / 정산
/instructor/qna              → 질문 관리
/instructor/qna/{id}         → 답변 등록 / 수정

/instructor/settlement       → 정산 내역
/instructor/account          → 계좌 정보 관리

### 📌 알림
/instructor/notifications    → 강사 알림

## 🟥 관리자 영역

(관리자 전용)

/admin/dashboard             → 관리자 대시보드

### 📌 회원 / 권한 관리
/admin/users                 → 전체 회원 관리
/admin/users/students        → 수강생 관리
/admin/users/instructors     → 강사 관리

### 📌 강좌 / 강의 승인
/admin/courses               → 강좌 관리
/admin/courses/pending       → 승인 대기 강좌
/admin/courses/{id}/approve  → 승인
/admin/courses/{id}/reject   → 반려

/admin/lectures              → 강의 관리

### 📌 결제 / 커뮤니티 / 시험
/admin/payments              → 결제 내역
/admin/refunds               → 환불 처리

/admin/community             → 커뮤니티 관리
/admin/reports               → 신고 처리

/admin/exams                 → 시험 관리
/admin/exams/questions       → 시험 문제 관리

### 📌 로그 / 공지 / 알림
/admin/logs                  → 로그 조회
/admin/logs/attendance       → 출석 로그
/admin/logs/exams            → 시험 로그

/admin/notices               → 공지 관리
/admin/notifications         → 알림 관리