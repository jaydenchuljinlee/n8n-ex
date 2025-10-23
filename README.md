# 싸다온라인 백엔드 시스템 및 고객 대응 플로우

## 📋 프로젝트 개요

이 프로젝트는 싸다온라인의 백엔드 시스템과 고객 컴플레인 대응 체계를 구축하는 것을 목표로 합니다.

### 주요 구성 요소

- **[Frontend](./frontend/README.md)** - Next.js 15 + TypeScript + Tailwind CSS
- **[Backend API Server](./backend_sarda_online/README.md)** - NestJS + Prisma + PostgreSQL
- **[Database](./database/README.md)** - PostgreSQL 16 스키마 및 설계 문서
- **[MCP Server](./mcp_server_practice/README.md)** - AI 에이전트 통합 레이어
- **n8n Workflow Automation** - MCP 서버 연동
- **고객 컴플레인 대응 시스템**
- **JIRA 연동 티켓 관리**

> 💡 **서비스별 상세 문서**: 각 서비스의 상세한 개발 가이드, API 문서, 트러블슈팅은 위 링크를 클릭하세요.

## 🏗️ 시스템 아키텍처

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                          🌐 External Layer                                   │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                               │
│   ┌──────────────┐    ┌──────────────┐    ┌──────────────┐                 │
│   │   🔒 ngrok   │    │  📋 JIRA     │    │  🤖 Claude   │                 │
│   │   (HTTPS)    │    │   System     │    │     AI       │                 │
│   │  Port: 4040  │    │  Port: 8081  │    │              │                 │
│   └──────┬───────┘    └──────┬───────┘    └──────┬───────┘                 │
│          │                   │                    │                          │
└──────────┼───────────────────┼────────────────────┼──────────────────────────┘
           │                   │                    │
           │                   │                    │
┌──────────▼───────────────────▼────────────────────▼──────────────────────────┐
│                          🌍 User Access Layer                                 │
├───────────────────────────────────────────────────────────────────────────────┤
│                                                                               │
│   ┌─────────────────────────────────────────────────────────────────┐       │
│   │  💻 Frontend (Next.js 15)                                        │       │
│   │  http://localhost:3002                                           │       │
│   │  ├─ React Hook Form + Zod Validation                            │       │
│   │  ├─ CS 민원 접수 폼                                               │       │
│   │  └─ 자동 ID 생성 (상품/주문)                                      │       │
│   └───────────────────────┬─────────────────────────────────────────┘       │
│                           │ HTTP/JSON (CORS)                                 │
└───────────────────────────┼──────────────────────────────────────────────────┘
                            │
                            │
┌───────────────────────────▼──────────────────────────────────────────────────┐
│                    🐳 Docker Network (Backend Services)                      │
├───────────────────────────────────────────────────────────────────────────────┤
│                                                                               │
│   ┌─────────────────────────────────────────────────────────────────┐       │
│   │  🚀 Backend API (NestJS)                                         │       │
│   │  Port: 3003 (External) / 3000 (Internal)                         │       │
│   │  ├─ REST API Endpoints                                           │       │
│   │  ├─ Prisma ORM                                                   │       │
│   │  ├─ CORS Configuration                                           │       │
│   │  └─ Swagger Documentation                                        │       │
│   └───────────────┬─────────────────┬───────────────────────────────┘       │
│                   │                 │                                        │
│                   │                 │                                        │
│   ┌───────────────▼─────────────────▼─────────────────────────────┐         │
│   │  🗄️  PostgreSQL Database                                        │         │
│   │  Port: 5432                                                     │         │
│   │  ├─ 17 Tables (complaints, users, logs, etc.)                  │         │
│   │  ├─ Prisma Schema & Migrations                                 │         │
│   │  ├─ Seed Data (50+ dummy records)                              │         │
│   │  └─ pgvector Extension (AI Embeddings)                         │         │
│   └──────┬──────────────────────────┬───────────────────────────────┘        │
│          │                          │                                        │
│          │                          │                                        │
│   ┌──────▼──────────┐    ┌──────────▼──────────┐    ┌────────────────┐     │
│   │  🔧 pgAdmin     │    │  🔌 Postgres MCP    │    │  🎯 MCP Server │     │
│   │  Port: 5050     │    │  Port: 8003         │    │  Port: 8001    │     │
│   │  └─ DB 관리 UI  │    │  └─ SQL via MCP     │    │  └─ AI Tools   │     │
│   └─────────────────┘    └─────────────────────┘    └────────┬───────┘     │
│                                                                │              │
│                                                                │              │
│   ┌────────────────────────────────────────────────────────────▼───────┐    │
│   │  ⚙️  n8n Workflow Automation                                       │    │
│   │  Port: 5680                                                        │    │
│   │  ├─ MCP Server Integration                                        │    │
│   │  ├─ Webhook Processing                                            │    │
│   │  ├─ Complaint Auto-Classification                                │    │
│   │  └─ JIRA Ticket Creation                                          │    │
│   └────────────────────────────────────────────────────────────────────┘    │
│                                                                               │
└───────────────────────────────────────────────────────────────────────────────┘

