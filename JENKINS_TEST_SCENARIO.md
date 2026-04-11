# 팀원 Jenkins CI/CD 테스트 시나리오

## 🎯 목적
각 팀원이 Jenkins CI/CD 파이프라인을 실제로 사용해보고 익히기 위한 테스트

---

## 👥 테스트 대상

- **Backend 팀** (2명)
- **AI Eye 팀** (1명)
- **AI Skin 팀** (1명)
- **Frontend 팀** (2명)

**총 6명 - 각자 자신의 Repository에서 테스트**

---

## 📋 사전 준비 (인프라 담당)

### ✅ 체크리스트
- [ ] ngrok 실행 중 (`https://relic-unsecured-shrine.ngrok-free.dev`)
- [ ] Jenkins 실행 중 (http://localhost:8090)
- [ ] Docker 컨테이너 정상 작동 확인
- [ ] GitHub Webhook 4개 모두 설정됨
- [ ] `JENKINS_USER_GUIDE.md` 팀원에게 공유

---

## 🧪 테스트 시나리오 (팀원별)

### 📌 **Level 1: 기본 Push 테스트** (필수)

**목표:** 코드 Push → 자동 빌드 확인

**단계:**

1. **Repository Clone (처음이면)**
   ```bash
   git clone https://github.com/KoreatIT125/pet-backend.git
   cd pet-backend
   ```

2. **간단한 파일 수정**
   ```bash
   echo "- 개발자: [본인이름]" >> README.md
   ```

3. **Git Push**
   ```bash
   git add README.md
   git commit -m "test: Add my name to README"
   git push origin master
   ```

4. **Jenkins 확인**
   - http://localhost:8090 접속
   - 자신의 프로젝트 클릭
   - Build History에서 새 빌드 생성 확인 (1~2분 대기)

**성공 조건:**
- ✅ 자동으로 새 빌드가 생성됨 (Started by GitHub push by [본인ID])
- ✅ 빌드가 초록색으로 성공

**실패 시:**
- Console Output 확인
- 인프라 담당자에게 로그 공유

---

### 📌 **Level 2: 의도적 빌드 실패 테스트** (선택)

**목표:** 빌드 실패 시 에러 확인 및 수정 경험

**Backend 팀:**

```java
// src/main/java/com/example/demo/controller/TestController.java
package com.example.demo.controller;

import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/test")
public class TestController {
    
    @GetMapping
    public String test() {
        return "test"  // 세미콜론 누락 (의도적 에러)
    }
}
```

**AI 팀:**

```python
# app/main.py에 추가
@app.get("/test")
def test():
    return {"message": "test"  # 중괄호 닫기 누락
```

**Frontend 팀:**

```typescript
// App.tsx에 추가
const test = () => {
  console.log("test"  // 괄호 닫기 누락
}
```

**단계:**

1. 위 코드 추가 (에러 포함)
2. Git Push
3. Jenkins에서 빌드 실패 확인 (빨간색)
4. Console Output에서 에러 메시지 찾기
5. 코드 수정 (세미콜론/괄호 추가)
6. 다시 Push → 성공 확인

**성공 조건:**
- ❌ 첫 빌드 실패 (에러 메시지 확인 가능)
- ✅ 수정 후 빌드 성공

---

### 📌 **Level 3: 수동 빌드 테스트** (필수)

**목표:** Push 없이 Jenkins에서 직접 빌드 실행

**단계:**

1. Jenkins → 자신의 프로젝트 클릭
2. 왼쪽 **"지금 빌드"** 클릭
3. Build History에서 새 빌드 생성 확인
4. Console Output 확인

**성공 조건:**
- ✅ 수동으로 빌드 실행 가능
- ✅ "Started by user admin" 메시지 확인

---

### 📌 **Level 4: Console Output 분석** (필수)

**목표:** 빌드 로그 읽고 이해하기

**단계:**

1. 가장 최근 빌드 클릭
2. **Console Output** 클릭
3. 아래 항목 찾아보기:
   - Git checkout 메시지 (`Checking out Revision ...`)
   - 빌드 명령어 실행 (`./gradlew build`, `npm install` 등)
   - 성공 메시지 (`BUILD SUCCESSFUL`, `✅ 빌드 및 배포 성공!`)

**성공 조건:**
- ✅ Console Output에서 주요 단계 확인 가능
- ✅ 에러 발생 시 어디서 발생했는지 파악 가능

---

## 📊 테스트 결과 기록

### **팀원별 체크리스트**

| 이름 | 팀 | Level 1 | Level 2 | Level 3 | Level 4 | 완료일 |
|------|-----|---------|---------|---------|---------|--------|
| 홍길동 | Backend | ✅ | ✅ | ✅ | ✅ | 2026-04-12 |
| 김철수 | Backend |  |  |  |  |  |
| 이영희 | AI Eye |  |  |  |  |  |
| 박민수 | AI Skin |  |  |  |  |  |
| 최지우 | Frontend |  |  |  |  |  |
| 정수진 | Frontend |  |  |  |  |  |

---

## 🎤 테스트 후 피드백 회의

### **질문 리스트**

각 팀원에게 물어볼 것:

1. **Jenkins 접속은 문제없었나요?**
2. **자동 빌드가 제대로 동작했나요?**
3. **Console Output을 이해할 수 있었나요?**
4. **어려웠던 부분은?**
5. **개선이 필요한 부분은?**

---

## ⚠️ 주의사항

### **테스트 중 문제 발생 시**

1. **즉시 멈추고** 인프라 담당자에게 알림
2. **Console Output 캡처** (스크린샷 또는 복사)
3. **에러 메시지 공유**

### **여러 명이 동시에 Push하면?**

- Jenkins가 순차 처리 (큐에 쌓임)
- 먼저 Push한 사람부터 빌드
- 걱정 안 해도 됨!

---

## 🎯 테스트 완료 기준

**전체 팀원이 아래 항목 완료:**

- [ ] Level 1 성공 (자동 빌드)
- [ ] Level 3 성공 (수동 빌드)
- [ ] Level 4 성공 (로그 이해)
- [ ] Level 2 (선택사항)

**완료 시:**
- ✅ Jenkins CI/CD 파이프라인 사용 준비 완료
- ✅ 본격적인 개발 시작 가능

---

## 📝 예상 소요 시간

- **1인당 테스트 시간:** 15~20분
- **전체 팀원 (6명):** 1~2시간 (동시 진행 시)
- **피드백 회의:** 30분

**총 예상 시간: 2~3시간**

---

## 📞 긴급 연락

**Jenkins 문제:**
- 인프라 담당: 환웅님

**Git 문제:**
- 각 팀 리더

**테스트 진행:**
- 전체 회의 또는 Discord 채널

---

**업데이트:** 2026-04-12
**작성자:** Infra팀 (환웅)
