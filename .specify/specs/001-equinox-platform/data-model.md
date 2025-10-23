# Data Model - Project Equinox

## Overview

This document defines the data models and database schemas for all three core modules of the Project Equinox platform:

- **GEMS (Global Engagement & Mobile Systems)** - Device fleet and persona management
- **CRIP (Client Relationship & Investment Portal)** - Client relationships and portfolio data
- **TDAMS (Treasury & Digital Asset Management System)** - Cryptocurrency wallets and transactions

## Database Strategy

### Primary Databases

**Relational Database (PostgreSQL)**
- Client profiles and relationships
- Portfolio configurations and administrative data
- Audit trails and compliance records
- User authentication and authorization

**NoSQL Database (MongoDB/DynamoDB)**
- Persona profiles (flexible schema for diverse platforms)
- Device metadata and configurations
- Campaign data and metrics
- Message history

**Time-Series Database (TimescaleDB/InfluxDB)**
- Portfolio performance history
- Device health metrics
- Campaign performance metrics

**Blockchain Data (Read from nodes)**
- Transaction history (Bitcoin, Ethereum)
- Wallet balances
- On-chain data verification

---

## GEMS Data Models

### Device

Represents a mobile device in the managed fleet.

```sql
CREATE TABLE devices (
    device_id VARCHAR(255) PRIMARY KEY,
    platform VARCHAR(20) NOT NULL CHECK (platform IN ('iOS', 'Android')),
    device_model VARCHAR(100),
    os_version VARCHAR(50),
    status VARCHAR(20) NOT NULL DEFAULT 'provisioning'
        CHECK (status IN ('online', 'offline', 'provisioning', 'error', 'maintenance')),
    label VARCHAR(255),
    assigned_persona_id UUID REFERENCES personas(persona_id),
    last_seen TIMESTAMPTZ,
    registered_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    CONSTRAINT device_platform_model CHECK (
        (platform = 'iOS' AND device_model LIKE 'iPhone%') OR
        (platform = 'Android')
    )
);

CREATE INDEX idx_devices_status ON devices(status);
CREATE INDEX idx_devices_assigned_persona ON devices(assigned_persona_id);
CREATE INDEX idx_devices_last_seen ON devices(last_seen);
```

### Installed Application

Tracks applications installed on devices.

```sql
CREATE TABLE installed_applications (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    device_id VARCHAR(255) NOT NULL REFERENCES devices(device_id) ON DELETE CASCADE,
    application_id VARCHAR(255) NOT NULL,
    application_name VARCHAR(255),
    version VARCHAR(50),
    status VARCHAR(20) NOT NULL DEFAULT 'installed'
        CHECK (status IN ('installed', 'updating', 'error')),
    configuration JSONB,
    installed_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    UNIQUE(device_id, application_id)
);

CREATE INDEX idx_installed_apps_device ON installed_applications(device_id);
CREATE INDEX idx_installed_apps_status ON installed_applications(status);
```

### Persona

Manager persona profiles with social media credentials.

```json
// MongoDB Collection: personas
{
    "_id": "uuid-v4",
    "name": "Alexandra Chen",
    "profile": {
        "occupation": "Investment Manager",
        "location": "Singapore",
        "bio": "Specialized in alternative asset management...",
        "photoUrl": "https://cdn.equinox.example/personas/photos/abc123.jpg",
        "backgroundUrl": "https://cdn.equinox.example/personas/backgrounds/def456.jpg"
    },
    "credentials": [
        {
            "platform": "linkedin",
            "username": "alexandra.chen.sg",
            "accessToken": "encrypted-token-data",
            "refreshToken": "encrypted-refresh-token",
            "expiresAt": "2025-12-31T23:59:59Z"
        },
        {
            "platform": "whatsapp",
            "phoneNumber": "+65XXXXXXXX",
            "apiKey": "encrypted-api-key"
        }
    ],
    "status": "active",  // active, inactive, assigned
    "assignedDevice": "IMEI-356938035643809",  // null if unassigned
    "createdAt": "2025-01-15T10:30:00Z",
    "updatedAt": "2025-10-20T14:22:00Z",
    "metadata": {
        "tags": ["high-value", "APAC"],
        "notes": "Experienced manager for crypto assets"
    }
}
```

