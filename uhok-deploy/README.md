# UHOK 배포 환경 (Docker Compose)

UHOK 프로젝트의 전체 스택을 Docker Compose로 관리하는 배포 환경입니다. 백엔드, 프론트엔드, ML 추론 서비스, Redis, Nginx를 포함한 마이크로서비스 아키텍처를 제공합니다.

## 📁 폴더 구조

```
uhok-deploy/
├── app/                            # 앱 서비스 (백엔드, 프론트엔드, Redis)
│   ├── .env
│   └── docker-compose.app.yml
├── ml/                             # ML 추론 서비스
│   └── docker-compose.ml.yml
└── public/                         # 공개 서비스 (Nginx, 전체 통합)
│   ├── .env
│   ├── docker-compose.public.yml
│   └── nginx.conf
├── Makefile
└── docker-compose-commands.md
```

### 주요 파일
- `public/docker-compose.public.yml` - 웹 서비스 (Nginx, 프론트엔드)
- `public/nginx.conf` - Nginx 리버스 프록시 설정
- `app/docker-compose.app.yml` - 앱 서비스 (백엔드, Redis)
- `ml/docker-compose.ml.yml` - ML 추론 서비스
- `Makefile` - 자주 사용하는 Docker Compose 명령어 단축키
- `docker-compose-commands.md` - Docker Compose 명령어 상세 가이드

## 🏗️ 아키텍처

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Frontend      │    │   Backend       │    │   ML Inference  │
│   (React)       │    │   (FastAPI)     │    │   (Python)      │
│   Port: 80      │    │   Port: 9000    │    │   Port: 8001    │
└─────────────────┘    └─────────────────┘    └─────────────────┘
         │                      │                       │
         └──────────────────────┼───────────────────────┘
                                │
                       ┌─────────────────┐
                       │     Nginx       │
                       │  (Reverse Proxy)│
                       │   Port: 80      │
                       └─────────────────┘
                                │
                       ┌─────────────────┐
                       │     Redis       │
                       │   Port: 6379    │
                       └─────────────────┘
```

## 🚀 서비스 구성

### 공개 서비스 (public/)
- **nginx** (nginx:1.25-alpine) - 리버스 프록시 및 로드 밸런서
- **frontend** (uhok-frontend:3.0.0) - React 프론트엔드 애플리케이션

### 앱 서비스 (app/)
- **backend** (uhok-backend:2.0.2) - Python FastAPI 백엔드 서비스
- **redis** (redis:7-alpine) - 캐시 및 세션 저장소 (프로필: `with-redis`)

### ML 서비스 (ml/)
- **ml-inference** (uhok-ml-inference:1.2.0) - Python ML 추론 서비스

## 🔧 빠른 시작

### 1. 네트워크 생성 (최초 1회)
```bash
# uhok_net 외부 네트워크 생성
docker network create uhok_net
```

### 2. 전체 서비스 실행
```bash
# uhok-deploy 루트에서 실행
make up-all

# 또는 개별 실행
make up          # 웹 서비스 (nginx + frontend)
make up-app      # 앱 서비스 (backend + redis)
make up-ml       # ML 서비스 (ml-inference)
```

### 3. 개별 서비스 실행
```bash
# 백엔드만 실행 (Redis 포함)
make up-backend

# ML 추론 서비스만 실행
make up-ml

# 프론트엔드만 실행
make up-frontend

# Nginx만 실행
make up-nginx
```

### 4. 접속 확인
- **웹 애플리케이션**: http://localhost
- **API 문서**: http://localhost/docs
- **API 헬스체크**: http://localhost/api/health
- **ML 서비스**: http://localhost:8001 (직접 접근)
- **ML API**: http://localhost/ml/ (Nginx 프록시)

## 🎯 주요 기능

### 백엔드 서비스 (uhok-backend)
- **사용자 관리**: JWT 기반 인증 시스템, 회원가입/로그인
- **홈쇼핑 서비스**: 상품 관리, 편성표 조회, 검색, 찜 기능, 라이브 스트리밍
- **콕 서비스**: 상품 관리, 할인 상품, 장바구니, 검색 기능
- **주문 관리**: 통합 주문 시스템, 결제 처리, 웹훅 방식 결제 확인
- **레시피 추천**: 재료 기반 레시피 추천, 하이브리드 추천, 벡터 유사도 검색
- **데이터베이스 연동**: MariaDB, PostgreSQL 지원
- **ML 서비스 연동**: 별도 ML 추론 서비스와 통신

### 프론트엔드 서비스 (uhok-frontend)
- **반응형 웹 UI**: 모바일/데스크톱 지원
- **인증 시스템**: 자동 토큰 갱신, 401 에러 처리
- **레시피 검색**: 키워드 및 재료 기반 검색
- **쇼핑 기능**: 홈쇼핑, 콕 상품 통합 쇼핑
- **사용자 인터페이스**: 직관적인 사용자 경험

### ML 추론 서비스 (uhok-ml-inference)
- **임베딩 생성**: SentenceTransformers 기반 레시피 임베딩
- **모델**: paraphrase-multilingual-MiniLM-L12-v2
- **성능 최적화**: CPU 전용, 컨테이너화된 배포

## 🔍 서비스 상세 정보

### Backend (uhok-backend)
- **포트**: 9000 (내부)
- **헬스체크**: `/api/health`
- **환경변수**: `../uhok-backend/.env` 파일 사용
- **의존성**: MariaDB, PostgreSQL (외부), Redis
- **이미지**: uhok-backend:2.0.2

### Frontend (uhok-frontend)
- **포트**: 80 (내부)
- **빌드**: React 애플리케이션
- **정적 파일**: Nginx를 통해 서빙
- **이미지**: uhok-frontend:3.0.0

### ML Inference (uhok-ml-inference)
- **포트**: 8001 (외부 노출)
- **헬스체크**: `/health`
- **환경변수**: `../uhok-ml-inference/.env` 파일 사용
- **역할**: 머신러닝 모델 추론 서비스
- **이미지**: uhok-ml-inference:1.2.0
- **볼륨**: `ml_cache` (모델 캐시)

### Redis
- **포트**: 6379 (내부)
- **볼륨**: `redis_data` (데이터 영속성)
- **프로필**: `with-redis`
- **이미지**: redis:7-alpine

### Nginx
- **포트**: 80 (외부 노출)
- **역할**: 리버스 프록시, 로드 밸런서
- **설정**: `nginx.conf`
- **이미지**: nginx:1.25-alpine

## 🌐 라우팅 설정

### API 요청
```
http://localhost/api/* → backend:9000/api/*
http://localhost/api/payment/* → backend:9000/api/payment/*
```

### 문서 및 스키마
```
http://localhost/docs → backend:9000/docs
http://localhost/redoc → backend:9000/redoc
http://localhost/openapi.json → backend:9000/openapi.json
```

### ML 서비스
```
http://localhost/ml/* → ml-inference:8001/*
http://localhost:8001/* → ml-inference:8001/* (직접 접근)
```

### 프론트엔드
```
http://localhost/ → frontend:80/
```

### 헬스체크
```
http://localhost/nginx-health → nginx 헬스체크
```

## 🔧 환경 설정

### 환경 변수 파일
각 서비스는 해당 디렉토리의 `.env` 파일을 사용합니다:
- `../uhok-backend/.env` - 백엔드 설정
- `../uhok-frontend/.env` - 프론트엔드 설정 (필요시)
- `../uhok-ml-inference/.env` - ML 추론 서비스 설정

### 네트워크
- **uhok_net**: 모든 서비스가 통신하는 외부 브리지 네트워크
- **외부 접근**: Nginx를 통해서만 가능 (포트 80)
- **네트워크 생성**: `docker network create uhok_net` (최초 1회)

## 📊 모니터링

### 로그 확인
```bash
# 웹 서비스 로그 (nginx + frontend)
make logs

