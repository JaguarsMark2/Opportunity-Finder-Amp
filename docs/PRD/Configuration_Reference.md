# Opportunity Finder - Configuration Reference

**Version:** 1.0
**Date:** 2026-01-19
**Status:** Implementation Specification
**Related Documents:**
- Opportunity_Finder_PRD_v3.0.md
- AI_Services_Platform_PRD_v1.4.md
- brainstorming-session-2026-01-19.md (Phase 1.5, Phase 2)

---

## Document Overview

This document provides the complete reference for all 100+ configurable parameters in the Opportunity Finder system. It serves as the authoritative source for:

1. **Application Configuration** - All parameters stored in `application_config` table
2. **Validation Rules** - Dependencies and constraints between parameters
3. **Configuration Presets** - Pre-built configurations for common use cases
4. **Source-Specific Parameters** - Parameters that apply only to specific data sources
5. **AI Prompt Templates** - Source-specific analysis prompts
6. **Data Contracts** - Unified schema and interface definitions

---

## Table of Contents

1. [Parameter Dimensions Overview](#parameter-dimensions-overview)
2. [Dimension 1: Data Source Configuration](#dimension-1-data-source-configuration)
3. [Dimension 2: Scoring Component Weights](#dimension-2-scoring-component-weights)
4. [Dimension 3: Scoring Thresholds & Ranges](#dimension-3-scoring-thresholds--ranges)
5. [Dimension 4: Validation Rules](#dimension-4-validation-rules)
6. [Dimension 5: Business Type Filters](#dimension-5-business-type-filters)
7. [Dimension 6: Category Classifications](#dimension-6-category-classifications)
8. [Dimension 7: Maturity Level Preferences](#dimension-7-maturity-level-preferences)
9. [Dimension 8: AI Model Configuration](#dimension-8-ai-model-configuration)
10. [Dimension 9: Geographic Configuration](#dimension-9-geographic-configuration)
11. [Dimension 10: Source-Specific Parameters](#dimension-10-source-specific-parameters)
12. [Dimension 11: Output Configuration](#dimension-11-output-configuration)
13. [Dimension 12: Notification & Alerting](#dimension-12-notification--alerting)
14. [Configuration Validation Rules](#configuration-validation-rules)
15. [Configuration Presets](#configuration-presets)
16. [Unified Data Schema Reference](#unified-data-schema-reference)
17. [Data Source Interface Contract](#data-source-interface-contract)
18. [AI Prompt Templates](#ai-prompt-templates)

---

## Parameter Dimensions Overview

| Dimension | Parameter Count | Category | Database Table |
|-----------|----------------|----------|----------------|
| 1. Data Source Configuration | 4 | Core | application_config |
| 2. Scoring Component Weights | 7 | Core | application_config |
| 3. Scoring Thresholds & Ranges | 8 | Core | application_config |
| 4. Validation Rules | 11 | Core | application_config |
| 5. Business Type Filters | 4 | Filtering | application_config |
| 6. Category Classifications | 3 | Filtering | application_config |
| 7. Maturity Level Preferences | 4 | Filtering | application_config |
| 8. AI Model Configuration | 7 | AI | application_config |
| 9. Geographic Configuration | 4 | Filtering | application_config |
| 10. Source-Specific Parameters | 26+ | Source | application_config |
| 11. Output Configuration | 6 | Display | application_config |
| 12. Notification & Alerting | 6 | Alerting | application_config |
| **TOTAL** | **90+** | - | - |

---

## Dimension 1: Data Source Configuration

**Category:** Core
**Validation:** Required
**Admin UI:** Data Sources tab

| Parameter | Type | Possible Values | Default | Dependency | Description |
|-----------|------|-----------------|---------|------------|-------------|
| `enabled_sources` | List[str] | ["reddit", "google_trends", "competition_search", "youtube", "gumroad", "etsy", "stackoverflow", "zapier", "make", "n8n", "indiehackers", "hackernews", "twitter"] | ["reddit", "google_trends", "competition_search"] | None | Active data sources for scans |
| `source_priority` | Dict[str, int] | Any positive integer (higher = more weight) | {"reddit": 3, "google_trends": 2, "competition_search": 1} | None | Weight of each source in scoring |
| `source_fallback_enabled` | bool | true/false | true | None | Enable/disable source fallback on failure |
| `min_confidence_threshold` | float | 0.0-100.0 | 50.0 | None | Filter out low-confidence data points |

**Database Schema:**
```sql
INSERT INTO application_config (application_id, config_key, config_value, value_type, category) VALUES
('opportunity-finder-app-id', 'enabled_sources', '["reddit", "google_trends", "competition_search"]', 'list', 'source'),
('opportunity-finder-app-id', 'source_priority', '{"reddit": 3, "google_trends": 2, "competition_search": 1}', 'dict', 'source'),
('opportunity-finder-app-id', 'source_fallback_enabled', 'true', 'bool', 'source'),
('opportunity-finder-app-id', 'min_confidence_threshold', '50.0', 'float', 'source');
```

---

## Dimension 2: Scoring Component Weights

**Category:** Core
**Validation:** Weight sum must equal 1.0
**Admin UI:** Scoring Weights tab

| Parameter | Type | Possible Values | Default | Dependency | Description |
|-----------|------|-----------------|---------|------------|-------------|
| `weight_demand` | float | 0.0-1.0 | 0.25 | All weights must sum to 1.0 | Overall demand component weight |
| `weight_demand_velocity` | float | 0.0-1.0 | 0.15 | Part of demand calculation | Weight for growth rate within demand |
| `weight_demand_volume` | float | 0.0-1.0 | 0.10 | Part of demand calculation | Weight for absolute volume within demand |
| `weight_competition` | float | 0.0-1.0 | 0.20 | All weights must sum to 1.0 | Competition score weight (inverted) |
| `weight_revenue` | float | 0.0-1.0 | 0.25 | All weights must sum to 1.0 | Revenue proof weight |
| `weight_feasibility` | float | 0.0-1.0 | 0.15 | All weights must sum to 1.0 | Technical feasibility weight |
| `weight_timing` | float | 0.0-1.0 | 0.10 | All weights must sum to 1.0 | Market timing weight |
| `weight_risk` | float | 0.0-1.0 | 0.05 | All weights must sum to 1.0 | Risk weight (inverted) |

**Scoring Formula:**
```
final_score = (
    (demand_score × weight_demand) +
    (competition_score_inverted × weight_competition) +
    (revenue_score × weight_revenue) +
    (feasibility_score × weight_feasibility) +
    (timing_score × weight_timing) +
    (risk_score_inverted × weight_risk)
)

Where demand_score = (velocity_score × weight_demand_velocity) + (volume_score × weight_demand_volume)
Where competition_score_inverted = (100 - competition_score) [lower competition = higher score]
Where risk_score_inverted = (100 - risk_score) [lower risk = higher score]
```

**Database Schema:**
```sql
INSERT INTO application_config (application_id, config_key, config_value, value_type, category, validation_rule) VALUES
('opportunity-finder-app-id', 'weight_demand', '0.25', 'float', 'scoring', '{"min": 0.0, "max": 1.0}'),
('opportunity-finder-app-id', 'weight_demand_velocity', '0.15', 'float', 'scoring', '{"min": 0.0, "max": 1.0}'),
('opportunity-finder-app-id', 'weight_demand_volume', '0.10', 'float', 'scoring', '{"min": 0.0, "max": 1.0}'),
('opportunity-finder-app-id', 'weight_competition', '0.20', 'float', 'scoring', '{"min": 0.0, "max": 1.0}'),
('opportunity-finder-app-id', 'weight_revenue', '0.25', 'float', 'scoring', '{"min": 0.0, "max": 1.0}'),
('opportunity-finder-app-id', 'weight_feasibility', '0.15', 'float', 'scoring', '{"min": 0.0, "max": 1.0}'),
('opportunity-finder-app-id', 'weight_timing', '0.10', 'float', 'scoring', '{"min": 0.0, "max": 1.0}'),
('opportunity-finder-app-id', 'weight_risk', '0.05', 'float', 'scoring', '{"min": 0.0, "max": 1.0}');
```

---

## Dimension 3: Scoring Thresholds & Ranges

**Category:** Core
**Validation:** Threshold ordering required
**Admin UI:** Thresholds tab

| Parameter | Type | Possible Values | Default | Dependency | Description |
|-----------|------|-----------------|---------|------------|-------------|
| `score_threshold_build` | float | 0-100 | 75 | Must be > score_threshold_validate | Minimum score for "build immediately" |
| `score_threshold_validate` | float | 0-100 | 60 | Must be > score_threshold_investigate | Minimum score for "validate first" |
| `score_threshold_investigate` | float | 0-100 | 40 | Must be < score_threshold_validate | Minimum score for "investigate" |
| `velocity_threshold_high` | float | 1.0-10.0+ | 2.0 | None | Growth rate considered "high velocity" |
| `velocity_threshold_medium` | float | 1.0-10.0+ | 1.5 | None | Growth rate considered "medium velocity" |
| `volume_threshold_high` | int | 100-1000000+ | 10000 | None | Search volume considered "high" (per month) |
| `volume_threshold_medium` | int | 100-1000000+ | 1000 | None | Search volume considered "medium" (per month) |

**Database Schema:**
```sql
INSERT INTO application_config (application_id, config_key, config_value, value_type, category, validation_rule) VALUES
('opportunity-finder-app-id', 'score_threshold_build', '75', 'float', 'threshold', '{"min": 0, "max": 100}'),
('opportunity-finder-app-id', 'score_threshold_validate', '60', 'float', 'threshold', '{"min": 0, "max": 100}'),
('opportunity-finder-app-id', 'score_threshold_investigate', '40', 'float', 'threshold', '{"min": 0, "max": 100}'),
('opportunity-finder-app-id', 'velocity_threshold_high', '2.0', 'float', 'threshold', '{"min": 1.0}'),
('opportunity-finder-app-id', 'velocity_threshold_medium', '1.5', 'float', 'threshold', '{"min": 1.0}'),
('opportunity-finder-app-id', 'volume_threshold_high', '10000', 'int', 'threshold', '{"min": 100}'),
('opportunity-finder-app-id', 'volume_threshold_medium', '1000', 'int', 'threshold', '{"min": 100}');
```

---

## Dimension 4: Validation Rules

**Category:** Core
**Validation:** Business type mutual exclusion
**Admin UI:** Validation Rules tab

| Parameter | Type | Possible Values | Default | Dependency | Description |
|-----------|------|-----------------|---------|------------|-------------|
| `require_paid_solution` | bool | true/false | true | None | Require at least 1 paid competitor |
| `min_competitor_mrr` | float | 0-100000+ | 1000 | Currency: USD | Minimum competitor MRR for validation |
| `max_competitor_count` | int | 0-1000+ | 50 | None | Maximum competitors to consider validated |
| `require_b2b` | bool | true/false | false | Mutually exclusive with require_b2c | Only show B2B opportunities |
| `require_b2c` | bool | true/false | false | Mutually exclusive with require_b2b | Only show B2C opportunities |
| `min_mention_count` | int | 1-10000+ | 20 | None | Minimum mentions across sources |
| `min_engagement_score` | float | 0-100 | 30 | None | Minimum engagement score |
| `max_content_age_days` | int | 1-3650+ | 365 | None | Maximum age of content to consider |
| `require_correlation` | bool | true/false | true | None | Require cross-source correlation |
| `min_correlation_sources` | int | 2-10 | 2 | If require_correlation = true | Minimum sources for correlation |
| `min_correlation_strength` | float | 0.0-1.0 | 0.3 | If require_correlation = true | Minimum correlation strength |

**Database Schema:**
```sql
INSERT INTO application_config (application_id, config_key, config_value, value_type, category, validation_rule) VALUES
('opportunity-finder-app-id', 'require_paid_solution', 'true', 'bool', 'validation', NULL),
('opportunity-finder-app-id', 'min_competitor_mrr', '1000', 'float', 'validation', '{"min": 0}'),
('opportunity-finder-app-id', 'max_competitor_count', '50', 'int', 'validation', '{"min": 0}'),
('opportunity-finder-app-id', 'require_b2b', 'false', 'bool', 'validation', NULL),
('opportunity-finder-app-id', 'require_b2c', 'false', 'bool', 'validation', NULL),
('opportunity-finder-app-id', 'min_mention_count', '20', 'int', 'validation', '{"min": 1}'),
('opportunity-finder-app-id', 'min_engagement_score', '30', 'float', 'validation', '{"min": 0, "max": 100}'),
('opportunity-finder-app-id', 'max_content_age_days', '365', 'int', 'validation', '{"min": 1}'),
('opportunity-finder-app-id', 'require_correlation', 'true', 'bool', 'validation', NULL),
('opportunity-finder-app-id', 'min_correlation_sources', '2', 'int', 'validation', '{"min": 2, "max": 10}'),
('opportunity-finder-app-id', 'min_correlation_strength', '0.3', 'float', 'validation', '{"min": 0.0, "max": 1.0}');
```

---

## Dimension 5: Business Type Filters

**Category:** Filtering
**Validation:** None
**Admin UI:** Business Type Filters tab

| Parameter | Type | Possible Values | Default | Dependency | Description |
|-----------|------|-----------------|---------|------------|-------------|
| `business_type_filter` | str | "all", "b2b_only", "b2c_only", "both" | "all" | None | Filter by business type |
| `b2b_boost_multiplier` | float | 1.0-5.0 | 1.5 | Applied when business_type = "B2B" | Score multiplier for B2B opportunities |
| `b2c_penalty_multiplier` | float | 0.0-1.0 | 0.8 | Applied when business_type = "B2C" | Score multiplier for B2C opportunities |
| `b2b_confidence_threshold` | float | 0-100 | 60 | None | Minimum AI confidence for B2B classification |
| `b2c_confidence_threshold` | float | 0-100 | 60 | None | Minimum AI confidence for B2C classification |

**Database Schema:**
```sql
INSERT INTO application_config (application_id, config_key, config_value, value_type, category, validation_rule) VALUES
('opportunity-finder-app-id', 'business_type_filter', '"all"', 'string', 'filter', '{"enum": ["all", "b2b_only", "b2c_only", "both"]}'),
('opportunity-finder-app-id', 'b2b_boost_multiplier', '1.5', 'float', 'filter', '{"min": 1.0, "max": 5.0}'),
('opportunity-finder-app-id', 'b2c_penalty_multiplier', '0.8', 'float', 'filter', '{"min": 0.0, "max": 1.0}'),
('opportunity-finder-app-id', 'b2b_confidence_threshold', '60', 'float', 'filter', '{"min": 0, "max": 100}'),
('opportunity-finder-app-id', 'b2c_confidence_threshold', '60', 'float', 'filter', '{"min": 0, "max": 100}');
```

---

## Dimension 6: Category Classifications

**Category:** Filtering
**Validation:** None
**Admin UI:** Category Filters tab

| Parameter | Type | Possible Values | Default | Dependency | Description |
|-----------|------|-----------------|---------|------------|-------------|
| `category_filter` | List[str] | ["SaaS", "E-commerce", "DevTool", "Marketplace", "Content", "Agency", "Consulting", "Course", "App", "Other"] | ["all"] | Empty list = no filter | Filter by opportunity category |
| `category_boosts` | Dict[str, float] | {"SaaS": 1.2, "DevTool": 1.3, ...} | {} | Multiplier for final score | Score boost per category |
| `category_blacklist` | List[str] | Any category | [] | Exclude these entirely | Categories to exclude |

**Database Schema:**
```sql
INSERT INTO application_config (application_id, config_key, config_value, value_type, category) VALUES
('opportunity-finder-app-id', 'category_filter', '["all"]', 'list', 'filter'),
('opportunity-finder-app-id', 'category_boosts', '{}', 'dict', 'filter'),
('opportunity-finder-app-id', 'category_blacklist', '[]', 'list', 'filter');
```

---

## Dimension 7: Maturity Level Preferences

**Category:** Filtering
**Validation:** None
**Admin UI:** Maturity Filters tab

| Parameter | Type | Possible Values | Default | Dependency | Description |
|-----------|------|-----------------|---------|------------|-------------|
| `maturity_filter` | str | "all", "emerging_only", "growing_only", "established_only", "early_stage" | "emerging_only" | None | Filter by opportunity maturity |
| `emerging_definition_max_competitors` | int | 0-100 | 10 | Competitors <= this = "emerging" | Max competitors for "emerging" classification |
| `emerging_definition_max_age_months` | int | 0-120 | 12 | Topic age <= this = "emerging" | Max age for "emerging" classification |
| `growing_definition_min_velocity` | float | 1.0-5.0 | 1.5 | Velocity >= this = "growing" | Min velocity for "growing" classification |
| `established_definition_min_competitors` | int | 10-1000 | 50 | Competitors >= this = "established" | Min competitors for "established" |

**Database Schema:**
```sql
INSERT INTO application_config (application_id, config_key, config_value, value_type, category, validation_rule) VALUES
('opportunity-finder-app-id', 'maturity_filter', '"emerging_only"', 'string', 'filter', '{"enum": ["all", "emerging_only", "growing_only", "established_only", "early_stage"]}'),
('opportunity-finder-app-id', 'emerging_definition_max_competitors', '10', 'int', 'filter', '{"min": 0, "max": 100}'),
('opportunity-finder-app-id', 'emerging_definition_max_age_months', '12', 'int', 'filter', '{"min": 0, "max": 120}'),
('opportunity-finder-app-id', 'growing_definition_min_velocity', '1.5', 'float', 'filter', '{"min": 1.0, "max": 5.0}'),
('opportunity-finder-app-id', 'established_definition_min_competitors', '50', 'int', 'filter', '{"min": 10, "max": 1000}');
```

---

## Dimension 8: AI Model Configuration

**Category:** AI
**Validation:** Model must exist in platform
**Admin UI:** AI Models tab

| Parameter | Type | Possible Values | Default | Dependency | Description |
|-----------|------|-----------------|---------|------------|-------------|
| `analysis_model` | str | "gpt-4o", "claude-3.5-sonnet", "glm-4.7", "deepseek-chat", "llama-3.1-70b", "custom" | "gpt-4o" | Must be available in AI platform | Primary model for opportunity analysis |
| `classification_model` | str | (same as above) | "gpt-4o-mini" | Faster/cheaper for classification | Model for B2B/B2C/category classification |
| `temperature` | float | 0.0-2.0 | 0.3 | None | AI temperature (lower = more deterministic) |
| `max_tokens` | int | 100-8000 | 2000 | None | Maximum tokens for AI response |
| `enable_model_fallback` | bool | true/false | true | None | Enable fallback to secondary model |
| `fallback_model` | str | (same as analysis_model) | "claude-3.5-sonnet" | If enable_model_fallback = true | Secondary model if primary fails |

**Database Schema:**
```sql
INSERT INTO application_config (application_id, config_key, config_value, value_type, category, validation_rule) VALUES
('opportunity-finder-app-id', 'analysis_model', '"gpt-4o"', 'string', 'ai', '{"enum": ["gpt-4o", "claude-3.5-sonnet", "glm-4.7", "deepseek-chat", "llama-3.1-70b", "custom"]}'),
('opportunity-finder-app-id', 'classification_model', '"gpt-4o-mini"', 'string', 'ai', '{"enum": ["gpt-4o", "claude-3.5-sonnet", "glm-4.7", "deepseek-chat", "llama-3.1-70b", "custom"]}'),
('opportunity-finder-app-id', 'temperature', '0.3', 'float', 'ai', '{"min": 0.0, "max": 2.0}'),
('opportunity-finder-app-id', 'max_tokens', '2000', 'int', 'ai', '{"min": 100, "max": 8000}'),
('opportunity-finder-app-id', 'enable_model_fallback', 'true', 'bool', 'ai', NULL),
('opportunity-finder-app-id', 'fallback_model', '"claude-3.5-sonnet"', 'string', 'ai', '{"enum": ["gpt-4o", "claude-3.5-sonnet", "glm-4.7", "deepseek-chat", "llama-3.1-70b", "custom"]}');
```

---

## Dimension 9: Geographic Configuration

**Category:** Filtering
**Validation:** None
**Admin UI:** Geographic Filters tab

| Parameter | Type | Possible Values | Default | Dependency | Description |
|-----------|------|-----------------|---------|------------|-------------|
| `target_geographies` | List[str] | ["US", "UK", "CA", "AU", "EU", "global", ...] | ["US", "UK", "CA", "AU"] | None | Target geographic regions |
| `geography_filter_mode` | str | "any", "all", "primary" | "any" | None | How to match multiple geographies |
| `language_filter` | List[str] | ["en", "es", "fr", "de", ...] | ["en"] | None | Content language filter |
| `exclude_geographies` | List[str] | Any geography code | [] | None | Geographies to exclude |

**Database Schema:**
```sql
INSERT INTO application_config (application_id, config_key, config_value, value_type, category, validation_rule) VALUES
('opportunity-finder-app-id', 'target_geographies', '["US", "UK", "CA", "AU"]', 'list', 'geo', NULL),
('opportunity-finder-app-id', 'geography_filter_mode', '"any"', 'string', 'geo', '{"enum": ["any", "all", "primary"]}'),
('opportunity-finder-app-id', 'language_filter', '["en"]', 'list', 'geo', NULL),
('opportunity-finder-app-id', 'exclude_geographies', '[]', 'list', 'geo', NULL);
```

---

## Dimension 10: Source-Specific Parameters

**Category:** Source
**Validation:** Only applies when source is enabled
**Admin UI:** Source-Specific tab (subsections per source)

### Reddit Parameters

| Parameter | Type | Possible Values | Default | Dependency | Description |
|-----------|------|-----------------|---------|------------|-------------|
| `reddit_subreddits` | List[str] | Any subreddit | ["SaaS", "Entrepreneur", "SideProject", "freelance", "smallbusiness"] | source = "reddit" | Subreddits to scan |
| `reddit_min_upvotes` | int | 0-10000 | 10 | source = "reddit" | Minimum upvotes for inclusion |
| `reddit_min_comments` | int | 0-5000 | 5 | source = "reddit" | Minimum comments for inclusion |
| `reddit_post_age_max_days` | int | 1-3650 | 365 | source = "reddit" | Maximum post age to consider |
| `reddit_sort_by` | str | "hot", "new", "top", "relevance" | "hot" | source = "reddit" | Sort order for posts |

### Google Trends Parameters

| Parameter | Type | Possible Values | Default | Dependency | Description |
|-----------|------|-----------------|---------|------------|-------------|
| `trends_timeframe` | str | "today 3-m", "today 12-m", "today 5-y", "all" | "today 12-m" | source = "google_trends" | Search timeframe |
| `trends_category` | int | 0-1000+ | 0 | source = "google_trends" | Google Trends category ID (0 = all) |
| `trends_geocode` | str | "US", "GB", "CA", "AU", "" | "" | source = "google_trends" | Geographic code (empty = worldwide) |
| `trends_property` | str | "", "news", "images", "youtube", "froogle" | "" | source = "google_trends" | Search property (empty = web) |

### YouTube Parameters

| Parameter | Type | Possible Values | Default | Dependency | Description |
|-----------|------|-----------------|---------|------------|-------------|
| `youtube_min_views` | int | 0-10000000 | 1000 | source = "youtube" | Minimum views for inclusion |
| `youtube_max_results` | int | 1-100 | 20 | source = "youtube" | Maximum results to fetch |
| `youtube_published_after` | str | ISO date | "2024-01-01" | source = "youtube" | Only videos after this date |
| `youtube_duration` | str | "any", "short", "medium", "long" | "any" | source = "youtube" | Video duration filter |

### Gumroad/Etsy Parameters

| Parameter | Type | Possible Values | Default | Dependency | Description |
|-----------|------|-----------------|---------|------------|-------------|
| `marketplace_min_sales` | int | 0-100000 | 10 | source = "gumroad" or "etsy" | Minimum sales for inclusion |
| `marketplace_min_price` | float | 0-10000 | 5.0 | source = "gumroad" or "etsy" | Minimum price point |
| `marketplace_max_price` | float | 0-10000 | 500.0 | source = "gumroad" or "etsy" | Maximum price point (janky detection) |
| `marketplace_keywords` | List[str] | "template", "system", "tool", "automation" | ["template", "system"] | source = "gumroad" or "etsy" | Keywords to search for |

### Stack Overflow Parameters

| Parameter | Type | Possible Values | Default | Dependency | Description |
|-----------|------|-----------------|---------|------------|-------------|
| `stackoverflow_min_score` | int | 0-10000 | 5 | source = "stackoverflow" | Minimum question score |
| `stackoverflow_min_answers` | int | 0-100 | 1 | source = "stackoverflow" | Minimum answer count |
| `stackoverflow_tags` | List[str] | Any SO tag | ["api", "web-scraping", "automation"] | source = "stackoverflow" | Tags to filter by |
| `stackoverflow_question_age_max_days` | int | 1-3650 | 365 | source = "stackoverflow" | Maximum question age |

### Zapier/Make/n8n Parameters

| Parameter | Type | Possible Values | Default | Dependency | Description |
|-----------|------|-----------------|---------|------------|-------------|
| `integration_min_uses` | int | 0-1000000 | 1000 | source = "zapier" or "make" or "n8n" | Minimum usage count |
| `integration_min_steps` | int | 2-10+ | 3 | source = "zapier" or "make" or "n8n" | Minimum workflow steps |
| `integration_popular_apps` | List[str] | Popular app names | ["slack", "gmail", "sheets", "notion"] | source = "zapier" or "make" or "n8n" | Apps to monitor |

**Database Schema:**
```sql
-- Reddit
INSERT INTO application_config (application_id, config_key, config_value, value_type, category) VALUES
('opportunity-finder-app-id', 'reddit_subreddits', '["SaaS", "Entrepreneur", "SideProject", "freelance", "smallbusiness"]', 'list', 'source_reddit'),
('opportunity-finder-app-id', 'reddit_min_upvotes', '10', 'int', 'source_reddit'),
('opportunity-finder-app-id', 'reddit_min_comments', '5', 'int', 'source_reddit'),
('opportunity-finder-app-id', 'reddit_post_age_max_days', '365', 'int', 'source_reddit'),
('opportunity-finder-app-id', 'reddit_sort_by', '"hot"', 'string', 'source_reddit');

-- Google Trends
INSERT INTO application_config (application_id, config_key, config_value, value_type, category) VALUES
('opportunity-finder-app-id', 'trends_timeframe', '"today 12-m"', 'string', 'source_google_trends'),
('opportunity-finder-app-id', 'trends_category', '0', 'int', 'source_google_trends'),
('opportunity-finder-app-id', 'trends_geocode', '""', 'string', 'source_google_trends'),
('opportunity-finder-app-id', 'trends_property', '""', 'string', 'source_google_trends');

-- YouTube
INSERT INTO application_config (application_id, config_key, config_value, value_type, category) VALUES
('opportunity-finder-app-id', 'youtube_min_views', '1000', 'int', 'source_youtube'),
('opportunity-finder-app-id', 'youtube_max_results', '20', 'int', 'source_youtube'),
('opportunity-finder-app-id', 'youtube_published_after', '"2024-01-01"', 'string', 'source_youtube'),
('opportunity-finder-app-id', 'youtube_duration', '"any"', 'string', 'source_youtube');

-- Marketplace (Gumroad/Etsy)
INSERT INTO application_config (application_id, config_key, config_value, value_type, category) VALUES
('opportunity-finder-app-id', 'marketplace_min_sales', '10', 'int', 'source_marketplace'),
('opportunity-finder-app-id', 'marketplace_min_price', '5.0', 'float', 'source_marketplace'),
('opportunity-finder-app-id', 'marketplace_max_price', '500.0', 'float', 'source_marketplace'),
('opportunity-finder-app-id', 'marketplace_keywords', '["template", "system"]', 'list', 'source_marketplace');

-- Stack Overflow
INSERT INTO application_config (application_id, config_key, config_value, value_type, category) VALUES
('opportunity-finder-app-id', 'stackoverflow_min_score', '5', 'int', 'source_stackoverflow'),
('opportunity-finder-app-id', 'stackoverflow_min_answers', '1', 'int', 'source_stackoverflow'),
('opportunity-finder-app-id', 'stackoverflow_tags', '["api", "web-scraping", "automation"]', 'list', 'source_stackoverflow'),
('opportunity-finder-app-id', 'stackoverflow_question_age_max_days', '365', 'int', 'source_stackoverflow');

-- Integration (Zapier/Make/n8n)
INSERT INTO application_config (application_id, config_key, config_value, value_type, category) VALUES
('opportunity-finder-app-id', 'integration_min_uses', '1000', 'int', 'source_integration'),
('opportunity-finder-app-id', 'integration_min_steps', '3', 'int', 'source_integration'),
('opportunity-finder-app-id', 'integration_popular_apps', '["slack", "gmail", "sheets", "notion"]', 'list', 'source_integration');
```

---

## Dimension 11: Output Configuration

**Category:** Display
**Validation:** None
**Admin UI:** Output Options tab

| Parameter | Type | Possible Values | Default | Dependency | Description |
|-----------|------|-----------------|---------|------------|-------------|
| `max_results_returned` | int | 1-1000 | 50 | None | Maximum opportunities to return per query |
| `sort_by` | str | "score", "velocity", "volume", "recency", "correlation" | "score" | None | Primary sort field |
| `sort_order` | str | "desc", "asc" | "desc" | None | Sort order |
| `include_score_breakdown` | bool | true/false | false | Admin-only visibility | Show component scores in output |
| `include_source_links` | bool | true/false | true | None | Include direct links to sources |
| `include_raw_data` | bool | true/false | false | Debug mode only | Include raw source data |
| `include_ai_reasoning` | bool | true/false | false | Debug mode only | Include AI analysis reasoning |

**Database Schema:**
```sql
INSERT INTO application_config (application_id, config_key, config_value, value_type, category, validation_rule) VALUES
('opportunity-finder-app-id', 'max_results_returned', '50', 'int', 'output', '{"min": 1, "max": 1000}'),
('opportunity-finder-app-id', 'sort_by', '"score"', 'string', 'output', '{"enum": ["score", "velocity", "volume", "recency", "correlation"]}'),
('opportunity-finder-app-id', 'sort_order', '"desc"', 'string', 'output', '{"enum": ["desc", "asc"]}'),
('opportunity-finder-app-id', 'include_score_breakdown', 'false', 'bool', 'output', NULL),
('opportunity-finder-app-id', 'include_source_links', 'true', 'bool', 'output', NULL),
('opportunity-finder-app-id', 'include_raw_data', 'false', 'bool', 'output', NULL),
('opportunity-finder-app-id', 'include_ai_reasoning', 'false', 'bool', 'output', NULL);
```

---

## Dimension 12: Notification & Alerting

**Category:** Alerting
**Validation:** None
**Admin UI:** Notifications tab

| Parameter | Type | Possible Values | Default | Dependency | Description |
|-----------|------|-----------------|---------|------------|-------------|
| `alert_on_new_opportunities` | bool | true/false | true | None | Enable new opportunity alerts |
| `alert_threshold` | float | 0-100 | 70 | None | Only alert if score >= this |
| `alert_velocity_spike` | bool | true/false | true | None | Alert on search growth spikes |
| `alert_velocity_spike_threshold` | float | 1.5-10.0 | 3.0 | None | Velocity growth multiplier for alerts |
| `alert_digest_frequency` | str | "immediate", "hourly", "daily", "weekly" | "daily" | None | Alert digest frequency |
| `alert_channels` | List[str] | ["email", "webhook", "slack", "dashboard"] | ["dashboard"] | None | Alert delivery channels |

**Database Schema:**
```sql
INSERT INTO application_config (application_id, config_key, config_value, value_type, category, validation_rule) VALUES
('opportunity-finder-app-id', 'alert_on_new_opportunities', 'true', 'bool', 'alert', NULL),
('opportunity-finder-app-id', 'alert_threshold', '70', 'float', 'alert', '{"min": 0, "max": 100}'),
('opportunity-finder-app-id', 'alert_velocity_spike', 'true', 'bool', 'alert', NULL),
('opportunity-finder-app-id', 'alert_velocity_spike_threshold', '3.0', 'float', 'alert', '{"min": 1.5, "max": 10.0}'),
('opportunity-finder-app-id', 'alert_digest_frequency', '"daily"', 'string', 'alert', '{"enum": ["immediate", "hourly", "daily", "weekly"]}'),
('opportunity-finder-app-id', 'alert_channels', '["dashboard"]', 'list', 'alert', NULL);
```

---

## Configuration Validation Rules

### 1. Weight Sum Validation

**Rule:** `weight_demand + weight_competition + weight_revenue + weight_feasibility + weight_timing + weight_risk === 1.0`

**Error Message:** "Weight sum must equal 1.0. Current sum: {actual_sum}"

**Validation Code:**
```python
def validate_weight_sum(config: Dict) -> ValidationResult:
    weight_params = ['weight_demand', 'weight_competition', 'weight_revenue',
                     'weight_feasibility', 'weight_timing', 'weight_risk']
    total = sum(config.get(p, 0.0) for p in weight_params)
    if abs(total - 1.0) > 0.001:  # Allow floating point tolerance
        return ValidationResult(
            valid=False,
            error=f"Weight sum must equal 1.0. Current sum: {total:.2f}"
        )
    return ValidationResult(valid=True)
```

### 2. Threshold Ordering Validation

**Rule:** `score_threshold_build > score_threshold_validate > score_threshold_investigate`

**Error Message:** "Thresholds must be ordered: build ({build}) > validate ({validate}) > investigate ({investigate})"

**Validation Code:**
```python
def validate_threshold_order(config: Dict) -> ValidationResult:
    build = config.get('score_threshold_build', 75)
    validate = config.get('score_threshold_validate', 60)
    investigate = config.get('score_threshold_investigate', 40)

    if not (build > validate > investigate):
        return ValidationResult(
            valid=False,
            error=f"Thresholds must be ordered: build ({build}) > validate ({validate}) > investigate ({investigate})"
        )
    return ValidationResult(valid=True)
```

### 3. Business Type Mutual Exclusion

**Rule:** `NOT (require_b2b = true AND require_b2c = true)`

**Error Message:** "Cannot require both B2B and B2C exclusively"

**Validation Code:**
```python
def validate_business_type_exclusion(config: Dict) -> ValidationResult:
    if config.get('require_b2b', False) and config.get('require_b2c', False):
        return ValidationResult(
            valid=False,
            error="Cannot require both B2B and B2C exclusively"
        )
    return ValidationResult(valid=True)
```

### 4. Correlation Requirements

**Rule:** IF `require_correlation = true`:
- `min_correlation_sources >= 2`
- `min_correlation_strength > 0.0`
- `enabled_sources.length >= min_correlation_sources`

**Error Message:** "Correlation requires {min_sources}+ sources, only {enabled} enabled"

**Validation Code:**
```python
def validate_correlation_requirements(config: Dict) -> ValidationResult:
    if config.get('require_correlation', True):
        enabled = len(config.get('enabled_sources', []))
        min_sources = config.get('min_correlation_sources', 2)

        if enabled < min_sources:
            return ValidationResult(
                valid=False,
                error=f"Correlation requires {min_sources}+ sources, only {enabled} enabled"
            )

        min_strength = config.get('min_correlation_strength', 0.3)
        if min_strength <= 0.0:
            return ValidationResult(
                valid=False,
                error="min_correlation_strength must be > 0.0 when correlation is required"
            )

    return ValidationResult(valid=True)
```

### 5. Source-Specific Validation

**Rule:** Source-specific parameters must be valid when source is enabled

**Validation Code:**
```python
def validate_source_config(source: str, config: Dict) -> ValidationResult:
    if source == "reddit":
        subreddits = config.get('reddit_subreddits', [])
        if not subreddits:
            return ValidationResult(
                valid=False,
                error="reddit_subreddits must not be empty when reddit is enabled"
            )

    elif source == "google_trends":
        # Check for valid API key (implementation-specific)
        if not has_valid_google_trends_api_key():
            return ValidationResult(
                valid=False,
                error="Google Trends API key must be configured"
            )

    # Add validation for other sources...

    return ValidationResult(valid=True)
```

---

## Configuration Presets

### Preset 1: "Aggressive Emerging Hunter"

**Focus:** EARLY mover, high growth, low competition

**Configuration:**
```yaml
# Maturity Settings
maturity_filter: "emerging_only"
emerging_definition_max_competitors: 5
emerging_definition_max_age_months: 6

# Velocity Thresholds
velocity_threshold_high: 3.0  # 3x growth = high

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

**When to Use:**
- Looking for blue ocean opportunities
- Willing to take calculated risks
- Want to be first to market
- Focus on emerging trends

---

### Preset 2: "Validated B2B SaaS Finder"

**Focus:** Proven demand, B2B, existing revenue

**Configuration:**
```yaml
# Business Type Filter
business_type_filter: "b2b_only"
b2b_boost_multiplier: 2.0  # Strong B2B preference

# Category Filter
category_filter: ["SaaS"]
category_boosts: {"SaaS": 1.5}

# Validation Rules (strict)
require_paid_solution: true
min_competitor_mrr: 5000  # Must have proven revenue ($5k+ MRR)
max_competitor_count: 20  # But not too crowded
require_correlation: true
min_correlation_sources: 3  # Require strong validation

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

**When to Use:**
- Building B2B SaaS specifically
- Want proven market validation
- Prefer opportunities with existing revenue
- Lower risk tolerance

---

### Preset 3: "Janky Solution Upgrader"

**Focus:** People paying for primitive solutions

**Configuration:**
```yaml
# Data Sources (marketplace focus)
enabled_sources: ["gumroad", "etsy", "reddit", "google_trends"]

# Marketplace Settings
marketplace_min_sales: 50  # Must have meaningful sales
marketplace_max_price: 100  # Low price point = primitive/janky solution
marketplace_keywords: ["template", "system", "tool", "automation", "spreadsheet", "notion"]

# Scoring Weights (revenue is proof)
weight_revenue: 0.50  # Revenue is the strongest signal
weight_demand: 0.30
weight_competition: 0.10
weight_feasibility: 0.10
weight_timing: 0.0
weight_risk: 0.0

# Validation
require_paid_solution: true
min_competitor_mrr: 0  # Looking for primitive, not established
require_correlation: true
min_correlation_sources: 2  # Reddit + marketplace confirms opportunity
```

**When to Use:**
- Looking to upgrade existing solutions
- Want proof of payment (janky solutions with sales)
- Focus on product improvement opportunities
- Lower technical risk (problem validated)

---

## Unified Data Schema Reference

### OpportunityMention Schema

Every data source normalizes output to this schema:

```python
from pydantic import BaseModel
from typing import Dict, Any, List
from datetime import datetime

class OpportunityMention(BaseModel):
    """Unified schema for all data source outputs."""

    # ========== Identity ==========
    mention_id: str                          # Unique identifier (UUID)
    source_type: str                         # "reddit", "google_trends", "youtube", etc.
    source_url: str                          # Direct link to original
    raw_data: Dict[str, Any]                 # Original source data (preserve for debugging)

    # ========== Content ==========
    title: str                               # Headline/subject
    description: str                         # Main content/body
    extracted_entities: List[str]            # NLP extracted: tools, platforms, categories

    # ========== Engagement (Normalized 0-100 across all sources) ==========
    engagement_score: float                  # Combined upvotes/views/likes/etc.
    engagement_breakdown: Dict[str, int]     # Source-specific metrics preserved
    # Example Reddit: {"upvotes": 150, "comments": 45, "awards": 2}
    # Example Google Trends: {"search_volume": 5000, "growth_rate": 2.5}

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
    # Example Reddit: {"subreddit": "r/SaaS", "is_self_post": true}
    # Example Google Trends: {"related_queries": [...], "geocode": "US"}

    # ========== Cross-Reference ==========
    correlated_sources: List[str]            # Other sources mentioning same opportunity
    correlation_strength: float              # 0-1 (how strong is the correlation?)

    class Config:
        json_schema_extra = {
            "example": {
                "mention_id": "550e8400-e29b-41d4-a716-446655440000",
                "source_type": "reddit",
                "source_url": "https://reddit.com/r/SaaS/comments/abc123",
                "raw_data": {...},
                "title": "Looking for a better CRM export tool",
                "description": "I hate how I can't easily export my contacts from my CRM...",
                "extracted_entities": ["CRM", "export", "contacts", "integration"],
                "engagement_score": 72.5,
                "engagement_breakdown": {"upvotes": 150, "comments": 45},
                "published_at": "2026-01-15T10:30:00Z",
                "collected_at": "2026-01-19T14:20:00Z",
                "recency_score": 85.0,
                "category": "DevTool",
                "business_type": "B2B",
                "maturity_level": "Emerging",
                "confidence_score": 82.0,
                "signals": {"subreddit": "SaaS", "is_self_post": true},
                "correlated_sources": ["google_trends"],
                "correlation_strength": 0.65
            }
        }
```

### Database Schema for OpportunityMention

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

## Data Source Interface Contract

Every data source MUST implement this interface:

```python
from abc import ABC, abstractmethod
from typing import Dict, Any, List, Optional, Tuple
from datetime import datetime
from pydantic import BaseModel

class OpportunityMention(BaseModel):
    """(See schema definition above)"""
    pass


class DataSourceInterface(ABC):
    """Contract that all data sources must implement."""

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

---

## AI Prompt Templates

### Reddit Analysis Prompt

```python
REDDIT_PROMPT = """You are analyzing a Reddit post for opportunity discovery.

POST CONTENT:
{content}

POST METADATA:
- Subreddit: {subreddit}
- Upvotes: {upvotes}
- Comments: {comment_count}
- Posted: {timestamp}

TASK:
1. Classify business type: Is this B2B or B2C? Look for:
   - B2B indicators: "my boss", "our team", "company", "client", "agency", "we need", "our business"
   - B2C indicators: "I", "personal", "home", "for myself", "personal use"

2. Extract the core problem being discussed (one sentence).

3. Extract any mentioned tools, platforms, or workarounds (list).

4. Score components (0-100 each):
   - DEMAND: How many people care? (engagement + specificity + comment activity)
   - PAIN: How urgent is this problem? (emotion language + frequency + "hate", "frustrated", "tired of")
   - FEASIBILITY: How solvable is this technically? (complexity assessment based on described needs)
   - B2B_POTENTIAL: Likelihood this is B2B opportunity (0-100)

5. Identify signals:
   - Multi-tool workaround mentioned? (yes/no + which tools)
   - Willingness to pay mentioned? (yes/no + evidence like "would pay for", "looking for a tool that does X for $Y/month")
   - Existing solutions mentioned? (yes/no + which ones)

6. Categorize the opportunity:
   - category: One of "SaaS", "E-commerce", "DevTool", "Marketplace", "Content", "Agency", "Consulting", "Course", "App", "Other"
   - maturity: One of "Emerging" (new/undiscovered), "Growing" (gaining traction), "Established" (saturated)

OUTPUT JSON:
{{
    "business_type": "B2B|B2C|Both|Unclear",
    "category": "SaaS|E-commerce|DevTool|Marketplace|Content|Agency|Consulting|Course|App|Other",
    "core_problem": "One sentence summary of the problem",
    "mentioned_tools": ["tool1", "tool2", "tool3"],
    "component_scores": {{
        "demand": 0-100,
        "pain": 0-100,
        "feasibility": 0-100,
        "b2b_potential": 0-100
    }},
    "signals": {{
        "multi_tool_workaround": {{"found": true, "tools": ["tool1", "tool2"]}},
        "willingness_to_pay": {{"found": true, "evidence": "quote about willingness to pay"}},
        "existing_solutions": {{"found": true, "solutions": ["solution1", "solution2"]}}
    }},
    "maturity_level": "Emerging|Growing|Established"
}}
"""
```

### Google Trends Analysis Prompt

```python
GOOGLE_TRENDS_PROMPT = """You are analyzing Google Trends search data for opportunity discovery.

SEARCH TERM: {query}
SEARCH VOLUME: {volume}
VELOCITY: {velocity}x growth (vs 90 days ago)
GEOGRAPHIC DISTRIBUTION: {geographies}

TASK:
1. Assess commercial intent: Is this informational curiosity or potential commercial interest?
   - Learning interest: "how to", "tutorial", "best", "what is", "guide"
   - Solution-seeking interest: "software", "tool", "platform", "app", "service", "solution"
   - Comparison interest: "vs", "alternative", "like", "similar to"

2. Score components (0-100 each):
   - DEMAND_VELOCITY: Growth rate score (higher velocity = higher score)
   - DEMAND_VOLUME: Absolute volume score (normalize 0-100 based on typical ranges)
   - COMMERCIAL_INTENT: Likelihood this translates to purchases (solution-seeking = high)

3. Cross-reference hypothesis: If this term is trending, what type of product would satisfy this search intent?
   - Describe the product/service that would address this search intent

4. Categorize:
   - category: Best fit from "SaaS", "E-commerce", "DevTool", "Marketplace", "Content", "Agency", "Consulting", "Course", "App", "Other"
   - maturity: Based on velocity - "Emerging" (low velocity, new), "Growing" (high velocity), "Established" (high volume, low velocity)

OUTPUT JSON:
{{
    "commercial_intent": 0-100,
    "intent_type": "learning|solution_seeking|comparison",
    "hypothesis": "People searching for this are looking for [product/service description]",
    "component_scores": {{
        "demand_velocity": 0-100,
        "demand_volume": 0-100,
        "commercial_intent": 0-100
    }},
    "category": "SaaS|E-commerce|DevTool|Marketplace|Content|Agency|Consulting|Course|App|Other",
    "maturity_level": "Emerging|Growing|Established"
}}
"""
```

### Indie Hackers Analysis Prompt

```python
INDIEHACKERS_PROMPT = """You are analyzing an Indie Hackers post/discussion for opportunity discovery.

CONTENT:
{content}

METADATA:
- Type: {type} (product, discussion, interview)
- Engagement: {engagement_metrics}
- Revenue Data: {revenue_data}

TASK:
1. Classify business type and extract revenue validation if present.

2. Extract core problem and any mentioned solutions/tools.

3. Score components focusing on:
   - REVENUE_PROOF: Is there MRR/ARR data? How much?
   - DEMAND: Community engagement
   - FEASIBILITY: Technical complexity

4. Identify validation signals:
   - Founder revenue mentioned?
   - Customer pain discussed?
   - Market size indicators?

OUTPUT JSON:
{{
    "business_type": "B2B|B2C|Both|Unclear",
    "category": "SaaS|E-commerce|DevTool|Other",
    "core_problem": "...",
    "mentioned_tools": [...],
    "revenue_data": {{"found": true, "mrr": 5000, "source": "interview"}},
    "component_scores": {{
        "demand": 0-100,
        "revenue_proof": 0-100,
        "feasibility": 0-100
    }},
    "maturity_level": "Emerging|Growing|Established"
}}
"""
```

### Product Hunt Analysis Prompt

```python
PRODUCT_HUNT_PROMPT = """You are analyzing a Product Hunt launch for opportunity discovery.

PRODUCT: {product_name}
TAGLINE: {tagline}
DESCRIPTION: {description}
UPVOTES: {upvotes}
COMMENTS: {comment_count}

TASK:
1. Extract the problem this product solves.

2. Assess market validation:
   - Upvote count indicates interest
   - Comments reveal user pain points
   - Is this early (validation opportunity) or established (competitive)?

3. Score components:
   - DEMAND: Upvote count + comment activity
   - COMPETITION: Is this a crowded space?
   - FEASIBILITY: Technical complexity based on product type

4. Identify differentiation opportunities:
   - What are users complaining about?
   - What features are missing?

OUTPUT JSON:
{{
    "business_type": "B2B|B2C|Both|Unclear",
    "category": "...",
    "core_problem": "Problem this product solves",
    "component_scores": {{
        "demand": 0-100,
        "competition": 0-100,
        "feasibility": 0-100
    }},
    "signals": {{
        "upvotes": 1234,
        "user_complaints": ["complaint1", "complaint2"],
        "missing_features": ["feature1", "feature2"]
    }},
    "maturity_level": "Emerging|Growing|Established"
}}
"""
```

### Hacker News Analysis Prompt

```python
HACKER_NEWS_PROMPT = """You are analyzing a Hacker News post for opportunity discovery.

POST TYPE: {post_type} (Ask HN, Show HN, Launch)
CONTENT:
{content}

METADATA:
- Upvotes: {upvotes}
- Comments: {comment_count}

TASK:
1. Identify if this is Ask HN (problem/opportunity) or Show HN (validation).

2. Extract technical pain points and developer needs.

3. Score components:
   - DEMAND: HN engagement
   - FEASIBILITY: Technical complexity (dev tools = typically feasible)
   - B2B_POTENTIAL: Developer tools often B2B

4. Identify opportunities:
   - What are developers complaining about?
   - What tools do they wish existed?

OUTPUT JSON:
{{
    "business_type": "B2B|B2C|Both|Unclear",
    "category": "DevTool|SaaS|Other",
    "core_problem": "...",
    "mentioned_tools": [...],
    "component_scores": {{
        "demand": 0-100,
        "pain": 0-100,
        "feasibility": 0-100,
        "b2b_potential": 0-100
    }},
    "signals": {{
        "post_type": "ask|show|launch",
        "developer_request": true/false,
        "complaints": [...]
    }},
    "maturity_level": "Emerging|Growing|Established"
}}
"""
```

---

## Appendix A: Parameter Quick Reference

### All Parameters by Category

**Core Parameters (19):**
- enabled_sources, source_priority, source_fallback_enabled, min_confidence_threshold
- weight_demand, weight_demand_velocity, weight_demand_volume, weight_competition, weight_revenue, weight_feasibility, weight_timing, weight_risk
- score_threshold_build, score_threshold_validate, score_threshold_investigate
- velocity_threshold_high, velocity_threshold_medium, volume_threshold_high, volume_threshold_medium

**Validation Parameters (11):**
- require_paid_solution, min_competitor_mrr, max_competitor_count, require_b2b, require_b2c, min_mention_count, min_engagement_score, max_content_age_days, require_correlation, min_correlation_sources, min_correlation_strength

**Filtering Parameters (19):**
- business_type_filter, b2b_boost_multiplier, b2c_penalty_multiplier, b2b_confidence_threshold, b2c_confidence_threshold
- category_filter, category_boosts, category_blacklist
- maturity_filter, emerging_definition_max_competitors, emerging_definition_max_age_months, growing_definition_min_velocity, established_definition_min_competitors
- target_geographies, geography_filter_mode, language_filter, exclude_geographies

**AI Parameters (7):**
- analysis_model, classification_model, temperature, max_tokens, enable_model_fallback, fallback_model

**Source-Specific Parameters (26+):**
- Reddit: reddit_subreddits, reddit_min_upvotes, reddit_min_comments, reddit_post_age_max_days, reddit_sort_by
- Google Trends: trends_timeframe, trends_category, trends_geocode, trends_property
- YouTube: youtube_min_views, youtube_max_results, youtube_published_after, youtube_duration
- Marketplace: marketplace_min_sales, marketplace_min_price, marketplace_max_price, marketplace_keywords
- Stack Overflow: stackoverflow_min_score, stackoverflow_min_answers, stackoverflow_tags, stackoverflow_question_age_max_days
- Integration: integration_min_uses, integration_min_steps, integration_popular_apps

**Output Parameters (7):**
- max_results_returned, sort_by, sort_order, include_score_breakdown, include_source_links, include_raw_data, include_ai_reasoning

**Alerting Parameters (6):**
- alert_on_new_opportunities, alert_threshold, alert_velocity_spike, alert_velocity_spike_threshold, alert_digest_frequency, alert_channels

**Total: 90+ parameters**

---

## Appendix B: Implementation Checklist

### Database Setup

- [ ] Create `application_config` table
- [ ] Create `opportunity_mentions` table
- [ ] Create `mention_correlations` table
- [ ] Add `application_id` foreign key to `ai_costs` table
- [ ] Create indexes for performance
- [ ] Seed default configuration values

### Backend Implementation

- [ ] Implement `ConfigurationValidator` class
- [ ] Implement `DataSourceInterface` abstract class
- [ ] Implement `OpportunityMention` Pydantic model
- [ ] Implement `CorrelationEngine` class
- [ ] Create configuration preset loader
- [ ] Implement source-specific AI prompt templates

### Admin Panel

- [ ] Create 12-tab configuration interface
- [ ] Implement weight sum validation UI
- [ ] Implement threshold ordering validation UI
- [ ] Create preset selector and save functionality
- [ ] Create validation error display
- [ ] Create source health monitoring page

### Testing

- [ ] Test all validation rules
- [ ] Test configuration preset loading
- [ ] Test source-specific parameter validation
- [ ] Test AI prompt template selection
- [ ] Test OpportunityMention normalization

---

**Document Version:** 1.0
**Last Updated:** 2026-01-19
**Next Review:** After PRD updates completed
