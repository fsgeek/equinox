# Quickstart: MCEP Development Environment

**Feature**: 001-mcep-engagement
**Date**: 2025-10-27
**Purpose**: Setup local development environment for MCEP platform

## Prerequisites

- Docker Desktop 4.0+ and Docker Compose 2.0+
- Go 1.21+ (for backend development)
- Python 3.11+ (for ML services)
- Node.js 18+ (for frontend development)
- Git

## Quick Start (5 minutes)

```bash
# 1. Clone repository
git clone https://github.com/equinox/equinox-spec-kit.git
cd equinox-spec-kit
git checkout 001-mcep-engagement

# 2. Start infrastructure services
docker-compose up -d postgres redis kafka

# 3. Run database migrations
cd backend && go run cmd/migrate/main.go up

# 4. Start backend services
go run cmd/api-gateway/main.go &
go run cmd/campaign-service/main.go &

# 5. Start ML service
cd ../ml-services/lead-scoring
uvicorn main:app --reload &

# 6. Start frontend
cd ../../frontend
npm install && npm run dev

# 7. Access dashboard
open http://localhost:3000
```

## Detailed Setup

### 1. Infrastructure Services

**docker-compose.yml** configures:
- PostgreSQL 15 (port 5432)
- TimescaleDB extension for metrics
- Redis (port 6379)
- Kafka + Zookeeper (port 9092)
- RabbitMQ (port 5672, management UI 15672)

```bash
docker-compose up -d
docker-compose ps  # Verify all services running
```

### 2. Database Setup

```bash
cd backend

# Run migrations
go run cmd/migrate/main.go up

# Seed test data (10 PMs, 50 profiles, 100 devices, 1000 prospects)
go run cmd/seed/main.go

# Verify
psql postgresql://localhost:5432/mcep_dev -c "SELECT COUNT(*) FROM portfolio_managers;"
```

### 3. Backend Services

**Microservices** (each runs independently):

```bash
# API Gateway (port 8080)
cd backend/cmd/api-gateway
go run main.go

# Campaign Service (port 8081)
cd backend/cmd/campaign-service
go run main.go

# Message Router (port 8082)
cd backend/cmd/message-router
go run main.go

# Device Orchestrator (port 8083)
cd backend/cmd/device-orchestrator
go run main.go
```

**Environment Variables** (create `.env`):
```
DATABASE_URL=postgresql://localhost:5432/mcep_dev
REDIS_URL=redis://localhost:6379
KAFKA_BROKERS=localhost:9092
JWT_SECRET=development-secret-change-in-production
WHATSAPP_API_KEY=<your-key>
TELEGRAM_BOT_TOKEN=<your-token>
```

### 4. ML Services

```bash
cd ml-services/lead-scoring

# Create virtual environment
python -m venv .venv
source .venv/bin/activate  # Windows: .venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt

# Start FastAPI service (port 8000)
uvicorn main:app --reload

# Test ML endpoint
curl http://localhost:8000/api/v1/score \
  -H "Content-Type: application/json" \
  -d '{"prospect_id": "uuid-here"}'
```

### 5. Frontend Dashboard

```bash
cd frontend

# Install dependencies
npm install

# Start development server (port 3000)
npm run dev

# Build for production
npm run build
```

### 6. Testing

**Backend Tests**:
```bash
cd backend
go test ./...  # All packages
go test -v ./internal/adapters/whatsapp  # Specific package
```

**ML Tests**:
```bash
cd ml-services/lead-scoring
pytest tests/ -v
```

**Frontend Tests**:
```bash
cd frontend
npm test  # Jest unit tests
npm run test:e2e  # Playwright end-to-end
```

## API Documentation

**Swagger UI**: http://localhost:8080/swagger

**GraphQL Playground**: http://localhost:8080/graphql

**API Spec**: See `specs/001-mcep-engagement/contracts/mcep-api.yaml`

## Development Workflow

### Creating a New Campaign (Example)

```bash
# 1. Authenticate
TOKEN=$(curl -X POST http://localhost:8080/api/v1/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"pm@example.com","password":"dev123"}' \
  | jq -r '.token')

# 2. Create engagement profile
PROFILE_ID=$(curl -X POST http://localhost:8080/api/v1/profiles \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Crypto Investors",
    "language": "en",
    "branding_config": {"colors": {"primary": "#FF6B35"}},
    "messaging_templates": {
      "whatsapp": "Hi {name}, check out our crypto fund..."
    }
  }' | jq -r '.id')

# 3. Create campaign
CAMPAIGN_ID=$(curl -X POST http://localhost:8080/api/v1/campaigns \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d "{
    \"name\": \"Q4 Crypto Campaign\",
    \"target_segment\": {\"interests\": [\"crypto\"]},
    \"message_content\": \"Test message\",
    \"profile_ids\": [\"$PROFILE_ID\"]
  }" | jq -r '.id')

# 4. Launch campaign
curl -X PUT http://localhost:8080/api/v1/campaigns/$CAMPAIGN_ID/launch \
  -H "Authorization: Bearer $TOKEN"

# 5. Monitor analytics (GraphQL)
curl -X POST http://localhost:8080/graphql \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "query": "query { campaign(id: \"'$CAMPAIGN_ID'\") { messagesSent messagesDelivered responseRate } }"
  }'
```

## Troubleshooting

**Database Connection Failed**:
```bash
docker-compose ps postgres  # Verify running
docker-compose logs postgres  # Check logs
```

**Device Pool Empty**:
```bash
# Seed test devices
go run cmd/seed/main.go --devices-only
```

**ML Service Timeout**:
```bash
# Check model files exist
ls ml-services/lead-scoring/models/
# Re-train if missing
python train.py
```

## Configuration

**Constitution Alignment**: All configuration follows Principle V (Architecture Documentation)

**Backend**: `backend/config/config.yaml`
**ML Service**: `ml-services/lead-scoring/.env`
**Frontend**: `frontend/.env.local`

## Next Steps

1. Review API contracts: `contracts/mcep-api.yaml`
2. Review data model: `data-model.md`
3. Run `/speckit.tasks` to see implementation tasks
4. Start with P1 tasks (Unified Communication Infrastructure)

---

**Version**: 1.0.0 | **Generated**: 2025-10-27 | **Purpose**: Development environment setup for SDD demonstration
