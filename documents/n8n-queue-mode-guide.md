# n8n Queue Mode 가이드

## 📋 개념

**Queue Mode**는 n8n의 확장 가능한 실행 모드로, 워크플로우 실행을 메시지 큐를 통해 여러 워커 프로세스에 분산시킵니다.

---

## 🏗️ 아키텍처

```
┌─────────────┐
│   Main      │  ← 워크플로우 관리 & UI
│   n8n       │
└──────┬──────┘
       │
       ├──→ Redis (메시지 큐)
       │
       ├──→ PostgreSQL (공유 DB)
       │
┌──────┴──────┐
│  Workers    │  ← 실제 워크플로우 실행
│  (1~N개)    │
└─────────────┘
```

**핵심 구성 요소:**
1. **Main Instance**: 워크플로우 조정 및 UI 제공
2. **Redis**: 작업 큐 및 태스크 분배
3. **PostgreSQL**: 워크플로우 상태 공유
4. **Worker Instances**: 병렬 작업 처리
5. **Webhook Handler** (선택): 웹훅 전용 처리

---

## ⚙️ 핵심 설정

### 환경 변수
```bash
# Queue 모드 설정
EXECUTIONS_MODE=queue

# Redis 설정
QUEUE_BULL_REDIS_HOST=n8n-redis
QUEUE_BULL_REDIS_PORT=6379
QUEUE_BULL_REDIS_DB=0

# PostgreSQL 설정
DB_TYPE=postgresdb
DB_POSTGRESDB_HOST=postgres
DB_POSTGRESDB_PORT=5432
DB_POSTGRESDB_DATABASE=n8n
DB_POSTGRESDB_USER=n8n
DB_POSTGRESDB_PASSWORD=your_password

# 암호화 키 (모든 인스턴스 동일해야 함)
N8N_ENCRYPTION_KEY=shared-encryption-key

# 워커 설정
EXECUTIONS_PROCESS=main        # Main 인스턴스용
EXECUTIONS_PROCESS=worker      # Worker 인스턴스용
```

---

## 🚀 장점

1. **확장성**: 워커 추가로 수평 확장 가능
2. **성능**: 메인 프로세스 병목 현상 제거
3. **안정성**: 작업 분산으로 시스템 안정성 향상
4. **비동기 처리**: 워크플로우 독립 실행
5. **격리**: 워크플로우 실패가 다른 작업에 영향 없음

---

## 💻 최소 시스템 요구사항

### 기본 구성
- **CPU**: 1 vCPU (Main)
- **RAM**: 1-2 GB (Main)
- **권장 워커**: 2-3개 프로세스

### 프로덕션 구성
- **Main**: 2 vCPU, 4 GB RAM
- **Worker (각)**: 2 vCPU, 2 GB RAM
- **Redis**: 1 vCPU, 1 GB RAM
- **PostgreSQL**: 2 vCPU, 4 GB RAM

---

## 📦 Docker Compose 구성 예시

