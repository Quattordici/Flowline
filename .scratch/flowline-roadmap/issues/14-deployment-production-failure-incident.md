# What counts as a deployment, production, a failure and an incident

Type: grilling (HITL)
Status: open
Blocked by: 02, 05, 07
Label: wayfinder:grilling

## Question

How does Flowline recognize a deployment, the production environment, a failed deployment and a production incident across different pipeline setups (classic releases, YAML environments, plain builds)? Where do incidents come from when Azure DevOps has no incident concept (a work item type, a tag, a hotfix branch convention)? Decide the configuration-driven rules and the presets shipped.

## Comments

2026-11-11. Mattias accepted these choices. They are fixed inputs to this ticket, which stays open for the configuration rules, the presets and the incident source in the reference environment.

- **Deployment.** A deployment is the successful completion of a stage or environment that the team has mapped as production. Failed attempts are not deployments. Constraint from research: pipeline environments are preview on Server 2020 and absent on Server 2019, and no read-only token scope is published for them.
- **Failed deployment.** The source of truth is an incident marker: a configured work item type or tag, attributed to the latest production deployment before it. A pipeline heuristic (rollback or hotfix deployment within a time window) is an optional preset, not the default.
- **Rework versus failure.** Change fail rate counts the deployments that caused a problem. Deployment rework rate counts the deployments that were the fix.
