# OPPORTUNITY FINDER

## Product Requirements Document

**Version 3.0 - Phase 1: Single-User MVP (Updated with Brainstorming Decisions)**

**19 January 2026**

| Field | Value |
|-------|-------|
| Status | READY FOR IMPLEMENTATION |
| Owner | Mark |
| Target Delivery | As soon as possible via AI |
| Phase | Phase 1: Single-user MVP for product owner. Phase 2: Multi-user SaaS production model. |

---

## EXECUTIVE SUMMARY

### Product Vision

Opportunity Finder is a systematic validation platform for micro-SaaS entrepreneurs delivered as a subscription SaaS product with **admin-configurable pricing**. It eliminates guesswork by collecting real market demand signals from multiple sources, validating them against existing solutions and revenue data, and producing scored opportunities with clear build/no-build recommendations. Built from day one as a multi-tenant, payment-enabled platform with mobile-ready architecture. Pricing is set by the admin through the Admin Panel and can be changed at any time without code deployment.

### What's New in Version 3.0

This version incorporates architectural decisions made during the brainstorming session (January 19, 2026):

- **Unified Data Schema**: All data sources normalize to `OpportunityMention` schema
- **Data Source Interface Contract**: Every source implements `DataSourceInterface`
- **100+ Configurable Parameters**: Complete morphological matrix for system tuning
- **MVP Scope Clarified**: 6 data sources for MVP (Reddit, Indie Hackers, Product Hunt, Hacker News, Google Trends, Competition Search)
- **Configuration Validation**: Complete validation rules and dependency mapping
- **Configuration Presets**: 3 pre-built configurations for common use cases
- **Cross-Source Correlation**: Engine identifies patterns across data sources
- **Source-Specific AI Prompts**: Tailored analysis for each data source type

### Problem Statement

Indie developers and entrepreneurs waste months building products nobody wants because:

- They guess at problems rather than finding validated demand
- Manual research across multiple platforms is time-consuming and inconsistent
- No systematic way to validate if people will actually pay
- Opportunity assessment is subjective and emotionally driven

### Solution

Automated system that monitors Reddit, Indie Hackers, Product Hunt, Hacker News, Google Trends, and other sources for pain points, validates them against existing solutions and revenue data, and produces scored opportunities 0-100 based on 7 weighted components (demand, revenue, competition, feasibility, timing, risk), with clear build/no-build recommendations.

### Success Metrics

| Priority | Metric |
|----------|--------|
| Primary | System accurately scores opportunities based on defined criteria |
| Secondary | Product owner identifies 1+ buildable opportunity per week |
| Tertiary | System collects and validates data from all 6 MVP sources reliably |

---

## 1. PHASE SCOPE DEFINITION

### Phase 1: Single-User MVP (Current Document)
- **Purpose:** Validate product concept with product owner (Mark) as sole user
- **Scope:** Full system for single user, all features functional
- **Delivery:** Complete working system running on local/VPS
- **Data Model:** Global scan runs, tier-based visibility (single tier for product owner)
- **Post-Phase 1:** Decision to proceed to Phase 2 based on system effectiveness

### Phase 2: Multi-User SaaS (Future)
- **Purpose:** Offer as subscription service to external users
- **Scope:** Add multi-user onboarding, trial flows, full production support
- **Delivery:** Separate production planning phase
- **Decision:** Triggered by successful Phase 1 validation

**This PRD documents Phase 1 requirements. Phase 2 will require separate planning based on Phase 1 outcomes.**

---

## 2. PRODUCT OVERVIEW

### 2.1 Target User

**Primary:**
- Solo indie developers with React/Python skills building first SaaS
- Former employees with 3-6 month runway seeking validated opportunities
- Willing to pay the configured subscription price for systematic validation vs months of guessing
- Need to identify £1k+ MRR opportunities quickly before runway expires

### 2.2 Use Cases

| Use Case | Description |
|----------|-------------|
| Weekly Opportunity Discovery | User runs weekly scan, reviews top 10 opportunities, identifies 1-3 to investigate further |
| Idea Validation | User has idea, searches system to see if demand exists and validates score |
| Market Research | User explores specific niche (e.g., 'freelancer tools') to find gaps |

### 2.3 Core Value Proposition

> "Stop guessing. Build what people are actively looking for and willing to pay for."

---

## 3. TECHNICAL ARCHITECTURE

### 3.1 System Components

**Backend (Python/Flask)**
- **Authentication:** JWT-based auth with secure session management
- **Payment Processing:** Stripe integration for subscriptions (price configured by admin via Admin Panel)
- **Data Collectors:** Modular data sources implementing `DataSourceInterface`
- **Validation Engine:** Checks for existing solutions and revenue
- **Scoring Engine:** Calculates 0-100 score based on 7 weighted components
- **Correlation Engine:** Cross-source pattern identification
- **REST API:** Exposes endpoints for frontend (with auth middleware)
- **Database:** PostgreSQL with multi-tenant architecture
- **Caching:** Redis for session storage, API response caching, and rate limiting
- **Background Jobs:** Celery with Redis broker for async scan jobs, email sending, PDF generation

**Frontend (React)**
- **Authentication:** Login/signup/password reset
- **Subscription Management:** Stripe checkout integration, billing portal
- **Dashboard:** Overview stats and top opportunities
- **Opportunity Cards:** Filterable, sortable grid view
- **Detail Modal:** Full opportunity breakdown with sources
- **Search & Filters:** Real-time filtering by score, keyword, time range
- **Theme Toggle:** Dark mode (default) with light mode option

---

### 3.1.1 Unified Data Schema

**Every data source normalizes output to the `OpportunityMention` schema:**

```python
from pydantic import BaseModel
from typing import Dict, Any, List, Optional
from datetime import datetime

class OpportunityMention(BaseModel):
    """Unified schema for all data source outputs.

    Reference: Configuration_Reference.md - Unified Data Schema Reference
    """

    # ========== Identity ==========
    mention_id: str                          # Unique identifier (UUID)
    source_type: str                         # "reddit", "google_trends", "indiehackers", etc.
    source_url: str                          # Direct link to original
    raw_data: Dict[str, Any]                 # Original source data (preserve for debugging)

    # ========== Content ==========
    title: str                               # Headline/subject
    description: str                         # Main content/body
    extracted_entities: List[str]            # NLP extracted: tools, platforms, categories

    # ========== Engagement (Normalized 0-100 across all sources) ==========
    engagement_score: float                  # Combined upvotes/views/likes/etc.
    engagement_breakdown: Dict[str, int]     # Source-specific metrics preserved

    # ========== Timing ==========
    published_at: datetime                   # When original was posted
    collected_at: datetime                   # When we scraped it
    recency_score: float                     # 0-100 based on age (newer = higher)

    # ========== Classification ==========
    category: str                            # "SaaS", "E-commerce", "DevTool", etc.
    business_type: str                       # "B2B", "B2C", "Both", "Unclear"
    maturity_level: str                      # "Emerging", "Growing", "Established"

    # ========== Confidence ==========
    confidence_score: float                  # 0-100 (how reliable is this data?)

    # ========== Source-Specific Signals (extensible map) ==========
    signals: Dict[str, Any]                  # Any source-specific metadata

    # ========== Cross-Reference ==========
    correlated_sources: List[str]            # Other sources mentioning same opportunity
    correlation_strength: float              # 0-1 (how strong is the correlation?)

    class Config:
        json_schema_extra = {
            "example": {
                "mention_id": "550e8400-e29b-41d4-a716-446655440000",
                "source_type": "reddit",
                "source_url": "https://reddit.com/r/SaaS/comments/abc123",
                "title": "Looking for a better CRM export tool",
                "description": "I hate how I can't easily export my contacts...",
                "extracted_entities": ["CRM", "export", "contacts"],
                "engagement_score": 72.5,
                "engagement_breakdown": {"upvotes": 150, "comments": 45},
                "published_at": "2026-01-15T10:30:00Z",
                "collected_at": "2026-01-19T14:20:00Z",
                "recency_score": 85.0,
                "category": "DevTool",
                "business_type": "B2B",
                "maturity_level": "Emerging",
                "confidence_score": 82.0,
                "signals": {"subreddit": "SaaS"},
                "correlated_sources": ["google_trends"],
                "correlation_strength": 0.65
            }
        }
```

**Database Schema:**

