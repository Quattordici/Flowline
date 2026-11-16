# Deployment and failure rules

Status: decided 2026-11-16, version 1 of these rules. Milestone: v0.2. Documentation only.

These rules say how Flowcompass recognizes a production deployment, a production incident, a failed deployment and a fixing deployment. The five DORA metric definitions build on them and are written separately.

## Summary

- A **production deployment** is a pipeline run in Azure DevOps in which the stage mapped as production finished successfully.
- A **production incident** is an Azure DevOps work item that matches the team's incident marker. ServiceNow is not read. An incident that exists only in ServiceNow does not exist for Flowcompass.
- A **failed deployment** is a production deployment with at least one incident attributed to it.
- A **fixing deployment** is the production deployment that remediated a failed one.

## What these rules rest on

Facts from the reference environment, given by Mattias on 2026-11-16:

- Deployment to production goes through Azure DevOps pipelines.
- Production incidents go through ServiceNow and Azure DevOps.

Choices accepted on 2026-11-11: a deployment is the successful completion of a stage or environment mapped as production; the incident marker is the source of truth for failure; recovery runs from the failed deployment finishing to the fixing deployment finishing; rework counts the fixes and change fail counts the causes.

Assumptions that are not yet confirmed and are settled by configuration at setup, not by redesign:

- Whether production is a stage in a YAML pipeline, a YAML environment, or a classic release environment. The rules cover all three.
- How the Azure DevOps side of an incident looks: its work item type, tag or field, and whether every ServiceNow incident gets an Azure DevOps work item.

Unverified against Microsoft documentation, because the pages were not reachable during research: how stage results are read on each Server version (build timeline), the release deployments list, and environment deployment records. Environments are documented as preview on Server 2020 and are not documented for Server 2019. On Server 2019 the stage-name mapping or the classic release mapping is the only expected route.

## Rule 1: production deployment

**Definition.** One production deployment is recorded per service per pipeline run when every production-mapped stage that ran in that run has succeeded. Its timestamp is the finish time of the last of those stages.

**Inputs.** Pipeline runs with their stages and stage results, the source version each run deployed, and the production mapping.

**Production mapping.** Configured per service. A service is the unit DORA measures: one application or service, owned by one team. The mapping names the pipelines that deploy the service, and for each one how production is recognized:

| Setup | Production is recognized by |
|---|---|
| YAML pipeline with stages | Stage name |
| YAML pipeline with environments | Environment name |
| Classic release | Release environment name |

Preset: a stage or environment whose name matches `prod` or `production`, ignoring case. The preset is a starting point that the team confirms. It is never applied silently.

A plain build with no production stage is not a deployment, even on the main branch.

**Edge cases.**

| Case | Rule |
|---|---|
| Stage failed or was cancelled | Not a deployment. Recorded as a failed attempt for visibility. It enters no DORA metric, because DORA does not count a change stopped before production as a failure |
| Stage succeeded with issues | Counts as succeeded. Configurable |
| Stage retried within the same run | One deployment, timestamped at the successful attempt |
| Several production stages in one run (regions, canary then full) | One deployment, timestamped when the last one finishes. A run where some succeeded and some failed is not a deployment and is flagged as partial |
| One run deploys several services | One deployment per service |
| Same version deployed again while it is already in production | Recorded and flagged as a redeploy. Excluded from deployment frequency by default, because nothing changed |
| An earlier version deployed again (rollback) | A deployment. It changes production |
| Approval wait before the production stage | Part of the run. The wait is real delay and stays inside change lead time |
| Deployment performed outside Azure DevOps | Invisible. Stated as a limitation |

## Rule 2: production incident

**Definition.** A production incident is an Azure DevOps work item that matches the service's incident marker. It opens at the work item's created time, or at a configured "occurred" date field when the team has one. It closes when the work item first enters a state in the Completed or Resolved state category.

**Incident marker.** Configured per team: a work item type, a tag, or both, optionally narrowed by area path. There is no default marker, because no Azure DevOps process template has an incident concept that can be assumed. Until a marker is configured, change fail rate, recovery time and rework rate are shown as not available, not as zero.

**ServiceNow.** Flowcompass reads Azure DevOps only. The rule for a team that works in both systems:

- An incident counts when it has an Azure DevOps work item matching the marker.
- A field can be mapped as the external reference, for example the ServiceNow incident number. Two work items with the same external reference are one incident. The reference is also what lets a person audit a number against ServiceNow.
- An incident that lives only in ServiceNow is not counted. Change fail rate is then a lower bound, and the product says so wherever the metric is shown.

A ServiceNow connector is outside the scope before v1.0. Whether it is a candidate for the second connector belongs to the ticket "Services and the second connector".

**Edge cases.**

| Case | Rule |
|---|---|
| Work item created long after the incident began | Use the "occurred" field when mapped. Otherwise created time is used and the recovery time by incident is understated |
| Incident reopened | The first close stands for recovery. The reopen is recorded |
| Work item moved to Removed | Not an incident |
| Marker added to an existing work item later | It is an incident from its opening time, and derived results are recalculated |
| Incident not tied to one service | Attributed by area path mapping. If no service matches, it is unattributed |

## Rule 3: failed deployment

**Definition.** A production deployment is failed when at least one production incident is attributed to it.

**Attribution.** In this order:

