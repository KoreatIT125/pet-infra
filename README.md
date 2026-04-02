# 🐟 Infra - 북태평양 연어 지능형 양식장 인프라

Docker Compose 기반 전체 시스템 배포

---

## 📋 **목차**

- [개요](#-개요)
- [시스템 구성](#-시스템-구성)
- [빠른 시작](#-빠른-시작)
- [서비스 상세](#-서비스-상세)
- [설정 파일](#-설정-파일)

---

## 🎯 **개요**

**Docker Compose**로 전체 시스템을 한 번에 실행할 수 있습니다.

### **포함 서비스**
- **Frontend** (React) - 포트 3000
- **Backend** (Spring Boot) - 포트 8080
- **AI Model** (Flask) - 포트 5000
- **MySQL** - 포트 3306

---

## 🏗️ **시스템 구성**

```
┌─────────────────────────────────────────────┐
│           사용자 (웹 브라우저)              │
└────────────────┬────────────────────────────┘
                 │ :3000
     ┌───────────▼───────────┐
     │   Frontend (React)    │
     │  - Nginx (Production) │
     └───────────┬───────────┘
                 │ :8080
     ┌───────────▼───────────┐
     │ Backend (Spring Boot) │
     │  - REST API           │
     │  - WebSocket          │
     └─────┬─────────────┬───┘
           │ :5000       │ :3306
   ┌───────▼──────┐  ┌──▼───────────┐
   │  AI Model    │  │  MySQL 8.0   │
   │  (Flask)     │  │  Database    │
   │  - YOLOv8    │  │              │
   └──────────────┘  └──────────────┘
```

---

## 🚀 **빠른 시작**

### **1. 전체 시스템 실행**
```bash
# docker-compose.yml이 있는 디렉토리에서
docker-compose up -d
```

### **2. 서비스 확인**
```bash
docker-compose ps
```

**예상 출력**:
```
NAME                  IMAGE                   STATUS
salmon-frontend       salmon-frontend:latest  Up
salmon-backend        salmon-backend:latest   Up
salmon-ai-model       salmon-ai-model:latest  Up
salmon-db             mysql:8.0               Up
```

### **3. 접속 테스트**
- Frontend: http://localhost:3000
- Backend API: http://localhost:8080/api/health
- AI Model API: http://localhost:5000/health
- MySQL: localhost:3306

---

### **4. 로그 확인**
```bash
# 전체 로그
docker-compose logs -f

# 특정 서비스 로그
docker-compose logs -f frontend
docker-compose logs -f backend
docker-compose logs -f ai-model
```

---

### **5. 시스템 종료**
```bash
# 컨테이너 중지
docker-compose stop

# 컨테이너 삭제
docker-compose down

# 볼륨까지 삭제 (데이터 초기화)
docker-compose down -v
```

---

## 📦 **서비스 상세**

### **Frontend (React + Nginx)**
```yaml
frontend:
  build: ../frontend
  ports:
    - "3000:80"
  environment:
    - VITE_API_BASE_URL=http://localhost:8080/api
  depends_on:
    - backend
```

**역할**:
- React 앱 정적 파일 서빙 (Nginx)
- Backend API 프록시

---

### **Backend (Spring Boot)**
```yaml
backend:
  build: ../backend
  ports:
    - "8080:8080"
  environment:
    - SPRING_DATASOURCE_URL=jdbc:mysql://db:3306/salmon_farm
    - SPRING_DATASOURCE_USERNAME=salmon_user
    - SPRING_DATASOURCE_PASSWORD=salmon_pass123
    - AI_MODEL_BASE_URL=http://ai-model:5000
  depends_on:
    - db
    - ai-model
```

**역할**:
- REST API 제공
- 데이터베이스 관리
- AI 모델 호출

---

### **AI Model (Flask + YOLOv8)**
```yaml
ai-model:
  build: ../ai-model
  ports:
    - "5000:5000"
  volumes:
    - ../ai-model/models:/app/models
  deploy:
    resources:
      reservations:
        devices:
          - driver: nvidia
            count: 1
            capabilities: [gpu]
```

**역할**:
- YOLO 모델 추론
- 연어 감지, 크기 측정, 질병 감지

**GPU 지원**:
- NVIDIA Docker Runtime 필요
- GPU 없이 실행 시 CPU로 자동 전환

---

### **MySQL Database**
```yaml
db:
  image: mysql:8.0
  ports:
    - "3306:3306"
  environment:
    - MYSQL_ROOT_PASSWORD=root_password
    - MYSQL_DATABASE=salmon_farm
    - MYSQL_USER=salmon_user
    - MYSQL_PASSWORD=salmon_pass123
  volumes:
    - mysql_data:/var/lib/mysql
    - ./init.sql:/docker-entrypoint-initdb.d/init.sql
```

**역할**:
- 연어 감지 이력 저장
- 센서 데이터 저장
- 사료 공급 이력 저장

---

## ⚙️ **설정 파일**

### **docker-compose.yml** (전체)
```yaml
version: '3.8'

services:
  frontend:
    build: ../frontend
    container_name: salmon-frontend
    ports:
      - "3000:80"
    environment:
      - VITE_API_BASE_URL=http://localhost:8080/api
    depends_on:
      - backend
    networks:
      - salmon-network

  backend:
    build: ../backend
    container_name: salmon-backend
    ports:
      - "8080:8080"
    environment:
      - SPRING_DATASOURCE_URL=jdbc:mysql://db:3306/salmon_farm
      - SPRING_DATASOURCE_USERNAME=salmon_user
      - SPRING_DATASOURCE_PASSWORD=salmon_pass123
      - AI_MODEL_BASE_URL=http://ai-model:5000
    depends_on:
      - db
      - ai-model
    networks:
      - salmon-network

  ai-model:
    build: ../ai-model
    container_name: salmon-ai-model
    ports:
      - "5000:5000"
    volumes:
      - ../ai-model/models:/app/models
    networks:
      - salmon-network

  db:
    image: mysql:8.0
    container_name: salmon-db
    ports:
      - "3306:3306"
    environment:
      - MYSQL_ROOT_PASSWORD=root_password
      - MYSQL_DATABASE=salmon_farm
      - MYSQL_USER=salmon_user
      - MYSQL_PASSWORD=salmon_pass123
    volumes:
      - mysql_data:/var/lib/mysql
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql
    networks:
      - salmon-network

volumes:
  mysql_data:

networks:
  salmon-network:
    driver: bridge
```

---

### **init.sql** (데이터베이스 초기화)
```sql
-- 연어 감지 이력 테이블
CREATE TABLE IF NOT EXISTS fish_detection (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    tank_id VARCHAR(50) NOT NULL,
    count INT NOT NULL,
    average_length_mm DECIMAL(6,2),
    average_weight_g DECIMAL(7,2),
    image_path VARCHAR(255),
    detected_at DATETIME NOT NULL,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    INDEX idx_tank_detected (tank_id, detected_at)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- 수질 센서 데이터 테이블
CREATE TABLE IF NOT EXISTS water_sensor (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    tank_id VARCHAR(50) NOT NULL,
    temperature_c DECIMAL(4,2),
    salinity_ppt DECIMAL(4,2),
    do_mg_l DECIMAL(4,2),
    ph DECIMAL(3,2),
    orp_mv DECIMAL(5,2),
    illuminance_lux DECIMAL(6,2),
    measured_at DATETIME NOT NULL,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    INDEX idx_tank_measured (tank_id, measured_at)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- 사료 공급 이력 테이블
CREATE TABLE IF NOT EXISTS feed_supply (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    tank_id VARCHAR(50) NOT NULL,
    feed_type VARCHAR(50) NOT NULL,
    amount_g DECIMAL(7,2) NOT NULL,
    protein_percent DECIMAL(4,2),
    fat_percent DECIMAL(4,2),
    supplied_at DATETIME NOT NULL,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    INDEX idx_tank_supplied (tank_id, supplied_at)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- 수조 정보 테이블
CREATE TABLE IF NOT EXISTS tank_info (
    tank_id VARCHAR(50) PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    type VARCHAR(50),
    radius_cm INT,
    height_cm INT,
    location VARCHAR(255),
    status VARCHAR(20) DEFAULT 'active',
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- 샘플 데이터 삽입
INSERT INTO tank_info (tank_id, name, type, radius_cm, height_cm, location) VALUES
('tank_001', '1번 수조', '유수식', 520, 120, '경북 영덕군'),
('tank_002', '2번 수조', '유수식', 520, 120, '경북 영덕군'),
('tank_003', '3번 수조', '유수식', 520, 120, '경북 영덕군'),
('tank_004', '4번 수조', '유수식', 520, 120, '경북 영덕군');
```

---

## 🔧 **고급 설정**

### **GPU 지원 (AI Model)**
```bash
# NVIDIA Docker Runtime 설치 필요
# https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html

# GPU 사용 여부 확인
docker-compose exec ai-model python -c "import torch; print(torch.cuda.is_available())"
```

---

### **환경 변수 커스터마이징**
`.env` 파일 생성:
```bash
# MySQL
MYSQL_ROOT_PASSWORD=custom_root_password
MYSQL_DATABASE=salmon_farm
MYSQL_USER=custom_user
MYSQL_PASSWORD=custom_password

# Backend
AI_MODEL_BASE_URL=http://ai-model:5000

# Frontend
VITE_API_BASE_URL=http://localhost:8080/api
```

---

## 📊 **헬스 체크**

```bash
# Backend
curl http://localhost:8080/api/health

# AI Model
curl http://localhost:5000/health

# MySQL
docker-compose exec db mysql -u salmon_user -psalmon_pass123 -e "SELECT 1"
```

---

## 📝 **라이선스**

MIT License - [LICENSE](../LICENSE) 참조

---

**🐟 Docker로 연어 양식장 시스템을 쉽게 배포하세요!**