```sql
CREATE TABLE opportunity_mentions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    -- Identity
    mention_id VARCHAR(255) UNIQUE NOT NULL,
    source_type VARCHAR(50) NOT NULL,
    source_url TEXT NOT NULL,
    raw_data JSONB,

    -- Content
    title TEXT NOT NULL,
    description TEXT,
    extracted_entities TEXT[] NOT NULL DEFAULT '{}',

    -- Engagement
    engagement_score FLOAT NOT NULL,
    engagement_breakdown JSONB,

    -- Timing
    published_at TIMESTAMP WITH TIME ZONE NOT NULL,
    collected_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    recency_score FLOAT NOT NULL,

    -- Classification
    category VARCHAR(100),
    business_type VARCHAR(20),
    maturity_level VARCHAR(20),

    -- Confidence
    confidence_score FLOAT NOT NULL,

    -- Signals
    signals JSONB,

    -- Cross-Reference
    correlated_sources TEXT[] NOT NULL DEFAULT '{}',
    correlation_strength FLOAT DEFAULT 0.0,

    -- Metadata
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),

    -- Indexes
    INDEX idx_mentions_source_type (source_type),
    INDEX idx_mentions_published_at (published_at DESC),
    INDEX idx_mentions_engagement (engagement_score DESC),
    INDEX idx_mentions_business_type (business_type),
    INDEX idx_mentions_category (category),
    INDEX idx_mentions_confidence (confidence_score)
);
```

---

### 3.1.2 Data Source Interface Contract

**Every data source MUST implement this interface:**

```python
from abc import ABC, abstractmethod
from typing import Dict, Any, List, Optional, Tuple
from datetime import datetime

class DataSourceInterface(ABC):
    """Contract that all data sources must implement.

    Reference: Configuration_Reference.md - Data Source Interface Contract
    """

    @property
    @abstractmethod
    def source_name(self) -> str:
        """Unique identifier for this source.

        Returns:
            str: Source identifier (e.g., "reddit", "google_trends")
        """
        pass

    @property
    @abstractmethod
    def source_type(self) -> str:
        """Type classification for this source.

        Returns:
            str: One of "discussion", "search", "marketplace", "launch", "qa"
        """
        pass

    @abstractmethod
    async def fetch_raw_data(
        self,
        query: str,
        filters: Dict[str, Any],
        date_range: Optional[Tuple[datetime, datetime]]
    ) -> List[Dict[str, Any]]:
        """Fetch raw data from the source API/web scrape.

        Args:
            query: Search term/topic
            filters: Source-specific filters (subreddit, region, etc.)
            date_range: Optional (start_date, end_date) tuple

        Returns:
            List[Dict]: Raw data dictionaries in source-specific format

        Raises:
            DataSourceError: If fetch fails
        """
        pass

    @abstractmethod
    def normalize_to_unified_schema(
        self,
        raw_data: Dict[str, Any]
    ) -> OpportunityMention:
        """Convert source-specific raw data to Unified Schema.

        Args:
            raw_data: Single item from fetch_raw_data()

        Returns:
            OpportunityMention: Normalized opportunity mention

        Raises:
            NormalizationError: If data cannot be normalized
        """
        pass

    @abstractmethod
    def extract_signals(
        self,
        raw_data: Dict[str, Any]
    ) -> Dict[str, Any]:
        """Extract source-specific signals/metrics.

        Args:
            raw_data: Single item from fetch_raw_data()

        Returns:
            Dict: Signals dictionary for the OpportunityMention.signals field
        """
        pass

    @abstractmethod
    def get_ai_prompt_template(self) -> str:
        """Return the AI prompt template for analyzing this source.

        The prompt should instruct AI on:
        - What to look for in this source type
        - How to classify (B2B vs B2C, category, etc.)
        - What signals indicate opportunity vs noise
        - How to score components (demand, competition, etc.)

        Returns:
            str: Prompt template string with placeholders:
                 - {content}: Main content to analyze
                 - {metadata}: Source-specific metadata dict
        """
        pass

    @abstractmethod
    def calculate_confidence_score(
        self,
        raw_data: Dict[str, Any],
        normalized: OpportunityMention
    ) -> float:
        """Calculate reliability score for this data point.

        Args:
            raw_data: Original source data
            normalized: Normalized OpportunityMention

        Returns:
            float: Confidence score 0-100

        Factors:
        - Data completeness
        - Source reliability
        - Engagement signals
        - Account/author reputation (if applicable)
        """
        pass

    @abstractmethod
    def get_rate_limit_info(self) -> Dict[str, Any]:
        """Return rate limit information for this source.

        Returns:
            Dict with keys:
                - requests_per_minute (int): Max requests per minute
                - requests_per_day (int): Max requests per day
                - requires_api_key (bool): Whether API key is needed
                - cost_per_request (Optional[float]): Cost per request if paid
                - retry_after_seconds (Optional[int]): Retry delay if rate limited
        """
        pass


class DataSourceError(Exception):
    """Base exception for data source errors."""
    pass


class FetchError(DataSourceError):
    """Raised when data fetch fails."""
    pass


class NormalizationError(DataSourceError):
    """Raised when data normalization fails."""
    pass


class RateLimitError(DataSourceError):
    """Raised when rate limit is exceeded."""
    def __init__(self, message: str, retry_after: int):
        super().__init__(message)
        self.retry_after = retry_after
```

**Implementation Requirements:**

1. All data sources must implement all 6 methods of `DataSourceInterface`
2. Each source returns `OpportunityMention` objects from `normalize_to_unified_schema()`
3. AI analysis uses source-specific prompts from `get_ai_prompt_template()`
4. Rate limit info returned by `get_rate_limit_info()` used for throttling
5. Confidence scores filter out low-quality data before correlation

---

### 3.2 Technology Stack

| Component | Technology | Version |
|-----------|------------|---------|
| Backend | Python + Flask | 3.11+ / 3.0+ |
| Frontend | React + Vite + TypeScript | 18+ / 5+ |
| Database | PostgreSQL | 15+ |
| ORM | SQLAlchemy | 2.0+ |
| Authentication | Flask-JWT-Extended | 4.6+ |
| Payments | Stripe Python SDK | 8.0+ |
| Email | SendGrid / Mailgun | Latest |
| PDF Generation | ReportLab | 4.0+ |
| Reddit | PRAW | 7.7+ |
| Web Scraping | BeautifulSoup4 | 4.12+ |
| API Calls | Requests | 2.31+ |
| Search | SerpAPI | Latest |
| CORS | Flask-CORS | 4.0+ |
| Password Hashing | bcrypt | 4.1+ |
| Caching | Redis | 7.0+ |
| Background Jobs | Celery | 5.3+ |
| Rate Limiting | Flask-Limiter | 3.5+ |

---

### 3.3 Deployment Architecture

| Environment | Configuration |
|-------------|---------------|
| Development | Local (localhost:3000 frontend, localhost:5000 backend) |
| Production | VPS deployment (DigitalOcean/Hetzner) |
| Frontend | Nginx serving React build |
| Backend | Gunicorn + Flask behind Nginx reverse proxy |
| Redis | Local instance for caching, sessions, job queue |
| Mobile Ready | API-first design allows future React Native/Flutter app using same backend |

---

### 3.4 Quality Architecture Requirements

#### Security Architecture

1. **JWT Implementation**: Short-lived access tokens (15min) + refresh tokens with Redis revocation capability
2. **Webhook Security**: Stripe signature verification on every request + idempotency table
3. **Data Isolation**: Row-level security at database level, not just application level
4. **Rate Limiting**: Per-user limits, not global (prevents DoS from single abusive user)

#### Performance Architecture

1. **Caching Strategy**: Redis cache opportunities by user_id + filter combination, invalidate on new scan
2. **Query Optimization**: Composite index `(user_id, score DESC)` for hot path queries
3. **Pagination**: Cursor-based using `id`, not offset (critical for scale)
4. **Collector Optimization**: Pre-calculate scores during collection, not on read

#### Operations Architecture

1. **Structured Logging**: JSON format with `timestamp, level, user_id (or 'anonymous'), request_id, event_type`
2. **Error Categorization**: User errors (400s) at INFO level, system errors (500s) at ERROR with full context
3. **Job Visibility**: Scan jobs track progress percentage, opportunities found, source-by-source completion
4. **Health Endpoints**: `/health` for load balancer, `/health/detailed` for admin with DB/Redis/collector status

#### Data Architecture

1. **Transaction Boundaries**: Scoring calculation atomic - calculate, validate, insert in single transaction
2. **Index Strategy**:
   - `(user_id, score DESC)` - main query
   - `(user_id, status)` - filtering
   - `(published_at DESC)` - time filtering
   - GIN tsvector for search
3. **Audit Integrity**: `old_values`/`new_values` JSONB captures full record snapshot, not partial
4. **Webhook Idempotency**: `webhook_events` table primary key on `event_id`, check before insert

---

### 3.5 Architectural Decisions

