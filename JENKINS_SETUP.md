# Jenkins CI/CD 설정 가이드

## 1. Jenkins 초기 설정

### 1.1 Jenkins 접속
```
http://localhost:8090
```

### 1.2 초기 비밀번호 확인
```bash
docker exec petmediscan-jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

비밀번호 복사 후 Jenkins 웹에 입력

### 1.3 플러그인 설치
- "Install suggested plugins" 선택
- 추가 설치 필요한 플러그인:
  - Git plugin
  - GitHub plugin
  - Docker plugin
  - Pipeline plugin
  - Gradle plugin

---

## 2. GitHub Webhook 설정

### 2.1 GitHub Personal Access Token 생성
1. GitHub → Settings → Developer settings → Personal access tokens → Tokens (classic)
2. "Generate new token (classic)" 클릭
3. 권한 선택:
   - `repo` (전체 체크)
   - `admin:repo_hook` (전체 체크)
4. Token 생성 후 복사 (한 번만 표시됨!)

### 2.2 Jenkins에 GitHub Credentials 추가
1. Jenkins → Manage Jenkins → Credentials
2. (global) → Add Credentials
3. 정보 입력:
   - Kind: `Username with password`
   - Username: GitHub 사용자명
   - Password: Personal Access Token (위에서 생성)
   - ID: `github-token`
   - Description: `GitHub Access Token`
4. OK 클릭

---

## 3. Pipeline Job 생성

### 3.1 Backend Pipeline

1. Jenkins 메인 → New Item
2. Item name: `PetMediScan-Backend`
3. Type: `Pipeline`
4. OK 클릭

#### 설정:
- **General**
  - [x] GitHub project
  - Project url: `https://github.com/KoreatIT125/pet-backend`

- **Build Triggers**
  - [x] GitHub hook trigger for GITScm polling

- **Pipeline**
  - Definition: `Pipeline script from SCM`
  - SCM: `Git`
  - Repository URL: `https://github.com/KoreatIT125/pet-backend.git`
  - Credentials: `github-token` (위에서 생성)
  - Branch Specifier: `*/main`
  - Script Path: `Jenkinsfile`

5. Save

### 3.2 AI Eye Pipeline

동일한 방법으로 생성:
- Item name: `PetMediScan-AI-Eye`
- Repository URL: `https://github.com/KoreatIT125/pet-ai-eye.git`
- Script Path: `Jenkinsfile`

### 3.3 AI Skin Pipeline

- Item name: `PetMediScan-AI-Skin`
- Repository URL: `https://github.com/KoreatIT125/pet-ai-skin.git`
- Script Path: `Jenkinsfile`

---

## 4. GitHub Webhook 연동

각 Repository에 Webhook 추가:

### 4.1 Backend Repository Webhook
1. https://github.com/KoreatIT125/pet-backend/settings/hooks
2. "Add webhook" 클릭
3. 정보 입력:
   - Payload URL: `http://<서버-IP>:8090/github-webhook/`
   - Content type: `application/json`
   - Which events: `Just the push event`
   - [x] Active
4. Add webhook

### 4.2 AI Eye, AI Skin도 동일하게 설정

---

## 5. GitHub Branch Protection 설정

### 5.1 Backend Repository 보호
1. https://github.com/KoreatIT125/pet-backend/settings/branches
2. "Add rule" 클릭
3. 설정:
   - Branch name pattern: `main`
   - [x] Require a pull request before merging
     - [x] Require approvals: `1`
   - [x] Require status checks to pass before merging
     - [x] Require branches to be up to date before merging
   - [x] Do not allow bypassing the above settings
4. Create

### 5.2 AI Eye, AI Skin, Frontend, Dataset, Infra도 동일하게 설정

---

## 6. 워크플로우 테스트

### 6.1 Feature 브랜치 생성
```bash
git checkout -b feature/backend-test
echo "test" > test.txt
git add test.txt
git commit -m "test: Jenkins 테스트"
git push origin feature/backend-test
```

### 6.2 Pull Request 생성
1. GitHub에서 "Compare & pull request" 클릭
2. 제목: `[Test] Jenkins CI/CD 테스트`
3. "Create pull request" 클릭

### 6.3 팀원 승인 대기
- 팀원이 Code Review 후 Approve

### 6.4 Merge
1. "Merge pull request" 클릭
2. "Confirm merge" 클릭

### 6.5 Jenkins 자동 빌드 확인
- Jenkins 대시보드에서 빌드 시작 확인
- 로그 확인: Console Output

---

## 7. Jenkins 도구 설정

### 7.1 JDK 설정
1. Manage Jenkins → Global Tool Configuration
2. JDK → Add JDK
   - Name: `JDK17`
   - [x] Install automatically
   - Version: `jdk-17.0.2+8`
3. Save

### 7.2 Gradle 설정 (선택)
- 각 프로젝트에 `gradlew` 포함되어 있으므로 별도 설정 불필요

---

## 8. 트러블슈팅

### 8.1 Docker 권한 오류
```bash
# Jenkins 컨테이너 내부에서 Docker 실행 권한 추가
docker exec -u root petmediscan-jenkins chmod 666 /var/run/docker.sock
```

### 8.2 Git Credentials 오류
- Jenkins Credentials에서 GitHub Token 재확인
- Token 권한 확인 (repo, admin:repo_hook)

### 8.3 Webhook이 작동하지 않을 때
- GitHub Webhook 설정 → Recent Deliveries 확인
- Response 200이 아니면 Jenkins 로그 확인

---

## 9. 알림 설정 (선택 사항)

### 9.1 Slack 알림
1. Slack에서 Incoming Webhook URL 생성
2. Jenkins → Manage Jenkins → Configure System
3. Slack 섹션 설정
4. Jenkinsfile에 추가:
```groovy
post {
    success {
        slackSend color: 'good', message: "배포 성공: ${env.JOB_NAME}"
    }
}
```

---

## 10. 정기 점검

### 주기적으로 확인:
- [ ] Jenkins 업데이트
- [ ] 플러그인 업데이트
- [ ] 디스크 용량
- [ ] 빌드 이력 삭제 (Build History)

---

**완료!** 🎉

이제 팀원이 PR을 생성하고 승인을 받으면, Jenkins가 자동으로:
1. 빌드
2. 테스트
3. Docker 이미지 생성
4. 컨테이너 재배포

를 수행합니다!
