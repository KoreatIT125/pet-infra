# Fish Detection Infrastructure

배포 및 인프라 설정

## 🚀 Tech Stack

- Docker & Docker Compose
- GitHub Actions (CI/CD)
- AWS / GCP (선택)
- Nginx
- PostgreSQL

## 📁 Directory Structure

```
infra/
├── docker/
│   ├── frontend/
│   │   └── Dockerfile
│   ├── backend/
│   │   └── Dockerfile
│   └── nginx/
│       └── nginx.conf
├── docker-compose.yml      # 로컬 개발 환경
├── docker-compose.prod.yml # 프로덕션 환경
├── .github/
│   └── workflows/
│       ├── frontend-ci.yml
│       ├── backend-ci.yml
│       └── ai-model-ci.yml
├── kubernetes/              # K8s (선택)
│   ├── deployment.yaml
│   └── service.yaml
├── scripts/
│   ├── deploy.sh
│   └── backup.sh
└── README.md
```

## 🐳 Docker Compose

### 로컬 개발 환경

```bash
# 전체 서비스 실행
docker-compose up -d

# 특정 서비스만
docker-compose up frontend backend

# 로그 확인
docker-compose logs -f

# 종료
docker-compose down
```

**Services**:
- `frontend` - http://localhost:3000
- `backend` - http://localhost:8000
- `database` - PostgreSQL (port 5432)
- `nginx` - Reverse proxy (port 80)

### 프로덕션 환경

```bash
docker-compose -f docker-compose.prod.yml up -d
```

## 🔄 CI/CD Pipeline

### GitHub Actions Workflows

**Frontend CI**:
- Lint & Type check
- Unit tests
- Build check
- Auto deploy (main 브랜치)

**Backend CI**:
- Lint & Format (Black, Flake8)
- Unit tests (pytest)
- Integration tests
- Docker build

**AI Model CI**:
- Model validation
- Performance benchmark
- Export to ONNX

## 🌐 Deployment

### AWS EC2 배포

```bash
# 서버 접속
ssh -i key.pem ubuntu@<server-ip>

# 코드 업데이트
git pull origin main

# Docker 재시작
docker-compose -f docker-compose.prod.yml up -d --build
```

### 환경 변수

`.env` 파일 생성:
```env
# Backend
DATABASE_URL=postgresql://user:pass@db:5432/fishdb
SECRET_KEY=your-secret-key

# Frontend
REACT_APP_API_URL=http://localhost:8000

# AI Model
MODEL_PATH=/app/models/best.pt
```

## 📊 Monitoring

- **Logs**: `docker-compose logs`
- **Metrics**: Prometheus + Grafana (선택)
- **Health Check**: `http://backend:8000/health`

## 🔒 Security

- SSL/TLS 인증서 (Let's Encrypt)
- 환경 변수로 민감 정보 관리
- `.env` 파일은 **절대 Git에 커밋 금지**

## 🛠️ Useful Commands

```bash
# 전체 재빌드
docker-compose build --no-cache

# DB 백업
docker exec -t postgres pg_dump -U user fishdb > backup.sql

# DB 복원
docker exec -i postgres psql -U user fishdb < backup.sql

# 컨테이너 정리
docker system prune -a
```

## 👥 Team

Infra / DevOps Team

## 📚 References

- [Docker Documentation](https://docs.docker.com/)
- [GitHub Actions](https://docs.github.com/en/actions)
