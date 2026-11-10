# Milestone 4 Frontend Gap Analysis & Architecture Plan

## Executive Summary
This document provides a comprehensive audit of the SentinelAI React frontend across Milestones 1, 2, and 3, identifying existing components, available APIs, and the exact gaps required to deliver the final Milestone 4 SOC experience.

---

## 1. Inventory of Existing Screens & Routes

| Route | Page Component | Current Milestone Scope | Status & M4 Role |
|---|---|---|---|
| `/` | `Login.jsx` | M1 Authentication UI | Complete; preserve as SOC analyst login gate |
| `/dashboard` | `Overview.jsx` | M1/M2 Telemetry & Predictions | **Requires Major Upgrade:** Upgrade to primary SOC Overview with dynamic M4 KPIs, Security Posture widget, severity/status distributions, time range filter (24h/7d/30d), and clickable Critical Incidents queue |
| `/dashboard/risk-overview` | `RiskOverview.jsx` | M3 Risk Intelligence & Weights | Complete; preserve dynamic weights configuration and factor sensitivity analysis |
| `/dashboard/incidents` | `PriorityIncidents.jsx` | M3 Risk Prioritization Queue | **Extend:** Add M4 cross-domain filters (`cve`, `ioc_status`, `severity`, `time_range`) |
| `/dashboard/incidents/:id` | `IncidentDetails.jsx` | M3 Incident Investigation | **Extend:** Prominently display "WHY IS THIS INCIDENT RISKY?" with backend-provided reasons, link correlated events to `/dashboard/events/:id` |
| `/dashboard/attack-chains` | `AttackChains.jsx` | M3 Multi-Stage Adversary Chains | Complete; extend stage inspection and navigation |
| `/dashboard/detection` | `ThreatDetection.jsx` | M2 ML Anomaly & Classification | Complete; preserve ML confidence inspection |
| `/dashboard/events/:id` | `EventDetails.jsx` | M2 Prediction & Event Detail | **Extend:** Support atomic event fallback via `/events/{event_id}` |
| `/dashboard/events` | `SecurityEvents.jsx` | M1 Raw Telemetry Stream | Complete; preserve baseline telemetry table |
| `/dashboard/threat-intel` | `ThreatIntel.jsx` | M1 Static IOC Table | **Requires Upgrade:** Upgrade to M4 Intelligence view with summary metrics, active indicators, correlation match status, and actor attribution |
| `/dashboard/vulnerabilities`| `Vulnerabilities.jsx` | M1 CVE Table | **Requires Upgrade:** Upgrade to M4 Vulnerability Intelligence with severity breakdown, affected assets, patch status, and CVSS distributions |
| `/dashboard/analytics` | `Analytics.jsx` | M1/M2 Aggregate Analytics | Complete; preserve exploratory analytics |
| `/dashboard/executive` | **MISSING (New)** | M4 Executive Overview | **New Screen:** Executive Security Overview answering "How healthy is the organization's security posture?" with posture gauge, volume trends, and risk summaries |
| `/dashboard/reports` | **MISSING (New)** | M4 Reporting & Export | **New Screen/Modal:** Executive report generator with instant RFC 4180 CSV streaming download and JSON report view |

---

## 2. Reusable Component Inventory

| Component | Path | Current Features | M4 Extension Strategy |
|---|---|---|---|
| `KpiCard` | `components/KpiCard.jsx` | Stat card with status badge and icon | Reuse in SOC Overview and Executive Overview for dynamic metrics |
| `ErrorBanner` | `components/ErrorBanner.jsx` | Standardized error banner with retry | Reuse across all pages for consistent error handling |
| `RiskBadge` | `components/m3/RiskBadge.jsx` | `RiskLevelBadge`, `PriorityBadge`, `StatusBadge`, `RiskScoreBadge` | Reuse throughout all incident tables, overview cards, and reports |
| `RiskBreakdownCard` | `components/m3/RiskBreakdownCard.jsx`| 5-factor mathematical score breakdown | Preserve intact; display values, weights, and factor contributions |
| `RiskScoreComparisonCard`| `components/m3/RiskScoreComparisonCard.jsx`| Before vs After correlation delta | Preserve intact; explains $\Delta = 0$ non-inflation |
| `AttackChainTimeline` | `components/m3/AttackChainTimeline.jsx` | Multi-stage chronological animation | Extend with clickable stage inspection linking to raw events |
| `RecommendationsPanel`| `components/m3/RecommendationsPanel.jsx`| Tailored SOC action advisories | Preserve intact; non-destructive guidance |
| `AnalystFeedbackModal`| `components/m3/AnalystFeedbackModal.jsx`| False positive feedback reason modal | Preserve intact; wire to backend persistence |
| `AnalystFeedbackCard` | `components/m3/AnalystFeedbackCard.jsx` | Incident feedback audit history | Preserve intact; display analyst review history |
| `RiskWeightConfigModal`| `components/m3/RiskWeightConfigModal.jsx`| Dynamic factor weight tuner | Preserve intact in Risk Overview |
| `Sidebar` | `components/layout/Sidebar.jsx`| Navigation sidebar | **Update Branding & Structure:** Categorize into SOC MONITORING, RISK & INVESTIGATION, INTELLIGENCE, and EXECUTIVE |
| `Header` | `components/layout/Header.jsx` | Top header with dark/light toggle | **Update Branding:** SentinelAI AI-Assisted Security Operations, add polling indicator |

---

## 3. Existing vs Missing API Functions in `src/services/api.js`

### Existing Functions (Preserve Intact):
- **M1:** `getEvents`, `getStats`, `getThreats`, `getThreatIntel`, `getVulnerabilities`, `createEvent`
- **M2:** `getPredictions`, `getPredictionById`, `getAnomalies`, `getThreatSummary`, `getModelPerformance`, `getPredictionTrend`, `getTopPredictions`, `predictEvent`
- **M3:** `getRiskSummary`, `getHighRiskIncidents`, `getIncidents`, `getIncidentById`, `getAttackChains`, `getRecommendations`, `updateIncidentStatus`, `calculateRisk`, `getRiskWeights`, `updateRiskWeights`, `resetRiskWeights`, `submitAnalystFeedback`, `getAnalystFeedback`


---

## 4. Components That Should NOT Be Duplicated
- **No Client-Side Risk Scoring:** Do not recalculate 5-factor risk, attack-chain confidence, or security posture in React. Always consume backend API values.
- **No Synthetic Mock Telemetry:** Do not generate random numbers or fake attack chains in frontend state.
- **No Parallel Incident Tables:** Re-use the established incident table/card patterns from M3.
- **No Duplicate Status Lifecycle Handlers:** Centralize status transitions through `updateIncidentStatus()` and `submitAnalystFeedback()`.

---

## 5. Visual Language & Theme Invariants
- **Dark Theme Dominant:** Dark navy background `#0F172A`, slate borders `#1F2937`, cards `#111827`, sub-boxes `#1E293B`.
- **Light Theme Compatibility:** Tested CSS overrides in `global.css` for clean high-contrast presentation.
- **Risk Severity Colors:**
  - Critical: Crimson `#EF4444`
  - High / Severe: Orange `#F97316`
  - Medium / Moderate: Amber `#FBBF24`
  - Low / Clean: Emerald `#10B981`
  - Informational / Active: Blue `#3B82F6`