| Category | Decision | Rationale |
|----------|----------|-----------|
| **Project Structure** | Manual Flask + React Vite | Flask provides flexibility for multi-tenant architecture; no single template handles both appropriately |
| **Authentication** | Redis-based refresh tokens | PRD requires Redis revocation capability; supports immediate account suspension for God-mode admin |
| **Data Collection** | Unified schema + interface contract | Extensible architecture supports unlimited sources without code changes |
| **Scoring Algorithm** | Admin-configurable 7-component system | Essential for testing different scoring approaches; Configuration_Reference.md defines 100+ parameters |
| **API Design** | Cursor-based pagination | Required for scale; industry standard for large datasets |
| **Background Jobs** | Celery + Redis broker | Industry standard for Flask; Redis already required for caching |
| **Admin Panel** | Full God-mode with audit trail | Required for user impersonation, emergency overrides, and production incident handling |
| **Deployment** | Direct VPS (DigitalOcean/Hetzner) | Phase 1 single-user MVP - containers add complexity; Docker can be Phase 2 consideration |
| **Configuration** | application_config table | All 100+ parameters stored in database, fully admin-configurable |

---

## 4. DATA SOURCES

> **MVP Sources (6):** Reddit, Indie Hackers, Product Hunt, Hacker News, Google Trends, Competition Search
> **Phase 2 Sources (Additional):** YouTube, Gumroad/Etsy, Stack Overflow, Zapier/Make/n8n
> **Scan Architecture:** Global scan runs on schedule set by admin. All users share same opportunity pool. User's subscription tier determines which opportunities they can view (not unique per-user data).
> **Scan Schedule:** Admin configures scan frequency via Admin Panel. Default: Daily scans, potentially running twice daily. Scan progress displayed to users but not user-triggered (system-decided execution).

### 4.1 MVP Data Sources (6 Sources)

#### 4.1.1 Reddit

| Field | Value |
|-------|-------|
| **Priority** | P0 (MVP) |
| **Purpose** | Primary pain point discovery from diverse communities |
| **Implementation** | PRAW (Python Reddit API Wrapper) |
| **Source Type** | Discussion |
| **Rate Limits** | 60 requests/minute. Implement exponential backoff. |

**Configuration Parameters:**
- `reddit_subreddits`: ["SaaS", "Entrepreneur", "SideProject", "freelance", "smallbusiness"]
- `reddit_min_upvotes`: 10
- `reddit_min_comments`: 5
- `reddit_post_age_max_days`: 365
- `reddit_sort_by`: "hot"

**Search Keywords:**
- "looking for a tool"
- "need software for"
- "wish there was"
- "hate that I have to"
- "tired of manually"
- "paying too much for"

**What Changed in v3.0:**
- Now implements `DataSourceInterface` contract
- Returns `OpportunityMention` objects with normalized schema
- Uses source-specific AI prompt for B2B/B2C classification
- Configurable parameters stored in `application_config` table

---

#### 4.1.2 Indie Hackers

| Field | Value |
|-------|-------|
| **Priority** | P0 (MVP) |
| **Purpose** | Founder community discussions, revenue validation |
| **Implementation** | BeautifulSoup4 web scraping |
| **Source Type** | Discussion/Marketplace |
| **Update Frequency** | Weekly scan of new products |

**Data to Collect:**
- Product names and descriptions
- MRR/ARR figures
- Founder interviews (pain points)
- Discussion threads

**What Changed in v3.0:**
- Now implements `DataSourceInterface` contract
- Returns `OpportunityMention` objects with normalized schema
- Uses source-specific AI prompt for revenue data extraction

---

#### 4.1.3 Product Hunt

| Field | Value |
|-------|-------|
| **Priority** | P0 (MVP) |
| **Purpose** | Product launches, validated ideas, early traction signals |
| **Implementation** | GraphQL API (free tier) |
| **Source Type** | Launch |

**Data to Collect:**
- Daily top launches
- Comment threads (pain points)
- Upvote counts (demand signal)

**What Changed in v3.0:**
- Now implements `DataSourceInterface` contract
- Returns `OpportunityMention` objects with normalized schema
- Uses source-specific AI prompt for launch validation analysis

---

#### 4.1.4 Hacker News

| Field | Value |
|-------|-------|
| **Priority** | P0 (MVP) |
| **Purpose** | Tech-focused discussions, developer pain points |
| **Implementation** | Algolia HN Search API (free) |
| **Source Type** | Discussion/QA |

**Query Patterns:**
- "Ask HN: What tool"
- "Ask HN: How do you"
- "Show HN" (for validation)

**What Changed in v3.0:**
- Now implements `DataSourceInterface` contract
- Returns `OpportunityMention` objects with normalized schema
- Uses source-specific AI prompt for developer pain point extraction

---

#### 4.1.5 Google Trends

| Field | Value |
|-------|-------|
| **Priority** | P0 (MVP) |
| **Purpose** | Demand velocity + volume validation |
| **Implementation** | Paid API |
| **Source Type** | Search |

**Configuration Parameters:**
- `trends_timeframe`: "today 12-m"
- `trends_category`: 0 (all categories)
- `trends_geocode`: "" (worldwide)
- `trends_property`: "" (web search)

**What Changed in v3.0:**
- Now implements `DataSourceInterface` contract
- Returns `OpportunityMention` objects with normalized schema
- Uses source-specific AI prompt for commercial intent assessment
- **NEW:** Calculates velocity scores and commercial intent

---

#### 4.1.6 Competition Search

| Field | Value |
|-------|-------|
| **Priority** | P0 (MVP) |
| **Purpose** | Basic saturation check |
| **Implementation** | SerpAPI (100 free searches/month) |
| **Source Type** | Search |
| **Search Pattern** | "[problem] software", "[problem] tool", "[problem] SaaS" |

**Use:** Validate existing solutions, count competitors

**What Changed in v3.0:**
- Now implements `DataSourceInterface` contract
- Returns `OpportunityMention` objects with normalized schema
- Renamed from "Google Search" to "Competition Search" for clarity

---

### 4.2 Phase 2 Data Sources (Deferred)

| Source | Priority | Type | Complexity | Notes |
|--------|----------|------|------------|-------|
| YouTube | P1 | Search | Medium | Tutorial gap detection |
| Gumroad/Etsy | P1 | Marketplace | High | Janky solution detection |
| Stack Overflow | P1 | QA | Low | B2B dev tool opportunities |
| Zapier/Make/n8n | P2 | Integration | Medium | Integration pain scanning |

---

### 4.3 Data Source Architecture

**Data Flow:**

```
┌─────────────────────────────────────────────────────────────────┐
│                     Admin Search Request                         │
│              (topic, category, filters, date range)              │
└────────────────────────┬────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────────┐
│                    Source Router                                 │
│           Routes query to all ENABLED data sources               │
└────────────────────────┬────────────────────────────────────────┘
                         │
         ┌───────────────┼───────────────┐
         ▼               ▼               ▼
    [Reddit]    [IndieHackers]  [ProductHunt]    [HackerNews]
    [GoogleTrends]   [Competition]                       │
         └───────────────┴───────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────────┐
│              Parallel Data Fetch                             │
│        Each source executes its fetch strategy                   │
└────────────────────────┬─────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────────┐
│               Source-Specific Normalizers                        │
│          Convert raw data to OpportunityMention schema           │
└────────────────────────┬─────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────────┐
│                Signal Extraction Layer                           │
│       Extract source-specific metrics (engagement, etc.)          │
└────────────────────────┬─────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────────┐
│            AI Analysis (Per Source)                              │
│    Each source analyzed with source-specific prompt template      │
│    Output: Component scores (demand, competition, etc.)           │
└────────────────────────┬─────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────────┐
│              Cross-Source Correlation Engine                     │
│     Merge signals, identify patterns, cross-reference            │
│     Example: "Trending on Google + Complaints on Reddit"         │
└────────────────────────┬─────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────────┐
│              Score Aggregation & Rules                           │
│      Combine component scores with admin-configured weights      │
└────────────────────────┬─────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────────┐
│              Validation & Recommendation                          │
│     Apply validation rules → generate recommendation              │
└────────────────────────┬─────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────────┐
│                Opportunity Output                                │
│        Final scored opportunities with evidence links             │
└─────────────────────────────────────────────────────────────────┘
```

---

## 5. SCORING ALGORITHM & DATA PROCESSING

### 5.1 Opportunity Detection Criteria

> **CORE FUNCTIONALITY:** These are the signals the engine looks for to identify valid opportunities.

**The 10 Detection Criteria:**

