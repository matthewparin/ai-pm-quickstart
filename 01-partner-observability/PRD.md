# PRD: Partner Health Dashboard

**Status:** Draft
**Author:** Matthew Parin
**Directory** PM Portfolio
**Last updated:** 2026-03-29

---

## Problem

ISV partners who build apps on the Salesforce platform struggle with visibility into how their app is performing post-launch. They cannot see how many customers have installed their app, whether users are active, or whether errors are degrading the experience. This creates three failure modes: partners can't detect regressions, can't prioritize fixes, and can't make a credible case to their own stakeholders for continued investment. Churn risk increases when partners feel flying blind.

---

## Users

**Primary: ISV partner developers and technical leads**
- Responsible for the app's uptime, quality, and feature roadmap
- Access the platform via a partner portal; comfortable with technical metrics; AppAnalytics is a tool that some ISVs use (~200), but it has a high barrier-to-entry; better would be a new framework that automatically evaluates an applcation and applies metrics at design time and pulls these metrics into the partner-centered UI that allows for deep exploration and debugging in real-time
- Need signal fast — they are not Salesforce employees and don't have internal escalation paths; excalation paths are via Salesforce cases, which take considerable skill to navigate

**Secondary: Partner business owners / partner account managers**
- Less technical; want a top-line health signal to report upward
- May not need raw error logs but need to know if there's a problem

---

## Goals

1. Give partners self-serve visibility into app adoption and engagement — reducing inbound support tickets asking "how is my app doing?"
2. Enable partners to detect error spikes within 1 hour without relying on Salesforce escalation
3. Surface a single health score partners can use as a shared language with their Salesforce partner manager
4. Success metrics:
   - ≥70% of active ISV partners log in to view dashboard at least once per month within 90 days of launch
   - ≥30% reduction in "app performance" support tickets from partners within 6 months
   - 99% of metrics data is available to view within 1 hour, with real-time the north star goal
   - Partner NPS (measured in quarterly survey) improves by ≥10 points among ISVs who use the dashboard

---

## Non-goals

- Real-time alerting or push notifications (not in v1)
- Comparison benchmarking against other partners' apps
- Customer-facing visibility (this is partner-only)
- Root cause analysis or log streaming (partners see aggregated error counts, not raw logs)
- Support for partners who have not yet published a live app

---

## Requirements

### App Install Counts Over Time
- Display cumulative install count and net new installs by day/week/month (toggle)
- Scope: all orgs where the partner's app is currently installed and active
- Exclude trial or sandbox installs from production count; show separately if requested
- Data freshness: T+1h is acceptable for v1

### Monthly Active Users (MAU)
- Define MAU as: unique users who invoked at least one feature of the app within a rolling 28-day window
- Display current MAU and trend (MoM delta, %)
- Exclude internal Salesforce test users

### Error Rates and Top Errors
- Show error rate as: errors per 1,000 app invocations, trended over time
- Surface top 5 errors by volume in the selected time window, with:
  - Error type / code
  - Count
  - First seen / last seen timestamps
- No PII in error display; org IDs must be masked unless partner has explicit data sharing agreement

### Composite Partner Health Score
- Single score (0–100) combining: install growth trend (25%), MAU retention (25%), error rate (25%), and app uptime/availability (25%)
- Score displayed as a number and a traffic-light band: Healthy (70–100), Needs Attention (40–69), At Risk (0–39)
- Score recalculates daily
- Provide a brief plain-language explanation of what is dragging the score down, if below 70

### Access and Permissions
- Dashboard accessible only to users with Partner Admin or Partner Developer role in the partner portal
- Data scoped to the authenticated partner's own apps only — no cross-partner data access

---

## Open Questions

1. **Data pipeline ownership:** Which team owns the event instrumentation on the platform side that feeds install counts and invocation events? Is this already collected, or does it require new instrumentation?

This requires new instrumentation. Be prepared for this. I want to see this done by an agent, if possible.

2. **Health score weighting:** Are the four components (install growth, MAU, error rate, uptime) equally weighted by default, or should we validate weights with partner data before hardcoding?

Error rate and uptime are most important for enterprise use cases. MAU is after that. Install growth is the last, given most installs are driven by sales processes and not organic growth. 

3. **Error masking and data agreements:** Do any enterprise ISV partners already have data sharing agreements that would permit org-level error attribution? If so, do we need a tiered view?

Assume data agreements do not exist, so a permission flow will need to be in place to enable this.

4. **MAU definition alignment:** Does "app invocation" map to an existing platform event, or do we need partners to instrument their apps? If the latter, adoption of the dashboard is gated on partner effort.

There should be default metrics that come out of the box, and are instrumented in an ISV app via an agent scanning an app at design time. It should be clear how these metrics are added to the codebase and then how these metrics will be visualized.

5. **Historical data availability:** How far back does platform telemetry go? Partners will expect at least 12 months of trend data at launch — is that feasible given current data retention policies?

One month is the default for now. Let's start with a one month (28 day) lookback.
