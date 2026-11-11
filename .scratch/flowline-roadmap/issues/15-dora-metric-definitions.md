# The five DORA metric definitions

Type: grilling (HITL)
Status: open
Blocked by: 05, 11, 14
Label: wayfinder:grilling

## Question

What are the written definitions of change lead time, deployment frequency, failed deployment recovery time, change fail rate and deployment rework rate for v0.2? Each follows the definition template with a worked example, states where it departs from the dora.dev wording and why, and says how squash merges affect the commit-to-production link.

## Comments

2026-11-11. Mattias accepted these choices. They are fixed inputs to this ticket, which stays open for the full written definitions and worked examples.

- **Start of change lead time.** The unit of change is the pull request. The clock starts at the first commit of the pull request, read from the pull request's commit list, so it survives squash merge and includes review time. The metric is shown in two parts: first commit to merge, and merge to production. This departs from a per-commit reading of DORA and the definition must say so.
- **Recovery clock.** Failed deployment recovery time runs from the failed deployment finishing to the fixing deployment finishing. Where incident records exist, incident created to incident resolved is shown beside it, not instead of it.
- **Statistic.** Median plus the 85th percentile. Never the mean. Same as the flow metrics.
- Deployment and failure are defined in the ticket "What counts as a deployment, production, a failure and an incident".