| # | Criteria | Signal Type | What to Look For | Strength |
|---|----------|-------------|------------------|----------|
| 1 | **Complaints/Pain Points** | Problem | "I hate", "frustrated with", "tired of", negative reviews, rants | High |
| 2 | **Solution Requests** | Direct Demand | "Looking for a tool", "Is there software that", "Need something to" | Very High |
| 3 | **Multi-tool Workarounds** | Gap Signal | "I use X and Y together", "I combine A + B + C", manual processes | Very High |
| 4 | **Willingness to Pay** | Revenue Signal | "I'd pay for this", "Shut up and take my money", "Would pay £X" | Very High |
| 5 | **Money Talk** | Market Signal | Budget discussions, "currently paying £X", pricing complaints | High |
| 6 | **Negative Reviews** | Improvement Opp | Poor reviews of existing tools on G2, App Store, Trustpilot | High |
| 7 | **Frequency** | Volume | Same problem mentioned by many people across sources | High |
| 8 | **Recency** | Timing | Recent complaints (last 30-90 days) vs old ones | Medium |
| 9 | **Search Demand** | Validation | Google Trends showing active searches for solutions | High |
| 10 | **Growing Trend** | Timing | Problem getting worse over time, increasing mentions | High |

**Signal Strength Combinations:**

The strongest opportunities show multiple overlapping criteria:
- **Excellent:** Complaint + Solution Request + Willingness to Pay + Search Demand
- **Strong:** Pain Point + Multi-tool Workaround + Frequency + Growing Trend
- **Good:** Negative Reviews + Search Demand + Recency

**Detection Keywords by Criteria:**

```python
DETECTION_KEYWORDS = {
    "complaints": [
        "I hate", "frustrated with", "tired of", "sick of",
        "annoying", "waste of time", "impossible to", "nightmare"
    ],
    "solution_requests": [
        "looking for a tool", "need software for", "is there an app",
        "wish there was", "anyone know of", "recommend a"
    ],
    "workarounds": [
        "I use X and Y", "combine", "workaround", "hack together",
        "manually", "spreadsheet for", "cobbled together"
    ],
    "willingness_to_pay": [
        "would pay", "I'd pay", "shut up and take my money",
        "happy to pay", "worth paying for", "would subscribe"
    ],
    "money_talk": [
        "currently paying", "costs us", "budget for", "pricing",
        "expensive", "cheaper alternative", "free alternative"
    ]
}
```

**Categorization (Like GummySearch):**

Each detected opportunity is categorized as:
- **Pain Point** - Complaints and frustrations
- **Solution Request** - Direct asks for tools
- **Money Talk** - Revenue/payment signals
- **Hot Discussion** - High engagement threads

---

### 5.2 Data Processing Pipeline

> **CORE FUNCTIONALITY:** This is how the system converts raw pain points into scored opportunities.

**Pipeline Steps:**

1. **Data Collection** (Section 4 - Data Sources)
   - Each source fetches raw data via `fetch_raw_data()`
   - Each source normalizes to `OpportunityMention` via `normalize_to_unified_schema()`
   - Each source extracts signals via `extract_signals()`

2. **Pain Point Clustering** (replaced by Correlation Engine)
   - See Section 5.1.4: Cross-Source Correlation Engine for multi-source clustering

3. **Opportunity Generation**
   - For each correlated cluster, create 1 opportunity record
   - Aggregate: total mentions, source list, source URLs
   - Track: first seen date, last seen date, mentions trend

4. **Data Enrichment**
   - Search competitors for opportunity via Competition Search source
   - Extract: competitor count, competitor URLs, example MRR data
   - Determine: build complexity (via AI assessment)

5. **AI Analysis** (Per Source)
   - Each mention analyzed with source-specific prompt template
   - Output: Component scores (demand, pain, feasibility, etc.)
   - Classification: business_type, category, maturity_level

6. **Scoring Calculation** (Section 5.2)
   - Apply 7-component weight formula
   - Store final score (0-100)

7. **Validation Check** (Section 5.4)
   - Check against validation rules (paid solution, MRR threshold, mentions, B2B)
   - Set "validated" boolean flag

8. **Recommendation Assignment** (Section 5.5)
   - Based on score + validation status
   - Store recommendation text

**Data Flow:**
```
Raw Data → Normalize to OpportunityMention → Extract Signals → AI Analysis →
Correlation → Aggregation → Scoring → Validation → Recommendation → Display
```

---

### 5.1.4 Cross-Source Correlation Engine

**Purpose:** Identifies when multiple sources are discussing the same opportunity.

**Correlation Logic:**

```python
class CorrelationEngine:
    """Identifies when multiple sources are discussing the same opportunity.

    Reference: Configuration_Reference.md - Unified Data Schema Reference
    """

    def correlate_mentions(
        self,
        mentions: List[OpportunityMention]
    ) -> Dict[str, List[OpportunityMention]]:
        """Group mentions by opportunity topic.

        Correlation signals:
        - Same extracted entities appear in multiple sources
        - Similar keywords across sources
        - Time proximity (mentions cluster around same time)

        Args:
            mentions: List of OpportunityMention objects to correlate

        Returns:
            Dictionary mapping opportunity IDs to lists of correlated mentions
        """

        # 1. Group by extracted entities
        entity_groups = self._group_by_entities(mentions)

        # 2. Within entity groups, cluster by semantic similarity
        semantic_clusters = self._semantic_cluster(entity_groups)

        # 3. Calculate correlation strength for each cluster
        for cluster_id, cluster in semantic_clusters.items():
            cluster.correlation_strength = self._calculate_correlation(cluster)

            # Update OpportunityMention objects
            for mention in cluster.mentions:
                mention.correlated_sources = [m.source_type for m in cluster.mentions if m.source_type != mention.source_type]
                mention.correlation_strength = cluster.correlation_strength

        return semantic_clusters

    def _group_by_entities(self, mentions: List[OpportunityMention]) -> Dict[str, List[OpportunityMention]]:
        """Group mentions that share extracted entities."""
        entity_groups = {}

        for mention in mentions:
            for entity in mention.extracted_entities:
                if entity not in entity_groups:
                    entity_groups[entity] = []
                entity_groups[entity].append(mention)

        return entity_groups

    def _semantic_cluster(self, entity_groups: Dict[str, List[OpportunityMention]]) -> Dict[str, Any]:
        """Cluster mentions by semantic similarity within entity groups."""
        # Implementation uses text similarity on titles/descriptions
        # Returns clusters with correlation strength calculated
        pass

    def _calculate_correlation(self, cluster) -> float:
        """Calculate correlation strength (0.0-1.0) for a cluster.

        Factors:
        - Number of different sources (more sources = higher strength)
        - Time proximity (mentions closer together = higher strength)
        - Entity overlap (more shared entities = higher strength)
        """
        source_count = len(set(m.source_type for m in cluster.mentions))
        time_spread = self._calculate_time_spread(cluster.mentions)
        entity_overlap = self._calculate_entity_overlap(cluster.mentions)

        # Weighted calculation
        correlation = (
            (source_count / len(cluster.mentions)) * 0.4 +
            (1.0 / max(time_spread, 1)) * 0.3 +
            entity_overlap * 0.3
        )

        return min(correlation, 1.0)
```

**Database Storage:**

```sql
CREATE TABLE mention_correlations (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    opportunity_id UUID NOT NULL REFERENCES opportunities(id),
    source_type VARCHAR(50) NOT NULL,
    mention_id VARCHAR(255) NOT NULL,
    correlation_strength FLOAT NOT NULL, -- 0.0 to 1.0
    correlation_method VARCHAR(100), -- 'entity_match', 'semantic_similarity', 'time_proximity'
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),

    INDEX idx_correlation_opportunity (opportunity_id),
    INDEX idx_correlation_strength (correlation_strength)
);
```

---

### 5.2 Scoring & Configuration System

> **Reference:** Configuration_Reference.md - Complete parameter reference with all 90+ parameters

**Scoring is based on 7 weighted components:**

| Component | Default Weight | Description |
|-----------|---------------|-------------|
| **Demand** | 25% | Overall demand signal (velocity + volume combined) |
| **Competition** | 20% | Market saturation (inverted scoring) |
| **Revenue** | 25% | Proof of payment willingness |
| **Feasibility** | 15% | Technical build complexity |
| **Timing** | 10% | Market timing readiness |
| **Risk** | 5% | Execution risk (inverted scoring) |

**Scoring Formula:**
```
final_score = (
    (demand_score × 0.25) +
    (competition_score_inverted × 0.20) +
    (revenue_score × 0.25) +
    (feasibility_score × 0.15) +
    (timing_score × 0.10) +
    (risk_score_inverted × 0.05)
)

Where:
- demand_score = (velocity_score × 0.6) + (volume_score × 0.4)
- competition_score_inverted = (100 - competition_score) [lower competition = higher score]
- risk_score_inverted = (100 - risk_score) [lower risk = higher score]
```

