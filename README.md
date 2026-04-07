# PetMediScan Infrastructure

반려동물 안구/피부질환 진단 애플리케이션 인프라 구성

## 기술 스택

- **Container**: Docker, Docker Compose
- **CI/CD**: Jenkins
- **Database**: MySQL 8.0
- **Reverse Proxy**: Nginx (선택)
- **Monitoring**: Prometheus + Grafana (선택)

## 포트 할당

| 서비스 | 포트 | 설명 |
|--------|------|------|
| MySQL | 3308 | 데이터베이스 |
| Spring Boot | 8080 | Backend API |
| FastAPI (Eye) | 5000 | 안구질환 AI 모델 |
| FastAPI (Skin) | 5001 | 피부질환 AI 모델 |
| Jenkins | 8090 | CI/CD 서버 |
| React Native Metro | 8081 | 모바일 개발 서버 |

## 프로젝트 구조

```
pet-infra/
├── docker/
│   ├── mysql/
│   │   ├── Dockerfile
│   │   └── init.sql           # 초기 스키마
│   ├── backend/
│   │   └── Dockerfile
│   ├── ai-eye/
│   │   └── Dockerfile
│   └── ai-skin/
│       └── Dockerfile
│
├── jenkins/
│   ├── Jenkinsfile.backend    # Backend CI/CD
│   ├── Jenkinsfile.ai-eye     # AI Eye CI/CD
│   └── Jenkinsfile.ai-skin    # AI Skin CI/CD
│
├── nginx/
│   └── nginx.conf             # Reverse Proxy 설정
│
├── docker-compose.yml         # 전체 서비스 구성
└── README.md
```

## Docker Compose 구성

### docker-compose.yml

```yaml
version: '3.8'

services:
  # MySQL 데이터베이스
  mysql:
    image: mysql:8.0
    container_name: petmediscan-mysql
    restart: always
    ports:
      - "3308:3306"
    environment:
      MYSQL_ROOT_PASSWORD: petmedi123
      MYSQL_DATABASE: petmediscan
      MYSQL_USER: petuser
      MYSQL_PASSWORD: petpass
    volumes:
      - mysql_data:/var/lib/mysql
      - ./docker/mysql/init.sql:/docker-entrypoint-initdb.d/init.sql
    networks:
      - petmediscan-network

  # Spring Boot Backend
  backend:
    build:
      context: ../pet-backend
      dockerfile: Dockerfile
    container_name: petmediscan-backend
    restart: always
    ports:
      - "8080:8080"
    environment:
      SPRING_DATASOURCE_URL: jdbc:mysql://mysql:3306/petmediscan
      SPRING_DATASOURCE_USERNAME: petuser
      SPRING_DATASOURCE_PASSWORD: petpass
      AI_EYE_SERVICE_URL: http://ai-eye:5000
      AI_SKIN_SERVICE_URL: http://ai-skin:5001
    depends_on:
      - mysql
    networks:
      - petmediscan-network

  # 안구질환 AI 모델 서버
  ai-eye:
    build:
      context: ../pet-ai-eye
      dockerfile: Dockerfile
    container_name: petmediscan-ai-eye
    restart: always
    ports:
      - "5000:5000"
    volumes:
      - ./models/eye:/app/models
    networks:
      - petmediscan-network

  # 피부질환 AI 모델 서버
  ai-skin:
    build:
      context: ../pet-ai-skin
      dockerfile: Dockerfile
    container_name: petmediscan-ai-skin
    restart: always
    ports:
      - "5001:5001"
    volumes:
      - ./models/skin:/app/models
    networks:
      - petmediscan-network

  # Jenkins CI/CD
  jenkins:
    image: jenkins/jenkins:lts
    container_name: petmediscan-jenkins
    restart: always
    ports:
      - "8090:8080"
      - "50000:50000"
    volumes:
      - jenkins_data:/var/jenkins_home
      - /var/run/docker.sock:/var/run/docker.sock
    networks:
      - petmediscan-network

volumes:
  mysql_data:
  jenkins_data:

networks:
  petmediscan-network:
    driver: bridge
```

## 사용 방법

### 1. 전체 서비스 시작

```bash
# 모든 서비스 시작
docker-compose up -d

# 로그 확인
docker-compose logs -f

# 특정 서비스만 시작
docker-compose up -d mysql backend
```

### 2. 서비스 중지

```bash
# 모든 서비스 중지
docker-compose down

# 볼륨까지 삭제
docker-compose down -v
```

### 3. 서비스 재시작

```bash
# 특정 서비스 재시작
docker-compose restart backend

# 전체 재시작
docker-compose restart
```

### 4. 빌드 및 재시작

```bash
# 이미지 재빌드
docker-compose build

# 재빌드 후 시작
docker-compose up -d --build
```

## Jenkins CI/CD 설정

### 1. Jenkins 초기 설정

```bash
# Jenkins 초기 비밀번호 확인
docker exec petmediscan-jenkins cat /var/jenkins_home/secrets/initialAdminPassword

# 브라우저에서 접속
http://localhost:8090
```

### 2. Jenkins Pipeline (Backend)