**Indexes:**
```javascript
db.personas.createIndex({ "status": 1 })
db.personas.createIndex({ "assignedDevice": 1 })
db.personas.createIndex({ "name": "text", "profile.occupation": "text" })
db.personas.createIndex({ "createdAt": -1 })
```

### Campaign

Outreach campaign configuration and execution.

```json
// MongoDB Collection: campaigns
{
    "_id": "uuid-v4",
    "name": "Q4 Alternative Assets Outreach",
    "scriptId": "uuid-of-script",
    "targetCriteria": {
        "platforms": ["linkedin", "instagram"],
        "regions": ["NA", "EU", "APAC"],
        "keywords": ["investment", "crypto", "alternative assets"],
        "minConnectionLevel": 1
    },
    "personas": [
        "persona-uuid-1",
        "persona-uuid-2"
    ],
    "schedule": {
        "startTime": "2025-10-25T09:00:00Z",
        "endTime": "2025-12-31T23:59:59Z",
        "maxContactsPerDay": 100
    },
    "status": "active",  // draft, scheduled, active, paused, completed
    "metrics": {
        "totalTargets": 15420,
        "contacted": 3250,
        "responses": 487,
        "conversions": 62,
        "lastUpdated": "2025-10-23T16:00:00Z"
    },
    "createdAt": "2025-10-15T10:00:00Z",
    "createdBy": "manager-uuid",
    "updatedAt": "2025-10-23T16:00:00Z"
}
```

### Engagement Script

Template for automated outreach messaging.

```json
// MongoDB Collection: scripts
{
    "_id": "uuid-v4",
    "name": "Professional Introduction - Alternative Assets",
    "description": "Initial contact script for HNW individuals interested in alternative investments",
    "template": {
        "initialContact": "Hi {targetName}, I noticed your interest in {sharedTopic}. I specialize in alternative asset opportunities that might align with your investment goals...",
        "followUpSequence": [
            {
                "delayHours": 48,
                "message": "Following up on my previous message. Would you be interested in learning about current opportunities in {assetClass}?"
            },
            {
                "delayHours": 168,
                "message": "Just wanted to share that we've recently opened positions in {specificOpportunity}..."
            }
        ],
        "variables": [
            "targetName",
            "sharedTopic",
            "assetClass",
            "specificOpportunity"
        ]
    },
    "createdAt": "2025-09-01T10:00:00Z",
    "createdBy": "content-team-uuid",
    "version": "1.2",
    "status": "approved"
}
```

---

## CRIP Data Models

### Client

Client profile and relationship data.

```sql
CREATE TABLE clients (
    client_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(255) NOT NULL,
    email VARCHAR(255) NOT NULL UNIQUE,
    phone VARCHAR(50),
    jurisdiction VARCHAR(2) NOT NULL,  -- ISO 3166-1 alpha-2
    manager_id UUID NOT NULL,  -- References persona ID from GEMS
    profile JSONB,
    status VARCHAR(20) NOT NULL DEFAULT 'pending'
        CHECK (status IN ('active', 'inactive', 'pending')),
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    CONSTRAINT valid_email CHECK (email ~* '^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}$')
);

CREATE INDEX idx_clients_manager ON clients(manager_id);
CREATE INDEX idx_clients_jurisdiction ON clients(jurisdiction);
CREATE INDEX idx_clients_status ON clients(status);
CREATE INDEX idx_clients_email ON clients(email);
```

### Communication Channel

Encrypted communication channels between managers and clients.

```sql
CREATE TABLE communication_channels (
    channel_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    client_id UUID NOT NULL REFERENCES clients(client_id) ON DELETE CASCADE,
    manager_id UUID NOT NULL,  -- References persona ID
    type VARCHAR(20) NOT NULL CHECK (type IN ('text', 'voice', 'video')),
    encryption_enabled BOOLEAN NOT NULL DEFAULT true,
    encryption_key_id VARCHAR(255),  -- Reference to key management system
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    last_activity TIMESTAMPTZ,

    UNIQUE(client_id, manager_id, type)
);

CREATE INDEX idx_channels_client ON communication_channels(client_id);
CREATE INDEX idx_channels_manager ON communication_channels(manager_id);
```