**All 90+ parameters are admin-configurable via:**

1. **Configuration_Reference.md** - Complete parameter reference
2. **application_config** table - Stores all configuration
3. **Admin Panel → Scoring & Configuration** - 12-tab UI interface

**Key Configuration Sections:**
- Data Source Config (4 parameters)
- Scoring Weights (7 parameters)
- Scoring Thresholds (7 parameters)
- Validation Rules (11 parameters)
- Business Type Filters (5 parameters)
- Category Classifications (3 parameters)
- Maturity Preferences (5 parameters)
- AI Model Configuration (7 parameters)
- Geographic Configuration (4 parameters)
- Source-Specific Parameters (26+ parameters)
- Output Configuration (7 parameters)
- Notification & Alerting (6 parameters)

---

### 5.2.1 Configuration Validation Rules

> **Reference:** Configuration_Reference.md - Configuration Validation Rules

**Critical Validation Rules:**

1. **Weight Sum Validation:**
   ```python
   weight_demand + weight_competition + weight_revenue + weight_feasibility + weight_timing + weight_risk === 1.0
   ```
   - **Error:** "Weight sum must equal 1.0. Current sum: {actual_sum}"

2. **Threshold Ordering:**
   ```python
   score_threshold_build > score_threshold_validate > score_threshold_investigate
   ```
   - **Error:** "Thresholds must be ordered: build ({build}) > validate ({validate}) > investigate ({investigate})"

3. **Business Type Mutual Exclusion:**
   ```python
   NOT (require_b2b = true AND require_b2c = true)
   ```
   - **Error:** "Cannot require both B2B and B2C exclusively"

4. **Correlation Requirements:**
   ```python
   IF require_correlation = true:
       - min_correlation_sources >= 2
       - min_correlation_strength > 0.0
       - enabled_sources.length >= min_correlation_sources
   ``   ```

5. **Source-Specific Validation:**
   ```python
   IF "reddit" IN enabled_sources:
       - reddit_subreddits must not be empty

   IF "google_trends" IN enabled_sources:
       - Must have valid API key configured

   IF ANY marketplace source IN enabled_sources:
       - marketplace_keywords must not be empty
   ```

**Implementation:**

See Configuration_Reference.md for complete validation code in `ConfigurationValidator` class.

---

### 5.2.2 Configuration Presets

> **Reference:** Configuration_Reference.md - Configuration Presets

**Three pre-built configurations for common use cases:**

#### Preset 1: "Aggressive Emerging Hunter"

**Focus:** EARLY mover, high growth, low competition

```yaml
# Maturity Settings
maturity_filter: "emerging_only"
emerging_definition_max_competitors: 5
emerging_definition_max_age_months: 6

# Velocity Thresholds
velocity_threshold_high: 3.0  # 3x growth

# Scoring Weights (heavier on demand and competition)
weight_demand: 0.35
weight_competition: 0.30  # Heavy competition weight
weight_revenue: 0.15
weight_feasibility: 0.10
weight_timing: 0.07
weight_risk: 0.03

# Thresholds (lower bar to catch early opportunities)
score_threshold_build: 65
score_threshold_validate: 50
score_threshold_investigate: 30

# Validation (willing to be first)
require_paid_solution: false
require_correlation: false  # Don't require multiple sources for early detection
```

**When to Use:** Looking for blue ocean opportunities, willing to take calculated risks, want to be first to market

---

#### Preset 2: "Validated B2B SaaS Finder"

**Focus:** Proven demand, B2B, existing revenue

```yaml
# Business Type Filter
business_type_filter: "b2b_only"
b2b_boost_multiplier: 2.0

# Category Filter
category_filter: ["SaaS"]
category_boosts: {"SaaS": 1.5}

# Validation Rules (strict)
require_paid_solution: true
min_competitor_mrr: 5000  # £5k+ MRR
max_competitor_count: 20
require_correlation: true
min_correlation_sources: 3

# Scoring Weights (heavier on revenue)
weight_revenue: 0.40
weight_demand: 0.20
weight_competition: 0.15
weight_feasibility: 0.15
weight_timing: 0.05
weight_risk: 0.05

# Thresholds (high bar)
score_threshold_build: 80
score_threshold_validate: 70
score_threshold_investigate: 50
```

**When to Use:** Building B2B SaaS specifically, want proven market validation, prefer opportunities with existing revenue

---

#### Preset 3: "Janky Solution Upgrader"

**Focus:** People paying for primitive solutions

```yaml
# Data Sources (marketplace focus)
enabled_sources: ["gumroad", "etsy", "reddit", "google_trends"]

# Marketplace Settings
marketplace_min_sales: 50
marketplace_max_price: 100  # Low price = primitive
marketplace_keywords: ["template", "system", "tool"]

# Scoring Weights (revenue is proof)
weight_revenue: 0.50
weight_demand: 0.30
weight_competition: 0.10
weight_feasibility: 0.10
weight_timing: 0.0
weight_risk: 0.0

# Validation
require_paid_solution: true
min_competitor_mrr: 0  # Looking for primitive, not established
require_correlation: true
min_correlation_sources: 2
```

**When to Use:** Looking to upgrade existing solutions, want proof of payment (janky solutions with sales)

---

### 5.3 Qualitative Scores

Three qualitative criteria assessed via AI:

| Criterion | Scale | Assessment Method | Admin Config |
|-----------|-------|-------------------|---------------|
| **Feasibility** | 1-10 | AI assessment via source-specific prompts | Manual override available |
| **Problem Severity** | 1-10 | AI assessment of language/emotion | Manual override available |
| **Timing/Urgency** | 1-10 | AI assessment based on growth trends | Manual override available |

> **Note:** These scores inform the recommendation but are NOT part of the 0-100 base score

---

### 5.4 Validation Rules

> **CRITICAL:** Validation rules determine whether opportunity is MARKED AS VALIDATED. Scoring (0-100) happens independently.

**Validation Criteria (all must be true for "validated" status):**

- **Existing Paid Solution:** At least 1 competitor charging money (configured via `require_paid_solution`)
- **Revenue Proof:** Evidence of £1,000+ MRR in niche (configured via `min_competitor_mrr`)
- **Frequency:** Minimum 20 mentions across sources (configured via `min_mention_count`)
- **B2B Problem:** Businesses will pay, not just consumers (configured via `require_b2b`)

> **Important:** Opportunity can score highly (60-100) without being validated. Validation is a separate flag. Recommendation logic combines score + validation status.

---

### 5.5 Recommendation Logic

| Score | Validated? | Recommendation |
|-------|-------------|----------------|
| 80-100 | Must be validated | "Build immediately" - All signals green, revenue proof confirmed |
| 60-79 | Must be validated | "Strong candidate - validate with landing page before building" |
| 40-59 | May not be validated | "High risk - need unique angle, proceed with caution" |
| 20-39 | Unlikely validated | "Reject - insufficient validation, do not build" |
| 0-19 | Not validated | "Reject - minimal data, do not build" |

> **Note:** Recommendations are based on both score AND validation status. High scores without validation still require landing page testing.

---

### 5.6 Evaluation Methodology

**Methodology Areas:**

| Area | Approach |
|--------|-------------------|
| **Revenue Extraction** | Algorithm parsing of Indie Hackers listings + AI from text + Manual annotation from founder interviews |
| **Build Complexity Classification** | AI assessment via source-specific prompts + Manual tags by admins |
| **B2B vs B2C Detection** | AI classification via source-specific prompts (B2B keywords vs B2C keywords) |
| **Validation Threshold Calibration** | Admin-configurable via Admin Panel (Configuration_Reference.md for all parameters) |

---

## 6. FEATURE SPECIFICATIONS

### 6.1 Backend API Endpoints

#### API Versioning

All API endpoints are prefixed with `/api/v1/`. This allows future breaking changes via `/api/v2/` without disrupting existing clients.

```
Base URL: https://opportunityfinder.app/api/v1/
Example:  GET /api/v1/opportunities
```

**Version Deprecation Policy:**
- New versions announced 3 months before old version sunset
- Old versions supported for 6 months after new version release
- Breaking changes only in major version bumps

#### Pagination Standard

All list endpoints support cursor-based pagination for consistent performance at scale:

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| limit | int | 50 | Items per page (max: 100) |
| cursor | string | null | Opaque cursor for next page |

**Response Format:**
```json
{
  "data": [...],
  "pagination": {
    "has_more": true,
    "next_cursor": "eyJpZCI6MTAwfQ==",
    "total_count": 500
  }
}
```

#### Authentication Endpoints

**POST /api/v1/auth/register**
- Purpose: Create new user account
- Body: email, password
- Action: Sends verification email to user

