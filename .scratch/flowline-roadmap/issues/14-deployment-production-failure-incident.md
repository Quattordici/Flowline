# What counts as a deployment, production, a failure and an incident

Type: grilling (HITL)
Status: resolved
Blocked by: 02, 05, 07
Label: wayfinder:grilling

## Question

How does Flowline recognize a deployment, the production environment, a failed deployment and a production incident across different pipeline setups (classic releases, YAML environments, plain builds)? Where do incidents come from when Azure DevOps has no incident concept (a work item type, a tag, a hotfix branch convention)? Decide the configuration-driven rules and the presets shipped.

## Comments

2026-11-11. Mattias accepted these choices. They are fixed inputs to this ticket, which stays open for the configuration rules, the presets and the incident source in the reference environment.

- **Deployment.** A deployment is the successful completion of a stage or environment that the team has mapped as production. Failed attempts are not deployments. Constraint from research: pipeline environments are preview on Server 2020 and absent on Server 2019, and no read-only token scope is published for them.
- **Failed deployment.** The source of truth is an incident marker: a configured work item type or tag, attributed to the latest production deployment before it. A pipeline heuristic (rollback or hotfix deployment within a time window) is an optional preset, not the default.
- **Rework versus failure.** Change fail rate counts the deployments that caused a problem. Deployment rework rate counts the deployments that were the fix.

## Answer

2026-11-16. Resolved with two facts from Mattias: deployment to production goes through Azure DevOps pipelines, and production incidents go through ServiceNow and Azure DevOps. The full rules, edge cases and a worked example are in [docs/definitions/deployment-and-failure-rules.md](../../../docs/definitions/deployment-and-failure-rules.md).

- Production deployment: one per service per pipeline run, when every production-mapped stage that ran has succeeded. Production is mapped per service by stage name, environment name or classic release environment name. A plain build is not a deployment.
- Production incident: an Azure DevOps work item matching a configured marker (type, tag or both). ServiceNow is not read. An incident only in ServiceNow is not counted, and the product says the failure metrics are a lower bound.
- Failed deployment: a production deployment with an attributed incident. Attribution is by explicit build or release link first, otherwise the latest deployment of the same service within a three-day window, otherwise unattributed.
- Fixing deployment: the deployment carrying a pull request linked to the incident, otherwise the first rollback, otherwise the last deployment before the incident closed.

Resolved while the tickets "REST API surface per Server version" and "Reference environment and test instances" were still open. The rules work on normalized events, so they do not depend on those answers, but how stage results are read on each Server version is still unverified. Four configuration values remain to be confirmed at setup and are listed at the end of the rules document.