### Message

Encrypted messages within communication channels.

```json
// MongoDB Collection: messages
{
    "_id": "uuid-v4",
    "channelId": "channel-uuid",
    "senderId": "persona-or-client-uuid",
    "senderType": "manager",  // manager or client
    "content": "encrypted-message-content",
    "encryptionAlgorithm": "AES-256-GCM",
    "iv": "initialization-vector",
    "attachments": [
        {
            "fileId": "uuid",
            "fileName": "document.pdf",
            "fileType": "application/pdf",
            "fileUrl": "encrypted-url",
            "fileSize": 125440
        }
    ],
    "timestamp": "2025-10-23T14:30:00Z",
    "read": false,
    "readAt": null
}
```

### Portfolio

Investment portfolio configuration.

```sql
CREATE TABLE portfolios (
    portfolio_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    client_id UUID NOT NULL REFERENCES clients(client_id) ON DELETE CASCADE,
    name VARCHAR(255) NOT NULL,
    description TEXT,
    strategy TEXT,
    total_value DECIMAL(20,2),
    currency VARCHAR(3) NOT NULL DEFAULT 'USD',
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    last_calculated TIMESTAMPTZ
);

CREATE INDEX idx_portfolios_client ON portfolios(client_id);
```

### Holding

Individual holdings within a portfolio.

```sql
CREATE TABLE holdings (
    holding_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    portfolio_id UUID NOT NULL REFERENCES portfolios(portfolio_id) ON DELETE CASCADE,
    asset VARCHAR(50) NOT NULL,  -- Ticker or crypto symbol
    asset_name VARCHAR(255),
    quantity DECIMAL(30,10) NOT NULL,
    current_value DECIMAL(20,2),
    cost_basis DECIMAL(20,2),
    unrealized_gain DECIMAL(20,2),
    unrealized_gain_percent DECIMAL(10,4),
    last_updated TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    UNIQUE(portfolio_id, asset)
);

CREATE INDEX idx_holdings_portfolio ON holdings(portfolio_id);
CREATE INDEX idx_holdings_asset ON holdings(asset);
```

### Portfolio Transaction

Historical transactions for portfolio.

```sql
CREATE TABLE portfolio_transactions (
    transaction_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    portfolio_id UUID NOT NULL REFERENCES portfolios(portfolio_id) ON DELETE CASCADE,
    date TIMESTAMPTZ NOT NULL,
    type VARCHAR(20) NOT NULL
        CHECK (type IN ('buy', 'sell', 'deposit', 'withdrawal', 'dividend', 'fee')),
    asset VARCHAR(50),
    quantity DECIMAL(30,10),
    value DECIMAL(20,2) NOT NULL,
    fee DECIMAL(20,2) DEFAULT 0,
    notes TEXT,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_portfolio_txns_portfolio ON portfolio_transactions(portfolio_id);
CREATE INDEX idx_portfolio_txns_date ON portfolio_transactions(date);
CREATE INDEX idx_portfolio_txns_type ON portfolio_transactions(type);
```

### Administrative Adjustment

Audit trail for administrative portfolio adjustments.

```sql
CREATE TABLE portfolio_adjustments (
    adjustment_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    portfolio_id UUID NOT NULL REFERENCES portfolios(portfolio_id) ON DELETE CASCADE,
    adjusted_by VARCHAR(255) NOT NULL,  -- User ID
    adjustment_type VARCHAR(50) NOT NULL
        CHECK (adjustment_type IN ('totalValue', 'holdingValue', 'performance', 'transaction')),
    adjustment_data JSONB NOT NULL,
    rationale TEXT,
    effective_date TIMESTAMPTZ NOT NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    -- Audit trail immutability
    CONSTRAINT no_delete CHECK (false)  -- Prevent DELETE via constraint
);

-- Enable row-level security for additional protection
ALTER TABLE portfolio_adjustments ENABLE ROW LEVEL SECURITY;

CREATE INDEX idx_adjustments_portfolio ON portfolio_adjustments(portfolio_id);
CREATE INDEX idx_adjustments_date ON portfolio_adjustments(effective_date);
CREATE INDEX idx_adjustments_user ON portfolio_adjustments(adjusted_by);
```