**GET /api/v1/auth/verify-email/:token**
- Purpose: Verify email address via link in email
- Action: Sets user.email_verified=true, redirects to login

**POST /api/v1/auth/login**
- Purpose: Authenticate and get JWT token
- Returns: access_token, refresh_token (only if email verified)

**POST /api/v1/auth/forgot-password**
- Purpose: Request password reset
- Body: email
- Action: Sends password reset email with token (expires in 1 hour)

**POST /api/v1/auth/reset-password**
- Purpose: Reset password with token
- Body: token, new_password

**POST /api/v1/auth/refresh**
- Purpose: Refresh access token using refresh token
- Returns: new access_token

#### Payment Endpoints

**POST /api/v1/payments/create-checkout**
- Purpose: Create Stripe checkout session for subscription based on selected tier price
- Returns: checkout_url
- Note: Tier price is fetched from subscription_tiers table, which is admin-controlled via Admin Panel

**POST /api/v1/payments/webhook**
- Purpose: Handle Stripe webhook events (payment success, subscription cancelled)
- Updates: user.subscription_status in database

#### Opportunity Endpoints

**GET /api/v1/opportunities**
- Purpose: Retrieve user's opportunities with filtering
- Auth: Requires valid JWT token. Returns only current user's opportunities.
- Query Parameters:
  - min_score (int, default: 0)
  - sort (string: 'score', 'revenue', 'mentions', default: 'score')
  - search (string: keyword search)
  - time_range (string: '3days', '1week', '1month', 'all', default: 'all')
  - status (string: 'new', 'researching', 'building', 'rejected', default: all)
  - limit (int, default: 50, max: 100)
  - cursor (string: pagination cursor)

**Search Implementation:**
- Uses PostgreSQL full-text search with `tsvector` on `title` and `problem` columns
- Search is case-insensitive and supports partial word matching
- Results ranked by relevance score combined with opportunity score
- Debounce on frontend: 300ms delay before triggering API call
- Minimum 2 characters required to trigger search

**GET /api/v1/opportunities/:id**
- Purpose: Get single opportunity with full details including source links

**PATCH /api/v1/opportunities/:id**
- Purpose: Update opportunity status and user notes
- Body:
  - status (optional): 'new', 'researching', 'building', 'rejected'
  - user_notes (optional): Research notes text

#### Scan Endpoints

**POST /api/v1/scan**
- Purpose: Trigger new opportunity scan
- Response: Job ID for tracking scan progress

**GET /api/v1/scan/:job_id**
- Purpose: Check scan progress (pending/running/complete)

#### User Endpoints

**GET /api/v1/stats**
- Purpose: Summary statistics (total, validated, high score, avg score)

**GET /api/v1/user/profile**
- Purpose: Get current user's profile (email, subscription tier, created date)

**PATCH /api/v1/user/profile**
- Purpose: Update user profile
- Body: email (optional), current_password, new_password (optional)

**GET /api/v1/user/export-data**
- Purpose: GDPR data export - download all user data
- Access: Pro and Premium tiers only
- Returns: JSON file with all opportunities, notes, searches, exports

#### Email & Landing Page Endpoints

**POST /api/v1/email/configure-alerts**
- Purpose: Configure email alert settings
- Body: alert_threshold (int), weekly_digest (bool)

**POST /api/v1/landing-page/generate**
- Purpose: Generate validation landing page for opportunity
- Body: opportunity_id, custom_headline (optional), custom_cta (optional)
- Returns: landing_page_url, slug

#### Export Endpoints

**GET /api/v1/export/csv**
- Purpose: Export filtered opportunities to CSV
- Query Parameters: Same as GET /api/v1/opportunities

**GET /api/v1/export/pdf**
- Purpose: Generate PDF report of top opportunities
- Query Parameters: top_n (int, default: 10)

#### Health Endpoints

**GET /health**
- Purpose: Simple health check for load balancer
- Returns: `{"status": "healthy"}`

**GET /health/detailed**
- Purpose: Detailed health check for admin
- Returns: Database status, Redis status, collector status, last scan time

---

### 6.2 Admin API Endpoints

> Authentication: Requires JWT with role='admin'
> Access Control: All endpoints return 403 Forbidden if user is not admin

**GET /api/v1/admin/tiers** - List all subscription tiers

**POST /api/v1/admin/tiers** - Create new subscription tier
- Body: name, price_monthly, max_sources, scan_frequency, export_limit_monthly, landing_pages_allowed, email_alerts_allowed

**PATCH /api/v1/admin/tiers/:id** - Update existing tier (price, features, enabled status)

**GET /api/v1/admin/sources** - List all data sources with stats

**POST /api/v1/admin/sources** - Add new data source
- Body: name, type, config (JSON with API keys, keywords, etc.), rate_limit_per_minute

**PATCH /api/v1/admin/sources/:id** - Update source config or enable/disable

**POST /api/v1/admin/sources/:id/test** - Test source connection and return sample data

**GET /api/v1/admin/users** - List all users with subscription details
- Query Parameters: search (email), status (active/cancelled), page, limit

**PATCH /api/v1/admin/users/:id** - Update user (cancel subscription, change role to admin)

**GET /api/v1/admin/analytics** - System analytics (users, MRR, opportunities, engagement)

**PATCH /api/v1/admin/settings** - Update system settings
- Body: currency (GBP/USD), scoring_weights (JSON), default_scan_frequency, validation_rules (JSON)
- Note: Changing currency triggers Stripe Price creation and tier price updates

**GET /api/v1/admin/trial-settings** - Get current free trial configuration

**PATCH /api/v1/admin/trial-settings** - Configure free trial settings
- Body: enabled (bool), duration_days (int), trial_tier_id (UUID), max_sources (int), max_exports (int), landing_pages_allowed (bool), email_alerts_allowed (bool), min_opportunity_rank (int), require_card (bool), convert_to_tier_id (UUID)

**GET /api/v1/admin/audit-logs** - View admin action audit trail
- Query Parameters: action_type, admin_id, start_date, end_date, limit, cursor
- Returns: Paginated list of admin actions with timestamps and details

#### Audit Logging

All admin actions are automatically logged for compliance and debugging:

| Action Type | Logged Data |
|-------------|-------------|
| tier_created | tier_id, name, price, admin_id, timestamp |
| tier_updated | tier_id, changed_fields, old_values, new_values, admin_id |
| user_updated | user_id, changed_fields, admin_id, timestamp |
| source_updated | source_id, changed_fields, admin_id, timestamp |
| settings_changed | setting_key, old_value, new_value, admin_id |

Audit logs retained for 1 year minimum.

---

### 6.3 Frontend Features

#### Authentication & Profile
- Registration form with email verification
- Login form
- "Forgot password" link → Email reset link
- Password reset page (token-based)
- User profile page:
  - Change email
  - Change password
  - View subscription details
  - Update payment method (Stripe portal)
  - Export data button (Pro/Premium only)

#### Dashboard View
- 4 stat cards: Total, Validated, High Score, Average
- Search bar with real-time filtering
- Sort dropdown (score/revenue/mentions)
- Filter toggle with min score slider

#### Opportunity Cards
- Score badge (color-coded by range)
- Title and problem statement
- Revenue, mentions, competition, complexity
- Recommendation text

#### Detail Modal
- Full opportunity breakdown
- Source links (clickable)
- Market size estimate
- Existing competitor examples with direct links
- Action buttons (Mark as Researching/Building/Rejected)

#### Time-Based Filtering
- **Default View:** "Top Opportunities" - All time opportunities sorted by score (highest first)
- **Purpose of Time Filters:** When you need more context or saw something that's no longer on main page
- **Filter options:** Last 3 days, Last week, Last month, All time
- "Last 3 days" shows recent discoveries if something dropped off top page
- Time filters complement score sorting, don't replace it

#### Historical Archive
- All opportunities permanently stored (never deleted)
- Track mentions over time (trending up/down indicators)
- See when opportunity was first discovered
- Research notes and status tracking per opportunity

#### Email Alerts
- Automatic email when opportunity scores 80+ found
- Weekly digest of top 5 opportunities
- Configurable alert threshold in settings

#### Landing Page Builder
Quick validation tool: Generate a "Coming Soon - Join Waitlist" page for any opportunity to test demand before building.
- One-click landing page generation from opportunity data
- Customizable headline, description, and CTA
- Email capture form with Mailchimp/ConvertKit integration
- Hosted on subdomain (validate.opportunityfinder.app/[slug])

#### Export Functionality
- Export filtered opportunities to CSV
- Generate PDF report with top opportunities
- Share link for specific opportunity

---

### 6.4 Frontend Page Structure

#### Dashboard/List Page

**URL Pattern:** `/dashboard` or `/browse`