```yaml
version: '3.8'

services:
  # Redis
  redis:
    image: redis:alpine
    restart: unless-stopped
    volumes:
      - redis_data:/data

  # PostgreSQL
  postgres:
    image: postgres:14
    restart: unless-stopped
    environment:
      POSTGRES_USER: n8n
      POSTGRES_PASSWORD: your_password
      POSTGRES_DB: n8n
    volumes:
      - postgres_data:/var/lib/postgresql/data

  # n8n Main Instance
  n8n-main:
    image: n8nio/n8n
    restart: unless-stopped
    ports:
      - "5678:5678"
    environment:
      - EXECUTIONS_MODE=queue
      - EXECUTIONS_PROCESS=main
      - QUEUE_BULL_REDIS_HOST=redis
      - QUEUE_BULL_REDIS_PORT=6379
      - DB_TYPE=postgresdb
      - DB_POSTGRESDB_HOST=postgres
      - DB_POSTGRESDB_PORT=5432
      - DB_POSTGRESDB_DATABASE=n8n
      - DB_POSTGRESDB_USER=n8n
      - DB_POSTGRESDB_PASSWORD=your_password
      - N8N_ENCRYPTION_KEY=shared-encryption-key
    volumes:
      - n8n_data:/home/node/.n8n
    depends_on:
      - redis
      - postgres

  # n8n Worker 1
  n8n-worker-1:
    image: n8nio/n8n
    restart: unless-stopped
    command: worker
    environment:
      - EXECUTIONS_MODE=queue
      - EXECUTIONS_PROCESS=worker
      - QUEUE_BULL_REDIS_HOST=redis
      - QUEUE_BULL_REDIS_PORT=6379
      - DB_TYPE=postgresdb
      - DB_POSTGRESDB_HOST=postgres
      - DB_POSTGRESDB_PORT=5432
      - DB_POSTGRESDB_DATABASE=n8n
      - DB_POSTGRESDB_USER=n8n
      - DB_POSTGRESDB_PASSWORD=your_password
      - N8N_ENCRYPTION_KEY=shared-encryption-key
    volumes:
      - n8n_data:/home/node/.n8n
    depends_on:
      - redis
      - postgres

  # n8n Worker 2
  n8n-worker-2:
    image: n8nio/n8n
    restart: unless-stopped
    command: worker
    environment:
      - EXECUTIONS_MODE=queue
      - EXECUTIONS_PROCESS=worker
      - QUEUE_BULL_REDIS_HOST=redis
      - QUEUE_BULL_REDIS_PORT=6379
      - DB_TYPE=postgresdb
      - DB_POSTGRESDB_HOST=postgres
      - DB_POSTGRESDB_PORT=5432
      - DB_POSTGRESDB_DATABASE=n8n
      - DB_POSTGRESDB_USER=n8n
      - DB_POSTGRESDB_PASSWORD=your_password
      - N8N_ENCRYPTION_KEY=shared-encryption-key
    volumes:
      - n8n_data:/home/node/.n8n
    depends_on:
      - redis
      - postgres

volumes:
  redis_data:
  postgres_data:
  n8n_data:
```

---

## 🔧 설정 단계

### 1. Docker 네트워크 생성 (선택사항)
```bash
docker network create n8n-network
```

### 2. 환경 변수 파일 작성
**queue.env**:
```bash
EXECUTIONS_MODE=queue
QUEUE_BULL_REDIS_HOST=redis
DB_TYPE=postgresdb
DB_POSTGRESDB_HOST=postgres
DB_POSTGRESDB_DATABASE=n8n
DB_POSTGRESDB_USER=n8n
DB_POSTGRESDB_PASSWORD=your_secure_password
N8N_ENCRYPTION_KEY=your-unique-encryption-key
```

### 3. Redis 실행
```bash
docker run -d \
  --name n8n-redis \
  --network n8n-network \
  redis:alpine
```

### 4. PostgreSQL 실행
```bash
docker run -d \
  --name n8n-postgres \
  --network n8n-network \
  -e POSTGRES_USER=n8n \
  -e POSTGRES_PASSWORD=your_password \
  -e POSTGRES_DB=n8n \
  postgres:14
```

### 5. Main 인스턴스 실행
```bash
docker run -d \
  --name n8n-main \
  --network n8n-network \
  -p 5678:5678 \
  --env-file queue.env \
  -e EXECUTIONS_PROCESS=main \
  -v n8n_data:/home/node/.n8n \
  n8nio/n8n
```

### 6. Worker 인스턴스 실행
```bash
# Worker 1
docker run -d \
  --name n8n-worker-1 \
  --network n8n-network \
  --env-file queue.env \
  -e EXECUTIONS_PROCESS=worker \
  -v n8n_data:/home/node/.n8n \
  n8nio/n8n worker

# Worker 2
docker run -d \
  --name n8n-worker-2 \
  --network n8n-network \
  --env-file queue.env \
  -e EXECUTIONS_PROCESS=worker \
  -v n8n_data:/home/node/.n8n \
  n8nio/n8n worker
```

---

## 📊 확장 전략

### 수평 확장
```bash
# 워커 3 추가
docker run -d \
  --name n8n-worker-3 \
  --network n8n-network \
  --env-file queue.env \
  -e EXECUTIONS_PROCESS=worker \
  -v n8n_data:/home/node/.n8n \
  n8nio/n8n worker
```

