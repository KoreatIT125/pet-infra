# Jenkins CI/CD 사용 가이드 (팀원용)

## 🎯 개요

**Jenkins가 뭐예요?**
- 코드를 Push하면 자동으로 빌드/테스트/배포해주는 시스템
- 여러분이 할 일: **코드 작성 → Git Push → 끝!**
- Jenkins가 할 일: 빌드, 테스트, 배포 (모두 자동)

---

## 🚀 기본 워크플로우

```
1. 코드 작성 (VSCode, IntelliJ 등)
   ↓
2. Git add → commit → push
   ↓
3. GitHub Webhook이 Jenkins에 알림
   ↓
4. Jenkins가 자동으로:
   - 코드 체크아웃
   - 빌드 (Gradle, npm 등)
   - 테스트 실행
   - Docker 이미지 생성
   - 컨테이너 재배포
   ↓
5. 결과 확인 (성공/실패)
```

**여러분은 1~2번만 하면 돼요!** ✅

---

## 📋 실전 사용법

### 1️⃣ 코드 작성 및 Push

**예시 - Backend 팀원:**

```bash
# 1. 코드 수정
# VSCode나 IntelliJ에서 MemberController.java 수정

# 2. Git 작업
git add .
git commit -m "feat: Add member registration API"
git push origin master

# 3. 끝! 나머지는 Jenkins가 알아서 처리
```

---

### 2️⃣ Jenkins에서 빌드 상태 확인

**Jenkins 접속:**
- URL: **http://localhost:8090**
- 계정: `admin` / `7b5950fa4bcc4e6aa5b90217a7e859bf`

**빌드 확인 방법:**

1. Jenkins 메인 페이지 접속
2. 자신의 프로젝트 클릭:
   - Backend 팀 → **PetMediScan-Backend**
   - AI Eye 팀 → **PetMediScan-AI-Eye**
   - AI Skin 팀 → **PetMediScan-AI-Skin**
   - Frontend 팀 → **PetMediScan-Frontend**

3. **Build History** 확인:
   - 🔵 파란색 = 진행 중
   - ✅ 초록색 = 성공
   - ❌ 빨간색 = 실패

---

### 3️⃣ 빌드 성공/실패 확인

#### ✅ **성공 시:**

- Build History에 초록색 체크 표시
- 자동으로 배포 완료!
- 서비스 재시작됨
- **아무것도 안 해도 돼요!**

#### ❌ **실패 시:**

