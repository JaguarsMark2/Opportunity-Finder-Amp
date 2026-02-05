# Minimum MVP Briefing Document

**Date:** 2026-01-22
**Purpose:** Define the absolute minimum to prove the Opportunity Finder engine works
**Status:** Ready for BMAD PRD workflow

---

## Objective

Prove the concept works by demonstrating:
1. Can read from all 6 data sources
2. Can find opportunities using the 10 detection criteria
3. Can cross-reference across sources to validate opportunities
4. Can display results in a usable dashboard

**NOT in scope:** Scoring/grading viability, customer auth, payments, multi-user

---

## Competitive Reference

Based on analysis of existing tools:
- **IdeaBrowser** ($500-$3000/year) - Full research + execution plans
- **BigIdeasDB** ($99 lifetime) - Scrapes negative reviews, focuses on pain points
- **GummySearch** (shutting down Dec 2026) - Reddit only, categorizes as Pain Points/Solution Requests/Money Talk/Hot Discussions

**Our approach:** Combine multiple sources (like IdeaBrowser), focus on complaints/pain points (like BigIdeasDB), smart categorization (like GummySearch)

---

## 10 Opportunity Detection Criteria

| # | Criteria | What to Look For |
|---|----------|------------------|
| 1 | Complaints/Pain Points | "I hate", "frustrated with", negative language |
| 2 | Solution Requests | "Looking for a tool that does X" |
| 3 | Multi-tool Workarounds | "I use A + B + C together" |
| 4 | Willingness to Pay | "I'd pay for this" |
| 5 | Money Talk | Budget discussions, current spending |
| 6 | Negative Reviews | Poor reviews of existing tools |
| 7 | Frequency | Same problem mentioned by many |
| 8 | Recency | Recent complaints (last 30-90 days) |
| 9 | Search Demand | Google Trends activity |
| 10 | Growing Trend | Problem getting worse over time |

**Strongest signal:** Multiple criteria overlap (complaint + solution request + willingness to pay + search demand)

---

## 6 Data Sources

| Source | Purpose | Auth Required |
|--------|---------|---------------|
| Reddit | Primary complaints/pain points | Yes (PRAW) |
| Indie Hackers | Founder discussions, revenue data | No (scraping) |
| Product Hunt | Launch signals, validated ideas | Yes (API) |
| Hacker News | Developer pain points | No (Algolia API) |
| Google Trends | Search demand validation | Paid API |
| Competition Search | Existing solutions check | Yes (SerpAPI) |

---

## Architecture

### Admin Page (Config Only)

| Section | Fields |
|---------|--------|
| AI Config | ZAI API key, model selector (GLM 4.7 + others) |
| Data Sources | Credentials for each of 6 sources |
| Connection Test | Test button per source with status indicator |

**No run scan button on admin - that's on dashboard**

### Dashboard Page

| Element | Description |
|---------|-------------|
| Theme Toggle | Dark/light mode switch |
| Run Scan Button | Triggers the engine, watch results appear |
| Opportunity Cards | Up to 20-30 opportunities displayed |
| Stats Bar | Total found, sources active, etc. |

### Opportunity Card Content

Each card shows the raw evidence:
- Problem/complaint found
- Category (Pain Point / Solution Request / Money Talk / Hot Discussion)
- Sources that mentioned it (badges)
- Mention count per source
- Links to actual references (so you can verify)
- Google Trends signal (if any)
- Search demand indicator

**No scoring yet** - just "here's what we found"

---

## Tech Stack

| Component | Technology |
|-----------|------------|
| Backend | Python + Flask |
| Frontend | Existing HTML (React) + connect to real data |
| Database | SQLite (simple, can upgrade later) |
| AI | ZAI GLM 4.7 |

---

## Success Criteria

The minimum MVP is successful when:
1. All 6 sources fetch data without errors
2. AI identifies opportunities from the data
3. Cross-source correlation shows overlapping signals
4. Dashboard displays 10-30 real opportunities
5. You can click references and verify they're real
6. You can judge by eye whether these are real opportunities

**Go/No-Go:** If you find 10+ opportunities that look genuinely buildable, the engine works.

---

## What's Deferred

| Feature | Why Deferred |
|---------|--------------|
| 0-100 Scoring | Prove detection works first |
| Scoring weights | Not needed for proof |
| Customer auth | Personal testing only |
| Payments | No customers yet |
| Email alerts | Dashboard viewing sufficient |
| Landing page builder | Phase 2 feature |
| PDF export | Phase 2 feature |

---

## Expansion Path

This minimum MVP is designed to expand into the full PRD v3.0:
1. Add scoring engine (7-component weighted formula)
2. Add validation rules (paid solution exists, MRR threshold, etc.)
3. Add recommendation logic (Build immediately / Validate first / Reject)
4. Add customer auth and payments
5. Add email alerts and exports

---

## Next Steps

1. Run BMAD `/prd` workflow using this briefing
2. Generate architecture document
3. Create implementation stories
4. Build

---

**Document Version:** 1.0
**Last Updated:** 2026-01-22