┌───────────────────────────────────────────────────────────────────────────────┐
│                          📊 Data Flow Diagram                                 │
├───────────────────────────────────────────────────────────────────────────────┤
│                                                                               │
│  1️⃣  Customer Request Flow:                                                  │
│      Frontend → Backend API → PostgreSQL → Response                          │
│                                                                               │
│  2️⃣  AI Agent Flow (via MCP):                                                │
│      Claude/n8n → MCP Server → Backend API → PostgreSQL                      │
│                                                                               │
│  3️⃣  Automation Flow:                                                        │
│      Webhook → ngrok → n8n → MCP Server → Backend API → JIRA                 │
│                                                                               │
│  4️⃣  Direct SQL Flow (for AI):                                               │
│      Claude → Postgres MCP → PostgreSQL (Direct SQL)                         │
│                                                                               │
└───────────────────────────────────────────────────────────────────────────────┘


┌───────────────────────────────────────────────────────────────────────────────┐
│                    🔄 CS 민원 처리 플로우 차트                                │
├───────────────────────────────────────────────────────────────────────────────┤
│                                                                               │
│                        👤 고객 민원 접수                                      │
│                               │                                               │
│                               ▼                                               │
│              ┌────────────────────────────────────┐                          │
│              │  📝 Frontend (Next.js)              │                          │
│              │  - CS 민원 접수 폼                  │                          │
│              │  - 자동 ID 생성                     │                          │
│              │  - Zod 검증                         │                          │
│              └────────────────┬───────────────────┘                          │
│                               │                                               │
│                               │ HTTP POST                                     │
│                               ▼                                               │
│              ┌────────────────────────────────────┐                          │
│              │  🚀 Backend API                     │                          │
│              │  POST /complaints                   │                          │
│              │  - 우선순위 자동 판단               │                          │
│              │  - 긴급도 자동 판단                 │                          │
│              │  - 티켓 번호 생성                   │                          │
│              └────────────────┬───────────────────┘                          │
│                               │                                               │
│                               ▼                                               │
│              ┌────────────────────────────────────┐                          │
│              │  🗄️  PostgreSQL                     │                          │
│              │  INSERT into complaints             │                          │
│              │  - status: "접수"                   │                          │
│              │  - 티켓 데이터 저장                 │                          │
│              └────────────────┬───────────────────┘                          │
│                               │                                               │
│          ┌───────────────────┼───────────────────┐                          │
│          │                   │                   │                           │
│          ▼                   ▼                   ▼                           │
│   ┌──────────┐        ┌──────────┐       ┌──────────┐                      │
│   │ n8n 자동 │        │ AI 에이전트│      │  Frontend │                      │
│   │ 워크플로우│        │  (Claude) │      │   응답     │                      │
│   └─────┬────┘        └─────┬────┘       └──────────┘                      │
│         │                   │                                                │
│         ▼                   ▼                                                │
│   ┌──────────────────┬──────────────────┐                                  │
│   │  🤖 MCP Server    │  🔌 Postgres MCP │                                  │
│   │  (Backend API)    │  (Direct SQL)    │                                  │
│   └─────┬─────────────┴──────┬───────────┘                                  │
│         │                    │                                               │
│         ▼                    ▼                                               │
│   ┌─────────────────────────────────┐                                       │
│   │     민원 분류 & 배정             │                                       │
│   │  ┌───────────────────────────┐  │                                       │
│   │  │  카테고리별 자동 분류:     │  │                                       │
│   │  │  - 가격정보    → CS 1팀   │  │                                       │
│   │  │  - 상품정보    → CS 2팀   │  │                                       │
│   │  │  - 배송구매    → 배송팀   │  │                                       │
│   │  │  - 시스템기술  → 기술팀   │  │                                       │
│   │  └───────────────────────────┘  │                                       │
│   └─────────────┬───────────────────┘                                       │
│                 │                                                            │
│                 ▼                                                            │
│   ┌─────────────────────────────────┐                                       │
│   │  🎯 담당자 배정 (MCP Tool)       │                                       │
│   │  PUT /complaints/:id             │                                       │
│   │  - assignedTo: agent_id          │                                       │
│   │  - assignedTeam: "CS 1팀"        │                                       │
│   │  - status: "처리중"              │                                       │
│   └─────────────┬───────────────────┘                                       │
│                 │                                                            │
│                 ▼                                                            │
│   ┌─────────────────────────────────┐                                       │
│   │  ❓ 지침서 존재 여부 확인        │                                       │
│   └──┬───────────────────────────┬──┘                                       │
│      │ YES                       │ NO                                        │
│      ▼                           ▼                                           │
│   ┌────────────────┐      ┌────────────────────┐                           │
│   │  📖 지침서     │      │  📋 JIRA 티켓 생성 │                           │
│   │  따라 응대     │      │  (MCP Tool)         │                           │
│   │                │      │  - update-complaint-│                           │
│   │  - 템플릿 사용 │      │    jira-ticket      │                           │
│   │  - 빠른 해결   │      │  - 개발팀 검토 요청│                           │
│   └───────┬────────┘      └─────────┬──────────┘                           │
│           │                         │                                        │
│           │                         ▼                                        │
│           │              ┌─────────────────────┐                            │
│           │              │  🔧 개발팀 검토     │                            │
│           │              │  - 신규 지침 작성  │                            │
│           │              │  - KB 업데이트      │                            │
│           │              └─────────┬───────────┘                            │
│           │                        │                                         │
│           └────────────────────────┘                                         │
│                        │                                                     │
│                        ▼                                                     │
│           ┌─────────────────────────────┐                                   │
│           │  💬 고객 응대                │                                   │
│           │  - 응답 작성                 │                                   │
│           │  - 상태 업데이트             │                                   │
│           │  - 만족도 조사               │                                   │
│           └─────────────┬───────────────┘                                   │
│                         │                                                    │
│                         ▼                                                    │
│           ┌─────────────────────────────┐                                   │
│           │  ✅ 민원 완료                │                                   │
│           │  - status: "해결"            │                                   │
│           │  - resolvedAt 기록           │                                   │
│           │  - 통계 업데이트             │                                   │
│           └──────────────────────────────┘                                   │
│                                                                               │
└───────────────────────────────────────────────────────────────────────────────┘


