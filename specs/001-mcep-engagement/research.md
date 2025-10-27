# Technical Research: Multi-Channel Client Engagement Platform (MCEP)

**Feature**: 001-mcep-engagement
**Date**: 2025-10-27
**Purpose**: Research and justify technical decisions for MCEP implementation

## Executive Summary

The MCEP platform requires managing 1,500-2,000 mobile devices, orchestrating messaging across 4 platforms (WhatsApp, Telegram, Signal, iMessage), handling 1,500 Portfolio Managers, and serving 250,000+ client relationships. The architecture must support real-time analytics, machine learning-based lead scoring, and integration with multiple CRM systems.

## Technology Stack Decisions

### 1. Backend Architecture

**Decision**: Microservices architecture with event-driven communication

**Rationale**:
- **Scale requirements**: 1,500 PMs × 1,000 recipients per campaign requires horizontal scalability
- **Fault isolation**: Device fleet management failures shouldn't impact analytics or CRM integration
- **Independent deployment**: Messaging, analytics, and ML scoring can be deployed independently
- **Team separation**: Aligns with separate compliance team developing multi-jurisdictional system

**Alternatives Considered**:
- **Monolithic**: Simpler initially but doesn't support independent scaling or team separation
- **Serverless**: Cost-effective for variable load but complex for stateful device management

**Technologies**:
- **Primary Language**: Go (Golang) 1.21+
  - Excellent concurrency for managing device pools
  - Low memory footprint for cost efficiency at scale
  - Strong ecosystem for messaging and API services
- **Secondary**: Python 3.11+ for ML/data science components
  - Lead scoring and A/B testing analytics
  - Integration with ML libraries (scikit-learn, pandas)

### 2. Device Fleet Management

**Decision**: Custom device orchestration service with mobile device management (MDM) integration

**Rationale**:
- **Scale**: 1,500-2,000 devices require automated provisioning and health monitoring
- **Platform requirements**: WhatsApp Business API, Telegram Bot API, Signal Protocol, iMessage (iOS-specific)
- **Dynamic allocation**: Devices must be assigned to campaigns based on load

**Alternatives Considered**:
- **Manual management**: Not feasible at 1,500+ device scale
- **Off-the-shelf MDM only**: Doesn't handle message routing and campaign assignment logic
- **Cloud-based device farms**: Messaging platforms may block cloud IPs; requires real consumer devices

**Technologies**:
- **MDM Platform**: Hexnode or ManageEngine for iOS/Android fleet management
- **Message Brokers**:
  - WhatsApp Business API (official cloud API)
  - Telegram Bot API
  - Signal-CLI for Signal Protocol
  - iMessage via Mac Mini servers with Apple Business Manager
- **Device Health Monitoring**: Prometheus + Grafana for real-time device status

### 3. Messaging Platform Integration

**Decision**: Adapter pattern with platform-specific clients behind unified interface

**Rationale**:
- **Platform diversity**: Each messaging platform has different APIs, rate limits, authentication
- **Abstraction**: Portfolio Managers interact with unified interface regardless of platform
- **Retry logic**: Exponential backoff (1min, 5min, 15min) implemented consistently across platforms

**Platform-Specific Considerations**:
- **WhatsApp Business API**: Official cloud API, 1,000 messages/day per number, webhook-based
- **Telegram Bot API**: HTTP-based, 30 messages/second per bot, long polling or webhooks
- **Signal**: Signal-CLI requires registration, rate limits less documented
- **iMessage**: iOS-only, requires physical iOS devices or Mac servers, no official API

**Technologies**:
- **WhatsApp**: Official WhatsApp Business Cloud API
- **Telegram**: Telegram Bot API (golang library: go-telegram-bot-api)
- **Signal**: signal-cli (Java-based CLI wrapper)
- **iMessage**: Custom service using AppleScript/Shortcuts automation on Mac Mini fleet

### 4. Real-Time Analytics & Metrics

**Decision**: Time-series database with streaming aggregation pipeline