**Layout Components:**
- Navigation header (logo, menu items)
- Filter bar (sort, filters, search)
- Opportunity grid/list of cards
- Pagination controls

**Card Structure:**
- Opportunity image/hero (optional)
- Title (clickable, links to detail modal)
- Badges/scores (Opportunity, Problem, Feasibility, Why Now)
- Brief description (2-3 sentences)
- Key metrics row (Volume, Growth, Competition level)
- Action buttons: View Report, Save, Building

**Data Displayed per Opportunity:**

| Metric | Example |
|---------|---------|
| Opportunity Score | 7/10 with color indicator |
| Problem Severity | High Pain to Low Pain 1-10 |
| Feasibility | Manageable to Very Hard 1-10 |
| Why Now | Perfect Timing to Poor Timing 1-10 |
| Volume | Search volume (e.g., "2.9K", "4.4K") |
| Growth | YoY percentage (e.g., "+303%") |
| Competition | LOW/MEDIUM/HIGH |

**Interactions:**
- Hover card shows expanded preview
- Click opportunity → detail modal
- Filter sidebar updates list in real-time
- Sort dropdown changes order

#### Opportunity Detail Modal/Page

**Layout Components:**
- Back to list link
- Large title with opportunity title
- Opportunity Score badge (large, prominent)

**Page Sections:**

**Header Section:**
- Four-score breakdown row:
  - Opportunity: X/10
  - Problem: X/10
  - Feasibility: X/10
  - Why Now: X/10
- Action buttons: Save, Building, Researching, Rejected

**Validation Evidence Section:**
- "Proof & Signals" heading
- List of proof sources with indicators:
  - Community: Reddit X/10, Facebook X/10, YouTube X/10
  - Market Data: Revenue proof, competitor examples, market size
  - Competitors section (competitor cards or list)
- "What's Missing?" analysis (market gap section)

**Content Sections:**
- "The Problem" section - emotional hook opening paragraph
- "The Solution" section - what the product does
- "Target Market" section - who pays and why
- "Pricing Model" section - pricing tiers and revenue potential

**Layout Pattern:**
- Full-width content container (max-width: 800-1200px)
- Left column (70%): All content sections
- Right column (30%): Proof & Signals + action buttons (sticky)

#### Public Landing Page

**URL Pattern:** `/` (root)

**Layout Components:**
- Hero section with value proposition
- Features list (3-4 key benefits)
- Social proof (user count, success stories)
- Email capture form (email field, submit button)
- CTA button: "Get Started"

**Hero Section:**
- Main headline: Product value proposition
- Subheadline: "Powered by Data, Not Guessing"
- Email capture form
- CTA button (prominent)

**Features Section:**
- 3-4 benefit cards (each with icon, title, bullet points)

**Social Proof Section:**
- Success stories with testimonials
- Member/idea count badge

---

### 6.5 Frontend Component Specifications

#### OpportunityCard Component

```typescript
interface OpportunityCardProps {
  title: string;
  score: number;
  problemScore: number;
  feasibilityScore: number;
  whyNowScore: number;
  volume: string;           // "2.9K"
  growth: string;           // "+303%"
  competitionLevel: 'LOW' | 'MEDIUM' | 'HIGH';
  description: string;
  actions: string[];        // ['view-report', 'save', 'building']
}
```

#### ActionButton Component

```typescript
interface ActionButtonProps {
  type: 'view-report' | 'save' | 'interested' | 'not-interested' | 'building' | 'researching' | 'rejected';
  icon?: string;
  label: string;
  onClick?: () => void;
}
```

#### Badge Component

```typescript
interface BadgeProps {
  type: 'opportunity' | 'problem' | 'feasibility' | 'why-now';
  score: number;
  label?: string;
}
```

#### Filter State

```typescript
interface Filters {
  sortBy: 'newest' | 'score-desc' | 'score-asc' | 'oldest';
  minScore?: number;
  maxScore?: number;
  validatedOnly?: boolean;
  status?: 'new' | 'researching' | 'building' | 'rejected';
  timeRange?: '3days' | '1week' | '1month' | 'all';
}
```

#### Opportunity List State

```typescript
interface OpportunityListState {
  opportunities: Opportunity[];
  loading: boolean;
  filters: Filters;
  pagination: {
    hasMore: boolean;
    nextCursor: string | null;
    totalCount: number;
  };
  error: string | null;
}
```

#### Loading State Component

```typescript
interface LoadingStateProps {
  message?: string;
  showProgress?: boolean;
  progress?: number;
}
```

---

### 6.6 UI Pages - Complete List

> **Important:** `docs/HTML/opportunity-finder.html` is a theme template showing visual style (colors, gradients, card design). It is NOT a complete set of pages. All pages below must be implemented.

**Required Pages for MVP (Phase 1):**

| Page | Description | Key Elements |
|-------|-------------|--------------|
| **Public Landing Page** | Product overview for visitors | Explains value prop, features, pricing tiers, CTA to register |
| **Register** | New user signup | Email field, password field, password confirm, submit button |
| **Login** | Existing user login | Email field, password field, "forgot password" link, submit button |
| **Forgot Password** | Request password reset | Email field, submit button |
| **Reset Password** | Set new password with token | New password field, confirm field, submit button |
| **Email Verification** | Handle email verification link | Auto-redirects to login after verification |
| **Dashboard** | Main opportunity view | Filter bar, opportunity grid, stat cards |
| **Opportunity Detail Modal** | Full opportunity breakdown | Scores, sources, competitors, action buttons |
| **User Profile** | Account management | Email, password change, subscription info, export data |
| **Admin Login** | Admin authentication | Same as user login but with role='admin' |
| **Admin Dashboard** | Admin overview | User management, system stats, configuration links |
| **Admin - Tiers** | Subscription tier management | Create/edit/delete tiers, pricing, features |
| **Admin - Sources** | Data source management | Enable/disable sources, configure parameters, test connections |
| **Admin - Users** | User management | View all users, edit subscriptions, change roles |
| **Admin - Analytics** | System analytics | Usage stats, MRR, opportunity metrics |
| **Admin - Settings** | System configuration | Scoring weights, validation rules, scan frequency |
| **Admin - Audit Logs** | Admin action history | Paginated log of all admin actions |

**Optional Pages (Phase 2):**
- Help/FAQ page
- Onboarding tutorial
- Referral program page

---

## 7. ADMIN PANEL SPECIFICATION

### 7.1 Admin Authentication & Authorization

**God-Mode Capabilities:**

1. **User Impersonation:** Admin can log in as any user without password
2. **Emergency Overrides:** Admin can manually trigger scans, adjust scores, change statuses
3. **Full Configuration:** Admin can modify all 90+ system parameters
4. **Audit Trail:** All admin actions logged with timestamps and changed values

**Implementation:**
- Admin users have `role='admin'` in users table
- `/api/v1/admin/impersonate/:user_id` endpoint starts impersonation session
- All admin actions logged to `admin_audit_logs` table
- Admin can view but not modify audit logs (compliance)

---

### 7.2 Admin Panel Configuration UI

**Scoring & Configuration Section (12 Tabs):**

| Tab | Parameters | Key Features |
|-----|-----------|--------------|
| **Data Sources** | enabled_sources, source_priority, source_fallback_enabled, min_confidence_threshold | Enable/disable sources, set priority weights |
| **Scoring Weights** | All 7 weight parameters | Sliders with sum validation (must equal 1.0) |
| **Thresholds** | All threshold parameters | Score thresholds, velocity/volume thresholds |
| **Validation Rules** | All 11 validation parameters | Toggle rules, set MRR thresholds, correlation requirements |
| **Business Type** | Business type filters, B2B/B2C multipliers | Filter by type, boost/penalty multipliers |
| **Category** | Category filters, boosts, blacklist | Filter by category, apply score multipliers |
| **Maturity** | Maturity filters, definition parameters | Emerging/growing/established configuration |
| **AI Models** | Model selection, temperature, tokens, fallback | Choose AI models, configure parameters |
| **Geographic** | Target geographies, language filters | Filter by region/language |
| **Source-Specific** | 26+ source-specific parameters | Configure parameters per enabled source |
| **Output** | Max results, sorting, include flags | Control output format and detail level |
| **Notifications** | Alert thresholds, digest frequency, channels | Configure alerting behavior |

---

### 7.3 Configuration Preset Management

**Preset Management UI:**
- Dropdown to select preset: "Default", "Aggressive Emerging Hunter", "Validated B2B SaaS Finder", "Janky Solution Upgrader", "Custom"
- Loading preset populates all tabs with preset values
- Admin can modify preset values and save as "Custom"
- Admin can create new presets from current configuration
- Presets stored in `application_config` table as JSON blobs

---