1. 빌드 번호 클릭 (예: #15)
2. 왼쪽 **Console Output** 클릭
3. 빨간색 에러 메시지 확인
4. 에러 수정 후 다시 Push

**주요 에러 예시:**

```
컴파일 에러:
[ERROR] /workspace/src/main/java/Controller.java:[25,10] error: ';' expected

→ 해결: 코드 문법 오류 수정

테스트 실패:
Tests run: 5, Failures: 1, Errors: 0, Skipped: 0

→ 해결: 실패한 테스트 수정

Docker 빌드 실패:
ERROR: failed to solve: process "/bin/sh -c ..." did not complete

→ 해결: Dockerfile 또는 의존성 문제 수정
```

---

## 🔄 자주 하는 작업

### 📌 **수동으로 빌드 실행하기**

**언제 필요?**
- Push 없이 테스트하고 싶을 때
- Webhook이 안 작동할 때

**방법:**
1. Jenkins → 프로젝트 선택
2. 왼쪽 **"지금 빌드"** 클릭
3. Build History에서 진행 상황 확인

---

### 📌 **빌드 로그 보는 법**

**상세 로그 확인:**
1. Build History에서 빌드 번호 클릭 (예: #15)
2. **Console Output** 클릭
3. 전체 빌드 과정 확인

**무엇을 볼 수 있나요?**
- Git checkout 내역
- 빌드 명령어 실행 과정
- 테스트 결과
- Docker 이미지 빌드 과정
- 배포 로그

---

### 📌 **이전 빌드 기록 보기**

1. 프로젝트 메인 페이지
2. **Build History** 섹션
3. 원하는 빌드 번호 클릭

**확인 가능한 정보:**
- 누가 Push했는지 (wpghksdnd 등)
- 언제 빌드됐는지
- 성공/실패 여부
- 소요 시간

---

## 🎯 팀별 가이드

### **Backend 팀 (Spring Boot)**

**빌드 과정:**
1. Git checkout
2. Gradle 빌드 (`./gradlew clean build`)
3. 테스트 실행 (`./gradlew test`)
4. Docker 이미지 빌드
5. 컨테이너 재배포 (포트 8080)

**확인 방법:**
- API 테스트: `http://localhost:8080/api/...`
- Swagger UI: `http://localhost:8080/swagger-ui/`

---

### **AI Eye 팀 (FastAPI - 안구질환)**

**빌드 과정:**
1. Git checkout
2. Docker 이미지 빌드 (Python + FastAPI)
3. 컨테이너 재배포 (포트 5000)

**확인 방법:**
- Health Check: `http://localhost:5000/health`
- API 문서: `http://localhost:5000/docs`

---

### **AI Skin 팀 (FastAPI - 피부질환)**

**빌드 과정:**
1. Git checkout
2. Docker 이미지 빌드
3. 컨테이너 재배포 (포트 5001)

**확인 방법:**
- Health Check: `http://localhost:5001/health`
- API 문서: `http://localhost:5001/docs`

---

### **Frontend 팀 (React Native)**

**빌드 과정:**
1. Git checkout
2. Node.js 설치 (자동)
3. npm 의존성 설치
4. Lint 검사
5. 빌드 (선택적)

**확인 방법:**
- 로컬 개발: `npm start` (포트 8081)
- 빌드 성공만 확인하면 됨

---

## ⚠️ 주의사항

### ❗ **ngrok 창 닫지 마세요!**

Jenkins 서버에서 **ngrok이 실행 중**이어야 Webhook이 작동합니다.

```
Forwarding: https://relic-unsecured-shrine.ngrok-free.dev -> http://localhost:8090
```

**ngrok 창을 닫으면:**
- ❌ Push해도 자동 빌드 안 됨
- ✅ "지금 빌드" 수동 실행은 가능

---

### ❗ **여러 명이 동시에 Push하면?**

**걱정 마세요!**
- Jenkins가 빌드를 **큐(Queue)**에 넣어 순차 실행
- 먼저 Push한 사람부터 빌드
- 각자의 빌드 번호로 구분

---

### ❗ **빌드 실패했는데 원인 모르겠어요**

**도움 요청 방법:**

1. Jenkins → 실패한 빌드 클릭
2. **Console Output** 전체 복사
3. 팀 채팅방 또는 인프라 담당자에게 공유
4. 에러 메시지 스크린샷 첨부

---

## 🧪 테스트 시나리오

### **신규 팀원 첫 사용 테스트**

**단계별 실습:**

#### 1. 간단한 파일 수정
```bash
# 자신의 Repository clone (이미 했다면 skip)
git clone https://github.com/KoreatIT125/pet-backend.git
cd pet-backend

# README 파일에 자기 이름 추가
echo "- 개발자: 홍길동" >> README.md

# Git 작업
git add README.md
git commit -m "docs: Add developer name"
git push origin master
```

#### 2. Jenkins 확인
- Jenkins 접속: http://localhost:8090
- PetMediScan-Backend 클릭
- Build History에 새 빌드 생성 확인 (1~2분 소요)

#### 3. 결과 확인
- 초록색 = 성공! 🎉
- 빨간색 = Console Output 확인

---

### **의도적으로 빌드 실패 만들기 (연습용)**

**Backend 예시:**

```java
// src/main/java/com/example/demo/TestController.java
public class TestController {
    public String test() {
        return "test"  // 세미콜론 빼먹음 (의도적 에러)
    }
}
```

**Push 후 빌드 실패 → Console Output에서 에러 확인 → 수정 → 다시 Push**

---

## 📊 빌드 시간 참고

| 프로젝트 | 평균 빌드 시간 |
|---------|--------------|
| Backend | 2~3분 |
| AI Eye | 1~2분 |
| AI Skin | 1~2분 |
| Frontend | 2~3분 (첫 빌드), 1분 (이후) |

**빌드가 5분 이상 걸리면 문제일 수 있어요!**

---

## 🆘 문제 해결

### **Q1: Push했는데 빌드가 안 시작돼요**

**해결:**
1. ngrok 실행 확인
2. GitHub Webhook 확인 (Settings → Webhooks → Recent Deliveries)
3. 수동 빌드 ("지금 빌드" 클릭)

---

### **Q2: 빌드는 성공했는데 변경사항이 안 보여요**

**Backend/AI 서비스:**
- Docker 컨테이너 재시작 확인: `docker ps | grep petmediscan`
- 로그 확인: `docker logs petmediscan-backend`

**Frontend:**
- 로컬 개발 서버 재시작: `npm start`

---

### **Q3: Jenkins 접속이 안 돼요**

**확인 사항:**
1. Docker 컨테이너 실행 중? `docker ps | grep jenkins`
2. 포트 충돌? `netstat -ano | findstr :8090`
3. 인프라 담당자에게 문의

---

## 📞 문의

**Jenkins 관련 문제:**
- 인프라 담당: 환웅님 (`@wpghksdnd`)

**Git/GitHub 문제:**
- 각 팀 리더에게 문의

**빌드 에러:**
- 각 팀 내에서 먼저 확인
- 해결 안 되면 전체 회의

---

## ✅ 체크리스트

**신규 팀원이 확인해야 할 사항:**

- [ ] Jenkins 접속 가능 (http://localhost:8090)
- [ ] 계정 정보 받음 (admin / 비밀번호)
- [ ] Git Repository clone 완료
- [ ] 테스트 Push → 자동 빌드 확인 완료
- [ ] Console Output 보는 법 숙지
- [ ] 수동 빌드 방법 숙지
- [ ] ngrok 창 닫으면 안 된다는 것 이해

---

## 🎓 추가 학습

**Jenkins 더 알아보기:**
- Pipeline 문법: https://www.jenkins.io/doc/book/pipeline/
- Docker 통합: https://www.jenkins.io/doc/book/pipeline/docker/

**Git 협업:**
- Feature Branch 전략
- Pull Request 워크플로우
- Merge Conflict 해결

---

**업데이트:** 2026-04-12
**작성자:** Infra팀 (환웅)