**Rationale**:
- **Real-time requirement**: SC-003 specifies <30 second delay for engagement analytics
- **Volume**: 1,500 PMs × 1,000 recipients × multiple metrics (delivery, reads, responses)
- **Query patterns**: Dashboard aggregations, A/B test comparisons, lead scoring inputs

**Alternatives Considered**:
- **Relational database only**: Query performance degrades with high write volume
- **Pre-aggregated tables**: Complex to maintain with real-time updates
- **NoSQL document store**: Good for writes but inefficient for time-range queries

**Technologies**:
- **Time-Series DB**: InfluxDB or TimescaleDB (PostgreSQL extension)
  - InfluxDB: Purpose-built for time-series, excellent retention policies
  - TimescaleDB: Leverages PostgreSQL reliability, familiar query language
  - **Recommendation**: TimescaleDB for consistency with relational data
- **Streaming Pipeline**: Apache Kafka + ksqlDB for real-time aggregation
- **Visualization**: Grafana for Portfolio Manager dashboards

### 5. Machine Learning Lead Scoring

**Decision**: Python-based ML service with feature store and model registry

**Rationale**:
- **Configurable thresholds**: 50-90 range per Portfolio Manager requires flexible scoring
- **85% accuracy target**: Requires iterative model training and evaluation
- **Real-time inference**: Scoring must occur as prospects engage (not batch)

**ML Architecture**:
- **Feature Engineering**: Extract engagement patterns (frequency, response time, message sentiment)
- **Model Type**: Gradient Boosting (XGBoost/LightGBM) for tabular data
- **Training Pipeline**: Periodic retraining with human advisor feedback loop
- **Serving**: FastAPI service with in-memory model for <100ms inference

**Technologies**:
- **Framework**: Python 3.11 with scikit-learn, XGBoost
- **Feature Store**: Feast (open-source feature store)
- **Model Registry**: MLflow for versioning and A/B testing models
- **Serving**: FastAPI + uvicorn for REST API

### 6. CRM Integration

**Decision**: Webhook/API gateway with adapter pattern for multiple CRM systems

**Rationale**:
- **Clarification decision**: Generic API/webhook approach supporting Salesforce, HubSpot, custom CRMs
- **Flexibility**: Portfolio Managers may use different CRM systems
- **Lead handoff**: Qualified leads (threshold met) trigger webhook with standardized payload

**Integration Pattern**:
- **Outbound**: Webhook push to CRM systems on qualified lead events
- **Standard Payload**: JSON schema with lead score, engagement history, contact info
- **Retry Logic**: Same exponential backoff pattern as messaging (1min, 5min, 15min)
- **Security**: HMAC signature validation, TLS 1.3, API key rotation

**Technologies**:
- **API Gateway**: Kong or AWS API Gateway for rate limiting and transformation
- **Webhook Framework**: Svix (webhook delivery infrastructure)
- **CRM Adapters**:
  - Salesforce: REST API + OAuth 2.0
  - HubSpot: REST API + OAuth 2.0
  - Generic: Configurable HTTP POST endpoint

### 7. Data Storage

**Decision**: Multi-database strategy based on data characteristics

**Rationale**:
- **Transactional data**: Portfolio Managers, profiles, campaigns require ACID guarantees
- **Time-series metrics**: Engagement analytics optimized for time-range queries
- **Audit logs**: Immutable compliance audit trail
- **Device state**: Frequently updated device health and allocation status

**Database Allocation**:
- **Primary RDBMS**: PostgreSQL 15+ for transactional data
  - Portfolio Managers, Engagement Profiles, Campaigns, Prospects
  - JSONB columns for flexible profile configurations
- **Time-Series**: TimescaleDB (PostgreSQL extension) for engagement metrics
- **Audit Trail**: PostgreSQL with append-only table (immutable records)
- **Cache Layer**: Redis for device allocation state and session management
- **Document Store**: MongoDB for flexible message templates and compliance rules (Phase 2)

### 8. Compliance Monitoring (Phase 1)

**Decision**: Rule engine with flagging system and manual review queue

**Rationale**:
- **Phase 1 scope**: Send-with-warning approach, flag violations for post-send review
- **Separation of concerns**: Compliance rules maintained separately by compliance team
- **Audit requirement**: Complete message audit trail with flagging metadata
- **Phase 2 readiness**: Architecture must accommodate future automated blocking