┌───────────────────────────────────────────────────────────────────────────────┐
│                    🚨 에스컬레이션 플로우 차트                                │
├───────────────────────────────────────────────────────────────────────────────┤
│                                                                               │
│                        📝 민원 접수                                           │
│                            │                                                  │
│                            ▼                                                  │
│              ┌──────────────────────────────┐                                │
│              │  🎯 우선순위 & 긴급도 판단   │                                │
│              │  (Backend 자동 판단)          │                                │
│              └────────┬─────────────────────┘                                │
│                       │                                                       │
│          ┌────────────┼────────────┐                                         │
│          │            │            │                                          │
│          ▼            ▼            ▼                                          │
│     ┌─────────┐  ┌─────────┐  ┌─────────┐                                  │
│     │  Low    │  │ Medium  │  │  High   │                                  │
│     │ Priority│  │ Priority│  │ Priority│                                  │
│     └────┬────┘  └────┬────┘  └────┬────┘                                  │
│          │            │            │                                          │
│          ▼            ▼            ▼                                          │
│   ┌──────────┐  ┌──────────┐  ┌──────────┐                                 │
│   │ Level 1  │  │ Level 1  │  │ Level 2  │                                 │
│   │ 일반상담원│  │ 일반상담원│  │ 선임상담원│                                 │
│   └────┬─────┘  └────┬─────┘  └────┬─────┘                                 │
│        │             │             │                                          │
│        │             │             │                                          │
│        ▼             ▼             ▼                                          │
│   ┌─────────────────────────────────────┐                                   │
│   │  ❓ 48시간 내 해결 가능?             │                                   │
│   └──┬───────────────────────┬──────────┘                                   │
│      │ YES                   │ NO                                            │
│      ▼                       ▼                                                │
│   ┌─────────┐     ┌──────────────────────┐                                 │
│   │  해결   │     │  🔺 Level 2 에스컬   │                                 │
│   │  완료   │     │  - escalationLevel: 2 │                                 │
│   └─────────┘     │  - isEscalated: true  │                                 │
│                   └──────────┬────────────┘                                 │
│                              │                                               │
│                              ▼                                               │
│                  ┌──────────────────────────┐                               │
│                  │  👔 팀장/매니저 검토     │                               │
│                  │  - 복잡한 케이스         │                               │
│                  │  - 법적 검토 필요        │                               │
│                  └──────────┬───────────────┘                               │
│                             │                                                │
│                   ┌─────────┼─────────┐                                     │
│                   │ 해결 가능│ 불가능  │                                     │
│                   ▼         ▼         ▼                                      │
│              ┌─────────┐ ┌────────────────┐                                │
│              │  해결   │ │ 🔺 Level 3     │                                │
│              │  완료   │ │ - 법무팀 협의  │                                │
│              └─────────┘ │ - 임원 보고     │                                │
│                          └────────┬────────┘                                │
│                                   │                                          │
│                                   ▼                                          │
│                        ┌────────────────────┐                               │
│                        │  👨‍💼 임원 레벨 결정  │                               │
│                        │  Level 4           │                               │
│                        │  - 중대 사안       │                               │
│                        │  - 최종 의사결정   │                               │
│                        └────────────────────┘                               │
│                                                                               │
└───────────────────────────────────────────────────────────────────────────────┘
```

### 구성 요소 설명

#### 🔵 Core Services
- **Frontend (Next.js 15)**: 고객 민원 접수 웹 애플리케이션 (Port: 3002)
  - Next.js 15 with App Router
  - TypeScript + Tailwind CSS
  - React Hook Form + Zod validation
  - CS 문의 입력 폼
  - 자동 ID 생성 (상품/주문)

- **Backend API (NestJS)**: 메인 백엔드 서버 (Port: 3003)
  - REST API 제공
  - CORS 설정 완료
  - Prisma ORM으로 DB 연동
  - 고객 컴플레인, 사용자 관리 등

- **PostgreSQL Database**: 데이터 저장소 (Port: 5432)
  - 17개 테이블 (customer_users, complaints, internal_users 등)
  - 더미 데이터 자동 생성
  - pgvector 확장 (AI embedding)

#### 🟢 Automation & AI
- **n8n Workflow**: 워크플로우 자동화 (Port: 5680)
  - 컴플레인 자동 분류 및 처리
  - MCP Server 연동
  - Webhook 기반 이벤트 처리

- **MCP Server (Custom)**: AI 에이전트 통합 레이어 (Port: 8001)
  - Tools: AI가 호출 가능한 함수들
  - Resources: 데이터 소스 제공
  - Prompts: 프롬프트 템플릿

- **Postgres MCP Server**: PostgreSQL 전용 MCP (Port: 8003)
  - SQL 쿼리 실행
  - 데이터베이스 스키마 탐색
  - AI 기반 SQL 생성

#### 🟡 External Integrations
- **ngrok**: HTTPS 터널링 (Port: 4040)
  - 로컬 n8n을 외부에 노출
  - Webhook URL 제공

- **JIRA**: 티켓 관리 시스템 (Port: 8081)
  - 컴플레인 티켓 자동 생성
  - 이슈 추적

- **Claude AI**: AI 어시스턴트
  - MCP Server 통해 백엔드 접근
  - 자동 응답 생성

#### 🟠 Management Tools
- **pgAdmin**: PostgreSQL 관리 UI (Port: 5050)
  - 데이터베이스 시각화
  - 쿼리 실행 및 관리

### 데이터 흐름

1. **고객 요청** → Frontend → Backend API → PostgreSQL
2. **자동화 워크플로우** → n8n → MCP Server → Backend API
3. **AI 어시스턴트** → Claude → MCP Server → PostgreSQL (via Postgres MCP)
4. **외부 Webhook** → ngrok → n8n → 워크플로우 처리

## 🚀 시작하기

### 1. 환경 설정

```bash
# 프로젝트 클론
git clone [repository-url]
cd backend-setup