### 7.4 Configuration Validation UI

**Real-Time Validation:**
- Invalid configurations highlighted in red
- Error messages displayed at top of configuration section
- Cannot save invalid configuration
- Validation occurs on:
  - Weight sum (must equal 1.0)
  - Threshold ordering (build > validate > investigate)
  - Business type mutual exclusion
  - Correlation requirements
  - Source-specific dependencies

**Error Display:**
```
┌─────────────────────────────────────────────────────────────────┐
│ ❌ Configuration Errors Detected                                     │
├─────────────────────────────────────────────────────────────────┤
│ • Weight sum is 0.85 (must equal 1.0)                          │
│ • score_threshold_investigate (40) must be < score_threshold_validate (60) │
│ • Cannot require both B2B and B2C exclusively                    │
└─────────────────────────────────────────────────────────────────┘
```

---

## 8. DATABASE SCHEMA

### 8.1 Core Tables

#### opportunities

```sql
CREATE TABLE opportunities (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(id),

    -- Core fields
    title TEXT NOT NULL,
    problem TEXT NOT NULL,
    score INTEGER NOT NULL CHECK (score >= 0 AND score <= 100),
    rank INTEGER NOT NULL,

    -- Metrics
    mentions INTEGER DEFAULT 0,
    mentions_trend VARCHAR(20) CHECK (mentions_trend IN ('up', 'down', 'stable')),
    revenue_amount INTEGER DEFAULT 0,
    revenue VARCHAR(100) DEFAULT '£0',
    competitors INTEGER DEFAULT 0,
    competition_level VARCHAR(20) CHECK (competition_level IN ('Low', 'Medium', 'High')),
    build_complexity VARCHAR(20) CHECK (build_complexity IN ('Low', 'Medium', 'High')),

    -- Sub-scores
    problem_score INTEGER CHECK (problem_score >= 1 AND problem_score <= 10),
    feasibility_score INTEGER CHECK (feasibility_score >= 1 AND feasibility_score <= 10),
    timing_score INTEGER CHECK (timing_score >= 1 AND timing_score <= 10),

    -- Classifications
    category VARCHAR(100),
    business_type VARCHAR(20) CHECK (business_type IN ('B2B', 'B2C', 'Both', 'Unclear')),
    maturity_level VARCHAR(20) CHECK (maturity_level IN ('Emerging', 'Growing', 'Established')),

    -- Validation
    validated BOOLEAN DEFAULT FALSE,
    recommendation TEXT,

    -- Market data
    market_size TEXT,
    keyword_volume VARCHAR(20),
    keyword_growth VARCHAR(20),
    keyword_competition VARCHAR(20),

    -- Sources
    sources TEXT[] NOT NULL DEFAULT '{}',
    source_urls TEXT[] NOT NULL DEFAULT '{}',

    -- Examples
    example VARCHAR(255),
    competitor_urls TEXT[] DEFAULT '{}',

    -- Status
    status VARCHAR(20) DEFAULT 'new' CHECK (status IN ('new', 'researching', 'building', 'rejected')),
    user_notes TEXT,

    -- Timestamps
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),

    -- Indexes
    INDEX idx_opportunities_user_score (user_id, score DESC),
    INDEX idx_opportunities_user_status (user_id, status),
    INDEX idx_opportunities_created (created_at DESC)
);
```

#### opportunity_mentions

```sql
-- Defined in Section 3.1.1
```

#### mention_correlations

```sql
-- Defined in Section 5.1.4
```

#### application_config

```sql
-- Defined in Configuration_Reference.md
```

---

## 9. IMPLEMENTATION ROADMAP

### MVP (Weeks 1-8) - Updated with Brainstorming Decisions

**Week 1-2: Foundation**
- [ ] Set up project structure (Flask + React Vite)
- [ ] Implement unified data schema (opportunity_mentions table)
- [ ] Implement DataSourceInterface abstract class
- [ ] Set up AI platform integration
- [ ] Create application_config table and seed default configuration
- [ ] Implement ConfigurationValidator class

**Week 3-4: Data Sources (6 MVP Sources)**
- [ ] Reddit source - Implement DataSourceInterface
- [ ] Indie Hackers source - Implement DataSourceInterface
- [ ] Product Hunt source - Implement DataSourceInterface
- [ ] Hacker News source - Implement DataSourceInterface
- [ ] Google Trends source - Implement DataSourceInterface
- [ ] Competition Search source - Implement DataSourceInterface
- [ ] All sources return OpportunityMention objects
- [ ] All sources use source-specific AI prompts

**Week 5-6: Scoring & Validation**
- [ ] Implement 7-component scoring engine
- [ ] Implement configuration validation (all 5 validation rules)
- [ ] Create admin configuration UI (12-tab interface)
- [ ] Implement configuration preset system
- [ ] Implement validation engine with configurable rules
- [ ] Connect to AI platform for analysis

**Week 7-8: Integration & Testing**
- [ ] Implement CorrelationEngine class
- [ ] End-to-end testing with real queries
- [ ] Test all 6 data sources end-to-end
- [ ] Test configuration validation
- [ ] Test preset loading/saving
- [ ] Bug fixes and refinement

**MVP Deliverable:**
- Working system with 6 data sources
- Admin-configurable scoring and validation (90+ parameters)
- Cross-source correlation engine
- Returns scored, validated opportunities
- Configuration presets available
- All data sources implement unified interface

---

### Phase 2 (Weeks 9-20) - Future Enhancement

**Week 9-10: Additional Data Sources**
- [ ] Stack Overflow source
- [ ] YouTube source
- [ ] Gumroad/Etsy source

**Week 11-12: Enhanced Features**
- [ ] Negative Space Analyzer (when 0 competitors)
- [ ] Investigation Briefs (auto-generated next steps)
- [ ] Quick Scan mode

**Week 13-14: Visualization & UX**
- [ ] Radar chart visualization
- [ ] Opportunity screener filters
- [ ] Emerging vs Established toggle

**Week 15-16: Monitoring**
- [ ] Opportunity watchlists
- [ ] Alert system (email, webhook)
- [ ] Alert digest (daily/weekly)

**Week 17-18: Advanced Correlation**
- [ ] Full correlation engine enhancements
- [ ] Opportunity clustering (basic)
- [ ] Cross-source pattern detection

**Week 19-20: Polish**
- [ ] Auto-categorization
- [ ] Query expansion
- [ ] Performance optimization
- [ ] Documentation

---

## 10. DOCUMENT CONTROL

### Version History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 10/01/2026 | Mark | Initial PRD - Core validation system |
| 1.1-1.9 | Various | Various | Multiple incremental updates (see full history in v2.0) |
| 2.0 | 14/01/2026 | Mark + Claude | Major consolidation release |
| **3.0** | **19/01/2026** | **Mark + Claude + Winston** | **Major update: Incorporated all brainstorming session architectural decisions** |

### Changes in Version 3.0

**Added Sections:**
- Section 3.1.1: Unified Data Schema (OpportunityMention)
- Section 3.1.2: Data Source Interface Contract (DataSourceInterface)
- Section 5.1.4: Cross-Source Correlation Engine
- Section 5.2.1: Configuration Validation Rules
- Section 5.2.2: Configuration Presets

**Updated Sections:**
- Section 3.1: System Components now references unified schema and interface contract
- Section 4: Data Sources now clearly distinguishes MVP (6 sources) from Phase 2 sources
- Section 5.2: Expanded to "Scoring & Configuration System" with reference to Configuration_Reference.md
- Section 13 (if exists): Implementation roadmap updated with 8-week MVP plan

**Key Architectural Decisions Incorporated:**
- All data sources normalize to OpportunityMention schema
- All data sources implement DataSourceInterface contract
- 90+ parameters stored in application_config table
- 7-component scoring system (vs 4-component in v2.0)
- Source-specific AI prompts for each data source
- Cross-source correlation engine
- Configuration validation prevents broken configurations
- Configuration presets for common use cases
- MVP scope clarified as 6 sources (Reddit, Indie Hackers, Product Hunt, Hacker News, Google Trends, Competition Search)

### Related Documents

- **Configuration_Reference.md** - Complete 90+ parameter reference
- **AI_Services_Platform_PRD_v1.4.md** - Platform infrastructure (to be updated)
- **Phase-8-Admin-Panel-UX-Design.md** - Admin panel UI (to be updated)
- **brainstorming-session-2026-01-19.md** - Source of architectural decisions

---

### Approvals

| Field | Value |
|-------|-------|
| Document Owner | Mark |
| Status | READY FOR IMPLEMENTATION |
| Date | 19/01/2026 |
| Review | Approved by Mark - incorporates all brainstorming session decisions |

---

**— END OF DOCUMENT —**