### Portfolio Performance (Time-Series)

```sql
-- TimescaleDB hypertable
CREATE TABLE portfolio_performance (
    time TIMESTAMPTZ NOT NULL,
    portfolio_id UUID NOT NULL,
    value DECIMAL(20,2) NOT NULL,
    day_change DECIMAL(20,2),
    day_change_percent DECIMAL(10,4),
    PRIMARY KEY (time, portfolio_id)
);

SELECT create_hypertable('portfolio_performance', 'time');

CREATE INDEX idx_perf_portfolio_time ON portfolio_performance (portfolio_id, time DESC);
```

---

## TDAMS Data Models

### Wallet

Cryptocurrency wallet management.

```sql
CREATE TABLE wallets (
    wallet_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    asset_type VARCHAR(20) NOT NULL CHECK (asset_type IN ('BTC', 'ETH', 'USDT', 'USDC')),
    label VARCHAR(255),
    master_public_key TEXT NOT NULL,  -- BIP32 extended public key
    derivation_path VARCHAR(100) NOT NULL,
    confirmed_balance DECIMAL(30,10) NOT NULL DEFAULT 0,
    pending_balance DECIMAL(30,10) NOT NULL DEFAULT 0,
    total_balance DECIMAL(30,10) GENERATED ALWAYS AS (confirmed_balance + pending_balance) STORED,
    metadata JSONB,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_wallets_asset_type ON wallets(asset_type);
CREATE INDEX idx_wallets_label ON wallets(label);
```

### Address

Individual addresses within a wallet.

```sql
CREATE TABLE addresses (
    address VARCHAR(255) PRIMARY KEY,
    wallet_id UUID NOT NULL REFERENCES wallets(wallet_id) ON DELETE CASCADE,
    label VARCHAR(255),
    derivation_path VARCHAR(100) NOT NULL,
    address_index INTEGER NOT NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    last_used TIMESTAMPTZ,

    UNIQUE(wallet_id, address_index)
);

CREATE INDEX idx_addresses_wallet ON addresses(wallet_id);
CREATE INDEX idx_addresses_created ON addresses(created_at);
```

### Transaction

Blockchain transactions.

```sql
CREATE TABLE transactions (
    tx_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tx_hash VARCHAR(255) NOT NULL UNIQUE,
    from_wallet_id UUID REFERENCES wallets(wallet_id),
    asset_type VARCHAR(20) NOT NULL,
    status VARCHAR(20) NOT NULL DEFAULT 'pending'
        CHECK (status IN ('pending', 'confirmed', 'failed')),
    confirmations INTEGER DEFAULT 0,
    fee DECIMAL(30,10),
    metadata JSONB,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    confirmed_at TIMESTAMPTZ,

    CONSTRAINT valid_confirmations CHECK (
        (status = 'confirmed' AND confirmations > 0) OR
        (status != 'confirmed' AND confirmations = 0)
    )
);

CREATE INDEX idx_transactions_wallet ON transactions(from_wallet_id);
CREATE INDEX idx_transactions_status ON transactions(status);
CREATE INDEX idx_transactions_hash ON transactions(tx_hash);
CREATE INDEX idx_transactions_created ON transactions(created_at);
```

### Transaction Output

Individual outputs within a transaction.

```sql
CREATE TABLE transaction_outputs (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tx_id UUID NOT NULL REFERENCES transactions(tx_id) ON DELETE CASCADE,
    output_index INTEGER NOT NULL,
    address VARCHAR(255) NOT NULL,
    amount DECIMAL(30,10) NOT NULL,
    label VARCHAR(255),

    UNIQUE(tx_id, output_index)
);

CREATE INDEX idx_tx_outputs_tx ON transaction_outputs(tx_id);
CREATE INDEX idx_tx_outputs_address ON transaction_outputs(address);
```

### Liquidity Workflow

Automated liquidity management workflows.