# 환경 변수 설정
cp .env.example .env
cp backend_sarda_online/.env.example backend_sarda_online/.env
```

#### 🔧 Backend DATABASE_URL 설정

Backend 서비스는 PostgreSQL 데이터베이스에 연결하기 위해 `DATABASE_URL` 환경 변수가 필요합니다.

**1. `.env` 파일 수정**

루트 디렉토리의 `.env` 파일에서 다음 변수들을 설정합니다:

```bash
# PostgreSQL 설정
POSTGRES_USER=sarda_online_user
POSTGRES_PASSWORD=sarda_online_password
POSTGRES_DB=sarda_online_db
POSTGRES_PORT=5432
```

**2. `backend_sarda_online/.env` 파일 수정**

Backend 디렉토리의 `.env` 파일에 `DATABASE_URL`을 추가합니다:

```bash
# Database Connection
DATABASE_URL="postgresql://${POSTGRES_USER}:${POSTGRES_PASSWORD}@postgres:5432/${POSTGRES_DB}"

# 또는 직접 값을 입력
DATABASE_URL="postgresql://sarda_online_user:sarda_online_password@postgres:5432/sarda_online_db"
```

**연결 문자열 형식:**
```
postgresql://[사용자명]:[비밀번호]@[호스트]:[포트]/[데이터베이스명]
```

**주의사항:**
- Docker 네트워크 내에서는 호스트명을 `postgres`로 사용
- 로컬 개발 시 (Docker 외부): `localhost` 또는 `127.0.0.1` 사용
- 비밀번호에 특수문자가 있으면 URL 인코딩 필요

**3. 연결 확인**

```bash
# Backend 컨테이너 로그 확인
docker logs sarda_online_backend

