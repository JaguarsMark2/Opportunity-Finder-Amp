# Opportunity Finder - Phase 8: Admin Panel UX Design

**Version:** 2.0
**Date:** 2026-01-19
**Designer:** Claude (AI)
**Status:** Ready for Implementation

---

## What's New in Version 2.0

This version incorporates architectural decisions from the Opportunity Finder brainstorming session (2026-01-19), expanding the configuration interface from 4 basic sliders to a comprehensive 12-tab configuration system.

**Major Updates:**
- **Section 7 Expanded**: From 4 sliders to 12 configuration tabs covering all 90+ parameters
- **Section 8 Added**: Additional Admin Pages (Correlation Engine, Presets Manager, Source Health, Configuration History)
- **Validation UI Added**: Real-time validation error display with actionable feedback
- **Preset Selector Added**: Quick-select interface for configuration presets
- **Source-Specific Tabs**: Dedicated configuration sections for each data source type

**Reference:** See [Configuration_Reference.md](./Configuration_Reference.md) for complete parameter documentation.

---

## Table of Contents

1. [Executive Summary](#executive-summary)
2. [Admin Persona & Goals](#admin-persona--goals)
3. [Design System](#design-system)
4. [Information Architecture](#information-architecture)
5. [Page-by-Page Specifications](#page-by-page-specifications)
6. [User Flows](#user-flows)
7. [Component Library](#component-library)
8. [Accessibility Standards](#accessibility-standards)
9. [Responsive Design](#responsive-design)
10. [Interaction Patterns](#interaction-patterns)
11. [Error States & Validation](#error-states--validation)
12. [Success Metrics](#success-metrics)
13. [Appendix A: Updated Section Reference](#appendix-a-updated-section-reference)

---

## Executive Summary

The Opportunity Finder Admin Panel provides system administrators with a comprehensive interface to manage all aspects of the platform without requiring code changes or database access. The design maintains visual consistency with the user-facing dashboard while introducing admin-specific patterns for data management, configuration, and analytics.

**Key Design Principles:**
- **Clarity over density**: Admin interfaces prioritize clarity of information over maximizing data density
- **Progressive disclosure**: Complex operations are revealed step-by-step to reduce cognitive load
- **Auditability**: All actions are traceable with clear confirmation and undo capabilities
- **Role-aware**: Admin-specific visual cues differentiate admin interfaces from user interfaces

---

## Admin Persona & Goals

### Primary Persona: System Administrator

**Name:** Alex (they/them)
**Role:** Platform Administrator / SaaS Owner
**Technical Proficiency:** High

#### Goals

1. **Operational Efficiency**
   - Manage user subscriptions and issues quickly
   - Adjust pricing without engineering intervention
   - Configure data sources as APIs change
   - Monitor system health and performance

2. **Business Intelligence**
   - Track revenue metrics and growth trends
   - Understand user behavior and engagement
   - Identify high-value users and churn risks
   - Monitor opportunity discovery effectiveness

3. **System Control**
   - Fine-tune scoring algorithms
   - Manage scan schedules and resources
   - Control email communication frequency
   - Grant/revoke administrative access

#### Pain Points

- Current tools require database queries for simple changes
- No visibility into real-time system metrics
- Difficult to troubleshoot user issues
- Pricing changes require code deployment
- Scoring algorithm tuning is opaque

#### Mental Model

Alex understands:
- Database concepts (users, subscriptions, relationships)
- API integration patterns
- SaaS business metrics (MRR, churn, LTV)
- The opportunity scoring algorithm

Alex expects:
- Fast load times for data tables
- Bulk operations for efficiency
- Clear confirmation before destructive actions
- Search and filtering on all list views

---

## Design System

### Visual Language

The admin panel extends the existing user dashboard design system with admin-specific variations:

#### Core Color Palette

| Usage | Dark Mode | Purpose |
|-------|-----------|---------|
| **Background** | `bg-gradient-to-br from-slate-900 to-slate-800` | Page background |
| **Card Background** | `bg-slate-800/50 border-slate-700/50` | Container backgrounds |
| **Primary Action** | `bg-blue-600 hover:bg-blue-700` | CTAs, primary buttons |
| **Destructive Action** | `bg-red-600 hover:bg-red-700` | Delete, cancel, remove |
| **Success** | `text-emerald-500 bg-emerald-500/10 border-emerald-500` | Confirmations |
| **Warning** | `text-amber-500 bg-amber-500/10 border-amber-500` | Alerts |
| **Error** | `text-red-500 bg-red-500/10 border-red-500` | Errors |
| **Info** | `text-blue-500 bg-blue-500/10 border-blue-500` | Information |

#### Admin-Specific Accent Colors

| Admin Context | Color | Tailwind Classes |
|---------------|-------|------------------|
| **Admin Badge** | Purple | `bg-purple-600 text-white` |
| **Revenue** | Green | `text-emerald-400` |
| **Active Users** | Blue | `text-blue-400` |
| **Churn Risk** | Orange | `text-orange-400` |
| **System Status** | Cyan | `text-cyan-400` |

#### Typography Scale

```css
/* Font: Inter, system-ui, sans-serif */
--font-display: 800 32px/1.2 'Inter';    /* Page titles */
--font-h1: 700 24px/1.3 'Inter';         /* Section headers */
--font-h2: 600 18px/1.4 'Inter';         /* Card titles */
--font-h3: 600 16px/1.4 'Inter';         /* Subsection headers */
--font-body: 400 15px/1.6 'Inter';       /* Body text */
--font-small: 400 13px/1.5 'Inter';      /* Labels, captions */
--font-micro: 500 11px/1.4 'Inter';      /* Tags, badges */
```

#### Spacing Scale

- **Base unit:** 4px
- **Scale:** 4, 8, 12, 16, 20, 24, 32, 40, 48, 64, 80px

#### Border Radius

- **Small:** 6px (tags, badges, small inputs)
- **Medium:** 10px (buttons, inputs, cards)
- **Large:** 16px (cards, modals)
- **X-Large:** 24px (large modals)

#### Shadows

```css
/* Elevation system for depth perception */
--shadow-sm: 0 1px 2px rgba(0, 0, 0, 0.3);
--shadow-md: 0 4px 6px rgba(0, 0, 0, 0.4);
--shadow-lg: 0 10px 15px rgba(0, 0, 0, 0.5);
--shadow-xl: 0 20px 25px rgba(0, 0, 0, 0.6);
--shadow-glow: 0 0 20px rgba(59, 130, 246, 0.3);
```

---

## Information Architecture

### Site Map

```
/admin (Admin Login)
├── /admin/dashboard (Analytics Dashboard)
├── /admin/users (User Management)
│   ├── /admin/users/:id (User Detail)
├── /admin/pricing (Pricing Management)
│   ├── /admin/pricing/new (Create Tier)
│   └── /admin/pricing/:id/edit (Edit Tier)
├── /admin/sources (Data Source Management)
├── /admin/scoring (Scoring Criteria)
├── /admin/scans (Scan Settings)
│   └── /admin/scans/:id (Scan Detail)
└── /admin/emails (Email Settings)
```

### Navigation Structure

**Primary Navigation** (Sidebar, always visible on desktop):

| Icon | Label | Route | Description |
|------|-------|-------|-------------|
| 📊 | Dashboard | `/admin/dashboard` | Analytics overview |
| 👥 | Users | `/admin/users` | User management |
| 💳 | Pricing | `/admin/pricing` | Subscription tiers |
| 🔌 | Sources | `/admin/sources` | Data source config |
| 🎯 | Scoring | `/admin/scoring` | Algorithm weights |
| 🔄 | Scans | `/admin/scans` | Scan management |
| ✉️ | Emails | `/admin/emails` | Email templates |

**Secondary Navigation** (Breadcrumbs + Tabs):

- Breadcrumbs show navigation path: `Dashboard > Users > user@example.com`
- Tabs for sub-sections: `[Overview | Subscription | Activity | Settings]`

---

## Page-by-Page Specifications

### 1. Admin Login Page

**Route:** `/admin`
**Layout:** Centered card, no navigation

```
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│                      [LOGO - 48px]                          │
│                  Opportunity Finder                         │
│                      Admin Panel                            │
│                                                             │
│   ┌─────────────────────────────────────────────────────┐   │
│   │                                                     │   │
│   │   Email                                            │   │
│   │   ┌─────────────────────────────────────────────┐   │   │
│   │   │ admin@opportunityfinder.com                 │   │   │
│   │   └─────────────────────────────────────────────┘   │   │
│   │                                                     │   │
│   │   Password                                    👁️   │   │
│   │   ┌─────────────────────────────────────────────┐   │   │
│   │   │ ••••••••••••••                               │   │   │
│   │   └─────────────────────────────────────────────┘   │   │
│   │                                                     │   │
│   │   ┌─────────────────────────────────────────────┐   │   │
│   │   │         Sign In to Admin Panel              │   │   │
│   │   └─────────────────────────────────────────────┘   │   │
│   │                                                     │   │
│   └─────────────────────────────────────────────────────┘   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

**Specifications:**

| Element | Specification |
|---------|---------------|
| **Card width** | 440px max, 100% on mobile |
| **Logo** | 48px gradient square with Target icon |
| **Title** | "Opportunity Finder" - 28px, gradient text |
| **Subtitle** | "Admin Panel" - 14px, slate-400 |
| **Input fields** | 48px height, full width, slate-700 bg |
| **Button** | Primary blue, full width, 48px height |
| **Error display** | Red banner above card for auth errors |
| **Loading state** | Spinner inside button, text changes to "Signing in..." |

**Validation:**
- Email: Required, valid email format
- Password: Required, min 8 characters
- Show inline validation errors on blur

---

### 2. Analytics Dashboard

**Route:** `/admin/dashboard`
**Layout:** Sidebar navigation + main content area

```
┌──────┬─────────────────────────────────────────────────────────┐
│      │ Header: Opportunity Finder Admin                        │
│      │ ───────────────────────────────────────────────────────│
│ SIDEBAR                                                      │
│                                                              │
│ 📊    │ ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐      │
│Dash   │ │ Total   │ │ Active  │ │  MRR    │ │ Churn   │      │
│       │ │ Users   │ │ Subs    │ │         │ │ Rate    │      │
│       │ │  1,247  │ │   834   │ │ $12.4k  │ │  2.3%   │      │
│ 👥    │ └─────────┘ └─────────┘ └─────────┘ └─────────┘      │
│Users  │                                                      │
│       │ ┌─────────────────────────────────────────────────┐   │
│ 💳    │ │         Revenue Trend (Last 30 Days)            │   │
│Pricing │ │  █                                            │   │
│       │ │  ██                               ████         │   │
│ 🔌    │ │  ████                            █████        │   │
│Sources │ │ █████                           ████████      │   │
│       │ │ ███████ ▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁  │   │
│ 🎯    │ └─────────────────────────────────────────────────┘   │
│Scoring│                                                      │
│       │ ┌─────────────────────────────────────────────────┐   │
│ 🔄    │ │         User Growth by Tier                     │   │
│Scans  │ │  ██████ Free  (847)                            │   │
│       │ │ │  ██████ Pro   (312)                         │   │
│ ✉️    │ │ │  ██████ Biz   (88)                          │   │
│Emails │ └─────────────────────────────────────────────────┘   │
│       │                                                      │
│       │ ┌─────────────────────────────────────────────────┐   │
│       │ │         Recent Activity                         │   │
│       │ │ • New user: john@email.com (2 min ago)         │   │
│       │ │ • Upgrade: jane@co.biz → Biz (15 min ago)      │   │
│       │ │ • Scan completed: 23 opportunities (1 hr ago)   │   │
│       │ └─────────────────────────────────────────────────┘   │
└──────┴─────────────────────────────────────────────────────────┘
```

**Specifications:**

| Component | Description |
|-----------|-------------|
| **Sidebar** | 240px wide, collapsible to icons on tablet |
| **Metric Cards** | 4 columns, gradient icons, large numbers |
| **Revenue Chart** | Line chart, 30-day window, hover tooltips |
| **Growth Chart** | Horizontal bar chart, tier breakdown |
| **Activity Feed** | Last 10 items, timestamped, clickable |

**Data Cards Layout:**

```html
<div class="admin-metric-card">
  <div class="metric-header">
    <span class="metric-label">Total Users</span>
    <div class="metric-icon blue">👥</div>
  </div>
  <div class="metric-value">1,247</div>
  <div class="metric-change positive">+12.5% this month</div>
</div>
```

**Chart Specifications:**

- **Revenue Trend:** Line chart, daily granularity, smooth curve
- **User Growth:** Stacked horizontal bar (Free/Pro/Biz)
- **Opportunity Discovery:** Donut chart (by source type)

---

### 3. User Management

**Route:** `/admin/users`
**Layout:** Sidebar + filter bar + data table

```
┌──────┬─────────────────────────────────────────────────────────┐
│      │ Header: User Management                    [Add User]  │
│      │ ───────────────────────────────────────────────────────│
│ SIDEBAR│                                                      │
│      │ ┌─────────────────────────────────────────────────┐   │
│ 📊    │ │ Search users...                        [Search] │   │
│Dash   │ └─────────────────────────────────────────────────┘   │
│      │                                                      │
│ 👥    │ Filter: [All ▼] [All Tiers ▼] [All Status ▼]       │
│Users  │                                                      │
│       │ ┌─────────────────────────────────────────────────┐   │
│ 💳    │ │ User            │ Tier  │ Status    │ Actions  │   │
│Pricing │ │─────────────────────────────────────────────────│   │
│       │ │ alice@email.com │ Free  │ Active    │ [View]   │   │
│ 🔌    │ │ bob@corp.io     │ Biz   │ Active    │ [View]   │   │
│Sources │ │ carl@start.up   │ Pro   │ Past Due  │ [View]   │   │
│       │ │ diana@co.net    │ Free  │ Cancelled│ [View]   │   │
│ 🎯    │ │ ...                                          │   │
│Scoring │ └─────────────────────────────────────────────────┘   │
│      │                                                      │
│ 🔄    │              ← 1 2 3 4 5 →                          │
│Scans  │           Showing 1-25 of 1,247                     │
│      │                                                      │
│ ✉️    │                                                      │
│Emails │                                                      │
└──────┴─────────────────────────────────────────────────────────┘
```

**Table Specifications:**

| Column | Width | Description |
|--------|-------|-------------|
| **User** | 30% | Email + avatar (clickable to detail) |
| **Tier** | 15% | Badge: Free (gray), Pro (blue), Biz (purple) |
| **Status** | 15% | Badge: Active (green), Past Due (orange), Cancelled (red) |
| **Created** | 15% | Date: "Jan 15, 2026" |
| **Actions** | 25% | [View] [Edit] [Cancel] [Make Admin] |

**Sort/Filter Options:**
- **Search:** Email, name
- **Sort:** Created date, tier, status, email
- **Filter by:** Tier, subscription status, email verified

**Bulk Actions Toolbar** (appears when rows selected):

```
☑ 3 selected → [Cancel Subscription] [Change Tier] [Export CSV] [Clear Selection]
```

---

### 4. User Detail Page

**Route:** `/admin/users/:id`
**Layout:** Sidebar + tabs + detail sections

```
┌──────┬─────────────────────────────────────────────────────────┐
│      │ ← Back to Users                    alice@email.com     │
│      │ ───────────────────────────────────────────────────────│
│ SIDEBAR│                                                      │
│      │ ┌─────────────────────────────────────────────────┐   │
│ 📊    │ │ [Overview] [Subscription] [Activity] [Settings] │   │
│Dash   │ └─────────────────────────────────────────────────┘   │
│      │                                                      │
│ 👥    │ ┌─────────────────────┐ ┌─────────────────────────┐   │
│Users  │ │                     │ │  Subscription           │   │
│       │ │   [Avatar: 80px]    │ │  ─────────────          │   │
│ 💳    │ │                     │ │  Plan: Pro              │   │
│Pricing │ │  alice@email.com    │ │  Status: Active         │   │
│       │ │                     │ │  Since: Jan 10, 2026    │   │
│ 🔌    │ │  Joined: Jan 5, 2026│ │  Renews: Feb 10, 2026   │   │
│Sources │ │  Role: User         │ │  MRR: $29.00            │   │
│       │ │                     │ │  [Cancel] [Change Plan] │   │
│ 🎯    │ │  [Edit Profile]     │ │                         │   │
│Scoring │ │  [Reset Password]   │ │                         │   │
│       │ │  [Make Admin]       │ │                         │   │
│ 🔄    │ │                     │ │                         │   │
│Scans  │ └─────────────────────┘ └─────────────────────────┘   │
│      │                                                      │
│ ✉️    │ ┌─────────────────────────────────────────────────┐   │
│Emails │ │ Recent Activity                                  │   │
│       │ │ • Jan 17: Logged in                              │   │
│       │ │ • Jan 16: Viewed "Testimonial Tool" opportunity │   │
│       │ │ • Jan 15: Saved 3 opportunities                 │   │
│       │ └─────────────────────────────────────────────────┘   │
└──────┴─────────────────────────────────────────────────────────┘
```

**Tab Contents:**

| Tab | Content |
|-----|---------|
| **Overview** | User info card + subscription card + quick actions |
| **Subscription** | Full billing history, invoices, payment methods |
| **Activity** | Timeline of user actions (login, views, saves, scans) |
| **Settings** | Email verified flag, password reset, admin role toggle, delete account |

---

### 5. Pricing Management

**Route:** `/admin/pricing`
**Layout:** Sidebar + card grid

```
┌──────┬─────────────────────────────────────────────────────────┐
│      │ Header: Pricing Tiers                   [+ New Tier]   │
│      │ ───────────────────────────────────────────────────────│
│ SIDEBAR│                                                      │
│      │ ┌─────────────────────────────────────────────────┐   │
│ 📊    │ │  FREE                                          │   │
│Dash   │ │  ─────────────────────────────────────────────  │   │
│       │ │  Price: $0/month                               │   │
│ 👥    │ │  Features:                                     │   │
│Users  │ │  ✓ 5 opportunities/month                       │   │
│       │ │  ✓ Basic scoring                              │   │
│ 💳    │ │  ✓ 1 data source                              │   │
│Pricing │ │  Status: [Enabled ●]                          │   │
│       │ │  [Edit] [Disable]                              │   │
│ 🔌    │ └─────────────────────────────────────────────────┘   │
│Sources│                                                      │
│       │ ┌─────────────────────────────────────────────────┐   │
│ 🎯    │ │  PRO                                           │   │
│Scoring │ │  ─────────────────────────────────────────────  │   │
│       │ │  Price: $29/month                              │   │
│ 🔄    │ │  Features:                                     │   │
│Scans  │ │  ✓ 50 opportunities/month                      │   │
│       │ │  ✓ Advanced scoring                           │   │
│ ✉️    │ │  ✓ All data sources                           │   │
│Emails │ │  ✓ Export opportunities                        │   │
│       │ │  Status: [Enabled ●]                          │   │
│       │ │  [Edit] [Disable]                              │   │
│       │ └─────────────────────────────────────────────────┘   │
│       │                                                      │
│       │ ┌─────────────────────────────────────────────────┐   │
│       │ │  BUSINESS                                      │   │
│       │ │  ─────────────────────────────────────────────  │   │
│       │ │  Price: $99/month                              │   │
│       │ │  Features:                                     │   │
│       │ │  ✓ Unlimited opportunities                     │   │
│       │ │  ✓ Custom scoring weights                     │   │
│       │ │  ✓ API access                                 │   │
│       │  ✓ Priority support                             │   │
│       │  Status: [Enabled ●]                            │   │
│       │  [Edit] [Disable]                               │   │
│       │ └─────────────────────────────────────────────────┘   │
└──────┴─────────────────────────────────────────────────────────┘
```

**Tier Card Specifications:**

```html
<div class="tier-card">
  <div class="tier-header">
    <h3 class="tier-name">PRO</h3>
    <span class="tier-status enabled">● Enabled</span>
  </div>
  <div class="tier-price">$29<span class="period">/month</span></div>
  <ul class="tier-features">
    <li>✓ 50 opportunities/month</li>
    <li>✓ Advanced scoring</li>
    <li>✓ All data sources</li>
    <li>✓ Export opportunities</li>
  </ul>
  <div class="tier-actions">
    <button>Edit Tier</button>
    <button class="destructive">Disable</button>
  </div>
</div>
```

**Create/Edit Tier Modal:**

```
┌────────────────────────────────────────────────────────┐
│ Edit Pricing Tier                                  [×] │
│ ───────────────────────────────────────────────────── │
│                                                        │
│ Tier Name                                              │
│ ┌──────────────────────────────────────────────────┐  │
│ │ Pro                                              │  │
│ └──────────────────────────────────────────────────┘  │
│                                                        │
│ Price ($/month)            Interval                   │
│ ┌──────────────┐          ┌──────────────┐           │
│ │ 29           │          │ Monthly  ▼   │           │
│ └──────────────┘          └──────────────┘           │
│                                                        │
│ Features (JSON)                                        │
│ ┌──────────────────────────────────────────────────┐  │
│ │ {                                                │  │
│ │   "opportunities": 50,                           │  │
│ │   "scoring": "advanced",                         │  │
│ │   "sources": "all",                              │  │
│ │   "export": true                                 │  │
│ │ }                                                │  │
│ └──────────────────────────────────────────────────┘  │
│                                                        │
│ Sources Allowed │ 5      Scans/Month │ 4              │
│ Export Limit    │ 100    Enabled       ● [Toggle]     │
│                                                        │
│ ┌────────────────────┐  ┌────────────────────┐        │
│ │ Cancel             │  │ Save Changes       │        │
│ └────────────────────┘  └────────────────────┘        │
└────────────────────────────────────────────────────────┘
```

---

### 6. Data Source Management

**Route:** `/admin/sources`
**Layout:** Sidebar + source cards

```
┌──────┬─────────────────────────────────────────────────────────┐
│      │ Header: Data Sources                                    │
│      │ ───────────────────────────────────────────────────────│
│ SIDEBAR│                                                      │
│      │ ┌─────────────────────────────────────────────────┐   │
│ 📊    │ │  🔌 REDDIT                                      │   │
│Dash   │ │  ─────────────────────────────────────────────  │   │
│       │ │  Status: [Connected ●]                          │   │
│ 👥    │ │  Client ID: •••••••••••                         │   │
│Users  │ │  Subreddits: r/Entrepreneur, r/SaaS, r/IndieHackers│
│       │ │  Last Sync: 2 hours ago                         │   │
│ 💳    │ │  [Configure] [Test Connection] [Disable]        │   │
│Pricing │ └─────────────────────────────────────────────────┘   │
│       │                                                      │
│ 🔌    │ ┌─────────────────────────────────────────────────┐   │
│Sources│ │  🔍 SERPAPI                                     │   │
│       │ │  ─────────────────────────────────────────────  │   │
│ 🎯    │ │  Status: [Connected ●]                          │   │
│Scoring │ │  API Key: •••••••••••••••                      │   │
│       │ │  Queries: 1,234 / 10,000 (12%)                  │   │
│ 🔄    │ │  Last Sync: 15 minutes ago                      │   │
│Scans  │ │  [Configure] [Test Connection] [Disable]        │   │
│       │ └─────────────────────────────────────────────────┘   │
│ ✉️    │                                                      │
│Emails │ ┌─────────────────────────────────────────────────┐   │
│       │ │  🛒 PRODUCT HUNT                                │   │
│       │ │  ─────────────────────────────────────────────  │   │
│       │ │  Status: [Not Configured ○]                     │   │
│       │ │  Token: Not set                                 │   │
│       │ │  [Configure]                                   │   │
│       │ └─────────────────────────────────────────────────┘   │
└──────┴─────────────────────────────────────────────────────────┘
```

**Source Card Specifications:**

| Element | Description |
|---------|-------------|
| **Icon** | Source-specific (48px) |
| **Name** | 20px, bold, uppercase |
| **Status** | Badge: Connected (green), Not Configured (gray), Error (red) |
| **Credentials** | Masked display, show/hide toggle |
| **Usage** | Progress bar for API quotas |
| **Last Sync** | Relative timestamp |
| **Actions** | Configure, Test, Enable/Disable |

**Configure Source Modal (Reddit example):**

```
┌────────────────────────────────────────────────────────┐
│ Configure Reddit                                 [×] │
│ ───────────────────────────────────────────────────── │
│                                                        │
│ Client ID                                             │
│ ┌──────────────────────────────────────────────────┐  │
│ │ pL8x...                                          │  │
│ └──────────────────────────────────────────────────┘  │
│                                                        │
│ Client Secret                [Show/Hide]              │
│ ┌──────────────────────────────────────────────────┐  │
│ │ •••••••••••••••••                                │  │
│ └──────────────────────────────────────────────────┘  │
│                                                        │
│ User Agent                                            │
│ ┌──────────────────────────────────────────────────┐  │
│ │ OpportunityFinder/1.0                            │  │
│ └──────────────────────────────────────────────────┘  │
│                                                        │
│ Subreddits (comma-separated)                          │
│ ┌──────────────────────────────────────────────────┐  │
│ │ Entrepreneur, SaaS, IndieHackers, smallbusiness  │  │
│ └──────────────────────────────────────────────────┘  │
│                                                        │
│ ┌────────────────────┐  ┌──────────────────────┐       │
│ │ Cancel             │  │ Test & Save          │       │
│ └────────────────────┘  └──────────────────────┘       │
│                                                        │
│ ✓ Connection successful!                              │
└────────────────────────────────────────────────────────┘
```

---

### 7. Scoring & Configuration System

**Route:** `/admin/scoring`
**Layout:** Sidebar + tabbed interface (12 tabs)

**Reference:** See [Configuration_Reference.md](./Configuration_Reference.md) for complete parameter documentation.

**Tab Navigation:**

| Tab | Parameters | Category |
|-----|-------------|----------|
| **Data Sources** | 7 parameters | Core |
| **Weights** | 8 parameters | Core |
| **Thresholds** | 8 parameters | Core |
| **Validation** | 5 parameters | Core |
| **Business Type** | 5 parameters | Filtering |
| **Categories** | 3 parameters | Filtering |
| **Maturity** | 4 parameters | Filtering |
| **AI Models** | 6 parameters | AI |
| **Geography** | 4 parameters | Filtering |
| **Sources** | 26+ parameters | Source-specific |
| **Output** | 7 parameters | Display |
| **Alerts** | 6 parameters | Alerting |

**Total Configurable Parameters:** 90+

---

#### 7.1 Tab 1: Data Sources

| Parameter | Type | Default | Control |
|-----------|------|---------|---------|
| `enabled_sources` | Multi-select | [All 6 MVP sources] | Checkbox list |
| `source_priority` | Drag-to-reorder | Custom list | Draggable list |
| `min_confidence_score` | Slider | 30 | 0-100 slider |
| `max_content_age_days` | Number input | 365 | 1-3650 input |
| `require_correlation` | Toggle | On | On/Off switch |
| `min_correlation_sources` | Number input | 2 | 2-10 input |
| `min_correlation_strength` | Slider | 0.3 | 0.0-1.0 slider |

**UI Layout:**
```
┌─────────────────────────────────────────────────────────┐
│ ENABLED DATA SOURCES                                    │
│ ─────────────────────────────────────────────────────── │
│                                                         │
│ Select which sources to scan for opportunities:         │
│                                                         │
│ ☑ Reddit (Discussion)                                   │
│ ☑ Indie Hackers (Discussion)                            │
│ ☑ Product Hunt (Launch)                                 │
│ ☑ Hacker News (Discussion/QA)                           │
│ ☑ Google Trends (Search)                [Phase 2 Badge] │
│ ☑ Competition Search (Search)                           │
│ ☐ YouTube (Search)                     [Phase 2 Badge] │
│ ☐ Gumroad/Etsy (Marketplace)              [Phase 2]     │
│ ☐ Stack Overflow (Q&A)                    [Phase 2]     │
│ ☐ Zapier/Make/n8n (Integration)           [Phase 2]     │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

---

#### 7.2 Tab 2: Scoring Weights

| Parameter | Type | Default | Range |
|-----------|------|---------|-------|
| `weight_demand` | Slider | 0.25 | 0.0-1.0 |
| `weight_competition` | Slider | 0.20 | 0.0-1.0 |
| `weight_revenue` | Slider | 0.25 | 0.0-1.0 |
| `weight_feasibility` | Slider | 0.15 | 0.0-1.0 |
| `weight_timing` | Slider | 0.10 | 0.0-1.0 |
| `weight_risk` | Slider | 0.05 | 0.0-1.0 |
| `weight_demand_velocity` | Slider | 0.60 | 0.0-1.0 |
| `weight_demand_volume` | Slider | 0.40 | 0.0-1.0 |

**Validation Display:**
- Live calculation shows current sum (target: 100%)
- Error indicator if sum ≠ 100%
- Warning message: "Weights must sum to 100%. Current: {sum}%"

---

#### 7.3 Tab 3: Scoring Thresholds

| Parameter | Type | Default | Range |
|-----------|------|---------|-------|
| `score_threshold_build` | Slider | 75 | 0-100 |
| `score_threshold_validate` | Slider | 60 | 0-100 |
| `score_threshold_investigate` | Slider | 40 | 0-100 |
| `velocity_threshold_high` | Slider | 3.0 | 1.0-10.0 |
| `velocity_threshold_medium` | Slider | 1.5 | 1.0-10.0 |
| `volume_threshold_high` | Slider | 5000 | 0-100000 |
| `volume_threshold_medium` | Slider | 1000 | 0-100000 |
| `engagement_threshold` | Slider | 30 | 0-100 |

**Validation:** Must be ordered: build > validate > investigate

---

#### 7.4 Tab 4: Validation Rules

| Parameter | Type | Default | Control |
|-----------|------|---------|---------|
| `require_paid_solution` | Toggle | On | On/Off |
| `min_competitor_mrr` | Number input | 1000 | 0-100000 |
| `max_competitor_count` | Number input | 50 | 0-1000 |
| `min_engagement_score` | Slider | 30 | 0-100 |
| `max_content_age_days` | Number input | 365 | 1-3650 |

---

#### 7.5 Tab 5: Business Type Filters

| Parameter | Type | Default | Control |
|-----------|------|---------|---------|
| `business_type_filter` | Dropdown | "all" | Select |
| `b2b_boost_multiplier` | Slider | 1.5 | 1.0-5.0 |
| `b2c_penalty_multiplier` | Slider | 0.8 | 0.0-1.0 |
| `b2b_confidence_threshold` | Slider | 60 | 0-100 |
| `b2c_confidence_threshold` | Slider | 60 | 0-100 |

---

#### 7.6 Tab 6: Category Classifications

| Parameter | Type | Default | Control |
|-----------|------|---------|---------|
| `category_filter` | Multi-select | ["all"] | Checkbox list |
| `category_boosts` | Dict editor | {} | Key-value pairs |
| `category_blacklist` | Multi-select | [] | Checkbox list |

---

#### 7.7 Tab 7: Maturity Level Preferences

| Parameter | Type | Default | Range |
|-----------|------|---------|-------|
| `maturity_filter` | Dropdown | "emerging_only" | Select |
| `emerging_definition_max_competitors` | Number | 10 | 0-100 |
| `emerging_definition_max_age_months` | Number | 12 | 0-120 |
| `growing_definition_min_velocity` | Slider | 1.5 | 1.0-5.0 |
| `established_definition_min_competitors` | Number | 50 | 10-1000 |

---

#### 7.8 Tab 8: AI Model Configuration

| Parameter | Type | Default | Control |
|-----------|------|---------|---------|
| `analysis_model` | Dropdown | "gpt-4o" | Select |
| `classification_model` | Dropdown | "gpt-4o-mini" | Select |
| `temperature` | Slider | 0.3 | 0.0-2.0 |
| `max_tokens` | Number input | 2000 | 100-8000 |
| `enable_model_fallback` | Toggle | On | On/Off |
| `fallback_model` | Dropdown | "claude-3.5-sonnet" | Select |

---

#### 7.9 Tab 9: Geographic Configuration

| Parameter | Type | Default | Control |
|-----------|------|---------|---------|
| `target_geographies` | Multi-select | ["US", "UK", "CA", "AU"] | Checkbox list |
| `geography_filter_mode` | Dropdown | "any" | Select |
| `language_filter` | Multi-select | ["en"] | Checkbox list |
| `exclude_geographies` | Multi-select | [] | Checkbox list |

---

#### 7.10 Tab 10: Source-Specific Parameters

Dynamically shows source-specific parameters for each enabled source.

**Reddit:** `reddit_subreddits`, `reddit_min_upvotes`, `reddit_min_comments`, `reddit_post_age_max_days`, `reddit_sort_by`

**Google Trends:** `trends_timeframe`, `trends_category`, `trends_geocode`, `trends_property`

**Product Hunt:** `product_hunt_min_upvotes`, `product_hunt_min_comments`, `product_hunt_launch_age_max_days`

**And more...** (See Configuration_Reference.md)

---

#### 7.11 Tab 11: Output Configuration

| Parameter | Type | Default | Control |
|-----------|------|---------|---------|
| `max_results_returned` | Number input | 50 | 1-1000 |
| `sort_by` | Dropdown | "score" | Select |
| `sort_order` | Dropdown | "desc" | Select |
| `include_score_breakdown` | Toggle | Off | On/Off (admin-only) |
| `include_source_links` | Toggle | On | On/Off |
| `include_raw_data` | Toggle | Off | On/Off (debug) |
| `include_ai_reasoning` | Toggle | Off | On/Off (debug) |

---

#### 7.12 Tab 12: Notification & Alerting

| Parameter | Type | Default | Control |
|-----------|------|---------|---------|
| `alert_on_new_opportunities` | Toggle | On | On/Off |
| `alert_threshold` | Slider | 70 | 0-100 |
| `alert_velocity_spike` | Toggle | On | On/Off |
| `alert_velocity_spike_threshold` | Slider | 3.0 | 1.5-10.0 |
| `alert_digest_frequency` | Dropdown | "daily" | Select |
| `alert_channels` | Multi-select | ["dashboard"] | Checkbox list |

---

### 8. Additional Admin Pages

---

#### 8.1 Presets Manager

**Route:** `/admin/presets`

**Purpose:** Quickly switch between pre-configured parameter sets.

**Available Presets:**

| Preset | Focus | Key Settings |
|--------|-------|--------------|
| **Aggressive Emerging Hunter** | Early mover, high growth | Emerging only, demand 35%, no correlation |
| **Validated B2B SaaS Finder** | Proven demand, B2B | B2B only, revenue 40%, min $5k MRR |
| **Janky Solution Upgrader** | Paying for primitive solutions | Marketplace focus, revenue 50% |

---

#### 8.2 Correlation Engine Monitor

**Route:** `/admin/correlation`

**Purpose:** View opportunities mentioned across multiple sources.

**Columns:** Opportunity name, final score, source count, correlation strength, entities, time cluster.

---

#### 8.3 Source Health Monitor

**Route:** `/admin/source-health`

**Purpose:** Real-time monitoring of data source health.

**Metrics per source:** Status, last sync, rate limit, response time, error rate.

---

#### 8.4 Configuration History

**Route:** `/admin/config-history`

**Purpose:** Track all configuration changes with audit trail.

**Columns:** Timestamp, changed by, action, changes, reason.

**Actions:** Rollback, compare, export.

---

### 9. Validation Error Display

Real-time validation feedback for all configuration parameters.

**Error Banner:**
```
⚠️ Configuration Validation Errors
❌ Weight sum must equal 100% (current: 95%)
❌ Thresholds must be ordered: Build (60) > Validate (60)
[Auto-Fix All]
```

**Inline Validation:**
```
Weight Demand: [━━━━━━━━━━●━━━━] 25%
                              ┃
                              ┗─ ❌ Must be ≤ 50% when using "Aggressive" preset
```

---


### 10. Scan Settings (Note: Formerly Section 8)
**Route:** `/admin/scans`
**Layout:** Sidebar + schedule cards + scan history

*(Note: Formerly Section 8. Renumbered to accommodate new Section 8 "Additional Admin Pages")*

```
┌──────┬─────────────────────────────────────────────────────────┐
│      │ Header: Scan Settings                    [Run Scan Now] │
│      │ ───────────────────────────────────────────────────────│
│ SIDEBAR│                                                      │
│      │ ┌─────────────────────────────────────────────────┐   │
│ 📊    │ │  SCHEDULE                                       │   │
│Dash   │ │  ─────────────────────────────────────────────  │   │
│       │ │  Frequency: [Daily ▼]                           │   │
│ 👥    │ │  Time: 02:00 UTC [Clock icon]                   │   │
│Users  │ │  Next scan: Jan 18, 2026 at 02:00 UTC           │   │
│       │ │  Sources: [✓ Reddit] [✓ SerpAPI] [✓ ProductHunt]│   │
│ 💳    │ │  [Update Schedule]                               │   │
│Pricing │ └─────────────────────────────────────────────────┘   │
│       │                                                      │
│ 🔌    │ ┌─────────────────────────────────────────────────┐   │
│Sources│ │  SCAN HISTORY                                   │   │
│       │ │  ─────────────────────────────────────────────  │   │
│ 🎯    │ │  ┌─────────────────────────────────────────┐    │   │
│Scoring │ │  │ Scan #47  │ 23 opportunities │ Success  │    │   │
│       │ │  │ Started:  Jan 17, 02:00              │    │   │
│       │ │  │ Completed: Jan 17, 02:14             │    │   │
│       │ │  │ Duration: 14 min                     │    │   │
│       │ │  │ Sources: 3/3                        │    │   │
│       │ │  │ [View Details]                     │    │   │
│       │ │  └─────────────────────────────────────────┘    │   │
│ 🔄    │ │                                                  │   │
│Scans  │ │  ┌─────────────────────────────────────────┐    │   │
│       │ │  │ Scan #46  │ 31 opportunities │ Success  │    │   │
│       │ │  │ Started:  Jan 16, 02:00              │    │   │
│ ✉️    │ │  │ Completed: Jan 16, 02:18             │    │   │
│Emails │ │  │ Duration: 18 min                     │    │   │
│       │ │  │ Sources: 3/3                        │    │   │
│       │ │  │ [View Details]                     │    │   │
│       │ │  └─────────────────────────────────────────┘    │   │
│       │ │                                                  │   │
│       │ │  ┌─────────────────────────────────────────┐    │   │
│       │ │  │ Scan #45  │ 18 opportunities │ Success  │    │   │
│       │ │  │ Started:  Jan 15, 02:00              │    │   │
│       │ │  │ Completed: Jan 15, 02:12             │    │   │
│       │ │  │ Duration: 12 min                     │    │   │
│       │ │  │ Sources: 3/3                        │    │   │
│       │ │  │ [View Details]                     │    │   │
│       │ │  └─────────────────────────────────────────┘    │   │
│       │                                                    │   │
│       │                    [View All History]              │   │
└──────┴─────────────────────────────────────────────────────────┘
```

**Scan Detail Modal:**

```
┌────────────────────────────────────────────────────────┐
│ Scan #47 Details                                 [×] │
│ ───────────────────────────────────────────────────── │
│                                                        │
│ Status: ✓ Success                                      │
│ Started: Jan 17, 2026 at 02:00 UTC                    │
│ Completed: Jan 17, 2026 at 02:14 UTC                  │
│ Duration: 14 minutes 23 seconds                        │
│                                                        │
│ ┌─────────────────────────────────────────────────┐   │
│ │ RESULTS                                         │   │
│ │ ─────────────────────────────────────────────   │   │
│ │                                                 │   │
│ │  Total opportunities found: 23                  │   │
│ │  New opportunities: 8                          │   │
│ │  Updated opportunities: 15                     │   │
│ │  High score (70+): 12                          │   │
│ │                                                 │   │
│ └─────────────────────────────────────────────────┘   │
│                                                        │
│ ┌─────────────────────────────────────────────────┐   │
│ │ SOURCES PROCESSED                               │   │
│ │ ─────────────────────────────────────────────   │   │
│ │                                                 │   │
│ │  Reddit      ✓ 12 opportunities (52%)          │   │
│ │  SerpAPI     ✓ 8 opportunities (35%)           │   │
│ │  ProductHunt ✓ 3 opportunities (13%)           │   │
│ │                                                 │   │
│ └─────────────────────────────────────────────────┘   │
│                                                        │
│ ┌─────────────────────────────────────────────────┐   │
│ │ ERRORS (if any)                                 │   │
│ │ ─────────────────────────────────────────────   │   │
│ │                                                 │   │
│ │  ProductHunt: Rate limit reached, used cache   │   │
│ │                                                 │   │
│ └─────────────────────────────────────────────────┘   │
│                                                        │
│                          [Close]  [Re-run Scan]        │
└────────────────────────────────────────────────────────┘
```

---

### 11. Email Settings

**Route:** `/admin/emails`
**Layout:** Sidebar + template list + editor

```
┌──────┬─────────────────────────────────────────────────────────┐
│      │ Header: Email Templates                    [+ Template] │
│      │ ───────────────────────────────────────────────────────│
│ SIDEBAR│                                                      │
│      │ ┌─────────────────────────────────────────────────┐   │
│ 📊    │ │  OPPORTUNITY DIGEST                             │   │
│Dash   │ │  ─────────────────────────────────────────────  │   │
│       │ │  Subject: "{{count}} new opportunities found"  │   │
│ 👥    │ │  Triggers: After scan, 5+ opportunities         │   │
│Users  │ │  Frequency per tier:                            │   │
│       │ │    • Free: Never                                │   │
│ 💳    │ │    • Pro: Weekly                                │   │
│Pricing │ │    • Business: Daily                            │   │
│       │ │  Status: [Enabled ●]                            │   │
│ 🔌    │ │  [Edit] [Preview] [Disable]                     │   │
│Sources │ └─────────────────────────────────────────────────┘   │
│       │                                                      │
│ 🎯    │ ┌─────────────────────────────────────────────────┐   │
│Scoring │ │  WELCOME EMAIL                                  │   │
│       │ │  ─────────────────────────────────────────────  │   │
│ 🔄    │ │  Subject: "Welcome to Opportunity Finder"       │   │
│Scans  │ │  Triggers: New user registration                │   │
│       │ │  Frequency per tier: All tiers                   │   │
│ ✉️    │ │  Status: [Enabled ●]                            │   │
│Emails │ │  [Edit] [Preview] [Disable]                     │   │
│       │ └─────────────────────────────────────────────────┘   │
│       │                                                      │
│       │ ┌─────────────────────────────────────────────────┐   │
│       │ │  SUBSCRIPTION EXPIRED                           │   │
│       │ │  ─────────────────────────────────────────────  │   │
│       │ │  Subject: "Your subscription has expired"       │   │
│       │ │  Triggers: Subscription past due 7+ days        │   │
│       │ │  Frequency per tier: All tiers                  │   │
│       │ │  Status: [Enabled ●]                            │   │
│       │ │  [Edit] [Preview] [Disable]                     │   │
│       │ └─────────────────────────────────────────────────┘   │
│       │                                                      │
│       │ ┌─────────────────────────────────────────────────┐   │
│       │ │  HIGH SCORE ALERT                               │   │
│       │ │  ─────────────────────────────────────────────  │   │
│       │ │  Subject: "Hot opportunity: {{title}}"          │   │
│       │ │  Triggers: Scan finds opportunity with 80+ score│   │
│       │ │  Frequency per tier:                            │   │
│       │ │    • Free: Never                                │   │
│       │ │    • Pro: Weekly digest                         │   │
│       │ │    • Business: Immediate                        │   │
│       │ │  Status: [Enabled ●]                            │   │
│       │ │  [Edit] [Preview] [Disable]                     │   │
│       │ └─────────────────────────────────────────────────┘   │
└──────┴─────────────────────────────────────────────────────────┘
```

**Email Template Editor Modal:**

```
┌────────────────────────────────────────────────────────┐
│ Edit: Opportunity Digest                          [×] │
│ ───────────────────────────────────────────────────── │
│                                                        │
│ Template Name                                          │
│ ┌──────────────────────────────────────────────────┐  │
│ │ Opportunity Digest                               │  │
│ └──────────────────────────────────────────────────┘  │
│                                                        │
│ Subject Line                                           │
│ ┌──────────────────────────────────────────────────┐  │
│ │ {{count}} new opportunities found                │  │
│ └──────────────────────────────────────────────────┘  │
│                                                        │
│ Trigger Conditions                                     │
│ • Run after: Scan completes                           │
│ • Minimum opportunities: 5                            │
│                                                        │
│ Frequency by Tier                                      │
│ ┌──────────────┬──────────────┬──────────────┐        │
│ │ Free         │ Pro          │ Business     │        │
│ │ Never    ▼   │ Weekly    ▼  │ Daily     ▼  │        │
│ └──────────────┴──────────────┴──────────────┘        │
│                                                        │
│ Email Body (Handlebars)                                │
│ ┌──────────────────────────────────────────────────┐  │
│ │ <h1>Hi {{name}}!</h1>                            │  │
│ │                                                  │  │
│ │ <p>We found {{count}} new opportunities:</p>     │  │
│ │                                                  │  │
│ │ {{#each opportunities}}                          │  │
│ │   <div>                                         │  │
│ │     <h2>{{title}}</h2>                          │  │
│ │     <p>Score: {{score}}</p>                     │  │
│ │     <a href="{{url}}">View Details</a>          │  │
│ │   </div>                                        │  │
│ │ {{/each}}                                       │  │
│ │                                                  │  │
│ │ <p>Happy hunting!</p>                           │  │
│ └──────────────────────────────────────────────────┘  │
│                                                        │
│ Available Variables: {name, email, count, url,        │
│ opportunities: [{title, score, url, description}]}    │
│                                                        │
│ ┌────────────────────┐  ┌──────────────────────┐       │
│ │ Cancel             │  │ Save & Test Send     │       │
│ └────────────────────┘  └──────────────────────┘       │
└────────────────────────────────────────────────────────┘
```

---

## User Flows

### Flow 1: Cancel User Subscription

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│ User List   │────▶│ User Detail │────▶│ Confirm     │
│ /admin/users│     │ /admin/users│     │ Modal       │
│             │     │ /:id        │     │             │
└─────────────┘     └─────────────┘     └─────────────┘
                                               │
                                               ▼
                                        ┌─────────────┐
                                        │ Success     │
                                        │ Toast       │
                                        └─────────────┘
```

**Steps:**
1. Navigate to Users page
2. Search or browse to find target user
3. Click "View" on user row
4. On User Detail > Subscription tab, click "Cancel Subscription"
5. Modal appears: "Cancel subscription for alice@email.com?"
   - Shows current plan, renewal date
   - Radio buttons: Immediate cancellation vs. End of period
   - Reason dropdown (optional)
6. Click "Cancel Subscription"
7. Toast notification: "Subscription cancelled. Access until Feb 10, 2026"
8. User status updates to "Cancel" (orange badge)

---

### Flow 2: Create New Pricing Tier

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│ Pricing     │────▶│ New Tier    │────▶│ Create/Edit │
│ /admin/     │     │ Modal       │     │ Modal       │
│ pricing     │     │             │     │             │
└─────────────┘     └─────────────┘     └─────────────┘
                                               │
                      ┌────────────────────────┘
                      ▼
               ┌─────────────┐
               │ Success     │
               │ Toast       │
               └─────────────┘
```

**Steps:**
1. Navigate to Pricing page
2. Click "+ New Tier" button
3. Fill in form:
   - Tier name (required)
   - Price (required, number)
   - Interval (Monthly/Yearly dropdown)
   - Features (JSON editor)
   - Sources allowed (number input)
   - Scans per month (number input)
   - Export limit (number input)
   - Enabled (toggle)
4. Click "Create Tier"
5. Validation: Check for duplicate names, valid JSON
6. Toast notification: "Enterprise tier created"
7. New card appears in pricing grid

---

### Flow 3: Adjust Scoring Weights

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│ Scoring     │────▶│ Adjust      │────▶│ Preview     │
│ /admin/     │     │ Weights     │     │ Impact      │
│ scoring     │     │             │     │             │
└─────────────┘     └─────────────┘     └─────────────┘
                                               │
                      ┌────────────────────────┘
                      ▼
               ┌─────────────┐
               │ Save Changes│
               │ Toast       │
               └─────────────┘
```

**Steps:**
1. Navigate to Scoring page
2. Adjust sliders for each weight category
3. Real-time validation: Sum must equal 100%
4. Click "Preview Changes"
5. Modal shows:
   - 5 sample opportunities
   - Current score vs. New score
   - Impact: "3 opportunities would move from 'Maybe' to 'Good'"
6. Click "Save" or "Cancel"
7. Toast: "Scoring weights updated. Next scan will use new weights."

---

### Flow 4: Configure Data Source

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│ Data Sources│────▶│ Configure   │────▶│ Test        │
│ /admin/     │     │ Modal       │     │ Connection  │
│ sources     │     │             │     │             │
└─────────────┘     └─────────────┘     └─────────────┘
                                               │
                      ┌────────────────────────┘
                      ▼
               ┌─────────────┐
               │ Save Config │
               │ Toast       │
               └─────────────┘
```

**Steps:**
1. Navigate to Data Sources page
2. Find source card (e.g., ProductHunt shows "Not Configured")
3. Click "Configure"
4. Fill in credentials:
   - API Token (required)
   - Additional settings per source
5. Click "Test Connection"
6. Spinner shows for 2-3 seconds
7. Success message: "✓ Connection successful!"
8. Click "Save"
9. Toast: "ProductHunt configured successfully"
10. Card updates: Status → "Connected ●", Last Sync → "Just now"

---

### Flow 5: Grant Admin Access

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│ User List   │────▶│ User Detail │────▶│ Confirm     │
│ /admin/users│     │ /admin/users│     │ Modal       │
│             │     │ /:id        │     │             │
└─────────────┘     └─────────────┘     └─────────────┘
                                               │
                                               ▼
                                        ┌─────────────┐
                                        │ Success     │
                                        │ Toast       │
                                        └─────────────┘
```

**Steps:**
1. Navigate to Users page
2. Search for user by email
3. Click "View" on user row
4. On User Detail > Settings tab, find "Admin Role" section
5. Current status: "Role: User [Make Admin]"
6. Click "Make Admin"
7. Confirmation modal:
   - "Make bob@corp.io an administrator?"
   - Warning: "Admins have full access to all settings and user data"
   - Confirm email input: Type user email to confirm
8. Click "Grant Admin Access"
9. Toast: "Admin access granted to bob@corp.io"
10. User badge changes to purple "ADMIN" badge
11. User can now access /admin routes

---

## Component Library

### Button Variants

```typescript
// Primary Action (CTA)
<button className="px-6 py-3 bg-blue-600 hover:bg-blue-700 text-white font-semibold rounded-lg transition-colors">
  Save Changes
</button>

// Secondary Action
<button className="px-6 py-3 bg-slate-700 hover:bg-slate-600 text-slate-200 font-semibold rounded-lg border border-slate-600 transition-colors">
  Cancel
</button>

// Destructive Action
<button className="px-6 py-3 bg-red-600 hover:bg-red-700 text-white font-semibold rounded-lg transition-colors">
  Delete
</button>

// Icon Button
<button className="p-2 hover:bg-slate-700 text-slate-400 hover:text-white rounded-lg transition-colors">
  <Edit size={18} />
</button>
```

### Input Field

```typescript
<div className="flex flex-col gap-2">
  <label className="text-sm font-medium text-slate-300">Email Address</label>
  <input
    type="email"
    placeholder="user@example.com"
    className="px-4 py-3 bg-slate-900 border border-slate-700 rounded-lg text-white placeholder-slate-500 focus:outline-none focus:ring-2 focus:ring-blue-500 focus:border-transparent"
  />
  <span className="text-xs text-slate-500">Required. Must be unique.</span>
</div>
```

### Status Badge

```typescript
// Active/Success
<span className="inline-flex items-center gap-1.5 px-3 py-1.5 bg-emerald-500/10 border border-emerald-500 text-emerald-500 text-xs font-semibold rounded-full">
  <span className="w-1.5 h-1.5 bg-emerald-500 rounded-full animate-pulse"></span>
  Active
</span>

// Warning
<span className="inline-flex items-center gap-1.5 px-3 py-1.5 bg-amber-500/10 border border-amber-500 text-amber-500 text-xs font-semibold rounded-full">
  Past Due
</span>

// Error
<span className="inline-flex items-center gap-1.5 px-3 py-1.5 bg-red-500/10 border border-red-500 text-red-500 text-xs font-semibold rounded-full">
  Cancelled
</span>
```

### Data Table

```typescript
<table className="w-full">
  <thead>
    <tr className="border-b border-slate-700">
      <th className="text-left py-3 px-4 text-sm font-semibold text-slate-400">
        Email
      </th>
      <th className="text-left py-3 px-4 text-sm font-semibold text-slate-400">
        Tier
      </th>
      {/* ... more headers */}
    </tr>
  </thead>
  <tbody>
    {users.map(user => (
      <tr key={user.id} className="border-b border-slate-700/50 hover:bg-slate-800/30 transition-colors">
        <td className="py-3 px-4 text-slate-200">{user.email}</td>
        <td className="py-3 px-4">{getTierBadge(user.tier)}</td>
        {/* ... more cells */}
      </tr>
    ))}
  </tbody>
</table>
```

### Modal

```typescript
<div className="fixed inset-0 z-50 flex items-center justify-center p-4">
  {/* Backdrop */}
  <div className="absolute inset-0 bg-black/80 backdrop-blur-sm" onClick={onClose} />

  {/* Modal */}
  <div className="relative w-full max-w-lg bg-slate-800 border border-slate-700 rounded-2xl shadow-2xl">
    {/* Header */}
    <div className="flex items-center justify-between p-6 border-b border-slate-700">
      <h2 className="text-xl font-bold text-white">Modal Title</h2>
      <button onClick={onClose} className="p-1 hover:bg-slate-700 rounded-lg transition-colors">
        <X size={20} className="text-slate-400" />
      </button>
    </div>

    {/* Content */}
    <div className="p-6">
      {children}
    </div>

    {/* Footer */}
    <div className="flex justify-end gap-3 p-6 border-t border-slate-700">
      <button onClick={onClose} className="px-4 py-2 text-slate-300 hover:text-white">Cancel</button>
      <button className="px-6 py-2 bg-blue-600 hover:bg-blue-700 text-white font-semibold rounded-lg">Confirm</button>
    </div>
  </div>
</div>
```

### Toast Notification

```typescript
// Success
<div className="flex items-center gap-3 px-4 py-3 bg-emerald-500/10 border border-emerald-500 text-emerald-500 rounded-lg">
  <CheckCircle size={20} />
  <span className="font-medium">Changes saved successfully</span>
</div>

// Error
<div className="flex items-center gap-3 px-4 py-3 bg-red-500/10 border border-red-500 text-red-500 rounded-lg">
  <AlertCircle size={20} />
  <span className="font-medium">Failed to save changes</span>
</div>
```

### Loading Spinner

```typescript
<div className="flex items-center justify-center p-8">
  <div className="w-8 h-8 border-3 border-blue-500 border-t-transparent rounded-full animate-spin" />
</div>
```

### Empty State

```typescript
<div className="flex flex-col items-center justify-center p-12 text-center">
  <div className="w-16 h-16 mb-4 text-slate-600">
    <Inbox size={64} />
  </div>
  <h3 className="text-xl font-semibold text-white mb-2">No users found</h3>
  <p className="text-slate-400 max-w-md">
    Try adjusting your search or filters to find what you're looking for.
  </p>
</div>
```

---

## Accessibility Standards

### WCAG 2.1 Level AA Compliance

**Color Contrast:**
- All text meets 4.5:1 contrast ratio against background
- Interactive elements meet 3:1 contrast ratio
- Focus indicators visible with 3:1 contrast

**Keyboard Navigation:**
- All functionality accessible via keyboard
- Tab order follows logical visual flow
- Skip links for main content
- Escape key closes modals

**Screen Reader Support:**
- Semantic HTML (header, nav, main, article)
- ARIA labels for icon-only buttons
- ARIA live regions for toasts and dynamic content
- Field labels and error announcements

**Focus Management:**
- Focus visible on all interactive elements (2px blue outline)
- Focus trapped in modals
- Focus returned to trigger after modal close
- Skip to main content link

**Reduced Motion:**
- Respects `prefers-reduced-motion` setting
- No auto-playing animations
- Smooth transitions disabled when requested

---

## Responsive Design

### Breakpoints

```css
/* Mobile First */
--bp-sm: 640px;   /* Large phones */
--bp-md: 768px;   /* Tablets */
--bp-lg: 1024px;  /* Small laptops */
--bp-xl: 1280px;  /* Desktops */
--bp-2xl: 1536px; /* Large screens */
```

### Layout Adaptations

| Screen Size | Sidebar | Content | Tables |
|-------------|---------|---------|--------|
| **< 640px** | Hidden (hamburger menu) | Single column | Card view (transform table rows to cards) |
| **640-1023px** | Icon-only (64px) | Single column | Card view or horizontal scroll |
| **≥ 1024px** | Full (240px) | Multi-column | Standard table |

### Mobile Navigation

```
┌─────────────────────────────────────┐
│ ☰  Opportunity Finder Admin         │
├─────────────────────────────────────┤
│                                     │
│  [Search...]                        │
│                                     │
│  ┌─────────────────────────────┐   │
│  │ alice@email.com             │   │
│  │ Pro | Active                │   │
│  │ [View] [Edit]               │   │
│  └─────────────────────────────┘   │
│  ┌─────────────────────────────┐   │
│  │ bob@corp.io                 │   │
│  │ Biz | Active                │   │
│  │ [View] [Edit]               │   │
│  └─────────────────────────────┘   │
│                                     │
└─────────────────────────────────────┘
```

---

## Interaction Patterns

### Confirmation Dialogs

All destructive actions require confirmation:

```
┌────────────────────────────────────────────────────────┐
│ Confirm Action                                   [×] │
│ ───────────────────────────────────────────────────── │
│                                                        │
│ ⚠️  Are you sure you want to cancel the subscription  │
│     for alice@email.com?                               │
│                                                        │
│     This action cannot be undone. The user will lose   │
│     access to premium features at the end of their     │
│     billing period (Feb 10, 2026).                     │
│                                                        │
│     Reason for cancellation (optional):                │
│     ┌────────────────────────────────────────────┐     │
│     │ Select a reason...                      ▼  │     │
│     └────────────────────────────────────────────┘     │
│                                                        │
│ ┌──────────────────┐  ┌──────────────────────────┐     │
│ │ Never Mind       │  │ Yes, Cancel Subscription │     │
│ └──────────────────┘  └──────────────────────────┘     │
└────────────────────────────────────────────────────────┘
```

### Form Validation

- **Real-time:** Validate on blur for individual fields
- **On submit:** Validate all fields, focus first error
- **Error display:** Red text below field, shake animation
- **Success:** Green checkmark icon appears

### Loading States

| Action | Loading Indicator | Location |
|--------|------------------|----------|
| **Button click** | Spinner inside button | Button text → spinner |
| **Page load** | Skeleton cards | Replace content area |
| **Table load** | Skeleton rows | 3-5 row skeletons |
| **Modal submit** | Full overlay | Dim modal + spinner |

### Optimistic Updates

For non-destructive actions:
- Update UI immediately
- Show "Saving..." indicator
- Rollback on error with toast

### Pagination

```
┌────────────────────────────────────────────────────────┐
│ Showing 1-25 of 1,247 users                            │
│                                                        │
│ ◀ Previous    1 2 3 4 5 ... 50    Next ▶              │
└────────────────────────────────────────────────────────┘
```

- Always show total count
- Current page highlighted
- Show first 5 + last page
- Jump to page input

---

## Error States & Validation

### Error Message Hierarchy

1. **Inline errors** (field validation)
2. **Banner errors** (page-level issues)
3. **Modal errors** (action failures)
4. **Toast errors** (background failures)

### Error Display Patterns

```typescript
// Inline Field Error
<div className="space-y-2">
  <label>Email Address</label>
  <input className="border-red-500" />
  <p className="text-red-500 text-sm">
    <AlertCircle size={14} className="inline mr-1" />
    Please enter a valid email address
  </p>
</div>

// Banner Error (top of page)
<div className="flex items-center gap-3 p-4 bg-red-500/10 border border-red-500 text-red-500 rounded-lg mb-6">
  <AlertCircle size={20} />
  <div>
    <p className="font-semibold">Failed to load users</p>
    <p className="text-sm">Please try again or contact support if the problem persists.</p>
  </div>
  <button className="ml-auto px-4 py-2 bg-red-500 hover:bg-red-600 text-white text-sm font-semibold rounded-lg">
    Retry
  </button>
</div>
```

### Validation Rules

| Field | Rule | Error Message |
|-------|------|---------------|
| **Email** | Valid format, unique | "Please enter a valid email address" |
| **Password** | Min 8 characters | "Password must be at least 8 characters" |
| **Price** | Positive number | "Price must be a positive number" |
| **Tier name** | Required, unique | "Tier name is required and must be unique" |
| **Weights** | Sum to 100% | "Weights must sum to 100% (currently: {sum}%)" |
| **API key** | Required, non-empty | "API key is required" |

---

## Success Metrics

### UX Performance Targets

| Metric | Target | Measurement |
|--------|--------|-------------|
| **Page load** | < 2 seconds | Lighthouse Performance Score |
| **Time to interactive** | < 3 seconds | Lighthouse TTI |
| **First input delay** | < 100ms | Lighthouse FID |
| **Task completion rate** | > 95% | Analytics for admin actions |
| **Error rate** | < 1% | Error tracking (Sentry) |
| **User satisfaction** | > 4.5/5 | Quarterly admin survey |

### Key User Journeys to Track

1. **User lookup:** Search → View profile → Action complete
2. **Pricing change:** Navigate → Edit tier → Save → Verify
3. **Scoring adjustment:** Navigate → Adjust weights → Preview → Save
4. **Source config:** Navigate → Configure → Test → Save
5. **Cancel subscription:** Find user → View detail → Cancel → Confirm

---

## Appendix: Tailwind CSS Reference

### Dark Mode Theme Classes (Admin-Specific)

| Element | Classes |
|---------|---------|
| **Admin badge** | `bg-purple-600/20 border-purple-500 text-purple-400` |
| **Revenue card** | `bg-gradient-to-br from-emerald-500/10 to-emerald-600/5 border-emerald-500/30` |
| **User count card** | `bg-gradient-to-br from-blue-500/10 to-blue-600/5 border-blue-500/30` |
| **Sidebar item (active)** | `bg-blue-600/20 border-l-2 border-blue-500 text-blue-400` |
| **Sidebar item (hover)** | `hover:bg-slate-700/50 text-slate-300` |
| **Destructive zone** | `bg-red-500/10 border-red-500/30` |

---

**Document End**

---

## Next Steps for Implementation

1. **Review this design** with stakeholders
2. **Create interactive prototype** (Figma/React)
3. **User testing** with 3-5 admin users
4. **Refine based on feedback**
5. **Hand off to development** with this spec
6. **Design QA** during implementation
7. **Usability testing** post-launch

---

*This UX design specification ensures the Admin Panel will be intuitive, efficient, and consistent with the existing Opportunity Finder design system while addressing the unique needs of system administrators.*