### 웹훅 전용 핸들러 (선택)
```bash
docker run -d \
  --name n8n-webhook \
  --network n8n-network \
  -p 5679:5678 \
  --env-file queue.env \
  -e EXECUTIONS_PROCESS=webhook \
  n8nio/n8n webhook
```

### 리소스 모니터링
```bash
# 컨테이너 리소스 사용량
docker stats n8n-main n8n-worker-1 n8n-worker-2

# Redis 모니터링
docker exec n8n-redis redis-cli INFO stats

# PostgreSQL 연결 수
docker exec n8n-postgres psql -U n8n -c "SELECT count(*) FROM pg_stat_activity;"
```

---

## ✅ 검증 방법

### 1. 컨테이너 상태 확인
```bash
docker ps --filter name=n8n
```

### 2. 로그 확인
```bash
# Main 로그
docker logs -f n8n-main

# Worker 로그
docker logs -f n8n-worker-1
docker logs -f n8n-worker-2

# Redis 로그
docker logs -f n8n-redis
```

### 3. 연결 테스트
```bash
# Redis 연결 확인
docker exec n8n-redis redis-cli ping

# PostgreSQL 연결 확인
docker exec n8n-postgres pg_isready -U n8n

# n8n 헬스 체크
curl http://localhost:5678/healthz
```

### 4. 워크플로우 테스트
1. n8n UI 접속 (http://localhost:5678)
2. 간단한 워크플로우 생성
3. 실행 모니터링
4. Worker 로그에서 실행 확인

---

## 🎯 베스트 프랙티스

### 1. 보안
- ✅ 강력하고 고유한 비밀번호 사용
- ✅ 모든 인스턴스에 동일한 `N8N_ENCRYPTION_KEY` 설정
- ✅ Redis 인증 활성화
- ✅ PostgreSQL SSL 연결 사용
- ✅ 네트워크 격리 구현

### 2. 성능
- ✅ 워커 수를 CPU 코어 수에 맞춤
- ✅ Redis 메모리 최적화
- ✅ PostgreSQL 연결 풀 설정
- ✅ 워크플로우 타임아웃 설정

### 3. 모니터링
- ✅ 로그 수집 및 분석 (ELK, Grafana)
- ✅ 메트릭 모니터링 (Prometheus)
- ✅ 알림 설정 (과부하, 에러)
- ✅ 정기적인 백업

### 4. 유지보수
- ✅ 정기적인 PostgreSQL 백업
- ✅ Redis 데이터 영속성 설정
- ✅ 컨테이너 업데이트 계획
- ✅ 디스크 공간 모니터링

---

## 🔍 문제 해결

### 워커가 작업을 가져오지 않음
```bash
# Redis 연결 확인
docker exec n8n-worker-1 env | grep REDIS

# 큐 상태 확인
docker exec n8n-redis redis-cli LLEN bull:n8n:jobs:wait
```

### 암호화 키 불일치
```bash
# 모든 컨테이너의 암호화 키 확인
docker exec n8n-main env | grep N8N_ENCRYPTION_KEY
docker exec n8n-worker-1 env | grep N8N_ENCRYPTION_KEY
```

### PostgreSQL 연결 실패
```bash
# 연결 문자열 확인
docker exec n8n-main env | grep DB_POSTGRESDB

# PostgreSQL 로그 확인
docker logs n8n-postgres | tail -50
```

---

## 📚 참고 자료

- [n8n Queue Mode 공식 문서](https://docs.n8n.io/hosting/scaling/queue-mode/)
- [n8n 환경 변수](https://docs.n8n.io/hosting/configuration/environment-variables/)
- [커뮤니티 가이드](https://community.n8n.io/t/queue-mode-guide-how-to-scale-up-n8n/75467)

---

## 🔑 핵심 요약

**Queue Mode는 n8n을 프로덕션 환경에서 확장 가능하게 만드는 핵심 기능**입니다:

- ✅ Redis 기반 메시지 큐
- ✅ 워커 프로세스 병렬 실행
- ✅ PostgreSQL 공유 상태 관리
- ✅ 수평 확장 가능
- ✅ 메인 프로세스 병목 해소
- ✅ 고가용성 및 안정성

---

## 📅 문서 정보
- **n8n 버전**: 1.113.3
- **작성일**: 2025-10-16
- **최종 업데이트**: 2025-10-16