# 데이터베이스 연결 성공 시 메시지
# "Database connection established successfully"
```

### 2. Docker 컨테이너 실행

```bash
# 모든 서비스 시작
docker-compose up -d

# 개별 서비스 시작
docker-compose up -d postgres    # PostgreSQL
docker-compose up -d pgadmin     # pgAdmin
docker-compose up -d n8n         # n8n
docker-compose up -d backend     # Backend API
```

### 3. 서비스 접속 정보

| 서비스           | URL                      | 인증 정보                     |
| ---------------- | ------------------------ | ----------------------------- |
| **Frontend**     | http://localhost:3002    | -                             |
| Backend API      | http://localhost:3003    | -                             |
| Swagger API Docs | http://localhost:3003/api | -                            |
| n8n              | http://localhost:5680    | admin / admin                 |
| pgAdmin          | http://localhost:5050    | admin@sarda-online.com / admin      |
| PostgreSQL       | localhost:5432           | sarda_online_user / sarda_online_password |
| MCP Server       | http://localhost:8001    | -                             |
| Postgres MCP     | http://localhost:8003    | -                             |
| ngrok Dashboard  | http://localhost:4040    | -                             |
| Jira             | http://localhost:8081    | (초기 설정 필요)              |

### 4. ngrok HTTPS 터널 설정

ngrok을 사용하여 로컬 n8n을 외부에서 HTTPS로 접근할 수 있습니다.

#### ngrok 설정 방법

1. **환경 변수 설정**

   ```bash
   # .env 파일에서 ngrok authtoken 설정
   NGROK_AUTHTOKEN=your_ngrok_authtoken_here
   ```

   authtoken은 [ngrok 대시보드](https://dashboard.ngrok.com/get-started/your-authtoken)에서 확인할 수 있습니다.

2. **Docker Compose 실행**

   ```bash
   docker-compose up -d ngrok
   ```

3. **HTTPS URL 확인**

   ngrok이 생성한 HTTPS URL을 확인하는 방법:

   **방법 1: 웹 인터페이스 (추천)**

   - 브라우저에서 http://localhost:4040 접속
   - "Tunnels" 섹션에서 HTTPS URL 확인 (예: https://abc123.ngrok.io)

   **방법 2: API 호출**

   ```bash
   curl http://localhost:4040/api/tunnels | jq '.tunnels[0].public_url'
   ```

   **방법 3: Docker 로그**

   ```bash
   docker logs sarda_online_ngrok
   ```

4. **n8n Webhook URL 업데이트**

   **방법 1: 자동 업데이트 스크립트 사용 (추천) ✨**

   ```bash
   # 실행 권한 부여 (최초 1회만)
   chmod +x update-n8n-url.sh

   # 스크립트 실행 - ngrok URL 자동 감지 및 .env 업데이트 + n8n 재시작
   ./update-n8n-url.sh
   ```

   스크립트가 자동으로:
   - ✅ ngrok API에서 현재 터널 URL 가져오기
   - ✅ `.env` 파일의 `N8N_WEBHOOK_URL`과 `N8N_HOST` 업데이트
   - ✅ n8n 컨테이너 자동 재시작

   **방법 2: 수동 업데이트**

   ngrok URL을 확인한 후 `.env` 파일 수정:

   ```bash
   # .env
   N8N_WEBHOOK_URL=https://abc123.ngrok.io
   N8N_HOST=abc123.ngrok.io
   ```

   그리고 n8n 컨테이너 재시작:

   ```bash
   docker-compose restart n8n
   ```

#### ngrok 설정 파일

`ngrok.yml` 파일 구조:

```yaml
version: "2"
tunnels:
  n8n:
    addr: n8n:56780 # n8n  docker 내부
    proto: http
    schemes:
      - https # HTTPS만 활성화