**Technologies**:
- **Rule Engine**: Drools (Java-based) or custom Go-based rule evaluator
- **Queue System**: RabbitMQ for compliance review queue
- **Audit Storage**: PostgreSQL with immutable audit log table
- **Review Interface**: React-based admin panel for compliance team

### 9. A/B Testing Framework

**Decision**: Feature flagging system with statistical significance calculation

**Rationale**:
- **User Story 2 requirement**: Optimize engagement strategies through A/B testing
- **SC-004**: Identify statistically significant improvements within 100 message sample size
- **Use cases**: Messaging templates, profile configurations, timing strategies

**Technologies**:
- **Feature Flags**: LaunchDarkly or open-source Unleash
- **Statistics**: Python scipy.stats for chi-square tests and confidence intervals
- **Experiment Tracking**: MLflow experiments for test configurations and results

### 10. API Design

**Decision**: RESTful API with OpenAPI 3.0 specification + GraphQL for analytics

**Rationale**:
- **REST for mutations**: Campaign creation, message sending, profile management
- **GraphQL for queries**: Flexible analytics queries for dashboards
- **API-first**: Contracts defined before implementation for parallel development

**API Structure**:
- **REST Endpoints**:
  - `/api/v1/campaigns` - Campaign CRUD
  - `/api/v1/profiles` - Engagement profile management
  - `/api/v1/messages` - Message sending and status
  - `/api/v1/devices` - Device fleet management (admin)
  - `/api/v1/webhooks` - CRM webhook registration
- **GraphQL Endpoint**: `/graphql` for analytics and reporting queries

**Technologies**:
- **REST Framework**: Go Gin or Echo framework
- **GraphQL Server**: gqlgen (Go code generation from schema)
- **API Documentation**: Swagger UI from OpenAPI spec

## Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                      MCEP Platform                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                   │
│  ┌──────────────────┐         ┌──────────────────┐             │
│  │  API Gateway     │◄────────┤  Portfolio       │             │
│  │  (Kong/AWS)      │         │  Manager UI      │             │
│  └────────┬─────────┘         └──────────────────┘             │
│           │                                                      │
│  ┌────────▼─────────────────────────────────────────────────┐  │
│  │              Core Services (Go Microservices)             │  │
│  ├───────────────────────────────────────────────────────────┤  │
│  │ Campaign Mgmt │ Message Router │ Profile Mgmt │ Analytics │  │
│  └────┬──────────┴──────┬─────────┴─────┬────────┴─────┬─────┘  │
│       │                 │               │              │        │
│  ┌────▼──────────┐  ┌───▼─────────┐  ┌─▼─────────┐  ┌▼────────┐│
│  │ Device Fleet  │  │  Message    │  │ Lead      │  │ CRM     ││
│  │ Orchestrator  │  │  Adapters   │  │ Scoring   │  │ Webhooks││
│  └───────┬───────┘  └───┬─────────┘  │ (Python)  │  └─────────┘│
│          │              │            └─────────────┘             │
│  ┌───────▼──────────────▼───────────────────────────────┐       │
│  │      Device Pool (1,500-2,000 iOS/Android)          │       │
│  │   WhatsApp│Telegram│Signal│iMessage                 │       │
│  └──────────────────────────────────────────────────────┘       │
│                                                                   │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │              Data Layer                                  │    │
│  ├─────────────────────────────────────────────────────────┤    │
│  │ PostgreSQL  │ TimescaleDB │ Redis │ Kafka │ RabbitMQ  │    │
│  └─────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────┘
```

## Performance Considerations

### Throughput Targets
- **SC-010**: 1,500 Portfolio Managers × 250,000 client relationships = 375M relationship capacity
- **Campaign launch**: 1,000 recipients in <5 minutes (SC-001) = ~3.3 messages/second per campaign
- **Concurrent campaigns**: Assume 10% of PMs active = 150 campaigns × 3.3 msg/s = ~500 msg/s sustained
- **Platform rate limits**:
  - WhatsApp Business: 1,000 messages/day per number (need ~500 numbers for burst)
  - Telegram: 30 messages/second per bot (need ~17 bots for 500 msg/s)
  - Signal/iMessage: Rate limits less documented, requires testing

### Latency Targets
- **API response**: <200ms p95 (from original requirements NFR-SCALE-3)
- **Analytics delay**: <30 seconds (SC-003)
- **Lead scoring inference**: <100ms (for real-time scoring)
- **Device allocation**: <2 seconds (to assign device to campaign)

### Scale Optimization
- **Device pooling**: Dynamic allocation reduces total devices needed vs. dedicated per-PM
- **Message batching**: Group messages to same platform for efficiency
- **Caching**: Redis cache for device state, profile lookups, recent metrics
- **Database partitioning**: Time-series data partitioned by date for query performance
- **Async processing**: Kafka streams for non-blocking analytics aggregation

## Security Considerations

### Authentication & Authorization
- **Portfolio Manager auth**: OAuth 2.0 with RBAC (admin vs. standard PM)
- **API keys**: Rotating keys for CRM webhook registration
- **Device access**: Secure credential storage (HashiCorp Vault or AWS Secrets Manager)
- **Audit logging**: All API calls logged with user ID and timestamp

### Data Protection
- **Encryption in transit**: TLS 1.3 for all API communication
- **Encryption at rest**: Database encryption (PostgreSQL pgcrypto)
- **Message content**: Temporary storage only, purged after audit retention period
- **PII handling**: GDPR/CCPA compliance (right to deletion in spec FR-018 from original requirements)

### Compliance (Phase 1)
- **Audit trail**: Immutable logs in PostgreSQL
- **Flagging system**: Violations flagged but messages sent (per clarification #5)
- **Manual review queue**: RabbitMQ queue for compliance team
- **Phase 2 preparation**: Architecture supports rule-based blocking

## Deployment Architecture

### Infrastructure
- **Cloud Provider**: AWS (or Azure/GCP - to be determined by infrastructure team)
- **Orchestration**: Kubernetes (EKS) for microservices
- **Container Registry**: Docker Hub or AWS ECR
- **CI/CD**: GitHub Actions for automated testing and deployment

### Environments
- **Development**: Local Docker Compose for full stack
- **Staging**: Kubernetes cluster with limited device pool (10-20 devices)
- **Production**: Multi-region Kubernetes with full device fleet

### Monitoring & Observability
- **Metrics**: Prometheus + Grafana
- **Logging**: ELK Stack (Elasticsearch, Logstash, Kibana) or AWS CloudWatch
- **Tracing**: Jaeger for distributed tracing
- **Alerting**: PagerDuty integration for critical failures

## Risk Mitigation

### Technical Risks
1. **Messaging platform rate limits**: Mitigated by device pool size and queuing
2. **Device fleet management complexity**: Start with 100 devices, scale gradually
3. **ML model accuracy**: Iterative training with human feedback loop
4. **Multi-jurisdiction compliance**: Phase 2 external system integration

### Operational Risks
1. **Device failures**: Automatic device health monitoring and failover
2. **Message delivery failures**: 3-retry exponential backoff (clarification #3)
3. **CRM webhook failures**: Same retry logic, dead letter queue for manual intervention
4. **Compliance violations**: Manual review team in Phase 1, automated in Phase 2

## Open Questions for Implementation

1. **Cloud provider preference**: AWS, Azure, or GCP?
2. **iMessage device strategy**: Mac Mini servers vs. iOS device farm?
3. **Device procurement**: Purchase vs. lease for 1,500-2,000 devices?
4. **Compliance team capacity**: How many reviewers for manual review queue?
5. **Phase 2 timeline**: When does external compliance system become available?
6. **Budget constraints**: Device costs, cloud infrastructure, third-party services?

## Next Steps

After approval of this research:
1. Generate data-model.md with entity schemas
2. Create OpenAPI contracts for REST endpoints
3. Create GraphQL schema for analytics queries
4. Generate quickstart.md for development environment setup
5. Update agent context files with selected technologies