# 앱 서비스 로그 (backend + redis)
make logs-app

# ML 추론 서비스 로그
make logs-ml

# 모든 서비스 로그
make logs-all

# 특정 서비스 로그
docker compose -f public/docker-compose.public.yml logs -f nginx
docker compose -f public/docker-compose.public.yml logs -f frontend
docker compose -f app/docker-compose.app.yml logs -f backend
docker compose -f app/docker-compose.app.yml logs -f redis
docker compose -f ml/docker-compose.ml.yml logs -f ml-inference
```

### 헬스체크
```bash
# 전체 헬스체크
make health

# 모든 서비스 상태
make status

# 개별 서비스 상태
make status-web      # 웹 서비스 (nginx + frontend)
make status-app      # 앱 서비스 (backend + redis)
make status-ml       # ML 추론 서비스
```

### 리소스 사용량
```bash
# 컨테이너 리소스 사용량
docker stats

# 특정 컨테이너 상세 정보
docker inspect uhok-backend
docker inspect uhok-frontend
docker inspect uhok-ml-inference
```

## 🚨 문제 해결

### 일반적인 문제
1. **포트 충돌**: 80번 포트가 사용 중인 경우
   ```bash
   netstat -tulpn | grep :80
   ```

2. **이미지 빌드 실패**: Dockerfile 경로 확인
   ```bash
   ls -la ../uhok-backend/Dockerfile
   ls -la ../uhok-frontend/Dockerfile
   ```

3. **서비스 연결 실패**: 네트워크 확인
   ```bash
   docker network ls
   docker network inspect uhok_net
   ```

### 로그 확인
```bash
# 에러 로그 필터링
docker compose -f public/docker-compose.public.yml logs | grep -i error
docker compose -f app/docker-compose.app.yml logs | grep -i error
docker compose -f ml/docker-compose.ml.yml logs | grep -i error

# 특정 서비스 에러
docker compose -f public/docker-compose.public.yml logs nginx | grep -i error
docker compose -f public/docker-compose.public.yml logs frontend | grep -i error
docker compose -f app/docker-compose.app.yml logs backend | grep -i error
docker compose -f ml/docker-compose.ml.yml logs ml-inference | grep -i error
```

## 🔄 개발 워크플로우

### 코드 변경 시
```bash
# 백엔드 변경 후
make restart-backend

# 프론트엔드 변경 후
make restart-frontend

# ML 추론 서비스 변경 후
make restart-ml

# Nginx 설정 변경 후
make nginx-reload
```

### 데이터베이스 마이그레이션
```bash
# 마이그레이션 실행
make migrate
```

## 📝 추가 정보

- **Docker Compose 버전**: 2.x 이상 필요
- **최소 메모리**: 4GB 권장 (ML 서비스 포함 시 8GB 권장)
- **디스크 공간**: 15GB 이상 권장 (ML 모델 캐시 포함)
- **지원 OS**: Linux, macOS, Windows (Docker Desktop)
- **네트워크**: 외부 네트워크 `uhok_net` 사용
- **볼륨**: Redis 데이터, ML 모델 캐시 영속성 보장

## 🔗 관련 문서

- [Docker Compose 명령어 가이드](docker-compose-commands.md)
- [Nginx 설정](public/nginx.conf)
- [백엔드 서비스](../uhok-backend/README.md)
- [프론트엔드 서비스](../uhok-frontend/README.md)
- [ML 추론 서비스](../uhok-ml-inference/README.md)