```

#### 주의사항

- **무료 플랜 제한**: ngrok 무료 플랜은 동시에 1개의 터널만 사용 가능합니다.
- **URL 변경**: ngrok을 재시작할 때마다 URL이 변경됩니다. 고정 URL이 필요한 경우 유료 플랜을 사용하세요.
- **보안**: authtoken은 `.env` 파일에 저장하고 절대 Git에 커밋하지 마세요.

#### 트러블슈팅

1. **ERR_NGROK_105 (인증 실패)**

   - `.env` 파일의 `NGROK_AUTHTOKEN` 값 확인
   - [ngrok 대시보드](https://dashboard.ngrok.com/get-started/your-authtoken)에서 올바른 토큰 확인

2. **터널이 생성되지 않음**

   - n8n 컨테이너가 실행 중인지 확인: `docker ps | grep n8n`
   - ngrok 로그 확인: `docker logs sarda_online_ngrok`

3. **Webhook이 작동하지 않음**
   - n8n의 `N8N_WEBHOOK_URL` 환경변수가 ngrok URL로 설정되었는지 확인
   - n8n 워크플로우에서 Webhook URL이 올바른지 확인

## 📦 서비스별 상세 문서

### 🌐 Frontend (Next.js 15)

고객이 CS 민원을 접수할 수 있는 웹 인터페이스입니다.

- **기술 스택**: Next.js 15, TypeScript, Tailwind CSS, React Hook Form, Zod
- **주요 기능**: CS 민원 접수 폼, 자동 ID 생성, 폼 검증, 접수 완료 정보 표시
- **포트**: 3002 (외부) → 3000 (내부)

**[📖 Frontend 상세 문서 보기](./frontend/README.md)**

### 🚀 Backend API (NestJS)

싸다온라인 CS 민원 관리 시스템의 백엔드 API 서버입니다.

- **기술 스택**: NestJS, Prisma, PostgreSQL 16, TypeScript
- **주요 API**: Customer Users, Internal Users, Complaints, User Logs
- **포트**: 3003 (외부) → 3000 (내부)
- **API 문서**: http://localhost:3003/api (Swagger)

**[📖 Backend 상세 문서 보기](./backend_sarda_online/README.md)**

### 🗄️ Database (PostgreSQL 16)

데이터베이스 스키마 설계 및 관리 문서입니다.

- **DBMS**: PostgreSQL 16
- **ORM**: Prisma
- **테이블**: 9개 (고객, 직원, 민원, 응답, 이력, 템플릿, SLA, KB 등)
- **주요 기능**: 마이그레이션, 시드 데이터, 백업/복원

**[📖 Database 상세 문서 보기](./database/README.md)**

### 🤖 MCP Server

AI 에이전트(Claude, n8n 등)가 Backend API와 상호작용할 수 있도록 하는 통합 레이어입니다.

- **기술 스택**: TypeScript, MCP Protocol, HTTP/SSE
- **주요 Tools**: 고객 관리, 직원 관리, 민원 관리 (Read/Write), 사용자 로그
- **포트**: 8001 (외부) → 3000 (내부)

**[📖 MCP Server 상세 문서 보기](./mcp_server_practice/README.md)**

---

### 빠른 시작 가이드

각 서비스를 개별적으로 실행하려면:

```bash
# Frontend 실행
docker compose up -d frontend

# Backend 실행
docker compose up -d backend