1. An explicit link wins. If the incident work item links to a build or a release, the incident is attributed to the production deployment made by that run. A link to a pull request is not a cause link. A pull request linked to an incident is read as the fix, under rule 4.
2. Otherwise the incident is attributed to the latest production deployment of the same service that finished before the incident opened, provided it finished within the attribution window.
3. Otherwise the incident is unattributed. It is listed separately and enters no rate.

**Attribution window.** Default three days. DORA says a failure needs "immediate intervention" and does not say how long immediate lasts, so the window is a stated Flowcompass choice and is configurable per team.

**Edge cases.**

| Case | Rule |
|---|---|
| Several incidents attributed to one deployment | The deployment is failed once |
| One incident, several candidate deployments | Only the latest one before the incident opened |
| A fixing deployment causes a new incident | It is itself a failed deployment |
| Incident caused by infrastructure, not by a deployment | Time-based attribution may blame an innocent deployment. Teams that see this should link incidents explicitly or mark such incidents out with the marker |

**Optional preset, off by default.** A pipeline heuristic that marks a deployment as failed when it is followed by a rollback within the window, even without an incident work item. It guesses, so it is opt-in and its results are labelled as inferred.

## Rule 4: fixing deployment

**Definition.** The fixing deployment of a failed deployment is, in this order:

1. The first production deployment of the same service, after the failed one, that carries a pull request linked to the incident work item.
2. Otherwise the first rollback of that service after the failed deployment.
3. Otherwise the last production deployment of that service that finished between the incident opening and the incident closing.
4. Otherwise none. The incident was resolved without a deployment, for example by a configuration or data change.

**Use in the metrics.**

- Failed deployment recovery time runs from the failed deployment finishing to the fixing deployment finishing. With no fixing deployment, the incident close time ends the clock and the value is flagged.
- Deployment rework rate counts fixing deployments found by steps 1 to 3.
- Both rates use all production deployments in the period as the denominator, fixing deployments included. DORA does not specify this.

## Worked example

One service, attribution window of three days. Times are on a 24-hour clock.

| When | Event |
|---|---|
| Day 1, 10:00 | D1: production stage succeeds, version 1.4 |
| Day 2, 14:00 | D2: production stage succeeds, version 1.5 |
| Day 2, 16:30 | I1 opened: work item with the incident marker and a ServiceNow reference |
| Day 2, 18:10 | D3: production stage succeeds, version 1.5.1, carrying a pull request linked to I1 |
| Day 2, 19:00 | I1 closed |
| Day 4, 09:00 | D4: production stage succeeds, version 1.6 |
| Day 5, 11:00 | Production stage fails for version 1.7 |
| Day 5, 13:00 | D5: production stage succeeds, version 1.7 |
| Day 10, 10:00 | I2 opened: work item with the incident marker, no links |

Reading it by the rules:

- Production deployments: D1, D2, D3, D4, D5. Five. The failed stage on day 5 is an attempt, not a deployment.
- I1 has no link to a build or release, and its linked pull request is the fix, not the cause. So it goes to the latest deployment before 16:30 on day 2, which is D2, two and a half hours earlier and inside the window. D2 is failed.
- D3 carries a pull request linked to I1, so D3 is the fixing deployment of D2.
- I2 opened four days and 21 hours after D5, outside the three-day window. I2 is unattributed.

Results for the period:

| Measure | Value |
|---|---|
| Production deployments | 5 |
| Change fail rate | 1 of 5, 20 % |
| Deployment rework rate | 1 of 5, 20 % |
| Failed deployment recovery time | D2 at 14:00 to D3 at 18:10, 4 hours 10 minutes |
| Incident open to close, shown beside it | 16:30 to 19:00, 2 hours 30 minutes |
| Unattributed incidents | 1 |

With a seven-day window I2 would be attributed to D5 and the change fail rate would be 2 of 5, 40 %. The window choice can double the number, which is why it is shown next to the metric.

## What would make these rules wrong or misleading

- **Incidents that never reach Azure DevOps.** If ServiceNow incidents are only sometimes mirrored, every failure metric is understated by an unknown amount. This is the largest risk in the reference environment.
- **A wrong production mapping.** A stage named for production that deploys to a staging slot inflates frequency and deflates lead time.
- **One pipeline, many services.** Without a per-service mapping, an incident in one service marks a deployment of all of them as failed.
- **Time-based attribution.** It blames the latest deployment, which is not always the cause.
- **Late incident records.** An incident logged the next morning shortens the incident view of recovery and can fall outside the window.
- **Teams that fix forward quietly.** No incident work item means no failure. The numbers reward not recording incidents, so they must never be used to compare teams or as targets.

## How to test

Each rule and each edge case above becomes a named synthetic timeline with an expected result, kept as a golden file. The worked example is the first of them. The window sensitivity case (three days against seven) is a second. No test needs a live Azure DevOps instance, because the rules work on normalized events only.

## Still to confirm at setup

1. Whether production in the reference environment is a YAML stage, a YAML environment or a classic release environment, and its name.
2. Which work item type, tag or field marks an incident in Azure DevOps, and which field holds the ServiceNow number.
3. Whether every ServiceNow production incident gets an Azure DevOps work item. If not, the failure metrics carry the lower-bound warning from day one.
4. Whether three days is the right attribution window for the team.
