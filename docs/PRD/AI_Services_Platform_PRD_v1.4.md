# AI Services Platform PRD v1.4

**Version:** 1.4
**Date:** 2026-01-19
**Status:** Ready for Implementation
**Type:** Standalone Reusable Platform

---

## What's New in Version 1.4

This version incorporates architectural decisions from the Opportunity Finder brainstorming session (2026-01-19), adding support for source-specific AI prompts and pipeline-to-source binding.

**New Sections:**
- **Section 7.9: Source-Specific Prompt Templates** - Custom AI prompts for each data source type (Reddit, Google Trends, Product Hunt, etc.)
- **Section 7.10: Pipeline-to-Source Binding** - Configuration for binding specific analysis pipelines to data sources
- **Section 13.4: Application Configuration Tables** - `application_config` table for multi-application configuration
- **Section 13.5: Correlation Tables** - `mention_correlations` table for cross-source correlation

**Database Updates:**
- Added `application_id` foreign key to `ai_request_logs` table
- Added `application_id` foreign key to `ai_cost_summaries` table
- Added `application_config` table for storing application-specific configuration
- Added `mention_correlations` table for storing cross-source correlation data

**Architecture Alignment:**
- Source-specific prompt templates align with `DataSourceInterface.get_ai_prompt_template()` contract
- Pipeline-to-source binding enables flexible AI analysis per data source
- Multi-application configuration support via `application_config` table