# MCP Server 실행
docker compose up -d mcp-server
```

더 자세한 개발 가이드는 각 서비스의 README를 참고하세요.


## 🗂️ 프로젝트 구조

```
n8n_with_mcp_server_example/
├── 📄 docker-compose.yml              # Docker Compose 설정
├── 📄 ngrok.yml                        # ngrok 터널 설정
├── 📄 .env                             # 환경 변수 (Git 제외)
├── 📄 .env.example                     # 환경 변수 템플릿
├── 📄 .gitignore                       # Git 제외 파일 목록
├── 📄 README.md                        # 프로젝트 문서
│
├── 📁 frontend/                        # Next.js Frontend 애플리케이션
│   ├── 📁 app/
│   │   ├── 📄 page.tsx                # 메인 페이지 (CS 폼)
│   │   ├── 📄 layout.tsx              # 루트 레이아웃
│   │   └── 📄 globals.css             # 전역 스타일
│   ├── 📁 components/
│   │   └── 📄 ComplaintForm.tsx       # CS 민원 접수 폼
│   ├── 📁 lib/
│   │   └── 📄 api.ts                  # API 클라이언트
│   ├── 📁 types/
│   │   └── 📄 complaint.ts            # 타입 정의
│   ├── 📄 Dockerfile                   # Frontend 도커 이미지
│   ├── 📄 .dockerignore                # Docker 빌드 제외
│   ├── 📄 .env.local                   # 로컬 환경변수
│   ├── 📄 next.config.ts               # Next.js 설정
│   ├── 📄 tailwind.config.ts           # Tailwind 설정
│   ├── 📄 package.json                 # 의존성 관리
│   └── 📄 tsconfig.json                # TypeScript 설정
│
├── 📁 backend_sarda_online/                  # NestJS Backend 애플리케이션
│   ├── 📁 src/
│   │   ├── 📁 users/                  # 고객 사용자 관리 모듈 (customer-users API)
│   │   ├── 📁 user-logs/              # 고객 사용자 로그 모듈 (customer-user-logs API)
│   │   ├── 📁 prisma/                 # Prisma 서비스
│   │   ├── 📄 main.ts                 # 애플리케이션 엔트리
│   │   ├── 📄 app.module.ts           # 루트 모듈
│   │   └── 📄 app.controller.ts       # 루트 컨트롤러
│   ├── 📁 prisma/
│   │   ├── 📁 migrations/             # Prisma 마이그레이션
│   │   ├── 📄 schema.prisma           # 데이터베이스 스키마
│   │   └── 📄 seed.ts                 # 시드 데이터
│   ├── 📁 test/                       # E2E 테스트
│   ├── 📄 Dockerfile                   # Backend 도커 이미지
│   ├── 📄 package.json                 # 의존성 관리
│   ├── 📄 tsconfig.json                # TypeScript 설정
│   └── 📄 .env.example                 # Backend 환경변수 템플릿
│
├── 📁 database/                        # 데이터베이스 관련 파일
│   ├── 📁 scripts/
│   │   ├── 📁 init/                   # 초기화 스크립트
│   │   │   └── 📄 01-init-databases.sql
│   │   ├── 📁 migrations/             # 마이그레이션 스크립트
│   │   │   └── 📄 02-create-cs-tables.sql
│   │   └── 📁 seeds/                  # 시드 데이터 스크립트
│   │       └── 📄 03-seed-dummy-data.sql
│   ├── 📁 backups/                    # 데이터베이스 백업
│   │   └── 📄 full-dump.sql           # 전체 데이터 덤프 (310MB)
│   └── 📄 README.md                    # 데이터베이스 문서
│
├── 📁 mcp_server_practice/             # MCP 서버 애플리케이션
│   ├── 📁 src/
│   │   ├── 📁 tools/                  # MCP 도구
│   │   └── 📄 index.ts                # MCP 서버 엔트리
│   ├── 📄 Dockerfile                   # MCP 서버 도커 이미지
│   ├── 📄 docker-compose.yml           # MCP 서버 전용 Compose
│   ├── 📄 package.json                 # 의존성 관리
│   ├── 📄 tsconfig.json                # TypeScript 설정
│   └── 📄 readme.md                    # MCP 서버 문서
│
└── 📁 documents/                       # 프로젝트 문서
    ├── 📄 customer-complaint-guideline.md   # 고객 대응 지침서
    ├── 📄 complaint-response-templates.md   # 응대 템플릿
    └── 📄 n8n-workflow-guide.md             # n8n 워크플로우 가이드
```

### 주요 디렉토리 설명

- **frontend/**: Next.js 15 기반 고객 CS 민원 접수 웹 애플리케이션
- **backend_sarda_online/**: NestJS 기반 Backend API 서버
- **database/**: PostgreSQL 초기화, 마이그레이션, 백업 관리
- **mcp_server_practice/**: Model Context Protocol 서버 (AI 연동)
- **documents/**: 고객 대응 가이드 및 워크플로우 문서

## 🔄 n8n 워크플로우 시나리오

### 1. 고객 문의 자동 분류

- 문의 내용 키워드 분석
- 카테고리 자동 분류
- 담당자 자동 배정

### 2. JIRA 티켓 자동 생성

- 지침서 없는 케이스 감지
- JIRA API 연동
- 우선순위 자동 설정

### 3. 고객 응대 자동화

- 템플릿 기반 초기 응답
- 상태 업데이트 알림
- 만족도 조사 발송

## 🛠️ 유지보수

### 로그 확인

```bash
# Backend 로그
docker logs sarda_online_backend -f

# n8n 로그
docker logs sarda_online_n8n -f

# PostgreSQL 로그
docker logs sarda_online_postgres -f
```

### 데이터베이스 백업

```bash
# 백업
docker exec sarda_online_postgres pg_dump -U sarda_online_user sarda_online_db > backup.sql

# 복원
docker exec -i sarda_online_postgres psql -U sarda_online_user sarda_online_db < backup.sql
```

## 🚨 트러블슈팅

### 1. 컨테이너 시작 실패

```bash
# 모든 컨테이너 중지 및 제거
docker-compose down

