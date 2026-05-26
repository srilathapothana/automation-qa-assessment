# Automation & QA Developer — Skills Assessment
**Candidate:** Srilatha Pothana  
**Date:** May 2026

---

## Files in this Repository

| File | Description |
|------|-------------|
| `Task1_QA_Report_SrilathaPothana.pdf` | Task 1 — QA Bug Report for demo.realworld.io (7 bugs + root-cause analysis) |
| `Task2_Workflow_SrilathaPothana.json` | Task 2 — n8n workflow JSON (import directly into n8n) |
| `Task2_and_Bonus_Workflows_SrilathaPothana.pdf` | Task 2 README + both workflow JSONs in PDF format |

---

## Task 1 — Web App QA & Debug Report
- **App tested:** [demo.realworld.io](https://demo.realworld.io)
- **Issues found:** 7 (2 Critical, 3 High, 2 Medium)
- Covers: functional bugs, security issues, UX problems, and performance issues
- Includes a full root-cause analysis for the CORS API failure bug

## Task 2 — n8n API Integration Workflow
- **Trigger:** Schedule (every 1 hour)
- **API 1:** HackerNews Firebase API — fetches top 5 stories
- **API 2:** Algolia HN Search API — enriches high-score stories (score > 100)
- **Output:** Formatted digest posted to Discord webhook
- **Error handling:** All HTTP nodes use `onError: continueErrorOutput` — no silent failures

## Bonus — Uptime Monitor
- Pings demo.realworld.io every 5 minutes
- Retry logic before alerting (avoids false positives)
- Discord alert on downtime + daily 08:00 summary report