```groovy
// Jenkinsfile.backend
pipeline {
    agent any
    
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/KoreatIT125/pet-backend.git'
            }
        }
        
        stage('Build') {
            steps {
                sh './gradlew clean build'
            }
        }
        
        stage('Test') {
            steps {
                sh './gradlew test'
            }
        }
        
        stage('Docker Build') {
            steps {
                sh 'docker build -t petmediscan-backend:${BUILD_NUMBER} .'
            }
        }
        
        stage('Deploy') {
            steps {
                sh '''
                    docker stop petmediscan-backend || true
                    docker rm petmediscan-backend || true
                    docker run -d -p 8080:8080 \
                        --name petmediscan-backend \
                        petmediscan-backend:${BUILD_NUMBER}
                '''
            }
        }
    }
    
    post {
        success {
            echo 'Backend deployment successful!'
        }
        failure {
            echo 'Backend deployment failed!'
        }
    }
}
```

### 3. Webhook 설정

GitHub Repository Settings → Webhooks:
- Payload URL: `http://<jenkins-url>:8090/github-webhook/`
- Content type: `application/json`
- Events: `Push events`, `Pull requests`

## 데이터베이스 초기화

### init.sql

```sql
-- 사용자 테이블
CREATE TABLE users (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,
    password VARCHAR(255) NOT NULL,
    name VARCHAR(100) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- 반려동물 테이블
CREATE TABLE pets (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    user_id BIGINT NOT NULL,
    name VARCHAR(100) NOT NULL,
    species ENUM('DOG', 'CAT') NOT NULL,
    breed VARCHAR(100),
    birth_date DATE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- 진단 이력 테이블
CREATE TABLE diagnoses (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    pet_id BIGINT NOT NULL,
    type ENUM('EYE', 'SKIN') NOT NULL,
    image_url VARCHAR(500) NOT NULL,
    result VARCHAR(100) NOT NULL,
    confidence DECIMAL(5,2) NOT NULL,
    diagnosis_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (pet_id) REFERENCES pets(id) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- 질환 정보 테이블
CREATE TABLE diseases (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    type ENUM('EYE', 'SKIN') NOT NULL,
    description TEXT,
    treatment TEXT
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- 초기 질환 데이터 삽입
INSERT INTO diseases (name, type, description, treatment) VALUES
('결막염', 'EYE', '눈의 결막에 발생한 염증으로 충혈, 눈곱, 눈물 등의 증상이 나타납니다.', '수의사 진료가 필요합니다. 항생제 안약으로 치료 가능합니다.'),
('백내장', 'EYE', '수정체가 혼탁해져 시력이 저하되는 질환입니다.', '수술적 치료가 필요할 수 있습니다.'),
('각막염', 'EYE', '각막에 염증이 발생하여 통증과 시력 저하가 나타납니다.', '항생제 치료 및 수의사 진료가 필요합니다.'),
('녹내장', 'EYE', '안압 상승으로 시신경이 손상되는 질환입니다.', '응급 치료가 필요하며, 안압 조절 약물 치료를 시행합니다.'),
('피부염', 'SKIN', '피부 염증으로 발적, 부종, 가려움증 등이 나타납니다.', '항염증제 및 항생제 치료가 필요할 수 있습니다.'),
('탈모', 'SKIN', '국소적 또는 전신적 털 손실이 발생합니다.', '원인 파악 후 적절한 치료가 필요합니다.'),
('습진', 'SKIN', '가려움과 발진이 나타나는 피부 질환입니다.', '항히스타민제 및 스테로이드 치료가 필요할 수 있습니다.'),
('곰팡이 감염', 'SKIN', '백선 등 곰팡이에 의한 피부 감염입니다.', '항진균제 치료가 필요합니다.');
```

## 모니터링 (선택 사항)

### Prometheus + Grafana

```yaml
# docker-compose.monitoring.yml 추가
  prometheus:
    image: prom/prometheus
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus/prometheus.yml:/etc/prometheus/prometheus.yml

  grafana:
    image: grafana/grafana
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin
```

## 트러블슈팅

### 1. 포트 충돌
```bash
# 사용 중인 포트 확인 (Windows)
netstat -ano | findstr :3308

# 프로세스 종료
taskkill /PID <PID> /F
```

### 2. Docker 네트워크 문제
```bash
# 네트워크 재생성
docker network rm petmediscan-network
docker network create petmediscan-network
```

### 3. MySQL 초기화 실패
```bash
# 볼륨 삭제 후 재시작
docker-compose down -v
docker-compose up -d mysql
```

## Git Workflow

### Branch 전략
- `main`: 프로덕션 설정
- `develop`: 개발 설정
- `feature/infra-기능명`: 인프라 기능 추가

### Commit Convention
```
infra: 인프라 설정 추가/수정
docker: Docker 설정 변경
ci: CI/CD 설정 변경
docs: 문서 수정
```

## 팀 구성원

- Infra 담당자 1: Docker, CI/CD 관리
- Infra 담당자 2: 데이터베이스, 모니터링

## 참고 자료

- [Docker Compose 공식 문서](https://docs.docker.com/compose/)
- [Jenkins 공식 문서](https://www.jenkins.io/doc/)
- [MySQL Docker 이미지](https://hub.docker.com/_/mysql)

## 라이선스

이 프로젝트는 교육 목적으로 제작되었습니다.