# 볼륨 삭제 (주의: 데이터 손실)
docker-compose down -v

# 재시작
docker-compose up -d
```

### 2. 데이터베이스 연결 오류

- PostgreSQL 컨테이너 상태 확인
- 네트워크 설정 확인
- 환경 변수 확인

### 3. n8n 워크플로우 오류

- 자격 증명 확인
- API 엔드포인트 확인
- 로그 분석

## 📈 향후 계획

### Phase 1 (현재)

- [x] Backend API 구축
- [x] 사용자 관리 시스템
- [x] 로그 시스템
- [x] 고객 대응 지침서

### Phase 2

- [ ] n8n 워크플로우 구현
- [ ] JIRA 연동
- [ ] 자동 응답 시스템

### Phase 3

- [ ] AI 기반 문의 분류
- [ ] 실시간 대시보드
- [ ] 성과 분석 시스템

## 🗄️ 데이터베이스 스키마

### Customer Users 테이블

- 기본 정보: id, email, password, firstName, lastName
- 추가 정보: phoneNumber, birthDate, role, profileImageUrl
- 설정: preferences (알림 설정, 관심 카테고리, 언어)
- 주소: address (도로명, 도시, 지역, 우편번호)
- 로그인 정보: loginCount, lastLoginAt, lastLoginIp
- 메타데이터: metadata (추가 정보 저장용)

### Customer User Logs 테이블

- 이벤트 정보: eventType, eventCategory, eventData
- 기기 정보: ipAddress, userAgent, deviceInfo
- 위치 정보: location (국가, 도시, 좌표)
- 세션 정보: sessionId, referrer, currentUrl
- 성능 정보: responseTime, httpMethod, statusCode
- 분류: tags, level (debug, info, warning, error, critical)

### InternalUsers 테이블 (내부 직원)

- 기본 정보: id, email, firstName, lastName, password, phoneNumber
- 직무 정보: department, position, employeeId, role
- 권한 및 레벨: accessLevel, permissions
- CS 전문성: specialties, maxConcurrentTickets
- 근무 정보: workSchedule, isAvailable, currentWorkload
- 성과 지표: totalTicketsHandled, avgResolutionTime, satisfactionRating
- 상태 정보: status, lastActiveAt
- 메타데이터: metadata, createdAt, updatedAt

### CustomerComplaints 테이블 (고객 컴플레인)

- 티켓 정보: id, ticketNumber
- 고객 정보: userId, customerName, customerEmail, customerPhone
- 문의 분류: category, subCategory, priority, urgency
- 내용: subject, description, attachments
- 상태 관리: status, escalationLevel, isEscalated
- 처리 정보: assignedTo, assignedTeam, firstResponseAt, resolvedAt, responseTime, resolutionTime
- 관련 정보: relatedProductId, relatedOrderId, relatedSellerId, jiraTicketKey
- 보상/조치: compensationType, compensationAmount, compensationNote
- 고객 만족도: satisfactionScore, feedbackComment
- 메타데이터: tags, metadata, createdAt, updatedAt

### ComplaintResponses 테이블 (컴플레인 응답)

- 기본 정보: id, complaintId, responderId, responderType
- 응답 내용: responseType, content, attachments
- 플래그: isInternal, isAutoResponse
- 타임스탬프: createdAt

### ComplaintHistory 테이블 (컴플레인 이력)

- 기본 정보: id, complaintId, actorId
- 변경 내역: action, fromValue, toValue, note
- 메타데이터: metadata, createdAt

### ComplaintTemplates 테이블 (응답 템플릿)

- 기본 정보: id, category, subCategory, templateName
- 템플릿 내용: templateContent, variables
- 상태 정보: isActive, usageCount
- 생성 정보: createdBy, createdAt, updatedAt

### ComplaintSlaRules 테이블 (SLA 규칙)

- 기본 정보: id, category, priority
- 시간 규칙: firstResponseTime, resolutionTime, escalationTime
- 상태 정보: isActive, createdAt, updatedAt

### ComplaintKnowledgeBase 테이블 (지식 베이스)

- 기본 정보: id, category, subCategory
- 콘텐츠: question, answer, keywords, relatedArticles
- 통계: viewCount, helpfulCount, notHelpfulCount
- 상태 정보: isPublished, createdBy, createdAt, updatedAt

## 👥 팀 구성

- **백엔드 개발**: Backend API, Database
- **워크플로우 개발**: n8n, MCP Server
- **CS 팀**: 고객 대응, 지침서 관리
- **QA 팀**: 테스트, 품질 관리

---

**Last Updated**: 2025-06-30