```sql
CREATE TABLE liquidity_workflows (
    workflow_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    type VARCHAR(20) NOT NULL CHECK (type IN ('consolidate', 'distribute')),
    status VARCHAR(20) NOT NULL DEFAULT 'pending'
        CHECK (status IN ('pending', 'in_progress', 'completed', 'failed')),
    source_wallets UUID[] NOT NULL,
    destination_wallet UUID,
    total_amount DECIMAL(30,10),
    transaction_count INTEGER DEFAULT 0,
    completed_transactions INTEGER DEFAULT 0,
    configuration JSONB,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    started_at TIMESTAMPTZ,
    completed_at TIMESTAMPTZ
);

CREATE INDEX idx_liquidity_workflows_status ON liquidity_workflows(status);
CREATE INDEX idx_liquidity_workflows_type ON liquidity_workflows(type);
CREATE INDEX idx_liquidity_workflows_created ON liquidity_workflows(created_at);
```

### Exchange

Integrated cryptocurrency exchanges.

```json
// MongoDB Collection: exchanges
{
    "_id": "exchange-identifier",
    "name": "Binance",
    "supportedAssets": ["BTC", "ETH", "USDT", "USDC"],
    "apiCapabilities": ["deposit", "withdrawal", "trading", "websocket"],
    "jurisdictions": ["US", "UK", "SG", "HK"],  // Permitted jurisdictions
    "apiConfiguration": {
        "baseUrl": "https://api.binance.com",
        "apiKeyRequired": true,
        "rateLimits": {
            "requestsPerSecond": 10,
            "requestsPerMinute": 1200
        }
    },
    "status": "active",
    "addedAt": "2025-01-10T00:00:00Z"
}
```

### Jurisdiction Configuration

Configuration for exchange jurisdictional compliance.

```json
// MongoDB Collection: jurisdiction_configs
{
    "_id": "jurisdiction-code",  // e.g., "US", "SG"
    "jurisdiction": "US",
    "jurisdictionName": "United States",
    "permittedExchanges": [
        {
            "exchangeId": "coinbase",
            "restrictions": ["KYC required", "volume limits apply"]
        },
        {
            "exchangeId": "kraken",
            "restrictions": []
        }
    ],
    "prohibitedExchanges": ["binance"],
    "complianceNotes": "Additional state-level restrictions may apply",
    "updatedAt": "2025-10-15T10:00:00Z",
    "updatedBy": "compliance-team-uuid"
}
```

---

## Cross-Module Relationships

### Client to Wallet Mapping

Links CRIP clients to TDAMS wallets.

```sql
CREATE TABLE client_wallets (
    client_id UUID NOT NULL REFERENCES clients(client_id) ON DELETE CASCADE,
    wallet_id UUID NOT NULL REFERENCES wallets(wallet_id) ON DELETE CASCADE,
    wallet_purpose VARCHAR(50),  -- e.g., "deposit", "trading", "cold_storage"
    assigned_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    PRIMARY KEY (client_id, wallet_id)
);

CREATE INDEX idx_client_wallets_client ON client_wallets(client_id);
CREATE INDEX idx_client_wallets_wallet ON client_wallets(wallet_id);
```

## Data Retention Policies

- **Messages**: 7 years (compliance requirement)
- **Transaction History**: Indefinite (blockchain immutability)
- **Portfolio Adjustments**: Indefinite (audit trail)
- **Device Logs**: 90 days
- **Campaign Metrics**: 2 years
- **Performance Data**: 10 years (compressed after 2 years)

## Encryption Strategy

- **At Rest**: AES-256 encryption for all databases
- **In Transit**: TLS 1.3 for all API communications
- **Field-Level**: Additional encryption for sensitive fields:
  - Persona credentials
  - Message content
  - Private keys (hardware security module)
  - Client PII

## Backup Strategy

- **Continuous**: Write-ahead log (WAL) archiving for PostgreSQL
- **Hourly**: Incremental backups
- **Daily**: Full backups retained for 30 days
- **Weekly**: Full backups retained for 1 year
- **Monthly**: Full backups retained for 7 years (compliance)

## Compliance Considerations

All audit-critical tables include:
- Immutability constraints where appropriate
- Row-level security policies
- Comprehensive indexing for audit queries
- Timestamp tracking (created_at, updated_at)
- User attribution for all modifications