---

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [Product Overview](#2-product-overview)
3. [Problem Statement](#3-problem-statement)
4. [Solution](#4-solution)
5. [Core Architecture](#5-core-architecture)
6. [AI Provider System](#6-ai-provider-system)
7. [Analysis Pipelines](#7-analysis-pipelines)
8. [Prompt Management System](#8-prompt-management-system)
9. [Data Source Management](#9-data-source-management)
10. [Scoring Integration](#10-scoring-integration)
11. [Admin Panel Specification](#11-admin-panel-specification)
12. [API Specification](#12-api-specification)
13. [Database Schema](#13-database-schema)
14. [Security Model](#14-security-model)
15. [Cost Management](#15-cost-management)
16. [Error Handling & Fallback](#16-error-handling--fallback)
17. [Monitoring & Observability](#17-monitoring--observability)
18. [Testing Strategy](#18-testing-strategy)
19. [Deployment](#19-deployment)
20. [Roadmap](#20-roadmap)
21. [Platform Authentication & Authorization](#21-platform-authentication--authorization)
22. [Multi-Application Support](#22-multi-application-support)
23. [Client Integration Guide](#23-client-integration-guide)
24. [Cache Management Strategy](#24-cache-management-strategy)
25. [Backup & Disaster Recovery](#25-backup--disaster-recovery)
26. [Webhook System](#26-webhook-system)
27. [Maintenance Mode](#27-maintenance-mode)
28. [Service Level Objectives](#28-service-level-objectives)
29. [Feature Flags System](#29-feature-flags-system)
30. [Capacity Planning](#30-capacity-planning)
31. [Compliance & Legal](#31-compliance--legal)
32. [Operational Runbooks](#32-operational-runbooks)
33. [Appendix A: Provider API Specifications](#appendix-a-provider-api-specifications)
34. [Appendix B: Error Codes](#appendix-b-error-codes)

---

## 1. Executive Summary

### 1.1 Product Vision

The AI Services Platform is a **standalone, reusable infrastructure layer** that enables any application to leverage multiple AI providers through a unified interface. It provides:

- **Multi-Provider Support**: OpenAI, Anthropic, ZAI, open-source models, and custom providers
- **Configurable Analysis Pipelines**: 5-stage, 21+ AI-powered analysis pipelines
- **Complete Admin Control**: Full configuration via web-based admin panel
- **Dynamic Data Sources**: Add/remove data sources without code changes
- **Cost Optimization**: Provider selection, rate limiting, cost tracking
- **Production Ready**: Fallback, retry, monitoring, alerting

### 1.2 Design Philosophy

| Principle | Implementation |
|-----------|----------------|
| **Provider Agnostic** | Switch providers via config, no code changes |
| **Pipeline Modular** | Each analysis stage is independently configurable |
| **Everything Configurable** | Prompts, models, parameters all admin-controlled |
| **Cost Aware** | Track every token, optimize provider selection |
| **Production Grade** | Fallbacks, retries, monitoring, alerting built-in |
| **Reusable** | Platform-agnostic, can power any application |

### 1.3 Target Users

| User Type | Needs |
|-----------|-------|
| **Application Developers** | Drop-in AI capabilities without building AI infrastructure |
| **Product Teams** | Configure AI behavior without engineering changes |
| **System Administrators** | Full control over providers, costs, monitoring |
| **DevOps Engineers** | Deploy, monitor, maintain AI services |

---

## 2. Product Overview

### 2.1 What Is The AI Services Platform?

The AI Services Platform is a **backend infrastructure service** that:

1. **Abstracts AI Provider Complexity**
   - Single API for multiple AI providers
   - Provider-agnostic interface
   - Automatic failover between providers

2. **Provides Analysis Pipelines**
   - Pre-built, configurable AI analysis stages
   - Pipeline orchestration
   - Result aggregation

3. **Enables Complete Control**
   - Web-based admin panel
   - Real-time configuration changes
   - Cost tracking and optimization

4. **Manages Data Sources**
   - Dynamic source addition/removal
   - Source templates for common platforms
   - Custom source builder

### 2.2 What It Is NOT

| Not This | Why |
|----------|-----|
| A frontend application | This is backend infrastructure |
| tied to Opportunity Finder | Fully reusable across applications |
| A simple AI wrapper | Complete platform with pipelines, monitoring, admin |
| A library/framework | Deployable service with its own infrastructure |

### 2.3 Use Cases

| Application | How It Uses The Platform |
|-------------|--------------------------|
| **Opportunity Finder** | Analyzes market signals, scores opportunities |
| **Content Moderation** | Analyzes user-generated content for policy violations |
| **Customer Support** | Classifies tickets, suggests responses, routes issues |
| **Market Research** | Analyzes trends, sentiment, competitive landscape |
| **Data Enrichment** | Enhances records with AI-extracted insights |

---

## 3. Problem Statement

### 3.1 Current Problems

Building AI-powered applications today requires solving the same problems repeatedly:

| Problem | Impact |
|---------|--------|
| **Provider Lock-in** | Tied to one AI vendor, can't switch without code changes |
| **Pipeline Complexity** | Each AI analysis stage requires custom code |
| **Cost Blindness** | No visibility into per-request or per-user AI costs |
| **Configuration Hardcoded** | Prompt changes, model selection requires deployment |
| **No Fallback Strategy** | Single provider failure = system failure |
| **Data Source Rigid** | Adding new data sources requires engineering |
| **Monitoring Gap** | Can't see AI performance, quality, or health |
| **Admin Burden** | No unified interface to manage AI infrastructure |

### 3.2 Why Existing Solutions Fall Short

| Solution Type | Limitation |
|---------------|------------|
| **Direct API Integration** | Provider-specific, no fallback, no cost tracking |
| **Simple AI Wrappers** | No pipelines, no orchestration, minimal config |
| **Enterprise AI Platforms** | Expensive, complex, vendor lock-in, over-engineered |
| **LLM Libraries (LangChain, etc.)** | Code-based configuration, no admin panel, infrastructure burden |

### 3.3 The Gap

**No open, standalone platform exists that:**
- Provides multi-provider AI with admin control
- Includes configurable analysis pipelines out of the box
- Offers complete cost visibility and optimization
- Enables dynamic data source management
- Can be deployed once and used by multiple applications

---

## 4. Solution

### 4.1 The AI Services Platform

A **deployable backend service** that provides:

```
┌─────────────────────────────────────────────────────────────────┐
│                     AI SERVICES PLATFORM                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │  UNIFIED AI API                                            │ │
│  │  Single interface for all AI capabilities                  │ │
│  └────────────────────────────────────────────────────────────┘ │
│                           ↓                                     │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │  ANALYSIS PIPELINES                                        │ │
│  │  5 stages, 21+ configurable AI analysis steps              │ │
│  └────────────────────────────────────────────────────────────┘ │
│                           ↓                                     │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │  MULTI-PROVIDER LAYER                                      │ │
│  │  OpenAI │ Anthropic │ ZAI │ Open-source │ Custom           │ │
│  └────────────────────────────────────────────────────────────┘ │
│                           ↓                                     │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │  INFRASTRUCTURE                                            │ │
│  │  Queue │ Cache │ Database │ Monitoring │ Admin Panel       │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 4.2 Key Capabilities

| Capability | Description |
|------------|-------------|
| **Provider Management** | Add/remove/configure AI providers via admin panel |
| **Pipeline Orchestration** | Chain multiple AI analysis steps automatically |
| **Prompt Configuration** | Edit system prompts without deployment |
| **Cost Tracking** | Real-time cost monitoring per provider, pipeline, user |
| **Fallback Strategy** | Automatic provider failover with configurable chains |
| **Data Source Management** | Dynamic source configuration without code changes |
| **Admin Control** | Web-based panel for all configuration |
| **Monitoring** | Health, performance, cost, quality metrics |

### 4.3 Integration Model

Applications integrate via **REST API**:

```python
# Application sends data to platform
response = POST /api/v1/analyze/pipeline
{
  "pipeline": "opportunity_analysis_full",
  "input": {
    "raw_data": "I wish there was a better way to export CRM contacts",
    "source": "reddit",
    "metadata": {...}
  }
}

# Platform runs all configured pipelines
# Returns unified result
{
  "analysis_id": "uuid",
  "results": {
    "pain_point": {...},
    "severity": 8,
    "solution_ideas": [...],
    "competitors": [...],
    "market_size": {...},
    "score": 78,
    "recommendation": "build_now"
  },
  "cost": 0.023,
  "providers_used": ["openai", "anthropic"],
  "duration_ms": 2340
}
```

---

## 5. Core Architecture

### 5.1 System Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                        AI SERVICES PLATFORM                         │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌────────────────────────────────────────────────────────────────┐ │
│  │  APPLICATION LAYER                                             │ │
│  │  Any application that needs AI capabilities                    │ │
│  └────────────────────────────────────────────────────────────────┘ │
│                           ↕ REST API                               │
│  ┌────────────────────────────────────────────────────────────────┐ │
│  │  API GATEWAY                                                   │ │
│  │  • Authentication (JWT)                                        │ │
│  │  • Rate limiting                                               │ │
│  │  • Request routing                                             │ │
│  │  • Response aggregation                                        │ │
│  └────────────────────────────────────────────────────────────────┘ │
│                           ↕                                        │
│  ┌────────────────────────────────────────────────────────────────┐ │
│  │  ORCHESTRATION LAYER                                           │ │
│  │  ┌────────────────┐  ┌────────────────┐  ┌────────────────┐   │ │
│  │  │ Pipeline       │  │ Fallback       │  │ Cost           │   │ │
│  │  │ Orchestrator   │  │ Manager        │  │ Tracker        │   │ │
│  │  └────────────────┘  └────────────────┘  └────────────────┘   │ │
│  └────────────────────────────────────────────────────────────────┘ │
│                           ↕                                        │
│  ┌────────────────────────────────────────────────────────────────┐ │
│  │  AI PROCESSING LAYER                                            │ │
│  │  ┌────────────────┐  ┌────────────────┐  ┌────────────────┐   │ │
│  │  │ Prompt         │  │ Provider       │  │ Result         │   │ │
│  │  │ Manager        │  │ Router         │  │ Aggregator     │   │ │
│  │  └────────────────┘  └────────────────┘  └────────────────┘   │ │
│  └────────────────────────────────────────────────────────────────┘ │
│                           ↕                                        │
│  ┌────────────────────────────────────────────────────────────────┐ │
│  │  PROVIDER ABSTRACTION LAYER                                     │ │
│  │  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐         │ │
│  │  │ OpenAI   │ │Anthropic │ │   ZAI    │ │  Custom  │         │ │
│  │  │ Adapter  │ │ Adapter  │ │ Adapter  │ │ Adapter  │         │ │
│  │  └──────────┘ └──────────┘ └──────────┘ └──────────┘         │ │
│  └────────────────────────────────────────────────────────────────┘ │
│                           ↕                                        │
│  ┌────────────────────────────────────────────────────────────────┐ │
│  │  DATA LAYER                                                    │ │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐        │ │
│  │  │ PostgreSQL   │  │ Redis        │  │ S3/Storage   │        │ │
│  │  │ • Config     │  │ • Cache      │  │ • Logs        │        │ │
│  │  │ • Prompts    │  │ • Queue      │  │ • Artifacts   │        │ │
│  │  │ • Costs      │  │ • Sessions   │  │              │        │ │
│  │  │ • Logs       │  │              │  │              │        │ │
│  │  └──────────────┘  └──────────────┘  └──────────────┘        │ │
│  └────────────────────────────────────────────────────────────────┘ │
│                           ↕                                        │
│  ┌────────────────────────────────────────────────────────────────┐ │
│  │  ADMIN PANEL                                                   │ │
│  │  Web-based configuration interface                            │ │
│  └────────────────────────────────────────────────────────────────┘ │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### 5.2 Request Flow

```
1. Application → Platform
   POST /api/v1/analyze/pipeline
   {pipeline: "name", input: {...}}

2. API Gateway
   • Validates JWT
   • Checks rate limits
   • Routes to orchestrator

3. Orchestrator
   • Loads pipeline configuration
   • Determines required AI providers
   • Queues pipeline stages

4. For Each Pipeline Stage:
   a. Prompt Manager
      • Loads system prompt
      • Injects input data
      • Applies parameters (temp, tokens, etc.)

   b. Provider Router
      • Selects provider/model
      • Checks rate limits
      • Sends request

   c. Provider Adapter
      • Transforms to provider format
      • Calls provider API
      • Handles errors/retries

   d. Fallback Manager
      • On failure: tries next provider
      • Logs failure
      • Continues or aborts

5. Cost Tracker
   • Records tokens used
   • Calculates cost
   • Updates budgets

6. Result Aggregator
   • Combines all stage outputs
   • Generates unified response
   • Returns to application

Total time: 2-30 seconds depending on pipeline complexity
```

### 5.3 Technology Stack

| Component | Technology | Rationale |
|-----------|-----------|-----------|
| **Backend** | Python 3.11+ | AI library ecosystem, async support |
| **API Framework** | FastAPI | Performance, auto-docs, async |
| **Database** | PostgreSQL 15+ | Relational, JSONB, reliable |
| **Cache** | Redis 7+ | Fast, pub/sub, queue |
| **Queue** | Celery + Redis | Proven, scalable |
| **Storage** | S3-compatible | Logs, artifacts, backups |
| **Admin Frontend** | React + TypeScript | Component reuse, type safety |
| **Monitoring** | Prometheus + Grafana | Metrics, alerting |
| **Logging** | Structlog + ELK | Queryable logs |
| **Deployment** | Docker + Kubernetes | Scalability, orchestration |

---

## 6. AI Provider System

### 6.1 Supported Providers

| Provider | Models | Status |
|----------|--------|--------|
| **OpenAI** | gpt-4-turbo, gpt-4, gpt-3.5-turbo | ✅ Core |
| **Anthropic** | claude-3-opus, claude-3-sonnet, claude-3-haiku | ✅ Core |
| **ZAI (Zhipu AI)** | glm-4.7, glm-4-plus, glm-4-air, glm-4-flash | ✅ Core |
| **Open-Source** | llama-3-70b, mistral-7b, qwen-72b (self-hosted) | ✅ Core |
| **Custom** | Any OpenAI-compatible endpoint | ✅ Supported |

### 6.2 Provider Configuration

Each provider requires:

| Setting | Type | Required | Description |
|---------|------|----------|-------------|
| `provider_id` | string | Yes | Unique identifier |
| `provider_name` | string | Yes | Display name |
| `api_endpoint` | string | Yes | API base URL |
| `api_key` | string | Yes | Encrypted at rest |
| `organization_id` | string | No | For org-level billing |
| `enabled` | boolean | Yes | Active/inactive |
| `priority` | integer | Yes | Fallback order (1=highest) |
| `models` | array | Yes | Available models |
| `rate_limits` | object | Yes | Requests/min, tokens/min |
| `cost_limits` | object | Yes | Monthly budget, alert threshold |

### 6.3 Model Configuration

Each model within a provider:

| Setting | Type | Description |
|---------|------|-------------|
| `model_id` | string | Unique model identifier |
| `model_name` | string | Display name |
| `enabled` | boolean | Available for use |
| `priority` | integer | Selection order within provider |
| `context_window` | integer | Max input tokens |
| `max_output_tokens` | integer | Max generation tokens |
| `input_cost_per_1k` | decimal | Cost per 1k input tokens |
| `output_cost_per_1k` | decimal | Cost per 1k output tokens |
| `supports_function_calling` | boolean | Function calling capability |
| `supports_streaming` | boolean | Streaming response support |

### 6.4 Provider Adapters

Each provider has an adapter that:

```python
class ProviderAdapter(ABC):
    @abstractmethod
    async def send_request(
        self,
        prompt: str,
        model: str,
        temperature: float,
        max_tokens: int,
        **kwargs
    ) -> ProviderResponse:
        """Send request to provider API"""
        pass

    @abstractmethod
    def calculate_cost(
        self,
        input_tokens: int,
        output_tokens: int,
        model: str
    ) -> decimal.Decimal:
        """Calculate request cost"""
        pass

    @abstractmethod
    async def test_connection(self) -> bool:
        """Verify API credentials work"""
        pass
```

### 6.5 Provider Selection Logic

```
1. Pipeline specifies required provider/model (optional)
   ↓
2. Admin configures provider priority per pipeline
   ↓
3. Provider Router selects:
   • If pipeline specifies: use that provider
   • If not configured: use highest priority enabled provider
   • Check rate limits before selection
   ↓
4. If selected provider fails:
   • Try next in priority list
   • Log failure
   • Continue until success or list exhausted
   ↓
5. If all fail:
   • Queue for retry (if configured)
   • Return error to application
```

---

## 7. Analysis Pipelines

### 7.1 Pipeline Architecture

Pipelines are **chained AI analysis stages** that process data sequentially:

```
Input → Stage 1 → Stage 2 → Stage 3 → Stage 4 → Stage 5 → Output
         ↓         ↓         ↓         ↓         ↓
       Result   Result   Result   Result   Result
         ↓         ↓         ↓         ↓         ↓
         └─────────┴─────────┴─────────┴─────────┘
                           ↓
                    Aggregated Output
```

### 7.2 The 5 Pipeline Stages

| Stage | Purpose | Input | Output |
|-------|---------|-------|--------|
| **Stage 1** | Raw Data Processing | Unstructured text | Structured pain points |
| **Stage 2** | Opportunity Analysis | Pain points | Solutions, feasibility |
| **Stage 3** | Market Research | Opportunities | Competitors, market size |
| **Stage 4** | Validation & Scoring | Research | Validation, scores |
| **Stage 5** | Output Generation | All analysis | User-facing output |

### 7.3 Stage 1: Raw Data Processing

**Purpose:** Extract structured data from unstructured sources

#### Pipeline 1.1: Pain Point Extractor

| Attribute | Value |
|-----------|-------|
| **ID** | `pain_point_extractor` |
| **Purpose** | Extract core problem statements from raw text |
| **Input** | Raw text content + metadata |
| **Output** | Structured pain point object |
| **Default Provider** | OpenAI gpt-4-turbo |
| **Temperature** | 0.3 (focused) |
| **Max Tokens** | 2000 |

**System Prompt Template:**
```
You are an expert product analyst. Extract pain points from user-generated content.

For each input, extract:
1. The core problem statement (concise summary)
2. Intent type: problem, wish, complaint, question, or other
3. Problem domain: technical, workflow, financial, operational, ux, or other
4. Exact quote from source text

Return JSON format:
{
  "problem": "Core issue in one sentence",
  "intent": "problem|wish|complaint|question|other",
  "domain": "technical|workflow|financial|operational|ux|other",
  "raw_quote": "Exact user text",
  "confidence": 0.0-1.0
}
```

**Output Schema:**
```json
{
  "problem": "CRM contact export is unreliable and complicated",
  "intent": "problem",
  "domain": "workflow",
  "raw_quote": "I hate how I can't easily export my contacts from my CRM",
  "confidence": 0.92
}
```

#### Pipeline 1.2: Emotional Severity Assessor

| Attribute | Value |
|-----------|-------|
| **ID** | `emotional_severity_assessor` |
| **Purpose** | Rate pain/urgency from user engagement signals |
| **Input** | Pain point + engagement metrics (upvotes, comments, etc.) |
| **Output** | Severity score 1-10 + reasoning |
| **Default Provider** | Anthropic claude-3-sonnet |
| **Temperature** | 0.2 (consistent) |
| **Max Tokens** | 1500 |

**System Prompt Template:**
```
Assess the emotional severity and urgency of a reported problem.

Consider:
- Language intensity (words like "hate", "urgent", "desperate")
- Engagement metrics (upvotes, comments shares)
- Frequency of mentions
- User persistence (multiple posts, follow-ups)

Rate severity: 1 (minor annoyance) to 10 (critical/blocking)

Return JSON:
{
  "severity_score": 1-10,
  "urgency_level": "low|medium|high|critical",
  "emotional_indicators": ["list", "of", "signals"],
  "reasoning": "Brief explanation",
  "confidence": 0.0-1.0
}
```

#### Pipeline 1.3: Problem Classifier

| Attribute | Value |
|-----------|-------|
| **ID** | `problem_classifier` |
| **Purpose** | Categorize problem into domain taxonomy |
| **Input** | Pain point text |
| **Output** | Primary category + secondary tags |
| **Default Provider** | ZAI GLM-4.7 |
| **Temperature** | 0.3 |
| **Max Tokens** | 1500 |

**System Prompt Template:**
```
Classify the problem into a domain category.

Primary Categories:
- SaaS/B2B Software
- E-commerce/Retail
- Mobile Apps
- Developer Tools
- Marketing/Sales
- Operations/Workflow
- Finance/Banking
- Healthcare
- Education
- Other (specify)

Return JSON:
{
  "primary_category": "category_name",
  "secondary_tags": ["tag1", "tag2", "tag3"],
  "target_audience": "b2b|b2c|both",
  "technical_complexity": "low|medium|high",
  "confidence": 0.0-1.0
}
```

#### Pipeline 1.4: Context Enricher

| Attribute | Value |
|-----------|-------|
| **ID** | `context_enricher` |
| **Purpose** | Add context from source, thread, surrounding content |
| **Input** | Pain point + source metadata |
| **Output** | Enriched pain point with context |
| **Default Provider** | OpenAI gpt-4-turbo |
| **Temperature** | 0.2 |
| **Max Tokens** | 2000 |

### 7.4 Stage 2: Opportunity Analysis

**Purpose:** Generate solution ideas and assess feasibility

#### Pipeline 2.1: Solution Ideator

| Attribute | Value |
|-----------|-------|
| **ID** | `solution_ideator` |
| **Purpose** | Generate potential solutions for the problem |
| **Input** | Pain point + context |
| **Output** | 3-5 solution concepts with descriptions |
| **Default Provider** | OpenAI gpt-4-turbo |
| **Temperature** | 0.7 (creative) |
| **Max Tokens** | 2500 |

**System Prompt Template:**
```
Generate 3-5 distinct solution concepts for the identified problem.

For each solution:
- Brief description (1-2 sentences)
- Key features (3-5 bullet points)
- Differentiation from existing approaches
- Estimated complexity (low/medium/high)

Return JSON:
{
  "solutions": [
    {
      "id": 1,
      "name": "Solution name",
      "description": "Brief description",
      "key_features": ["feature1", "feature2"],
      "differentiation": "What makes this unique",
      "complexity": "low|medium|high"
    }
  ]
}
```

#### Pipeline 2.2: Complexity Assessor

| Attribute | Value |
|-----------|-------|
| **ID** | `complexity_assessor` |
| **Purpose** | Evaluate technical difficulty, resources, timeline |
| **Input** | Solution concepts |
| **Output** | Complexity analysis per solution |
| **Default Provider** | Anthropic claude-3-opus |
| **Temperature** | 0.3 |
| **Max Tokens** | 2000 |

#### Pipeline 2.3: Market Gap Analyzer

| Attribute | Value |
|-----------|-------|
| **ID** | `market_gap_analyzer` |
| **Purpose** | Identify what existing solutions are missing |
| **Input** | Problem + competitor list (from Stage 3) |
| **Output** | Gap analysis with differentiation opportunities |
| **Default Provider** | OpenAI gpt-4-turbo |
| **Temperature** | 0.4 |
| **Max Tokens** | 2500 |

#### Pipeline 2.4: Differentiation Strategist

| Attribute | Value |
|-----------|-------|
| **ID** | `differentiation_strategist` |
| **Purpose** | Suggest how to make solution unique in market |
| **Input** | Solution + gaps |
| **Output** | Differentiation strategy |
| **Default Provider** | Anthropic claude-3-sonnet |
| **Temperature** | 0.6 |
| **Max Tokens** | 2000 |

### 7.5 Stage 3: Market Research

**Purpose:** Analyze competitive landscape and market size

#### Pipeline 3.1: Competitor Analyzer

| Attribute | Value |
|-----------|-------|
| **ID** | `competitor_analyzer` |
| **Purpose** | Analyze competitor websites, features, pricing |
| **Input** | Search results + scraped content |
| **Output** | Competitor analysis with feature matrix |
| **Default Provider** | OpenAI gpt-4-turbo |
| **Temperature** | 0.3 |
| **Max Tokens** | 3000 |

**System Prompt Template:**
```
Analyze the following competitor information.

For each competitor:
- Name and website
- Key features (what they do)
- Pricing model (if available)
- Target market
- Strengths
- Weaknesses/gaps

Return JSON:
{
  "competitors": [
    {
      "name": "Competitor name",
      "website": "URL",
      "key_features": ["feature1", "feature2"],
      "pricing_model": "subscription|one-time|freemium|unknown",
      "price_range": "$X-$Y" or "unknown",
      "target_market": "description",
      "strengths": ["strength1", "strength2"],
      "weaknesses": ["gap1", "gap2"]
    }
  ],
  "market_saturation": "low|medium|high",
  "common_features": ["feature1", "feature2"],
  "price_range_observed": "$low-$high"
}
```

#### Pipeline 3.2: Market Sizer

| Attribute | Value |
|-----------|-------|
| **ID** | `market_sizer` |
| **Purpose** | Estimate TAM/SAM/SOM from available data |
| **Input** | Problem domain + competitor revenue data |
| **Output** | Market size estimates with reasoning |
| **Default Provider** | Anthropic claude-3-opus |
| **Temperature** | 0.3 |
| **Max Tokens** | 2000 |

#### Pipeline 3.3: Trend Analyzer

| Attribute | Value |
|-----------|-------|
| **ID** | `trend_analyzer` |
| **Purpose** | Assess market growth, timing, trends |
| **Input** | Search volume trends + competitor growth |
| **Output** | Trend analysis with timing assessment |
| **Default Provider** | ZAI GLM-4.7 |
| **Temperature** | 0.3 |
| **Max Tokens** | 2000 |

#### Pipeline 3.4: Customer Persona Builder

| Attribute | Value |
|-----------|-------|
| **ID** | `customer_persona_builder` |
| **Purpose** | Create ideal customer profiles |
| **Input** | Problem + domain + market data |
| **Output** | Customer personas with pain points |
| **Default Provider** | OpenAI gpt-4-turbo |
| **Temperature** | 0.5 |
| **Max Tokens** | 2000 |

### 7.6 Stage 4: Validation & Scoring

**Purpose:** Validate demand and generate final scores

#### Pipeline 4.1: Revenue Validator

| Attribute | Value |
|-----------|-------|
| **ID** | `revenue_validator` |
| **Purpose** | Confirm proof people will pay |
| **Input** | Competitor data + market signals |
| **Output** | Validation assessment with confidence |
| **Default Provider** | Anthropic claude-3-opus |
| **Temperature** | 0.2 |
| **Max Tokens** | 2000 |

#### Pipeline 4.2: Feasibility Scorer

| Attribute | Value |
|-----------|-------|
| **ID** | `feasibility_scorer` |
| **Purpose** | Multi-factor buildability assessment |
| **Input** | All previous analysis |
| **Output** | Feasibility score with reasoning |
| **Default Provider** | OpenAI gpt-4-turbo |
| **Temperature** | 0.2 |
| **Max Tokens** | 2500 |

#### Pipeline 4.3: Risk Assessor

| Attribute | Value |
|-----------|-------|
| **ID** | `risk_assessor` |
| **Purpose** | Identify potential failure points |
| **Input** | Solution + market + competitive analysis |
| **Output** | Risk assessment with mitigation strategies |
| **Default Provider** | Anthropic claude-3-sonnet |
| **Temperature** | 0.3 |
| **Max Tokens** | 2000 |

#### Pipeline 4.4: Component Scoring Engine

| Attribute | Value |
|-----------|-------|
| **ID** | `component_scoring_engine` |
| **Purpose** | Analyze all inputs and generate component scores for each factor |
| **Input** | All previous pipeline outputs |
| **Output** | Component scores (0-100 each) + recommendations + reasoning |
| **Default Provider** | Anthropic claude-3-opus |
| **Temperature** | 0.2 |
| **Max Tokens** | 3000 |

**System Prompt Template:**
```
You are a senior product strategist and analyst. Analyze all available data and generate component scores for each factor.

For each factor, provide a score from 0-100 and brief reasoning:

**Components to Score:**
1. Demand Score (0-100): How much interest/need exists?
2. Revenue Score (0-100): Is there proof people will pay?
3. Competition Score (0-100): How saturated is the market? (lower = less competitive)
4. Feasibility Score (0-100): How buildable is this?
5. Timing Score (0-100): Is this the right time to enter?
6. Risk Score (0-100): What's the likelihood of failure? (lower = less risky)

Return JSON:
{
  "component_scores": {
    "demand": {"score": 85, "reasoning": "High mention frequency, growing trend"},
    "revenue": {"score": 70, "reasoning": "3 competitors with £5k+ MRR each"},
    "competition": {"score": 40, "reasoning": "Only 3 direct competitors, market not saturated"},
    "feasibility": {"score": 75, "reasoning": "Requires standard web stack, 2-3 month build"},
    "timing": {"score": 80, "reasoning": "Market growing, recent surge in discussions"},
    "risk": {"score": 30, "reasoning": "Low risk - proven demand, achievable complexity"}
  },
  "overall_assessment": {
    "summary": "Strong opportunity with validated demand and achievable complexity",
    "key_strengths": ["proven revenue", "growing market", "low competition"],
    "key_risks": ["execution risk", "time to market"],
    "recommendation": "build_now|validate_first|avoid",
    "confidence": 0.85
  }
}
```

**Note:** The application layer applies admin-configured weights to these component scores to calculate the final 0-100 score.

### 7.7 Stage 5: Output Generation

**Purpose:** Create user-facing output

#### Pipeline 5.1: Opportunity Summarizer

| Attribute | Value |
|-----------|-------|
| **ID** | `opportunity_summarizer` |
| **Purpose** | Create readable opportunity description |
| **Input** | All analysis results |
| **Output** | Human-readable summary |
| **Default Provider** | OpenAI gpt-4-turbo |
| **Temperature** | 0.5 |
| **Max Tokens** | 1500 |

#### Pipeline 5.2: Key Highlights Extractor

| Attribute | Value |
|-----------|-------|
| **ID** | `key_highlights_extractor` |
| **Purpose** | Pull 3-5 bullet points for quick scan |
| **Input** | All analysis |
| **Output** | Key highlights array |
| **Default Provider** | OpenAI gpt-3.5-turbo |
| **Temperature** | 0.3 |
| **Max Tokens** | 1000 |

#### Pipeline 5.3: Action Plan Generator

| Attribute | Value |
|-----------|-------|
| **ID** | `action_plan_generator` |
| **Purpose** | Create next steps for validation/building |
| **Input** | Recommendation + analysis |
| **Output** | Actionable next steps |
| **Default Provider** | Anthropic claude-3-sonnet |
| **Temperature** | 0.4 |
| **Max Tokens** | 1500 |

#### Pipeline 5.4: Landing Page Copywriter (Optional)

| Attribute | Value |
|-----------|-------|
| **ID** | `landing_page_copywriter` |
| **Purpose** | Generate validation landing page content |
| **Input** | Opportunity + recommendation |
| **Output** | Landing page headline, CTA, benefits |
| **Default Provider** | OpenAI gpt-4-turbo |
| **Temperature** | 0.7 |
| **Max Tokens** | 2000 |

### 7.8 Pipeline Configuration

Each pipeline stage is configurable via admin panel:

```json
{
  "pipeline_id": "opportunity_analysis_full",
  "name": "Complete Opportunity Analysis",
  "description": "5-stage analysis for opportunity evaluation",
  "stages": [
    {
      "stage_id": "pain_point_extractor",
      "enabled": true,
      "provider": "openai",
      "model": "gpt-4-turbo",
      "fallback_providers": ["anthropic", "zai"],
      "temperature": 0.3,
      "max_tokens": 2000,
      "timeout_seconds": 30,
      "retry_attempts": 3,
      "system_prompt": "Custom prompt override (optional)"
    }
    // ... 20 more pipeline configurations
  ],
  "orchestration": {
    "mode": "sequential",
    "continue_on_error": false,
    "cache_results": true,
    "cache_ttl_seconds": 86400
  }
}
```

### 7.9 Source-Specific Prompt Templates

Different data sources require different AI analysis approaches. Each source type has a specialized prompt template optimized for its content structure and signals.

**Reference:** See [Configuration_Reference.md](../Configuration_Reference.md) for complete prompt templates.

#### 7.9.1 Discussion Source Prompts

**Reddit Analysis Prompt:**
```python
REDDIT_PROMPT = """
You are analyzing a Reddit post for opportunity discovery.

POST CONTENT:
{content}

POST METADATA:
- Subreddit: {subreddit}
- Upvotes: {upvotes}
- Comments: {comment_count}
- Posted: {timestamp}

TASK:
1. Classify business type: Is this B2B or B2C? Look for:
   - B2B indicators: "my boss", "our team", "company", "client", "agency"
   - B2C indicators: "I", "personal", "home", "for myself"

2. Extract the core problem being discussed (one sentence).

3. Extract any mentioned tools, platforms, or workarounds (list).

4. Score components (0-100 each):
   - DEMAND: How many people care? (engagement + specificity)
   - PAIN: How urgent is this problem? (emotion + frequency)
   - FEASIBILITY: How solvable is this technically?
   - B2B_POTENTIAL: Likelihood this is B2B opportunity (0-100)

5. Identify signals:
   - Multi-tool workaround mentioned? (yes/no + which tools)
   - Willingness to pay mentioned? (yes/no + evidence)
   - Existing solutions mentioned? (yes/no + which ones)

OUTPUT JSON:
{
    "business_type": "B2B|B2C|Both|Unclear",
    "category": "SaaS|E-commerce|DevTool|Other",
    "core_problem": "...",
    "mentioned_tools": ["...", "..."],
    "component_scores": {
        "demand": 0-100,
        "pain": 0-100,
        "feasibility": 0-100,
        "b2b_potential": 0-100
    },
    "signals": {
        "multi_tool_workaround": {"found": true, "tools": [...]},
        "willingness_to_pay": {"found": true, "evidence": "..."},
        "existing_solutions": {"found": true, "solutions": [...]}
    },
    "maturity_level": "Emerging|Growing|Established"
}
"""
```

**Indie Hackers Analysis Prompt:**
```python
INDIEHACKERS_PROMPT = """
You are analyzing an Indie Hackers post for opportunity discovery.

POST CONTENT:
{content}

POST METADATA:
- Post type: {post_type}  # "question", "showcase", "discussion"
- Upvotes: {upvotes}
- Comments: {comment_count}

TASK:
1. Identify if this is a problem statement or product showcase.
   - Problem: Looking for solutions, asking "how to"
   - Showcase: Showing what they built, may indicate validated niche

2. Classify business type and category.

3. For showcase posts: Extract revenue metrics if mentioned (MRR, users, etc.)

4. Score components (0-100):
   - DEMAND: Community engagement + topic relevance
   - VALIDATION: Evidence of paying customers or strong interest
   - FEASIBILITY: Technical complexity assessment

OUTPUT JSON:
{
    "post_type": "problem|showcase|discussion",
    "business_type": "B2B|B2C|Both|Unclear",
    "category": "...",
    "revenue_signals": {
        "found": true,
        "metrics": {"mrr": ..., "users": ...}
    },
    "component_scores": {...}
}
"""
```

**Hacker News Analysis Prompt:**
```python
HACKERNEWS_PROMPT = """
You are analyzing a Hacker News post/comment for opportunity discovery.

CONTENT:
{content}

METADATA:
- Type: {type}  # "story" or "comment"
- Points: {points}
- Comment count: {comment_count}

TASK:
1. Distinguish between:
   - Technical pain points (developers struggling)
   - Tool requests (asking for recommendations)
   - Market discussions (industry trends)

2. Classify as B2B (developer tools, infrastructure) or B2C (consumer apps).

3. Extract technical context: languages, frameworks, platforms mentioned.

4. Score components:
   - DEMAND: Engagement + technical specificity
   - PAIN: Frustration indicators (language, workarounds)
   - FEASIBILITY: Based on technical context

OUTPUT JSON:
{
    "content_type": "pain_point|tool_request|market_discussion",
    "business_type": "B2B|B2C|Both",
    "technical_context": ["python", "api", "microservices"],
    "component_scores": {...}
}
"""
```

#### 7.9.2 Search Source Prompts

**Google Trends Analysis Prompt:**
```python
GOOGLE_TRENDS_PROMPT = """
You are analyzing Google Trends search data for opportunity discovery.

SEARCH TERM: {query}
SEARCH VOLUME: {volume}
VELOCITY: {velocity}x growth (vs 90 days ago)
GEOGRAPHIC DISTRIBUTION: {geographies}

TASK:
1. Assess commercial intent: Is this informational curiosity or potential commercial interest?
   - Look for "how to", "tutorial", "best" = learning interest
   - Look for "software", "tool", "platform" = solution-seeking interest

2. Score components (0-100 each):
   - DEMAND_VELOCITY: Growth rate score (higher velocity = higher score)
   - DEMAND_VOLUME: Absolute volume score
   - COMMERCIAL_INTENT: Likelihood this translates to purchases

3. Cross-reference hypothesis: If this term is trending, what type of product would satisfy this search intent?

OUTPUT JSON:
{
    "commercial_intent": 0-100,
    "intent_type": "learning|solution_seeking|comparison",
    "hypothesis": "People searching for this are looking for...",
    "component_scores": {
        "demand_velocity": 0-100,
        "demand_volume": 0-100,
        "commercial_intent": 0-100
    }
}
"""
```

**Competition Search Prompt:**
```python
COMPETITION_SEARCH_PROMPT = """
You are analyzing search results for existing competitors.

SEARCH RESULTS:
{results}

TASK:
1. Identify competitor types:
   - Established SaaS companies (funded, multi-year)
   - Indie projects (single developer, minimal features)
   - Marketplaces/templated solutions (low-end)

2. Assess market saturation:
   - How many direct competitors?
   - Are there clear market leaders?
   - Is there differentiation opportunity?

3. Score components:
   - COMPETITION: Inverse of saturation (fewer competitors = higher score)
   - REVENUE_POTENTIAL: Based on competitor pricing tiers
   - DIFFERENTIATION_OPPORTUNITY: Gaps in competitor features

OUTPUT JSON:
{
    "competitor_count": 15,
    "competitor_types": {
        "established_saas": 5,
        "indie_projects": 8,
        "marketplaces": 2
    },
    "market_saturation": "low|medium|high",
    "differentiation_gaps": ["feature_x", "segment_y"],
    "component_scores": {
        "competition": 40,  # Lower = less crowded
        "revenue": 70,
        "differentiation": 80
    }
}
"""
```

#### 7.9.3 Launch Source Prompts

**Product Hunt Analysis Prompt:**
```python
PRODUCT_HUNT_PROMPT = """
You are analyzing a Product Hunt launch for opportunity discovery.

PRODUCT:
{name}
TAGLINE: {tagline}
DESCRIPTION: {description}

LAUNCH METRICS:
- Upvotes: {upvotes}
- Comments: {comments}
- Launch date: {date}

TASK:
1. Assess validation level:
   - High upvotes + comments = validated interest
   - Recent launch = emerging market
   - Multiple similar launches = established category

2. Extract value proposition and target audience.

3. Score components:
   - DEMAND: Launch engagement (upvotes + comments)
   - REVENUE: Pricing tier mentioned (or estimate based on category)
   - COMPETITION: How similar to previous launches?

OUTPUT JSON:
{
    "validation_level": "emerging|growing|established",
    "target_audience": "developers|founders|consumers",
    "value_proposition": "...",
    "component_scores": {
        "demand": 75,
        "revenue": 60,
        "competition": 50
    },
    "similar_launches_count": 3
}
"""
```

#### 7.9.4 Prompt Template Storage

All source-specific prompts are stored in `ai_prompts` table:

```sql
-- Reddit analysis prompt
INSERT INTO ai_prompts (prompt_id, version, content, variables, status) VALUES
('reddit_analysis_system', '1.0.0', 'You are analyzing a Reddit post...', '["content", "subreddit", "upvotes", "comment_count"]', 'active');

-- Google Trends analysis prompt
INSERT INTO ai_prompts (prompt_id, version, content, variables, status) VALUES
('google_trends_analysis_system', '1.0.0', 'You are analyzing Google Trends data...', '["query", "volume", "velocity", "geographies"]', 'active');

-- Product Hunt analysis prompt
INSERT INTO ai_prompts (prompt_id, version, content, variables, status) VALUES
('product_hunt_analysis_system', '1.0.0', 'You are analyzing a Product Hunt launch...', '["name", "tagline", "description", "upvotes", "comments"]', 'active');
```

**Prompt Retrieval by Source:**
```python
async def get_source_prompt(source_type: str) -> str:
    """Retrieve the AI prompt template for a specific source type."""
    prompt_id = f"{source_type}_analysis_system"
    prompt = await db.execute(
        "SELECT content FROM ai_prompts WHERE prompt_id = ? AND status = 'active' ORDER BY version DESC LIMIT 1",
        prompt_id
    )
    return prompt["content"]
```

---

### 7.10 Pipeline-to-Source Binding

The platform supports binding specific analysis pipelines to data sources, enabling flexible AI analysis per source type.

#### 7.10.1 Binding Configuration

Each data source can be configured to use a specific pipeline:

```json
{
  "source_id": "reddit",
  "bound_pipeline_id": "discussion_source_analysis",
  "pipeline_config": {
    "stages": [
      {
        "stage_id": "source_classification",
        "enabled": true,
        "provider": "openai",
        "model": "gpt-4o",
        "system_prompt_id": "reddit_analysis_system"
      },
      {
        "stage_id": "component_scoring",
        "enabled": true,
        "provider": "openai",
        "model": "gpt-4o-mini"
      },
      {
        "stage_id": "signal_extraction",
        "enabled": true,
        "provider": "anthropic",
        "model": "claude-3.5-sonnet"
      }
    ]
  }
}
```

#### 7.10.2 Default vs Custom Pipelines

**Default Pipeline Behavior:**
- If no pipeline is explicitly bound, use `default_analysis_pipeline`
- Default pipeline includes: classification, scoring, signal extraction

**Custom Pipeline Binding:**
- Admin can bind specialized pipelines per source
- Example: `google_trends` source uses `search_volume_analysis` pipeline (velocity-focused)
- Example: `reddit` source uses `discussion_sentiment_analysis` pipeline (pain-focused)

#### 7.10.3 Pipeline Binding Schema

**Database Schema:**
```sql
CREATE TABLE source_pipeline_bindings (
    binding_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    source_id VARCHAR(50) NOT NULL REFERENCES data_sources(source_id),
    pipeline_id VARCHAR(100) NOT NULL REFERENCES ai_pipeline_configs(pipeline_id),
    priority INTEGER DEFAULT 0,
    enabled BOOLEAN DEFAULT true,
    binding_config JSONB,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW(),
    UNIQUE(source_id)
);
```

**API Endpoints:**
```
POST   /api/v1/admin/sources/{source_id}/bind-pipeline
GET    /api/v1/admin/sources/{source_id}/pipeline-binding
DELETE /api/v1/admin/sources/{source_id}/pipeline-binding
```

#### 7.10.4 Pipeline Selection Logic

```python
async def get_pipeline_for_source(source_id: str) -> str:
    """Get the bound pipeline for a source, or default if none bound."""
    binding = await db.fetch_one(
        "SELECT pipeline_id FROM source_pipeline_bindings WHERE source_id = ? AND enabled = true",
        source_id
    )
    if binding:
        return binding["pipeline_id"]
    return "default_analysis_pipeline"
```

#### 7.10.5 Pipeline Binding Examples

**Example 1: Reddit → Discussion Analysis Pipeline**
```json
{
  "source_id": "reddit",
  "pipeline_id": "discussion_source_analysis",
  "binding_config": {
    "prompt_overrides": {
      "classification": "reddit_analysis_system",
      "scoring": "component_scoring_v2"
    },
    "enable_correlation_check": true,
    "min_confidence_threshold": 60
  }
}
```

**Example 2: Google Trends → Search Volume Pipeline**
```json
{
  "source_id": "google_trends",
  "pipeline_id": "search_volume_analysis",
  "binding_config": {
    "prompt_overrides": {
      "classification": "google_trends_analysis_system"
    },
    "velocity_weight_multiplier": 1.5,
    "enable_trend_comparison": true
  }
}
```

**Example 3: Product Hunt → Launch Validation Pipeline**
```json
{
  "source_id": "product_hunt",
  "pipeline_id": "launch_validation_analysis",
  "binding_config": {
    "prompt_overrides": {
      "classification": "product_hunt_analysis_system"
    },
    "competitor_lookup_enabled": true,
    "similar_launches_window_days": 30
  }
}
```

---

## 8. Prompt Management System

### 8.1 Prompt Versioning

All prompts are versioned:

```json
{
  "prompt_id": "pain_point_extractor_system",
  "version": "1.2.0",
  "created_at": "2026-01-15T10:30:00Z",
  "created_by": "admin@example.com",
  "content": "You are an expert product analyst...",
  "variables": [
    {
      "name": "analysis_depth",
      "type": "select",
      "options": ["basic", "detailed", "comprehensive"],
      "default": "detailed"
    }
  ],
  "status": "active",
  "previous_versions": ["1.1.0", "1.0.0"]
}
```

### 8.2 Prompt Testing

Admin can test prompts before deployment:

```
Input: Test data
Provider: Select provider
Model: Select model
Parameters: Configure temp, tokens

→ Run Test
→ See output
→ Compare cost/tokens
→ A/B test against previous version
```

### 8.3 Prompt A/B Testing

Run two prompt versions simultaneously:

```json
{
  "experiment_id": "pain_point_extractor_v1_vs_v2",
  "control": {
    "prompt_version": "1.1.0",
    "traffic_percentage": 50
  },
  "variant": {
    "prompt_version": "1.2.0",
    "traffic_percentage": 50
  },
  "success_metrics": [
    "confidence_score",
    "output_quality",
    "cost_per_request"
  ],
  "status": "running",
  "started_at": "2026-01-18T00:00:00Z",
  "duration_days": 7
}
```

---

## 9. Data Source Management

### 9.1 Source Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│  DATA SOURCE MANAGEMENT                                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │  ACTIVE SOURCES                                            │ │
│  │  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐     │ │
│  │  │ Reddit   │ │ Product  │ │SerpAPI   │ │Indie      │     │ │
│  │  │          │ │ Hunt     │ │          │ │Hackers    │     │ │
│  │  └──────────┘ └──────────┘ └──────────┘ └──────────┘     │ │
│  │  ┌──────────┐ ┌──────────┐ ┌──────────┐                  │ │
│  │  │ Hacker   │ │ Microns  │ │ [Custom] │                  │ │
│  │  │ News     │ │          │ │ Sources  │                  │ │
│  │  └──────────┘ └──────────┘ └──────────┘                  │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │  SOURCE TEMPLATES (Quick Add)                              │ │
│  │  LinkedIn │ Facebook │ YouTube │ Quora │ Stack Overflow   │ │
│  │ Discourse │ TikTok │ Instagram │ Discord │ [Custom]        │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │  COLLECTION ENGINE                                          │ │
│  │  ┌──────────────────────────────────────────────────────┐  │ │
│  │  │ Scheduler │ Queue │ Workers │ Rate Limiter │ Parser │  │ │
│  │  └──────────────────────────────────────────────────────┘  │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 9.2 Source Configuration Schema

```json
{
  "source_id": "reddit_main",
  "name": "Reddit",
  "type": "api",
  "enabled": true,
  "priority": 1,
  "config": {
    "endpoint": "https://oauth.reddit.com",
    "auth_type": "oauth2",
    "credentials": {
      "client_id": "encrypted_value",
      "client_secret": "encrypted_value",
      "user_agent": "OpportunityFinder/1.0"
    },
    "collection_params": {
      "subreddits": ["Entrepreneur", "SaaS", "IndieHackers"],
      "post_limit": 100,
      "comment_depth": 3
    },
    "rate_limits": {
      "requests_per_minute": 60,
      "posts_per_scan": 100
    }
  },
  "keywords": [
    "I wish", "Does anyone know", "How do I",
    "I hate", "Why isn't there", "Need help with"
  ],
  "health": {
    "status": "healthy",
    "last_scan": "2026-01-18T14:30:00Z",
    "last_check": "2026-01-18T16:45:00Z",
    "success_rate": 99.8
  }
}
```

### 9.3 Custom Source Builder

For custom sources, admin specifies:

| Field | Description | Example |
|-------|-------------|---------|
| **Source Name** | Display name | "My Custom Forum" |
| **Source Type** | api, scraper, rss, webhook | api |
| **Base URL** | API endpoint | https://example.com/api/v1 |
| **Auth Type** | bearer_token, api_key, basic, oauth2 | bearer_token |
| **API Key** | Encrypted credential | •••••••••••• |
| **HTTP Method** | GET, POST | GET |
| **Headers** | Custom headers | {"Accept": "application/json"} |
| **Pagination** | Type and parameter | {"type": "cursor", "param": "after"} |
| **Data Extraction** | JSONPath or CSS selectors | Items: $.data.posts |
| **Rate Limit** | Requests per minute | 10 |

### 9.4 Source Health Monitoring

Each source is monitored:

```json
{
  "source_id": "reddit_main",
  "health": {
    "status": "healthy|degraded|unhealthy|disabled",
    "success_rate_24h": 99.8,
    "success_rate_7d": 98.5,
    "last_error": null,
    "last_error_time": null,
    "consecutive_failures": 0,
    "avg_response_time_ms": 234,
    "last_check": "2026-01-18T16:45:00Z"
  }
}
```

Alerts trigger when:
- Success rate < 95%
- Consecutive failures > 3
- Response time > 5 seconds
- Rate limit exceeded

---

## 10. Scoring Integration

### 10.1 Hybrid Scoring Architecture

**AI generates component scores → Rules combine them with weights:**

```
┌─────────────────────────────────────────────────────────────────┐
│  HYBRID SCORING ARCHITECTURE                                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  All Pipeline Outputs                                           │
│  • Pain point + severity                                        │
│  • Solutions + feasibility                                      │
│  • Competitors + market size                                    │
│  • Validation + risks                                          │
│         ↓                                                        │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │  AI Analysis Components (Stage 4.4)                        │ │
│  │  Generates RAW scores for each factor (0-100):             │ │
│  │  • Demand Score: How much interest exists                 │ │
│  │  • Revenue Score: Proof people will pay                    │ │
│  │  • Competition Score: How crowded the market is            │ │
│  │  • Feasibility Score: How buildable it is                  │ │
│  │  • Timing Score: Is this the right time?                    │ │
│  │  • Risk Score: What could go wrong?                         │ │
│  └────────────────────────────────────────────────────────────┘ │
│         ↓                                                        │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │  Scoring Rules (Application Layer)                         │ │
│  │  Admin-configurable weights combine AI scores:             │ │
│  │  Final Score = (Demand × 25%) + (Revenue × 35%)            │ │
│  │              - (Competition × 20%) + (Feasibility × 20%)    │ │
│  └────────────────────────────────────────────────────────────┘ │
│         ↓                                                        │
│  Final Score 0-100 + Recommendation + Confidence               │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 10.2 Score Composition

**AI generates component scores → Rules combine them:**

| AI Component | What AI Analyzes | Output |
|-------------|------------------|--------|
| **Demand Score** | Mention frequency, trend, engagement | 0-100 |
| **Revenue Score** | Competitor MRR, pricing proof | 0-100 |
| **Competition Score** | Number of competitors, saturation | 0-100 (inverted) |
| **Feasibility Score** | Technical complexity, resources needed | 0-100 |
| **Timing Score** | Market growth, trend direction | 0-100 |
| **Risk Score** | Potential failure points | 0-100 (inverted) |

**Rules then apply weights (admin-configurable):**

| Factor | Weight | Example AI Score | Contribution |
|--------|--------|-----------------|-------------|
| Demand | 25% | 85 | 85 × 0.25 = 21.25 |
| Revenue | 35% | 70 | 70 × 0.35 = 24.50 |
| Competition | 20% | 40 | (100-40) × 0.20 = 12.00 |
| Feasibility | 20% | 75 | 75 × 0.20 = 15.00 |

**Final Score Calculation:**
```
Final Score = (Demand × 0.25) + (Revenue × 0.35) + ((100-Competition) × 0.20) + (Feasibility × 0.20)

Example: (85 × 0.25) + (70 × 0.35) + ((100-40) × 0.20) + (75 × 0.20)
         = 21.25 + 24.50 + 12.00 + 15.00
         = 72.75
```

**Competition and Risk are inverted** (lower = better):
- Original AI score: 40 for competition, 30 for risk
- Inverted for formula: (100-40) = 60, (100-30) = 70
- Higher inverted score = less competition/risk = better opportunity

**Admin Controls:**
- Adjust weights via admin panel (e.g., prioritize revenue over demand)
- Add/remove scoring factors
- Set minimum thresholds for each factor
- **Manual Override:** Admin can manually override any AI-generated component score
  - Enables adding "manual bias" when needed
  - Manual override becomes the source of truth
  - Audit log tracks who changed what and when

| Factor | How AI Weighs It |
|--------|-----------------|
| **Problem Severity** | High severity → higher score |
| **Market Size** | Larger market → higher score |
| **Competition** | Less competition → higher score |
| **Revenue Validation** | Competitors making money → higher score |
| **Feasibility** | More buildable → higher score |
| **Timing** | Growing market → higher score |
| **Risk** | Lower risk → higher score |

### 10.3 Score Ranges

| Score Range | Label | Meaning |
|-------------|-------|---------|
| 80-100 | Excellent | Build immediately, high confidence |
| 60-79 | Good | Validate first, likely worth building |
| 40-59 | Maybe | Needs research, moderate potential |
| 0-39 | Reject | Poor opportunity, avoid |

### 10.4 Confidence Levels

Each score includes AI's confidence:

| Level | Range | Meaning |
|-------|-------|---------|
| **High** | 0.8-1.0 | AI is very confident in assessment |
| **Medium** | 0.5-0.8 | AI is moderately confident |
| **Low** | 0.2-0.5 | AI has low confidence, more research needed |
| **Very Low** | 0.0-0.2 | Insufficient data, score unreliable |

### 10.5 Score Display (User-Facing)

**Default User Experience:**
- Users see **final score only** (0-100)
- No breakdown of components shown by default
- Users trust the system through successful use, not transparency

**Admin-Optional Transparency:**
- Admin can enable "Show Score Breakdown" for debugging/analytics
- When enabled, displays: AI component scores + weight formula
- This is an **admin toggle**, not shown to end users by default
- Full details always logged internally for admin/audit purposes

### 10.6 Validation Rules (Admin-Configurable)

**All validation criteria are admin-configurable, not hardcoded.**

**Separation of Concerns:**

| Layer | Purpose | Example |
|-------|---------|---------|
| **AI Analysis** | Generates insights and component scores | "Demand: 85/100, 3 competitors with £5k+ MRR" |
| **Validation Rules** | Hard checks against configurable thresholds | "Has paid solution? ✓ MRR > £1k? ✓" |
| **Scoring Rules** | Weighted combination → 0-100 score | Formula with admin-configurable weights |
| **Recommendation** | Combines score + validation → action | "Build now" vs "Validate first" |

**Admin-Configurable Validation Thresholds:**

| Criterion | Default Threshold | Admin Can Adjust To |
|-----------|------------------|---------------------|
| Minimum MRR | £1,000 | Any amount |
| Minimum mentions | 20 | Any number |
| Requires paid solution | Yes | No |
| Competitor count limit | 50 | Any number |
| B2B requirement | Yes | No |

**Why Separation Matters:**
- An opportunity can score 90/100 but fail validation (no revenue proof)
- An opportunity can score 50/100 but pass validation (has revenue, lower score)
- Validation = "Is this real?" Scoring = "How good is it?"
- Recommendation considers both

---

## 11. Admin Panel Specification

### 11.1 Admin Panel Navigation

```
/admin
├── /admin/dashboard (Analytics Overview)
├── /admin/ai-services (AI Configuration)
│   ├── /admin/ai-services/providers (Provider Management)
│   ├── /admin/ai-services/pipelines (Pipeline Configuration)
│   ├── /admin/ai-services/prompts (Prompt Management)
│   ├── /admin/ai-services/costs (Cost Tracking)
│   ├── /admin/ai-services/fallback (Fallback Rules)
│   ├── /admin/ai-services/feature-flags (Feature Flags)
│   └── /admin/ai-services/testing (Testing Sandbox)
├── /admin/data-sources (Data Source Management)
│   ├── /admin/data-sources/active (Active Sources)
│   ├── /admin/data-sources/templates (Source Templates)
│   ├── /admin/data-sources/custom (Custom Source Builder)
│   ├── /admin/data-sources/schedule (Collection Schedule)
│   └── /admin/data-sources/health (Health Monitoring)
├── /admin/users (User Management)
├── /admin/pricing (Pricing Tiers)
├── /admin/scoring (Scoring Configuration)
├── /admin/scans (Scan Settings)
├── /admin/emails (Email Configuration)
└── /admin/system-settings (System Configuration)
```

### 11.2 AI Services Section

#### Provider Management Page

```
┌─────────────────────────────────────────────────────────────────────┐
│  AI PROVIDERS                                                        │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌────────────────────────────────────────────────────────────────┐ │
│  │  OPENAI                                          [Enabled ✓]     │ │
│  │  ────────────────────────────────────────────────────────────  │ │
│  │  API Key: sk-••••••••••••••• [Rotate] [Test Connection]        │ │
│  │  Organization: org-•••••••••                                    │ │
│  │                                                                  │ │
│  │  Models:                                                         │ │
│  │  ┌────────────────────────────────────────────────────────────┐ │ │
│  │  │ Model          │ Enbld │ Pri │ Tokens/Mo  │ Used   │ Cost││ │ │
│  │  │ gpt-4-turbo    │  ✓   │  1  │   10M     │  4.5M  │ $45 ││ │ │
│  │  │ gpt-4          │  ✓   │  2  │   5M      │  1.5M  │ $30 ││ │ │
│  │  │ gpt-3.5-turbo  │  ✓   │  3  │   50M     │  12.3M │ $6  ││ │ │
│  │  └────────────────────────────────────────────────────────────┘ │ │
│  │                                                                  │ │
│  │  Rate Limits: Requests/Min [1000]  Tokens/Min [150000]          │ │
│  │  Budget: $500/month  Current: $234 (47%)                        │ │
│  │                                                                  │ │
│  │  [Configure Models] [View Usage] [Edit Limits]                  │ │
│  └────────────────────────────────────────────────────────────────┘ │
│                                                                      │
│  [+ Add New Provider]                                                │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

#### Pipeline Configuration Page

```
┌─────────────────────────────────────────────────────────────────────┐
│  ANALYSIS PIPELINES                                                  │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  STAGE 1: RAW DATA PROCESSING                                        │
│  ┌────────────────────────────────────────────────────────────────┐ │
│  │ 1.1 Pain Point Extractor                                       │ │
│  │ Provider: [OpenAI ▼]  Model: [gpt-4-turbo ▼]                    │ │
│  │ Fallback: [Anthropic ▼] → [ZAI ▼]                              │ │
│  │ Temperature: [0.3]  Max Tokens: [2000]                          │ │
│  │ System Prompt: [View/Edit...] [Test] [Reset] [Save]             │ │
│  └────────────────────────────────────────────────────────────────┘ │
│                                                                      │
│  [21+ pipeline configurations - one per stage]                       │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### 11.3 Data Sources Section

```
┌─────────────────────────────────────────────────────────────────────┐
│  DATA SOURCE MANAGEMENT                                              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ACTIVE SOURCES                                                      │
│  ┌────────────────────────────────────────────────────────────────┐ │
│  │ REDDIT [Enabled ✓]                                              │ │
│  │ Last Scan: 2 hours ago  Items: 47  Status: Healthy             │ │
│  │ [Configure] [View Logs] [Edit] [Disable]                        │ │
│  └────────────────────────────────────────────────────────────────┘ │
│                                                                      │
│  [+ Add New Source]                                                  │
│                                                                      │
│  QUICK ADD TEMPLATES                                                 │
│  [LinkedIn] [Facebook] [YouTube] [Quora] [Stack Overflow]           │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 12. API Specification

### 12.1 Base URL

```
https://ai-platform.example.com/api/v1
```

### 12.2 Authentication

All requests require JWT token:

```
Authorization: Bearer <jwt_token>
```

### 12.3 Core Endpoints

#### Analyze with Pipeline

```http
POST /analyze/pipeline
Content-Type: application/json

{
  "pipeline": "opportunity_analysis_full",
  "input": {
    "raw_data": "I wish there was a better way to...",
    "source": "reddit",
    "metadata": {
      "subreddit": "Entrepreneur",
      "upvotes": 45,
      "comments": 12
    }
  },
  "options": {
    "include_reasoning": true,
    "cache": true
  }
}

Response:
{
  "analysis_id": "uuid",
  "status": "complete",
  "results": {
    "pain_point": {...},
    "solutions": [...],
    "competitors": [...],
    "market_size": {...},
    "score": 78,
    "recommendation": "validate_first",
    "reasoning": [...]
  },
  "pipeline_info": {
    "duration_ms": 2340,
    "stages_completed": 21,
    "providers_used": ["openai", "anthropic", "zai"]
  },
  "cost": {
    "total_cost": 0.023,
    "by_provider": {
      "openai": 0.015,
      "anthropic": 0.006,
      "zai": 0.002
    },
    "tokens_used": 8470
  }
}
```

#### Get Analysis Status

```http
GET /analyze/{analysis_id}

Response:
{
  "analysis_id": "uuid",
  "status": "running|complete|failed",
  "progress": {
    "current_stage": "competitor_analyzer",
    "stages_completed": 15,
    "total_stages": 21
  },
  "started_at": "2026-01-18T14:30:00Z",
  "estimated_completion": "2026-01-18T14:32:00Z"
}
```

### 12.4 Admin Endpoints

#### Provider Management

```http
# List Providers
GET /admin/providers

# Add Provider
POST /admin/providers
{
  "provider_id": "openai",
  "provider_name": "OpenAI",
  "api_endpoint": "https://api.openai.com/v1",
  "api_key": "sk-...",
  "enabled": true,
  "priority": 1
}

# Update Provider
PATCH /admin/providers/{provider_id}

# Delete Provider
DELETE /admin/providers/{provider_id}

# Test Provider Connection
POST /admin/providers/{provider_id}/test
```

#### Pipeline Configuration

```http
# List Pipelines
GET /admin/pipelines

# Get Pipeline Config
GET /admin/pipelines/{pipeline_id}

# Update Pipeline Config
PATCH /admin/pipelines/{pipeline_id}
{
  "stages": {
    "pain_point_extractor": {
      "provider": "anthropic",
      "model": "claude-3-opus",
      "temperature": 0.4
    }
  }
}

# Test Pipeline
POST /admin/pipelines/{pipeline_id}/test
{
  "test_input": "Test data..."
}
```

#### Prompt Management

```http
# List Prompts
GET /admin/prompts

# Get Prompt
GET /admin/prompts/{prompt_id}

# Update Prompt
PATCH /admin/prompts/{prompt_id}
{
  "content": "New prompt content...",
  "version": "1.3.0"
}

# Create A/B Test
POST /admin/prompts/{prompt_id}/ab-test
{
  "variant_content": "Alternative prompt...",
  "traffic_percentage": 50,
  "duration_days": 7
}
```

#### Cost Tracking

```http
# Get Cost Summary
GET /admin/costs
  ?period=month
  &provider=openai

Response:
{
  "period": "2026-01",
  "total_cost": 756.32,
  "by_provider": {...},
  "by_pipeline": {...},
  "by_user": {...}
}

# Set Budget Alert
POST /admin/costs/alerts
{
  "provider_id": "openai",
  "alert_threshold": 400,
  "alert_email": "admin@example.com"
}
```

### 12.5 Data Source Endpoints

```http
# List Sources
GET /admin/data-sources

# Add Source
POST /admin/data-sources
{
  "name": "Reddit",
  "type": "api",
  "config": {...}
}

# Update Source
PATCH /admin/data-sources/{source_id}

# Test Source
POST /admin/data-sources/{source_id}/test

# Get Source Health
GET /admin/data-sources/{source_id}/health
```

---

## 13. Database Schema

### 13.1 AI Configuration Tables

#### ai_providers

```sql
CREATE TABLE ai_providers (
    provider_id VARCHAR(50) PRIMARY KEY,
    provider_name VARCHAR(100) NOT NULL,
    api_endpoint VARCHAR(500) NOT NULL,
    api_key_encrypted TEXT NOT NULL,
    organization_id VARCHAR(100),
    enabled BOOLEAN DEFAULT true,
    priority INTEGER DEFAULT 0,
    rate_limits JSONB,
    cost_limits JSONB,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);
```

#### ai_models

```sql
CREATE TABLE ai_models (
    model_id VARCHAR(100) PRIMARY KEY,
    provider_id VARCHAR(50) REFERENCES ai_providers(provider_id),
    model_name VARCHAR(100) NOT NULL,
    enabled BOOLEAN DEFAULT true,
    priority INTEGER DEFAULT 0,
    context_window INTEGER,
    max_output_tokens INTEGER,
    input_cost_per_1k DECIMAL(10,6),
    output_cost_per_1k DECIMAL(10,6),
    supports_function_calling BOOLEAN DEFAULT false,
    supports_streaming BOOLEAN DEFAULT false,
    created_at TIMESTAMP DEFAULT NOW()
);
```

#### ai_pipeline_configs

```sql
CREATE TABLE ai_pipeline_configs (
    pipeline_id VARCHAR(100) PRIMARY KEY,
    pipeline_name VARCHAR(200) NOT NULL,
    description TEXT,
    stages JSONB NOT NULL,
    orchestration JSONB,
    enabled BOOLEAN DEFAULT true,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);
```

#### ai_prompts

```sql
CREATE TABLE ai_prompts (
    prompt_id VARCHAR(100) PRIMARY KEY,
    version VARCHAR(20) NOT NULL,
    content TEXT NOT NULL,
    variables JSONB,
    status VARCHAR(20) DEFAULT 'active',
    created_by VARCHAR(100),
    created_at TIMESTAMP DEFAULT NOW(),
    UNIQUE(prompt_id, version)
);
```

### 13.2 Cost Tracking Tables

#### ai_request_logs

```sql
CREATE TABLE ai_request_logs (
    request_id UUID PRIMARY KEY,
    application_id VARCHAR(100),  -- v1.4: Added for multi-application support
    pipeline_id VARCHAR(100),
    stage_id VARCHAR(50),
    provider_id VARCHAR(50),
    model_id VARCHAR(100),
    input_tokens INTEGER,
    output_tokens INTEGER,
    total_tokens INTEGER,
    cost DECIMAL(10,6),
    latency_ms INTEGER,
    status VARCHAR(20),
    error_message TEXT,
    user_id VARCHAR(100),
    created_at TIMESTAMP DEFAULT NOW()
);
```

#### ai_cost_summaries

```sql
CREATE TABLE ai_cost_summaries (
    summary_id UUID PRIMARY KEY,
    application_id VARCHAR(100),  -- v1.4: Added for multi-application support
    period_type VARCHAR(20),
    period_start TIMESTAMP,
    period_end TIMESTAMP,
    provider_id VARCHAR(50),
    pipeline_id VARCHAR(100),
    total_cost DECIMAL(10,6),
    total_requests INTEGER,
    total_tokens BIGINT,
    created_at TIMESTAMP DEFAULT NOW()
);
```

### 13.3 Data Source Tables

#### data_sources

```sql
CREATE TABLE data_sources (
    source_id VARCHAR(50) PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    type VARCHAR(20) NOT NULL,
    enabled BOOLEAN DEFAULT true,
    priority INTEGER DEFAULT 0,
    config JSONB NOT NULL,
    keywords TEXT[],
    health_status JSONB,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);
```

#### data_collection_runs

```sql
CREATE TABLE data_collection_runs (
    run_id UUID PRIMARY KEY,
    source_id VARCHAR(50) REFERENCES data_sources(source_id),
    started_at TIMESTAMP,
    completed_at TIMESTAMP,
    status VARCHAR(20),
    items_collected INTEGER,
    error_message TEXT,
    created_at TIMESTAMP DEFAULT NOW()
);
```

### 13.4 Application Configuration Tables

#### application_config

**v1.4:** New table for storing application-specific configuration parameters.

This table enables multi-application support by storing configuration per application. Each row represents a single configuration parameter.

```sql
CREATE TABLE application_config (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    application_id VARCHAR(100) NOT NULL,
    config_key VARCHAR(255) NOT NULL,
    config_value JSONB NOT NULL,
    value_type VARCHAR(50) NOT NULL,  -- 'float', 'int', 'string', 'list', 'dict', 'bool'
    category VARCHAR(100),  -- 'scoring', 'validation', 'source', 'ai', 'filter', 'output', 'alert'
    is_admin_configurable BOOLEAN DEFAULT true,
    validation_rule JSONB,  -- Optional validation rules
    description TEXT,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW(),
    UNIQUE(application_id, config_key)
);

-- Indexes for efficient lookup
CREATE INDEX idx_app_config_app_id ON application_config(application_id);
CREATE INDEX idx_app_config_category ON application_config(category);
CREATE INDEX idx_app_config_key ON application_config(config_key);
```

**Configuration Categories:**

| Category | Purpose | Example Keys |
|----------|---------|--------------|
| `scoring` | Scoring weights and thresholds | `weight_demand`, `score_threshold_build` |
| `validation` | Validation rules | `require_paid_solution`, `min_competitor_mrr` |
| `source_reddit` | Reddit-specific settings | `reddit_subreddits`, `reddit_min_upvotes` |
| `source_google_trends` | Google Trends settings | `trends_timeframe`, `trends_geocode` |
| `ai` | AI model configuration | `analysis_model`, `temperature` |
| `filter` | Business type/category filters | `business_type_filter`, `category_filter` |
| `output` | Output configuration | `max_results_returned`, `sort_by` |
| `alert` | Notification settings | `alert_threshold`, `alert_digest_frequency` |

**Example Data:**
```sql
-- Scoring weights
INSERT INTO application_config (application_id, config_key, config_value, value_type, category) VALUES
('opportunity-finder-app-id', 'weight_demand', '0.25', 'float', 'scoring'),
('opportunity-finder-app-id', 'weight_competition', '0.20', 'float', 'scoring'),
('opportunity-finder-app-id', 'weight_revenue', '0.25', 'float', 'scoring');

-- Validation rules
INSERT INTO application_config (application_id, config_key, config_value, value_type, category, validation_rule) VALUES
('opportunity-finder-app-id', 'require_paid_solution', 'true', 'bool', 'validation', NULL),
('opportunity-finder-app-id', 'min_competitor_mrr', '1000', 'int', 'validation', '{"min": 0, "max": 100000}');

-- Reddit source configuration
INSERT INTO application_config (application_id, config_key, config_value, value_type, category) VALUES
('opportunity-finder-app-id', 'reddit_subreddits', '["SaaS", "Entrepreneur"]', 'list', 'source_reddit'),
('opportunity-finder-app-id', 'reddit_min_upvotes', '10', 'int', 'source_reddit');
```

**Reference:** See [Configuration_Reference.md](../Configuration_Reference.md) for complete parameter documentation.

---

### 13.5 Correlation Tables

#### mention_correlations

**v1.4:** New table for storing cross-source correlation data.

This table tracks when multiple sources discuss the same opportunity, enabling correlation strength calculations.

```sql
CREATE TABLE mention_correlations (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    opportunity_id UUID NOT NULL,  -- References opportunities table (in application DB)
    source_type VARCHAR(50) NOT NULL,
    mention_id VARCHAR(255) NOT NULL,  -- References opportunity_mentions.mention_id
    correlation_strength FLOAT NOT NULL,  -- 0.0-1.0
    correlation_method VARCHAR(100),  -- 'entity_overlap', 'semantic_similarity', 'time_proximity'
    metadata JSONB,  -- Additional correlation details
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Indexes for correlation queries
CREATE INDEX idx_correlations_opportunity ON mention_correlations(opportunity_id);
CREATE INDEX idx_correlations_source ON mention_correlations(source_type);
CREATE INDEX idx_correlations_strength ON mention_correlations(correlation_strength DESC);
CREATE INDEX idx_correlations_mention ON mention_correlations(mention_id);
```

**Correlation Methods:**

| Method | Description | Strength Range |
|--------|-------------|----------------|
| `entity_overlap` | Shared extracted entities (tools, platforms) | 0.3-0.7 |
| `semantic_similarity` | AI-powered semantic matching | 0.5-1.0 |
| `time_proximity` | Mentions clustered in time | 0.2-0.5 |
| `combined` | Multiple methods combined | 0.0-1.0 |

**Example Data:**
```sql
-- Same opportunity found on Reddit and Google Trends
INSERT INTO mention_correlations (opportunity_id, source_type, mention_id, correlation_strength, correlation_method) VALUES
('550e8400-e29b-41d4-a716-446655440000', 'reddit', 'reddit_mention_123', 0.85, 'combined'),
('550e8400-e29b-41d4-a716-446655440000', 'google_trends', 'trends_mention_456', 0.82, 'combined');
```

**Related Table (in application database):**
```sql
-- This table exists in the application's database, not the AI platform
CREATE TABLE opportunities (
    id UUID PRIMARY KEY,
    title TEXT NOT NULL,
    final_score FLOAT NOT NULL,
    correlated_sources TEXT[] NOT NULL DEFAULT '{}',
    correlation_strength FLOAT DEFAULT 0.0,
    -- ... other fields
);
```

---

## 14. Security Model

### 14.1 API Key Storage

```
┌─────────────────────────────────────────────────────────────────┐
│  API KEY ENCRYCTION                                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  plaintext_key                                                  │
│       ↓                                                         │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ ENCRYPTION (Application Layer)                             │ │
│  │ • Algorithm: AES-256-GCM                                   │ │
│  │ • Key: Derived from master key (env var)                   │ │
│  │ • IV: Unique per key                                       │ │
│  └────────────────────────────────────────────────────────────┘ │
│       ↓                                                         │
│  encrypted_key (stored in database)                            │
│       ↓                                                         │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ DATABASE ENCRYPTION (At Rest)                              │ │
│  │ • PostgreSQL transparent data encryption (TDE)             │ │
│  │ • Or disk-level encryption                                 │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 14.2 Key Rotation

```python
# Rotation Process
1. Admin requests rotation
2. Generate new key
3. Encrypt new key
4. Store alongside old key (marked as deprecated)
5. Mark old key expires_at = now + 30 days
6. New requests use new key
7. Old key can decrypt existing encrypted data
8. After 30 days, old key permanently deleted
9. Re-encrypt any data still using old key
```

### 14.3 Access Control

| Role | AI Services | Data Sources | Costs | Prompts |
|------|-------------|--------------|-------|---------|
| **Super Admin** | Full | Full | Full | Full |
| **AI Admin** | Full | View | Full | Full |
| **Data Admin** | View | Full | View | View |
| **Billing Admin** | View | View | Full | View |
| **Developer** | View | View | View | View |

### 14.4 Audit Logging

All sensitive actions logged:

```json
{
  "audit_id": "uuid",
  "timestamp": "2026-01-18T14:30:00Z",
  "user_id": "admin@example.com",
  "action": "update_api_key",
  "resource_type": "ai_provider",
  "resource_id": "openai",
  "changes": {
    "before": "sk-...old",
    "after": "sk-...new"
  },
  "ip_address": "192.168.1.100",
  "user_agent": "Mozilla/5.0...",
  "success": true
}
```

---

## 15. Cost Management

### 15.1 Cost Calculation

Real-time cost tracking per request:

```python
def calculate_cost(provider, model, input_tokens, output_tokens):
    model_config = get_model_config(provider, model)
    input_cost = (input_tokens / 1000) * model_config.input_cost_per_1k
    output_cost = (output_tokens / 1000) * model_config.output_cost_per_1k
    return input_cost + output_cost
```

### 15.2 Cost Aggregation

Costs aggregated at multiple levels:

```
Per Request → Per Pipeline → Per Provider → Per User → Per Period
```

### 15.3 Budget Controls

| Control Type | Description |
|--------------|-------------|
| **Provider Budget** | Max spend per provider per month |
| **Pipeline Budget** | Max spend per pipeline per month |
| **User Budget** | Max spend per user per month |
| **Alert Thresholds** | Email alert at X% of budget |
| **Hard Limits** | Stop processing when budget exceeded |

### 15.4 Cost Optimization

Automatic optimization:

```python
def select_provider(pipeline, input_size, budget_remaining):
    # Get available providers for this pipeline
    providers = get_pipeline_providers(pipeline)

    # Filter by priority and rate limits
    available = [p for p in providers if p.enabled and p.within_rate_limits()]

    # Select based on cost if within budget
    if budget_remaining > 0:
        # Choose most cost-effective option that meets quality threshold
        return select_by_cost_effectiveness(available)

    # If over budget, use fallback/cheapest option
    return select_cheapest(available)
```

---

## 16. Error Handling & Fallback

### 16.1 Error Classification

| Error Type | Handling |
|------------|----------|
| **Rate Limit** | Retry after delay, try next provider |
| **Timeout** | Retry once, try next provider |
| **API Error (4xx)** | Log, don't retry |
| **API Error (5xx)** | Retry, try next provider |
| **Network Error** | Retry with exponential backoff |
| **Invalid Response** | Retry, try next provider |

### 16.2 Fallback Chain

```python
fallback_chain = [
    {"provider": "openai", "model": "gpt-4-turbo"},
    {"provider": "anthropic", "model": "claude-3-sonnet"},
    {"provider": "zai", "model": "glm-4.7"},
    {"provider": "open_source", "model": "llama-3-70b"}
]

for option in fallback_chain:
    try:
        result = call_provider(option)
        return result
    except Exception as e:
        log_failure(e)
        continue

# All failed - queue for retry or return error
```

### 16.3 Retry Strategy

```python
retry_config = {
    "max_retries": 3,
    "initial_delay": 1.0,
    "backoff_multiplier": 2.0,
    "max_delay": 30.0
}

# Retry with exponential backoff
for attempt in range(max_retries):
    try:
        return make_request()
    except RetryableError:
        delay = min(initial_delay * (backoff_multiplier ** attempt), max_delay)
        sleep(delay)
```

### 16.4 Dead Letter Queue

Failed requests go to dead letter queue:

```
Failed Request → DLQ → Manual Review → Retry or Discard
```

Admin can view and retry DLQ items via admin panel.

---

## 17. Monitoring & Observability

### 17.1 Metrics Collected

| Category | Metrics |
|----------|---------|
| **Request** | Total, success rate, error rate, latency |
| **Provider** | Requests per provider, success rate, avg latency |
| **Pipeline** | Executions per pipeline, stage success rate |
| **Cost** | Cost per request, cost per provider, trends |
| **Tokens** | Tokens per request, tokens per provider |
| **Queue** | Queue depth, processing time, worker utilization |

### 17.2 Health Checks

```http
GET /health

Response:
{
  "status": "healthy",
  "timestamp": "2026-01-18T14:30:00Z",
  "components": {
    "api": "healthy",
    "database": "healthy",
    "redis": "healthy",
    "queue": "healthy",
    "providers": {
      "openai": "healthy",
      "anthropic": "healthy",
      "zai": "degraded",
      "open_source": "healthy"
    }
  }
}
```

### 17.3 Alerts

Alert triggers:

| Alert | Condition | Action |
|-------|-----------|--------|
| **High Error Rate** | Error rate > 5% | Email + Slack |
| **Provider Down** | Provider failure rate > 50% | Email + Slack |
| **Cost Alert** | Spend > 80% budget | Email |
| **Slow Response** | P95 latency > 10s | Warning |
| **Queue Backlog** | Queue depth > 1000 | Alert |

---

## 18. Testing Strategy

### 18.1 Prompt Testing

Test prompts before deployment:

```python
test_cases = [
    {
        "input": "Test input data",
        "expected_output": {...},
        "evaluation_criteria": ["accuracy", "completeness"]
    }
]

results = run_prompt_test(new_prompt, test_cases)
compare_with_current(results)
```

### 18.2 Provider Testing

Test all providers:

```python
for provider in providers:
    result = provider.test_connection()
    assert result.success
    assert result.latency < threshold
```

### 18.3 Pipeline Testing

End-to-end pipeline tests:

```python
test_input = create_test_data()
result = run_pipeline("opportunity_analysis_full", test_input)

assert result.status == "complete"
assert result.results.score >= 0
assert result.results.score <= 100
assert result.cost > 0
```

### 18.4 Load Testing

Simulate production load:

```python
scenarios = [
    {"concurrent_users": 10, "duration": "5min"},
    {"concurrent_users": 50, "duration": "5min"},
    {"concurrent_users": 100, "duration": "10min"}
]

for scenario in scenarios:
    run_load_test(scenario)
    measure_metrics()
```

---

## 19. Deployment

### 19.1 Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│  PRODUCTION DEPLOYMENT                                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐         │
│  │ Load        │    │ API         │    │ Worker      │         │
│  │ Balancer    │───▶│ Servers     │───▶│ Processes   │         │
│  │ (Nginx)     │    │ (3 replicas)│    │ (5 workers) │         │
│  └─────────────┘    └─────────────┘    └─────────────┘         │
│         │                   │                   │               │
│         ▼                   ▼                   ▼               │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐         │
│  │ Redis       │    │ PostgreSQL  │    │ S3          │         │
│  │ (Cache)     │    │ (Primary)   │    │ (Storage)   │         │
│  └─────────────┘    └─────────────┘    └─────────────┘         │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 19.2 Deployment Checklist

- [ ] Database migrations run
- [ ] Redis configured and running
- [ ] Environment variables set
- [ ] API keys configured (encrypted)
- [ ] SSL certificates installed
- [ ] Load balancer configured
- [ ] Monitoring dashboards created
- [ ] Alert rules configured
- [ ] Backup schedules configured
- [ ] Log aggregation configured

### 19.3 Scaling

| Component | Horizontal Scale | Vertical Scale |
|-----------|------------------|----------------|
| API Servers | ✅ Add replicas | ✅ Increase CPU/RAM |
| Workers | ✅ Add workers | ✅ Increase capacity |
| Database | ✖️ Use read replicas | ✅ Increase instance size |
| Redis | ✅ Cluster mode | ✅ Increase memory |

---

## 20. Roadmap

### 20.1 Phase 1: MVP (Current Scope)

| Feature | Status |
|---------|--------|
| Multi-provider support (OpenAI, Anthropic, ZAI) | ✅ Specified |
| 5-stage pipeline architecture | ✅ Specified |
| 21+ configurable pipelines | ✅ Specified |
| Prompt management system | ✅ Specified |
| Data source management | ✅ Specified |
| Admin panel (all sections) | ✅ Specified |
| Cost tracking | ✅ Specified |
| Fallback & retry | ✅ Specified |
| Basic monitoring | ✅ Specified |

### 20.2 Phase 2: Enhanced Features

| Feature | Description |
|---------|-------------|
| **User Customization** | Users configure their own AI preferences |
| **Advanced Analytics** | Deeper cost and performance insights |
| **Prompt Marketplace** | Share prompts between organizations |
| **Multi-tenancy** | Full multi-tenant isolation |
| **Fine-tuning Support** | Use fine-tuned models |
| **Streaming Responses** | Real-time streaming output |
| **Batch Processing** | Process multiple items efficiently |

### 20.3 Phase 3: Enterprise

| Feature | Description |
|---------|-------------|
| **SSO Integration** | SAML, OAuth for enterprise auth |
| **RBAC Advanced** | Fine-grained permissions |
| **Compliance** | SOC2, HIPAA, GDPR features |
| **Private Deployment** | On-premise deployment option |
| **Custom Models** | Bring your own models |
| **Edge Deployment** | Deploy to edge locations |

---

## 21. Platform Authentication & Authorization

### 21.1 Platform Authentication

The AI Services Platform requires its own authentication system for admin panel access. This is **separate from application authentication** - applications use API keys, while admins use the platform's auth system.

#### Admin User Management

```sql
CREATE TABLE admin_users (
    user_id UUID PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    full_name VARCHAR(200),
    role VARCHAR(50) NOT NULL,
    mfa_enabled BOOLEAN DEFAULT false,
    mfa_secret VARCHAR(255),
    last_login_at TIMESTAMP,
    failed_login_attempts INTEGER DEFAULT 0,
    locked_until TIMESTAMP,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);
```

#### Admin Roles

| Role | Permissions |
|------|-------------|
| **super_admin** | Full access to everything, including user management |
| **ai_admin** | AI configuration (providers, pipelines, prompts) |
| **data_admin** | Data source management |
| **billing_admin** | Cost viewing, budget configuration |
| **operator** | View-only access to dashboards and logs |
| **developer** | API testing, prompt testing, sandbox access |

#### Authentication Flow

```
1. Admin accesses /admin
   ↓
2. Redirects to /admin/login
   ↓
3. Submits email + password
   ↓
4. Platform validates credentials
   - Check if account locked
   - Verify password hash
   - Check 2FA if enabled
   ↓
5. If 2FA enabled:
   - Prompt for TOTP code
   - Verify code
   ↓
6. Generate admin session token (JWT)
   - 15 minute expiry
   - Contains user_id, role, permissions
   - Stored in httpOnly cookie
   ↓
7. Redirect to /admin/dashboard
```

#### Security Requirements

| Requirement | Specification |
|-------------|----------------|
| **Password Hashing** | bcrypt with cost factor 12 |
| **Password Policy** | Min 12 chars, 1 uppercase, 1 lowercase, 1 number, 1 special |
| **Password Expiry** | 90 days, with 14-day grace period |
| **Session Timeout** | 15 minutes of inactivity |
| **Max Concurrent Sessions** | 3 per admin |
| **Brute Force Protection** | 5 failed attempts = 15 min lock, 10 attempts = 1 hour lock |
| **2FA Requirement** | Required for super_admin and ai_admin roles |
| **IP Whitelisting** | Optional per-admin IP whitelist |
| **Session Security** | httpOnly cookies, SameSite=Strict, secure flag (HTTPS only) |

#### Admin Session Management

```json
{
  "session_id": "uuid",
  "user_id": "uuid",
  "email": "admin@example.com",
  "role": "super_admin",
  "permissions": ["all"],
  "login_at": "2026-01-18T14:30:00Z",
  "expires_at": "2026-01-18T14:45:00Z",
  "ip_address": "192.168.1.100",
  "user_agent": "Mozilla/5.0...",
  "mfa_verified": true
}
```

### 21.2 Authorization Model

Permission checks on every admin action:

```python
def check_permission(user_id, required_permission, resource):
    user = get_admin_user(user_id)
    role_permissions = get_role_permissions(user.role)

    # Super admin has all permissions
    if user.role == "super_admin":
        return True

    # Check specific permission
    if required_permission in role_permissions:
        # Check resource-level access if applicable
        if requires_resource_access(required_permission):
            return check_resource_access(user_id, resource)
        return True

    return False
```

#### Permission Matrix

| Action | super_admin | ai_admin | data_admin | billing_admin | operator | developer |
|--------|-------------|----------|------------|---------------|----------|-----------|
| View dashboard | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| Configure providers | ✓ | ✓ | ✗ | ✗ | ✗ | ✗ |
| Edit pipelines | ✓ | ✓ | ✗ | ✗ | ✗ | ✗ |
| Edit prompts | ✓ | ✓ | ✗ | ✗ | ✗ | ✗ |
| Manage data sources | ✓ | ✗ | ✓ | ✗ | ✗ | ✗ |
| View costs | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| Set budgets | ✓ | ✓ | ✗ | ✓ | ✗ | ✗ |
| Manage admin users | ✓ | ✗ | ✗ | ✗ | ✗ | ✗ |
| View audit logs | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| Run tests | ✓ | ✓ | ✗ | ✗ | ✗ | ✓ |
| Access sandbox | ✓ | ✓ | ✗ | ✗ | ✗ | ✓ |

---

## 22. Multi-Application Support

### 22.1 Application Model

The platform serves multiple applications. Each application is isolated:

```sql
CREATE TABLE applications (
    application_id UUID PRIMARY KEY,
    application_name VARCHAR(200) NOT NULL,
    api_key_hash VARCHAR(255) UNIQUE NOT NULL,
    api_key_prefix VARCHAR(20) UNIQUE NOT NULL,
    organization_id VARCHAR(100),
    contact_email VARCHAR(255),
    status VARCHAR(20) DEFAULT 'active',
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE application_config (
    config_id UUID PRIMARY KEY,
    application_id UUID REFERENCES applications(application_id),
    pipeline_overrides JSONB,
    provider_preferences JSONB,
    rate_limits JSONB,
    cost_budgets JSONB,
    webhook_url VARCHAR(500),
    webhook_secret VARCHAR(255),
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);
```

### 22.2 Application Isolation

| Isolation Type | Implementation |
|----------------|----------------|
| **API Keys** | Each app has unique API key, hashed separately |
| **Requests** | All requests tagged with application_id |
| **Cost Tracking** | Costs tracked per application |
| **Rate Limiting** | Separate rate limits per application |
| **Configuration** | Apps can have custom pipelines/prompts or use defaults |
| **Logs** | Audit logs separated by application |
| **Data** | Application data isolated at database level |

### 22.3 Application API Keys

Format: `aiplat_<prefix>_<random>`

Example: `aiplat_oppxy_abc123def456...`

| Component | Purpose |
|-----------|---------|
| `aiplat_` | Identifies platform |
| `<prefix>` | App identifier (first 8 chars of hash) |
| `<random>` | Cryptographically random 64 characters |

**Storage:**
```json
{
  "api_key_id": "uuid",
  "application_id": "uuid",
  "api_key_hash": "bcrypt_hash_of_full_key",
  "api_key_prefix": "oppxy_abcd",
  "name": "Production Key",
  "created_at": "2026-01-18T14:30:00Z",
  "last_used_at": "2026-01-18T16:45:00Z",
  "status": "active"
}
```

### 22.4 Per-Application Configuration

Applications can:
- **Override pipelines** - Use different providers/models per stage
- **Set rate limits** - Requests per minute, tokens per minute
- **Set cost budgets** - Monthly maximum spend
- **Configure webhooks** - Receive async event notifications
- **Use custom prompts** - Override default system prompts

**Or use defaults** - If no override, platform defaults apply

### 22.5 Application Onboarding Flow

```
1. Admin registers new application
   ↓
2. Platform generates API key
   ↓
3. Admin receives API key (only shown once)
   ↓
4. Admin configures application:
   - Set rate limits
   - Set cost budget
   - Configure webhooks (optional)
   - Customize pipelines (optional)
   ↓
5. Application receives "onboarding complete" notification
   ↓
6. Application can now make API requests
```

---

## 23. Client Integration Guide

### 23.1 Integration Overview

Applications integrate with the AI Services Platform via REST API:

```
Application → API Key → Platform API → AI Processing → Response
```

### 23.2 Authentication

**All API requests require authentication via API key:**

```http
GET /api/v1/analyze/pipeline
Authorization: Bearer aiplat_oppxy_abc123...
Content-Type: application/json
```

**API Key Header (alternative):**
```http
X-API-Key: aiplat_oppxy_abc123...
```

### 23.3 Making Your First Request

**Step 1: Obtain API Key**
- Contact platform admin
- Receive API key (keep secret!)
- Store securely in environment variables

**Step 2: Test Connection**
```http
GET /api/v1/health
X-API-Key: your_api_key
```

**Step 3: Run Analysis**
```http
POST /api/v1/analyze/pipeline
X-API-Key: your_api_key
Content-Type: application/json

{
  "pipeline": "opportunity_analysis_full",
  "input": {
    "raw_data": "I wish there was a better way to export CRM contacts"
  }
}
```

### 23.4 SDKs and Client Libraries

**Official SDKs:**

| Language | Package | Installation |
|----------|---------|--------------|
| **Python** | `aiplatform-python` | `pip import aiplatform` |
| **JavaScript/TypeScript** | `@aiplatform/client` | `npm install @aiplatform/client` |
| **Go** | `github.com/aiplatform/go-sdk` | `go get github.com/aiplatform/go-sdk` |
| **Java** | `com.aiplatform:client` | Maven/Gradle |

**Python SDK Example:**
```python
from aiplatform import AIPlatformClient

client = AIPlatformClient(
    api_key="your_api_key",
    base_url="https://ai-platform.example.com"
)

result = client.analyze(
    pipeline="opportunity_analysis_full",
    input={
        "raw_data": "I wish there was a better way to export CRM contacts"
    }
)

print(f"Score: {result.score}")
print(f"Recommendation: {result.recommendation}")
```

**JavaScript SDK Example:**
```javascript
import { AIPlatformClient } from '@aiplatform/client';

const client = new AIPlatformClient({
  apiKey: 'your_api_key',
  baseUrl: 'https://ai-platform.example.com'
});

const result = await client.analyze({
  pipeline: 'opportunity_analysis_full',
  input: {
    raw_data: 'I wish there was a better way to export CRM contacts'
  }
});

console.log(`Score: ${result.score}`);
console.log(`Recommendation: ${result.recommendation}`);
```

### 23.5 Error Handling

**Always handle errors gracefully:**

```python
try:
    result = client.analyze(pipeline="...", input={...})
except APIKeyError:
    # Invalid API key
    logger.error("API key invalid")
except RateLimitError:
    # Rate limit exceeded - implement backoff
    time.sleep(60)
    retry_with_backoff()
except InsufficientBudgetError:
    # Application budget exceeded
    alert_admin("Budget exceeded")
except ProviderError as e:
    # All AI providers failed
    logger.error(f"Provider error: {e}")
    fallback_to_alternative()
except APIError as e:
    # Other API errors
    handle_generic_error(e)
```

### 23.6 Rate Limiting

**Your application has rate limits:**

| Limit Type | Default | Can Be Increased |
|------------|---------|------------------|
| Requests/minute | 100 | Yes, contact admin |
| Requests/day | 10,000 | Yes |
| Concurrent requests | 10 | Yes |
| Tokens/minute | 50,000 | Yes |

**Handling Rate Limits:**
```python
from aiplatform import RateLimitError

try:
    result = client.analyze(...)
except RateLimitError as e:
    # e.retry_after_seconds tells you when to retry
    retry_after = e.retry_after_seconds
    logger.info(f"Rate limited, retry after {retry_after}s")
    time.sleep(retry_after)
    # Retry request
```

### 23.7 Webhooks

**Configure webhook URL to receive async events:**

```http
POST /api/v1/webhooks
X-API-Key: your_api_key

{
  "url": "https://your-app.com/webhooks/ai-platform",
  "secret": "your_webhook_secret",
  "events": ["analysis.complete", "analysis.failed", "budget.alert"]
}
```

**Webhook Payload:**
```json
{
  "event_id": "uuid",
  "event_type": "analysis.complete",
  "timestamp": "2026-01-18T14:30:00Z",
  "application_id": "uuid",
  "data": {
    "analysis_id": "uuid",
    "pipeline": "opportunity_analysis_full",
    "results": {...},
    "cost": 0.023
  }
}
```

**Verify webhook signature:**
```python
import hmac
import hashlib

def verify_webhook_signature(payload, signature, secret):
    expected = hmac.new(
        secret.encode(),
        payload.encode(),
        hashlib.sha256
    ).hexdigest()
    return hmac.compare_digest(expected, signature)
```

### 23.8 Best Practices

| Practice | Recommendation |
|----------|----------------|
| **API Key Storage** | Environment variables, never commit to git |
| **Retry Logic** | Exponential backoff, max 3 retries |
| **Timeouts** | Set reasonable timeouts (30-60 seconds) |
| **Monitoring** | Track API usage, costs, error rates |
| **Testing** | Use sandbox endpoint for development |
| **Batch Requests** | Use batch endpoint for multiple analyses |
| **Caching** | Cache results when appropriate |

---

## 24. Cache Management Strategy

### 24.1 Cache Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│  CACHE LAYER (Redis)                                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ API Response Cache                                         │ │
│  │ • Key: api_response:{hash_of_request}                     │ │
│  │ • TTL: 3600s (1 hour)                                      │ │
│  │ • Invalidated on config change                             │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Prompt Cache                                               │ │
│  │ • Key: prompt:{prompt_id}:{version}                       │ │
│  │ • TTL: 86400s (24 hours)                                   │ │
│  │ • Loaded on startup                                        │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Configuration Cache                                        │ │
│  │ • Key: config:{provider_id}|{pipeline_id}                 │ │
│  │ • TTL: 300s (5 minutes)                                    │ │
│  │ • Instant invalidation on admin change                     │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Model Metadata Cache                                       │ │
│  │ • Key: model:{provider_id}:{model_id}                     │ │
│  │ • TTL: 3600s (1 hour)                                      │ │
│  │ • Includes costs, context window, etc.                     │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Rate Limit Cache                                           │ │
│  │ • Key: ratelimit:{application_id}:{window}                │ │
│  │ • TTL: 60s (1 minute)                                      │ │
│  │ • Sliding window counter                                   │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 24.2 Cache Key Patterns

| Cache Type | Key Pattern | Example |
|------------|-------------|---------|
| API Response | `api_response:{md5(request_body)}` | `api_response:a3f5c9d2...` |
| Prompt | `prompt:{prompt_id}:{version}` | `prompt:pain_point_extractor:1.2.0` |
| Provider Config | `config:provider:{provider_id}` | `config:provider:openai` |
| Pipeline Config | `config:pipeline:{pipeline_id}` | `config:pipeline:opportunity_analysis_full` |
| Model Info | `model:{provider_id}:{model_id}` | `model:openai:gpt-4-turbo` |
| Rate Limit | `ratelimit:{app_id}:{YYYYMMDDHHMM}` | `ratelimit:app_123:202601181430` |

### 24.3 Cache Invalidation Strategies

| Strategy | When Used | Implementation |
|----------|-----------|----------------|
| **TTL Expiration** | All caches | Automatic on expiry |
| **Immediate Invalidation** | Config changes | Admin changes → cache delete |
| **Tagged Invalidation** | Related items | Cache tags for grouping |
| **Cache Warming** | Cold starts | Pre-load critical items |

**Immediate Invalidation Example:**
```python
def update_provider_config(provider_id, new_config):
    # Update database
    db.execute("UPDATE ai_providers SET config = ? WHERE provider_id = ?",
              (new_config, provider_id))

    # Invalidate cache immediately
    cache.delete(f"config:provider:{provider_id}")

    # Invalidate related model caches
    for model in new_config['models']:
        cache.delete(f"model:{provider_id}:{model}")
```

### 24.4 Cache Performance Targets

| Metric | Target | Rationale |
|--------|--------|-----------|
| **Cache Hit Rate** | > 80% | Reduces API calls, improves latency |
| **Cache Latency** | < 5ms | Redis should be very fast |
| **Memory Usage** | < 70% of Redis | Leave room for growth |
| **Eviction Rate** | < 5% of requests/min | Healthy cache turnover |

### 24.5 Cache Monitoring

**Track these metrics:**
```python
cache_metrics = {
    "hit_rate": 0.85,  # 85% of requests hit cache
    "miss_rate": 0.15,  # 15% cache misses
    "evictions": 150,  # Items evicted
    "memory_used": "2.1GB",  # Redis memory usage
    "keys_total": 45230,  # Total keys stored
    "avg_item_size": "48KB"  # Average cached item size
}
```

---

## 25. Backup & Disaster Recovery

### 25.1 Backup Strategy

**What Gets Backed Up:**

| Asset | Frequency | Retention | Location |
|-------|-----------|-----------|----------|
| **Database** | Daily | 30 days | S3-compatible storage |
| **Redis Cache** | Daily | 7 days | S3-compatible storage |
| **Configuration** | On change | 90 days | Git repository + S3 |
| **Audit Logs** | Continuous | 365 days | S3 + cold storage |
| **Prompts** | On change | Indefinite | Git repository |

**Database Backup Schedule:**
```
03:00 UTC Daily - Full backup
Every 6 hours - Transaction log backup
Continuous - WAL archiving (7 days retention)
```

### 25.2 Recovery Objectives

| Metric | Target | Rationale |
|--------|--------|-----------|
| **RPO (Recovery Point Objective)** | < 1 hour | Max acceptable data loss |
| **RTO (Recovery Time Objective)** | < 4 hours | Max acceptable downtime |
| **Critical Systems RTO** | < 1 hour | Core platform services |
| **Data Loss** | Zero | Transactional integrity |

### 25.3 Backup Testing

**Monthly restore tests:**
1. Random backup selection
2. Restore to staging environment
3. Verify data integrity
4. Test application functionality
5. Document results

**Quarterly disaster recovery drills:**
1. Simulate complete platform failure
2. Execute recovery procedures
3. Measure actual RTO/RPO
4. Identify and fix gaps
5. Update runbooks

### 25.4 Disaster Recovery Runbooks

#### Scenario 1: Database Corruption

**Symptoms:**
- Database errors
- Query failures
- Data inconsistency

**Recovery Steps:**
```
1. Identify corruption extent
   ↓
2. Stop all application writes
   ↓
3. Restore from last known good backup
   ↓
4. Replay transaction logs
   ↓
5. Verify data integrity
   ↓
6. Restart applications
   ↓
7. Monitor for errors
```

#### Scenario 2: Redis Cache Failure

**Symptoms:**
- Cache misses = 100%
- Increased latency
- Database load spike

**Recovery Steps:**
```
1. Check Redis service status
   ↓
2. If down: Restart Redis
   ↓
3. If data loss: Applications will rebuild cache naturally
   ↓
4. Monitor cache hit rate recovery
   ↓
5. If persistent failure: Failover to backup Redis
```

#### Scenario 3: AI Provider Outage

**Symptoms:**
- Provider API errors
- Increased failure rate
- Cost alerts

**Recovery Steps:**
```
1. Fallback automatically activates
   ↓
2. Monitor fallback provider performance
   ↓
3. Check provider status page
   ↓
4. If extended outage: Adjust cost budgets
   ↓
5. Communicate with affected applications
   ↓
6. When restored: Revert to primary provider
```

### 25.5 Geographic Redundancy

**For production deployments:**

| Component | Primary | Secondary |
|-----------|---------|-----------|
| **Database** | Region A | Async replica in Region B |
| **Redis** | Region A | standalone instance in Region B |
| **Storage** | Region A | Cross-region replication |
| **Application** | Region A + Region B | Active-active |

**Failover Decision Tree:**
```
Primary region healthy → Use primary
         ↓
Primary region degraded → Assess severity
         ↓
Minor issue → Remain on primary
         ↓
Major issue → Failover to secondary
         ↓
Regional disaster → Declare disaster, activate DR plan
```

---

## 26. Webhook System

### 26.1 Webhook Architecture

Applications can register webhooks to receive async notifications:

```
Platform Event → Webhook Queue → Worker Process → HTTP POST → Application
                                    ↓
                              Retry on failure
                                    ↓
                              Dead Letter Queue
```

### 26.2 Webhook Events

| Event | Trigger | Payload |
|-------|---------|---------|
| `analysis.complete` | Pipeline finishes successfully | Full analysis results |
| `analysis.failed` | Pipeline fails | Error details, retry info |
| `analysis.progress` | Pipeline stage completes | Progress percentage |
| `budget.alert` | Cost threshold exceeded | Budget details, usage |
| `provider.warning` | Provider degraded | Provider status, impact |
| `provider.down` | Provider unavailable | Expected recovery time |
| `system.maintenance` | Scheduled maintenance | Window, impact |
| `system.alert` | System issue | Alert details, severity |

### 26.3 Webhook Configuration

```sql
CREATE TABLE webhooks (
    webhook_id UUID PRIMARY KEY,
    application_id UUID REFERENCES applications(application_id),
    url VARCHAR(500) NOT NULL,
    secret VARCHAR(255) NOT NULL,
    events JSONB NOT NULL,
    headers JSONB,
    retry_policy JSONB,
    active BOOLEAN DEFAULT true,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);
```

**Configuration Example:**
```json
{
  "webhook_id": "uuid",
  "application_id": "uuid",
  "url": "https://your-app.com/webhooks/ai-platform",
  "secret": "webhook_signature_secret",
  "events": [
    "analysis.complete",
    "analysis.failed",
    "budget.alert"
  ],
  "headers": {
    "X-Custom-Header": "value"
  },
  "retry_policy": {
    "max_retries": 5,
    "initial_delay": 30,
    "backoff_multiplier": 2.0,
    "max_delay": 3600
  }
}
```

### 26.4 Webhook Delivery

**Delivery Process:**
```
1. Event occurs
   ↓
2. Determine subscribed applications
   ↓
3. Create webhook delivery job
   ↓
4. Queue for worker
   ↓
5. Worker sends POST request
   ↓
6. Application responds with 2xx
   ↓
7. Mark as delivered
```

**Retry Logic:**
```
On failure:
├─ 1st retry: 30 seconds later
├─ 2nd retry: 60 seconds later (30 × 2)
├─ 3rd retry: 120 seconds later (60 × 2)
├─ 4th retry: 240 seconds later
└─ 5th retry: 480 seconds later
   ↓
   After max retries: Move to Dead Letter Queue
```

### 26.5 Webhook Signature

**Each webhook includes signature for verification:**

```
X-Webhook-Signature: sha256=<signature>
X-Webhook-Timestamp: <timestamp>
X-Webhook-Id: <unique_id>
```

**Signature Calculation:**
```
signature = HMAC-SHA256(webhook_secret, timestamp + payload)
```

**Verification:**
```python
def verify_webhook(payload, signature, timestamp, secret):
    # Check timestamp (prevent replay attacks)
    if abs(time.time() - timestamp) > 300:  # 5 minutes
        return False

    # Calculate expected signature
    expected = hmac.new(
        secret.encode(),
        f"{timestamp}{payload}".encode(),
        hashlib.sha256
    ).hexdigest()

    # Compare securely
    return hmac.compare_digest(f"sha256={expected}", signature)
```

### 26.6 Webhook Monitoring

**Track delivery metrics:**
```json
{
  "webhook_id": "uuid",
  "url": "https://your-app.com/webhooks/ai-platform",
  "delivery_stats": {
    "total_sent": 15230,
    "delivered": 15180,
    "failed": 50,
    "pending": 0,
    "delivery_rate": 99.67,
    "avg_delivery_time_ms": 234
  },
  "last_delivery_at": "2026-01-18T16:45:00Z",
  "last_success_at": "2026-01-18T16:45:00Z",
  "last_failure_at": "2026-01-18T14:23:00Z"
}
```

---

## 27. Maintenance Mode

### 27.1 Maintenance Mode Purpose

Planned maintenance for:
- Platform updates
- Database migrations
- Provider changes
- Scaling activities

### 27.2 Maintenance Mode States

| State | API Behavior | Admin Panel |
|-------|-------------|-------------|
| **Normal** | Full operation | Full operation |
| **Draining** | New requests rejected, existing complete | Read-only |
| **Maintenance** | All requests rejected | Maintenance banner |
| **Recovery** | Gradual traffic restoration | Read-only → Full |

### 27.3 Maintenance Mode Flow

```
1. Admin schedules maintenance
   ↓
2. System notifies affected applications (via webhook if configured)
   ↓
3. At scheduled time: Enter draining state
   • Grace period: 5 minutes
   • Allow in-flight requests to complete
   • Reject new requests with 503
   ↓
4. Enter maintenance state
   • All requests return 503
   • Retry-After header indicates expected恢复时间
   ↓
5. Perform maintenance tasks
   ↓
6. Enter recovery state
   • Gradually restore traffic
   • Monitor for errors
   ↓
7. Return to normal state
```

### 27.4 Maintenance Response

**During maintenance:**
```http
HTTP/1.1 503 Service Unavailable
Retry-After: 3600
Content-Type: application/json

{
  "error": {
    "code": "MAINTENANCE_MODE",
    "message": "Platform is under scheduled maintenance",
    "scheduled_start": "2026-01-18T20:00:00Z",
    "expected_end": "2026-01-18T22:00:00Z",
    "status_page": "https://status.aiplatform.example.com"
  }
}
```

### 27.5 Maintenance Best Practices

| Practice | Recommendation |
|----------|----------------|
| **Scheduling** | Low-traffic hours, communicate 48hrs ahead |
| **Duration** | Keep under 2 hours when possible |
| **Rolling Updates** | Update components sequentially when possible |
| **Rollback Plan** | Have documented rollback procedure |
| **Communication** | Status page, email notifications, webhooks |
| **Testing** | Test maintenance procedures in staging first |

---

## 28. Service Level Objectives (SLOs)

### 28.1 Service Level Commitments

| Metric | Objective | Measurement Period |
|--------|-----------|-------------------|
| **API Availability** | 99.5% uptime | Monthly (rolling 30 days) |
| **API Response Time (P95)** | < 2 seconds | Daily |
| **API Response Time (P99)** | < 5 seconds | Daily |
| **Error Rate** | < 0.5% | Daily |
| **Webhook Delivery Rate** | > 99% | Daily |
| **Data Freshness** | < 5 minutes | Continuous |

### 28.2 Availability Calculation

```
Availability = (Total Time - Downtime) / Total Time

99.5% availability = Maximum 3.65 hours downtime/month
99.9% availability = Maximum 43.2 minutes downtime/month
```

### 28.3 Excluded from Downtime

| Exclusion | Reason |
|-----------|--------|
| **Scheduled maintenance** | Communicated in advance |
| **Force majeure** | Natural disasters, out of control |
| **Third-party failures** | AI provider outages (have fallbacks) |
| **Application issues** | Problems with integrating applications |

### 28.4 Performance Targets by Endpoint

| Endpoint | P50 Latency | P95 Latency | P99 Latency |
|----------|-------------|-------------|-------------|
| `POST /analyze/pipeline` | 3s | 10s | 30s |
| `GET /analyze/{id}` | 100ms | 300ms | 500ms |
| `POST /admin/pipelines` | 200ms | 500ms | 1s |
| `GET /admin/costs` | 300ms | 800ms | 2s |

**Note:** Pipeline analysis latency depends on AI providers and is variable.

### 28.5 SLO Monitoring and Alerting

**Alert Thresholds:**
```
Error rate > 1% for 5 minutes → Page on-call
Error rate > 0.5% for 15 minutes → Email alert
P95 latency > 3s for 10 minutes → Warning
Availability < 99% for 1 hour → Critical alert
```

### 28.6 SLO Reporting

**Monthly SLO Report includes:**
- Actual availability vs target
- Performance metrics (P50, P95, P99)
- Incident summary
- Root causes of any breaches
- Improvement actions

---

## 29. Feature Flags System

### 29.1 Feature Flag Purpose

Runtime control over platform features without deployment:
- Gradual feature rollout
- A/B testing
- Emergency kill switches
- Beta feature access

### 29.2 Feature Flag Schema

```sql
CREATE TABLE feature_flags (
    flag_id UUID PRIMARY KEY,
    flag_name VARCHAR(100) UNIQUE NOT NULL,
    description TEXT,
    enabled BOOLEAN DEFAULT false,
    rollout_percentage INTEGER DEFAULT 0,
    environment VARCHAR(20),
    conditions JSONB,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE feature_flag_overrides (
    override_id UUID PRIMARY KEY,
    flag_id UUID REFERENCES feature_flags(flag_id),
    application_id UUID REFERENCES applications(application_id),
    enabled BOOLEAN,
    created_at TIMESTAMP DEFAULT NOW()
);
```

### 29.3 Feature Flag Evaluation

```python
def is_feature_enabled(flag_name, application_id=None):
    flag = get_feature_flag(flag_name)

    # Global disabled
    if not flag.enabled:
        return False

    # Check application-specific override
    if application_id:
        override = get_override(flag.flag_id, application_id)
        if override:
            return override.enabled

    # Rollout percentage
    if flag.rollout_percentage < 100:
        hash_value = hash(application_id + flag_name) % 100
        return hash_value < flag.rollout_percentage

    return True
```

### 29.4 Common Feature Flags

| Flag | Purpose | Default |
|------|---------|---------|
| `new_pipeline_orchestrator` | New pipeline execution engine | 0% (testing) |
| `enhanced_cost_tracking` | Per-token cost breakdown | 100% (enabled) |
| `streaming_responses` | Real-time streaming output | 0% (development) |
| `experimental_provider` | New AI provider | 10% (beta) |
| `advanced_caching` | New caching strategy | 50% (rolling out) |
| `webhook_v2` | Improved webhook system | 0% (testing) |

### 29.5 Feature Flag Administration

**Admin Panel Controls:**
- Enable/disable flags globally
- Set rollout percentage
- Configure application-specific overrides
- View flag usage statistics
- A/B test configurations

---

## 30. Capacity Planning

### 30.1 Capacity Metrics

**Track these metrics for capacity planning:**

| Metric | Current | Warning | Critical |
|--------|---------|---------|----------|
| **API Requests/Day** | 50,000 | 80,000 | 100,000 |
| **Concurrent Requests** | 25 | 40 | 50 |
| **Database CPU** | 35% | 70% | 85% |
| **Database Memory** | 45% | 75% | 90% |
| **Redis Memory** | 30% | 70% | 85% |
| **API Server CPU** | 25% | 70% | 85% |
| **Disk Usage** | 40% | 75% | 90% |

### 30.2 Scaling Decisions

**When to Scale Up:**
- Any metric enters "Warning" zone for > 24 hours
- Any metric enters "Critical" zone
- Growth trend shows > 20% month-over-month

**Scaling Actions:**
| Situation | Action |
|-----------|--------|
| API server CPU > 70% | Add API server replica |
| Database CPU > 70% | Upgrade DB instance |
| Redis memory > 70% | Add Redis shard or upgrade |
| Disk > 75% | Archive old data or expand |

### 30.3 Cost Modeling

**Per-Request Cost Model:**
```
Base cost = Infrastructure overhead + AI costs

Infrastructure overhead (per 1000 requests):
- API compute: $0.01
- Database: $0.005
- Redis: $0.002
- Storage: $0.001
- Total: ~$0.018 per 1000 requests

AI costs (varies by pipeline):
- Simple pipeline: $0.01-0.05 per request
- Complex pipeline: $0.05-0.30 per request
```

**Monthly Cost Projection:**
```
Requests per month = 50,000
Average cost per request = $0.10
Monthly AI costs = $5,000
Infrastructure costs = $900
Total = $5,900/month

At 100,000 requests/month:
Monthly AI costs = $10,000
Infrastructure costs = $1,800
Total = $11,800/month
```

### 30.4 Capacity Planning Timeline

**Review capacity monthly:**
```
Month 1-3: Monitor baseline
Month 4-6: Adjust for growth
Month 7-12: Plan for scale events
```

**Scale Triggers:**
- Request volume doubles
- New application onboarding
- New AI providers integrated
- New pipelines added

---

## 31. Compliance & Legal

### 31.1 GDPR Compliance

**Data Subject Rights:**

| Right | Implementation |
|-------|----------------|
| **Right to Access** | Application can export all their data |
| **Right to Rectification** | Application can update their data |
| **Right to Erasure** | Application data deleted within 30 days |
| **Right to Portability** | Data exported in machine-readable format |
| **Right to Object** | Processing can be disabled |

**Data Retention:**
| Data Type | Retention Period |
|-----------|-----------------|
| Active application data | Indefinite (while active) |
| Inactive application data | 90 days after deactivation |
| Audit logs | 365 days |
| AI request logs | 90 days |
| Error logs | 30 days |
| Backups | 30 days (per data retention policy) |

### 31.2 Data Processing Agreement

**Platform acts as Data Processor:**
- Processes data on behalf of applications (Data Controllers)
- Each application signs DPA
- DPA specifies:
  - Data processing purposes
  - Data types processed
  - Security measures
  - Sub-processor restrictions (AI providers)
  - Data deletion procedures
  - Breach notification requirements

### 31.3 AI Provider Data Usage

**Each AI provider has different data policies:**

| Provider | Uses Data for Training | Data Retention |
|----------|----------------------|----------------|
| **OpenAI** | No (API data not used) | 30 days for abuse monitoring |
| **Anthropic** | No | 30 days |
| **ZAI** | Check current policy | Varies |
| **Open-source** | Depends on hosting | Depends on hosting |

**Platform must:**
- Display current provider policies to applications
- Allow applications to opt-out of providers that train on data
- Document any sub-processor relationships

### 31.4 Security Compliance

**SOC 2 Type II (Future Target):**
- Implement SOC 2 controls
- Annual audit
- Compliance report available to customers

**HIPAA (If Required):**
- Business Associate Agreements
- PHI encryption at rest and in transit
- Audit logging for all PHI access
- BAA with sub-processors (AI providers)

### 31.5 Privacy Policy

**Platform privacy policy must disclose:**
- What data is collected
- How data is used
- Who data is shared with (AI providers)
- Data retention periods
- User rights
- Contact information

### 31.6 Audit Trail

**Complete audit trail required:**
- All admin actions
- All API requests
- All configuration changes
- All data access
- All cost-related activities

**Audit log retention:** 365 days minimum

---

## 32. Operational Runbooks

### 32.1 Incident Response Runbook

**Severity Levels:**

| Severity | Definition | Response Time |
|----------|-----------|---------------|
| **P1 - Critical** | Platform completely down | 15 minutes |
| **P2 - High** | Major functionality broken | 1 hour |
| **P3 - Medium** | Partial degradation | 4 hours |
| **P4 - Low** | Minor issues | 1 business day |

**Incident Response Process:**
```
1. Detect incident
   - Monitoring alert
   - User report
   ↓
2. Acknowledge incident
   - Assign severity
   - Notify on-call
   ↓
3. Investigate
   - Gather symptoms
   - Check recent changes
   - Review logs
   ↓
4. Mitigate
   - Implement temporary fix
   - Or rollback
   ↓
5. Resolve
   - Implement permanent fix
   - Verify resolution
   ↓
6. Post-incident review
   - Document timeline
   - Identify root cause
   - Create action items
   ↓
7. Improve
   - Update runbooks
   - Implement preventive measures
```

### 32.2 On-Call Procedures

**On-Call Responsibilities:**
- Respond to alerts within SLA
- Triage incidents
- Escalate when needed
- Document all actions
- Participate in post-incident reviews

**Escalation Path:**
```
Level 1: On-Call Engineer
   ↓ (30 min no response / can't resolve)
Level 2: Engineering Lead
   ↓ (1 hour no response / can't resolve)
Level 3: CTO / VP Engineering
   ↓ (Critical issue)
Level 4: Executive Team
```

### 32.3 Common Operational Procedures

#### Procedure: Scale API Servers

**Trigger:** API CPU > 70% or latency > 3s P95

**Steps:**
```
1. Verify actual load (not spike)
   ↓
2. Check available capacity
   ↓
3. Add new API server instance
   ↓
4. Verify instance healthy
   ↓
5. Add to load balancer
   ↓
6. Monitor metrics for 30 minutes
   ↓
7. Document scaling event
```

#### Procedure: Rotate API Keys

**Trigger:** Key compromise or scheduled rotation (90 days)

**Steps:**
```
1. Generate new API key
   ↓
2. Store new key (encrypted)
   ↓
3. Update application configurations
   ↓
4. Notify application owners
   ↓
5. Grace period: old key still works
   ↓
6. After 30 days: disable old key
   ↓
7. After 60 days: permanently delete old key
```

#### Procedure: Clear Redis Cache

**Trigger:** Cache corruption or stale data

**Steps:**
```
1. Identify affected cache keys
   ↓
2. If all cache: FLUSHDB (use with caution!)
   ↓
3. If specific keys: DELETE pattern
   ↓
4. Monitor cache rebuild
   ↓
5. Verify application performance
   ↓
6. Document cache clear event
```

### 32.4 Communication Templates

#### Template: Planned Maintenance

```
Subject: Scheduled Maintenance - [Date] [Time Window]

Dear Platform User,

We will be performing scheduled maintenance on the AI Services Platform.

Maintenance Window:
- Start: [Date] [Time] UTC
- End: [Date] [Time] UTC
- Expected Duration: [X] hours

Impact:
- API requests will be rejected during maintenance
- Existing requests will complete before maintenance starts
- Webhooks may be delayed

What You Need to Do:
- No action required
- Retry failed requests after maintenance ends

We apologize for any inconvenience.

Status Page: https://status.aiplatform.example.com
Support: support@aiplatform.example.com
```

#### Template: Incident Update

```
Subject: [URGENT] Platform Incident - [Brief Description]

Status: [Investigating | Identified | Monitoring | Resolved]

Impact:
- [Description of affected functionality]
- [Which applications/users are affected]

Current Status:
- [Time] - Detecting issue
- [Time] - Investigating cause
- [Time] - [Update]
- [Time] - [Update]

Next Update: [Time] UTC

Status Page: https://status.aiplatform.example.com
```

---

## Appendix A: Provider API Specifications

### A.1 OpenAI API

**Base URL:** `https://api.openai.com/v1`

**Models:**
- gpt-4-turbo: 128K context, $10/input (per 1M), $30/output (per 1M)
- gpt-4: 8K context, $30/input, $60/output
- gpt-3.5-turbo: 16K context, $0.50/input, $1.50/output

### A.2 Anthropic API

**Base URL:** `https://api.anthropic.com/v1`

**Models:**
- claude-3-opus: 200K context, $15/input, $75/output
- claude-3-sonnet: 200K context, $3/input, $15/output
- claude-3-haiku: 200K context, $0.25/input, $1.25/output

### A.3 ZAI (Zhipu AI) API

**Base URL:** `https://open.bigmodel.cn/api/paas/v4/`

**Models:**
- glm-4.7: 128K context, ¥0.5/input, ¥0.5/output
- glm-4-plus: 128K context, ¥0.4/input, ¥0.4/output
- glm-4-air: 200K context, ¥0.15/input, ¥0.15/output
- glm-4-flash: 128K context, ¥0.1/input, ¥0.1/output

---

## Appendix B: Error Codes

| Code | Meaning | Resolution |
|------|---------|------------|
| `AI_001` | Provider not found | Check provider configuration |
| `AI_002` | Model not available | Verify model enabled |
| `AI_003` | Rate limit exceeded | Wait or try different provider |
| `AI_004` | Invalid API key | Update credentials |
| `AI_005` | Provider timeout | Check provider status, retry |
| `AI_006` | Insufficient budget | Top up budget or use cheaper option |
| `AI_007` | Pipeline not found | Verify pipeline ID |
| `AI_008` | Prompt not found | Check prompt configuration |
| `AI_009` | All providers failed | Check system status |
| `AI_010` | Invalid input format | Correct request format |

---

## Document Metadata

| Field | Value |
|-------|-------|
| **Document ID** | AI_Services_Platform_PRD_v1.3 |
| **Version** | 1.3 |
| **Status** | Draft |
| **Created** | 2026-01-18 |
| **Last Updated** | 2026-01-18 |
| **Authors** | AI Platform Team |
| **Reviewers** | Pending |
| **Approved By** | Pending |

---

## Change Log

| Version | Date | Changes | Author |
|---------|------|---------|--------|
| 1.3 | 2026-01-18 | Added critical clarifications: (1) Manual override capability for AI scores - admin can add "manual bias" by overriding any component, (2) Score transparency is admin-optional - users see final score only by default, (3) Validation rules are admin-configurable not hardcoded - all thresholds adjustable, (4) Clear separation of AI analysis vs validation rules vs scoring rules vs recommendation | AI Platform Team |
| 1.2 | 2026-01-18 | Fixed scoring architecture: Changed from "AI generates final score" to hybrid "AI generates component scores + rules apply weights" to align with application requirements and enable admin fine-tuning control | AI Platform Team |
| 1.1 | 2026-01-18 | Added 12 critical production sections: Platform Auth, Multi-Application Support, Integration Guide, Cache Strategy, Backup/DR, Webhooks, Maintenance Mode, SLOs, Feature Flags, Capacity Planning, Compliance, Runbooks | AI Platform Team |
| 1.0 | 2026-01-18 | Initial creation - complete AI Services Platform specification | AI Platform Team |

---

**END OF DOCUMENT**
