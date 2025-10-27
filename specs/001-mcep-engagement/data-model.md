# Data Model: Multi-Channel Client Engagement Platform (MCEP)

**Feature**: 001-mcep-engagement
**Date**: 2025-10-27
**Source**: Extracted from spec.md Key Entities section

## Entity Schemas

### Portfolio Manager

**Description**: Platform user managing client outreach campaigns

**Attributes**:
- `id` (UUID, primary key)
- `email` (string, unique, required) - Authentication identifier
- `name` (string, required)
- `role` (enum: `standard`, `admin`, required)
- `lead_score_threshold` (integer, 50-90, required) - Configurable qualification threshold
- `created_at` (timestamp, required)
- `updated_at` (timestamp, required)
- `last_login` (timestamp, nullable)

**Relationships**:
- Has many `Engagement Profiles`
- Has many `Campaigns`

**Validation Rules**:
- `email` must be valid format
- `lead_score_threshold` must be between 50 and 90 (from clarification #4)
- Default `role` is `standard`

**Indexes**:
- Primary: `id`
- Unique: `email`
- Search: `created_at` (for reporting)

---

### Engagement Profile

**Description**: Personalized configuration for market segment

**Attributes**:
- `id` (UUID, primary key)
- `portfolio_manager_id` (UUID, foreign key, required)
- `name` (string, required) - Profile identifier
- `language` (string, ISO 639-1, required) - e.g., "en", "es", "zh"
- `branding_config` (JSONB, required) - Visual branding settings
- `messaging_templates` (JSONB, required) - Platform-specific message templates
- `compliance_rules` (JSONB, required) - Regional compliance configuration
- `target_demographics` (JSONB, nullable) - Segmentation criteria
- `is_active` (boolean, required, default true)
- `created_at` (timestamp, required)
- `updated_at` (timestamp, required)

**Relationships**:
- Belongs to one `Portfolio Manager`
- Has many `Campaigns` (many-to-many through campaign_profiles)

**Validation Rules**:
- Portfolio Manager can have maximum 50 profiles (FR-005)
- `language` must be valid ISO 639-1 code
- `branding_config` must include: colors, logos, fonts
- `messaging_templates` must have templates for each supported platform

**Indexes**:
- Primary: `id`
- Foreign key: `portfolio_manager_id`
- Composite: `(portfolio_manager_id, is_active)` for active profile queries

---

### Campaign

**Description**: Coordinated outreach initiative targeting specific client segments

**Attributes**:
- `id` (UUID, primary key)
- `portfolio_manager_id` (UUID, foreign key, required)
- `name` (string, required)
- `target_segment` (JSONB, required) - Segmentation criteria
- `message_content` (text, required)
- `status` (enum: `draft`, `scheduled`, `active`, `paused`, `completed`, required)
- `launch_time` (timestamp, nullable) - Scheduled start time
- `total_recipients` (integer, required) - Campaign size
- `messages_sent` (integer, default 0)
- `messages_delivered` (integer, default 0)
- `messages_failed` (integer, default 0)
- `created_at` (timestamp, required)
- `updated_at` (timestamp, required)

**Relationships**:
- Belongs to one `Portfolio Manager`
- Has many `Engagement Profiles` (many-to-many)
- Has many `Messages`

**Validation Rules**:
- SC-001: Must support 1,000+ recipients (no upper limit enforced)
- `status` transitions: draft → scheduled → active → (paused ↔ active) → completed
- Cannot modify `message_content` after status is `active`

**Indexes**:
- Primary: `id`
- Foreign key: `portfolio_manager_id`
- Composite: `(portfolio_manager_id, status)` for PM dashboard
- Search: `launch_time` for scheduled campaigns

---

### Prospect

**Description**: Potential client receiving outreach communications

**Attributes**:
- `id` (UUID, primary key)
- `phone_number` (string, unique, required) - Primary contact identifier
- `email` (string, nullable)
- `name` (string, nullable)
- `demographics` (JSONB, nullable) - Age, location, interests
- `asset_class_interests` (JSONB, nullable) - Investment preferences
- `engagement_history` (JSONB, required) - Interaction timestamps and types
- `lead_score` (integer, 0-100, default 0) - ML-generated score
- `last_engagement` (timestamp, nullable)
- `created_at` (timestamp, required)
- `updated_at` (timestamp, required)

**Relationships**:
- Has many `Messages`
- Associated with `Lead Score` calculations

**Validation Rules**:
- `phone_number` must be E.164 format (e.g., +1234567890)
- `lead_score` must be between 0 and 100
- `engagement_history` structure: `[{timestamp, platform, action_type, campaign_id}]`

**Indexes**:
- Primary: `id`
- Unique: `phone_number`
- Search: `lead_score DESC` for qualification queries
- Search: `last_engagement` for engagement recency

---

### Message

**Description**: Individual communication sent through messaging platform

**Attributes**:
- `id` (UUID, primary key)
- `campaign_id` (UUID, foreign key, required)
- `prospect_id` (UUID, foreign key, required)
- `engagement_profile_id` (UUID, foreign key, required)
- `device_id` (UUID, foreign key, required)
- `platform` (enum: `whatsapp`, `telegram`, `signal`, `imessage`, required)
- `content` (text, required)
- `delivery_status` (enum: `pending`, `sent`, `delivered`, `failed`, `retrying`, required)
- `retry_count` (integer, default 0) - Exponential backoff counter
- `next_retry_at` (timestamp, nullable) - Scheduled retry time
- `sent_at` (timestamp, nullable)
- `delivered_at` (timestamp, nullable)
- `read_at` (timestamp, nullable) - If platform supports read receipts
- `failed_reason` (text, nullable)
- `compliance_flags` (JSONB, nullable) - Flagged violations
- `created_at` (timestamp, required)

**Relationships**:
- Belongs to one `Campaign`
- Belongs to one `Prospect`
- Belongs to one `Engagement Profile`
- Belongs to one `Device`

**Validation Rules**:
- FR-016: Maximum 3 retries with exponential backoff (1min, 5min, 15min)
- `retry_count` must not exceed 3
- `delivery_status` transitions: pending → sent → delivered (or failed)
- `delivery_status` can transition to `retrying` up to 3 times

**Indexes**:
- Primary: `id`
- Foreign keys: `campaign_id`, `prospect_id`, `engagement_profile_id`, `device_id`
- Composite: `(delivery_status, next_retry_at)` for retry processing
- Search: `sent_at` for analytics (TimescaleDB hypertable)

---

### Device

**Description**: Physical mobile device managed by the platform

**Attributes**:
- `id` (UUID, primary key)
- `device_identifier` (string, unique, required) - IMEI or serial number
- `platform_type` (enum: `ios`, `android`, required)
- `supported_platforms` (array of enum, required) - e.g., [`whatsapp`, `telegram`]
- `health_status` (enum: `healthy`, `degraded`, `offline`, required)
- `last_health_check` (timestamp, required)
- `current_allocation` (UUID, nullable) - Campaign ID if allocated
- `messages_sent_today` (integer, default 0) - Rate limiting counter
- `rate_limit_reset_at` (timestamp, required) - Daily reset time
- `created_at` (timestamp, required)
- `updated_at` (timestamp, required)

**Relationships**:
- Has many `Messages` (historical)
- Currently allocated to zero or one `Campaign`

**Validation Rules**:
- FR-003: Total device count between 1,500-2,000 (system-wide constraint)
- `messages_sent_today` must respect platform rate limits:
  - WhatsApp: 1,000 messages/day
  - Telegram: Calculated based on 30 msg/s = ~2.6M/day (no daily limit)
  - Signal/iMessage: Conservative limit of 500/day (less documented)
- `health_status` transitions: healthy ↔ degraded ↔ offline → healthy

**Indexes**:
- Primary: `id`
- Unique: `device_identifier`
- Composite: `(health_status, current_allocation)` for device pool queries
- Search: `messages_sent_today` for rate limit checking

---

### Lead Score (Computed/Cached)

**Description**: Calculated value representing prospect qualification level

**Attributes**:
- `prospect_id` (UUID, primary key, foreign key)
- `score` (integer, 0-100, required)
- `score_factors` (JSONB, required) - Feature contributions (explainability)
- `model_version` (string, required) - ML model version used
- `calculated_at` (timestamp, required)
- `last_engagement_weight` (float, nullable) - Feature: engagement recency
- `response_quality_weight` (float, nullable) - Feature: response quality
- `frequency_weight` (float, nullable) - Feature: engagement frequency

**Relationships**:
- Belongs to one `Prospect` (one-to-one)

**Validation Rules**:
- FR-009: Score must be 0-100 scale
- Updated in real-time as prospect engages
- `score_factors` provides ML explainability for Portfolio Managers

**Indexes**:
- Primary: `prospect_id`
- Search: `calculated_at` for model drift monitoring

---

## Relationships Diagram

```
Portfolio Manager (1) ──< (M) Engagement Profile
      │
      │ (1)
      │
      ▼ (M)
   Campaign ──< (M) Message ──> (1) Prospect ──(1-1)── Lead Score
      │                │
      │ (M)            │ (1)
      │                ▼
      └──< (M) ────> Device (shared pool)
```

## Database Implementation Notes

### PostgreSQL Tables

**Primary Database**: PostgreSQL 15+ with JSONB support

**Table Partitioning**:
- `messages` table partitioned by `created_at` (monthly partitions) for performance
- TimescaleDB hypertable on `messages` for time-series analytics

**JSONB Columns**:
- `engagement_profile.branding_config`: `{colors: {...}, logos: {...}, fonts: {...}}`
- `engagement_profile.messaging_templates`: `{whatsapp: "...", telegram: "...", signal: "...", imessage: "..."}`
- `engagement_profile.compliance_rules`: `{jurisdiction: "...", rules: [...]}`
- `campaign.target_segment`: `{age_range: [...], location: [...], interests: [...]}`
- `prospect.demographics`: `{age: 35, location: "New York", interests: ["crypto", "real estate"]}`
- `prospect.engagement_history`: `[{timestamp, platform, action, campaign_id}]`
- `message.compliance_flags`: `[{rule_id, severity, message}]`
- `lead_score.score_factors`: `{engagement_frequency: 0.3, response_quality: 0.5, recency: 0.2}`

### TimescaleDB Hypertables

**Analytics Tables** (optimized for time-range queries):
- `message_metrics` - Aggregated delivery/read/response metrics
- `campaign_performance` - Real-time campaign statistics
- `device_health_history` - Device status over time

### Redis Caching

**Cached Data**:
- Device pool availability (key: `device:pool:available`, value: sorted set by health score)
- Portfolio Manager session data (key: `session:{pm_id}`, TTL: 24h)
- Recent lead scores (key: `lead_score:{prospect_id}`, TTL: 1h)
- Rate limit counters (key: `rate_limit:{device_id}:{date}`, TTL: 24h)

## Migration Strategy

**Phase 1 Migrations**:
1. Create core tables: `portfolio_managers`, `engagement_profiles`, `campaigns`, `prospects`, `devices`
2. Create `messages` table with TimescaleDB hypertable
3. Create `lead_scores` computed table
4. Add indexes for performance
5. Create partitions for `messages` table (6 months initial range)

**Data Seeding** (for development/demo):
- 10 Portfolio Managers with varied thresholds (50, 60, 70, 80, 90)
- 50 Engagement Profiles across different languages/segments
- 100 mock Devices (50 iOS, 50 Android)
- 1,000 Prospects with varied lead scores
- Sample campaigns with different statuses

---

**Data Model Version**: 1.0.0 | **Generated**: 2025-10-27 | **Source**: spec.md entities + clarifications
