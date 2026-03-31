# Infrastructure - Disaster Safety System

Docker Compose 및 배포 설정

## 🚀 Quick Start

### 전체 서비스 실행

```bash
docker-compose up -d
```

### 서비스 접속

- **Frontend**: http://localhost:3000
- **Backend API**: http://localhost:8080
- **AI Model**: http://localhost:5000
- **MySQL**: localhost:3306

### 서비스 중지

```bash
docker-compose down
```

### 로그 확인

```bash
docker-compose logs -f
```

## 📦 Services

### Frontend
- **Port**: 3000
- **Image**: disaster-safety-frontend
- **Depends**: backend

### Backend
- **Port**: 8080
- **Image**: disaster-safety-backend
- **Depends**: db, ai-model

### AI Model
- **Port**: 5000
- **Image**: disaster-safety-ai-model
- **GPU**: Required (NVIDIA)

### Database (MySQL)
- **Port**: 3306
- **Database**: safety_db
- **User**: safety_user
- **Password**: safety_pass

## 🐳 Docker Commands

### 개별 서비스 빌드

```bash
docker-compose build frontend
docker-compose build backend
docker-compose build ai-model
```

### 개별 서비스 재시작

```bash
docker-compose restart frontend
```

### 볼륨 삭제

```bash
docker-compose down -v
```

## 🔧 Environment Variables

### Backend

```env
SPRING_DATASOURCE_URL=jdbc:mysql://db:3306/safety_db
SPRING_DATASOURCE_USERNAME=safety_user
SPRING_DATASOURCE_PASSWORD=safety_pass
AI_MODEL_URL=http://ai-model:5000
```

### Frontend

```env
VITE_API_URL=http://localhost:8080
```

## 📊 Health Checks

```bash
# Frontend
curl http://localhost:3000

# Backend
curl http://localhost:8080/api/health

# AI Model
curl http://localhost:5000/health

# Database
docker-compose exec db mysql -u safety_user -p safety_db
```

## 🚀 Production Deployment

### AWS EC2

1. Install Docker & Docker Compose
2. Clone repository
3. Update environment variables
4. Run `docker-compose up -d`

### Kubernetes (Optional)

See `k8s/` directory for Kubernetes manifests.

## 📝 License

MIT
