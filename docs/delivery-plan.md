# Flowline delivery plan

Status: draft for review, 2026-11-10. Planning only. Nothing here is built, and nothing here is a script or a pipeline definition.

This plan was charted with the wayfinder method: a map of decision tickets that are resolved one at a time until the way to building is clear. The map is [`.scratch/flowline-roadmap/map.md`](../.scratch/flowline-roadmap/map.md) and the tickets are in [`.scratch/flowline-roadmap/issues/`](../.scratch/flowline-roadmap/issues/). This document is the readable summary. Where the two differ, the map and its tickets win.

## Recommendation

Do not write code yet. Take three steps in order, and let each one gate the next.

1. **Verify.** Close the six research tickets. Two of them decide the architecture: whether Analytics and OData can be relied on across Server versions, and which REST endpoints exist on each version. Until they are answered, the data access path and the minimum supported Server version are guesses.
2. **Decide and define.** Settle the data access path, the minimum Server version, the name, and the event model with its glossary. Then write the flow metric definitions with worked examples. The name comes early because it becomes the namespace and NuGet prefix of every project.
3. **Build v0.1 as a thin slice against the one Server version you actually run.** Work items in, cycle time and throughput out, one team, one database. Support for further Server versions is added by recorded-response tests after the slice works, not designed in up front.

Three things in the current roadmap deserve a challenge before they are accepted:

- **Builds in v0.1.** The roadmap lists a connector for work items and builds in v0.1, but cycle time and throughput need work items only. I recommend moving builds to v0.2 with deployments, unless a build signal is wanted purely to prove the connector. This is the ticket "v0.1 scope and exit criteria".
- **The second connector in v1.0.** The roadmap puts a second connector in v1.0 while the non-goals exclude GitHub, GitLab and Jira from v1. Either the second connector is Azure DevOps Services, or it should leave v1.0. This is the ticket "Services and the second connector".
- **"Older versions still in the field."** The pitch promises this before anyone has checked what Server 2019 and 2020 can deliver. Both are already out of active support. Treat the promise as unverified until the ticket "Minimum supported Server version and test strategy" is resolved.

## Why this order

- The project rules say metric definitions come before code and that claims about Azure DevOps are verified, not recalled. The order above is those two rules applied.
- The riskiest unknowns are facts, not design choices. Facts are cheap to look up and expensive to discover after a connector exists.
- A single-version slice gives real data early, and real data is the only thing that will test the definitions against work item hygiene as it really is.

## Stage 0: verification

Research tickets. None needs a decision from you. Findings go to `docs/research/`, one file per ticket, with a source per claim and the Server version each claim applies to.

| Ticket | What it settles | Decisions waiting on it |
|---|---|---|
| Analytics and OData on Azure DevOps Server | Availability, entity sets and limits per Server version | Data access path, minimum Server version |
| REST API surface per Server version | Endpoints and api-versions for work items, Git, builds, releases, environments | Data access path, minimum Server version, event model, deployment definition, Services |
| Authentication options on Server | What an unattended read-only service can use, and least privilege | Authentication model, identity data |
| Landscape gaps | DevLake issue 8985 status, LinearB, Swarmia, Jellyfish, Oobeya, MiddlewareHQ, Four Keys, Microsoft's own guidance | Whether the differentiation holds, license |
| DORA metric wording | Exact current wording of the five metrics and what DORA leaves open | Flow and DORA definitions, deployment definition |
| Name availability | NuGet, GitHub and existing products named Flowline | Name |

One task ticket sits beside them and needs you: **Reference environment and test instances**. It records which Server version and update you run, whether Analytics is on, how iterations are structured, whether deployments are classic releases or YAML environments, and which instances can be tested against.

Status on 2026-11-11:

- **Analytics and OData on Azure DevOps Server: resolved.** Analytics exists on every version but is optional or can be switched off on Server, the released OData versions cover work tracking only, and pipeline data is preview only. It cannot be the only data path. See [the findings](research/01-analytics-odata-on-server.md).
- **Landscape gaps: resolved, with a finding that changes the pitch.** Apache DevLake merged Azure DevOps Server support in August 2026, so issue 8985 is no longer evidence of a gap. DevLake's plugin does not cover Azure Boards work items. What remains open is the combination of open source, Server, work item flow metrics and PI reporting in a small deployment. The brief and the pitch need rewording before they are used again. See [the findings](research/04-landscape-gaps.md).
- **REST API surface per Server version: partly answered.** Server 2019 supports api-version 5.0, 2020 supports 6.0, 2022 supports 7.0. The work item and pull request endpoints needed for v0.1 exist at 5.0. About half the endpoints, mostly the v0.2 deployment ones, were not read. See [the findings](research/02-rest-api-surface-per-server-version.md).
- **Authentication, DORA metric wording, name availability: still open and unverified.**

None of this was tested against a live server. Vendor sites and parts of Microsoft's documentation were blocked from the research environment, and each findings file lists what it could not verify.

## Stage 1: decisions and definitions

Each of these is a conversation with you, one ticket per session. The recommended order follows the blocking edges in the tickets.

| Order | Ticket | Gates |
|---|---|---|
| 1 | Does the differentiation hold | Everything. A go, narrow, contribute or stop answer |
| 2 | Name | Solution scaffold |
| 3 | Data access path | Event model, sync design |
| 4 | Minimum supported Server version and test strategy | v0.1 scope |
| 5 | Event model and glossary | Every metric definition |
| 6 | Identity data and team-level guardrails | What the event store may hold |
| 7 | Flow metric definitions | v0.1 |
| 8 | Authentication model | v0.1 |
| 9 | Storage support matrix | v0.1 |
| 10 | UI approach | v0.1 |
| 11 | v0.1 scope and exit criteria | Start of building |
| 12 | What counts as a deployment, production, a failure and an incident | v0.2 |
| 13 | The five DORA metric definitions | v0.2 |
| 14 | Services and the second connector | v1.0 |
| 15 | License | First outside contribution, v1.0 at the latest |

Storage, UI and the two research-free tickets are unblocked today and can be taken in any order while research runs.

Every metric definition follows one template: definition, inputs, assumptions, edge cases, what would make it wrong or misleading, how to test it, and a worked example on a synthetic event timeline. The worked examples later become the golden-file tests.

## Milestones

Scope per milestone is taken from the brief. The gates are the tickets that must be resolved before that milestone's code starts.

| Milestone | Scope | Gates | Done when |
|---|---|---|---|
| v0.1 | Azure DevOps connector for work items, cycle time and throughput, simple UI | Stage 1 rows 1 to 11 | Cycle time and throughput percentiles for one real team match a hand calculation from the written definitions |
| v0.2 | Deployment tracking, the five DORA metrics, configurable mappings | Rows 12 and 13, plus the mapping configuration fog | Each DORA metric reproduces its worked example and runs on the reference environment's real pipelines |
| v0.3 | PI and iteration model, feature roll-ups, PI burn-up, predictability | PI model and PI metric definitions, both still fog | A finished Program Increment from the reference environment is reported and agrees with what the train recorded |
| v0.4 | Per-team configuration, pseudonymization, CSV and OpenMetrics export | Identity ticket (decided before v0.1, shipped here), export fog | A second team with a different process is onboarded by configuration alone |
| v1.0 | Stable, versioned metric definitions, every formula documented, second connector if it survives | Definition versioning fog, row 14, row 15 | Definitions carry versions, and every number in the UI links to its formula |

The "done when" column is a proposal. The real exit criteria are set per milestone, starting with the ticket "v0.1 scope and exit criteria".

## What is deliberately not planned yet

The map keeps a "Not yet specified" section for questions that are in scope but cannot be phrased sharply until earlier answers exist: the PI and iteration model, the PI metric definitions, the mapping configuration, metric definition versioning, sync design, hosting and packaging, export formats, data quality reporting, and how to test demand. They become tickets as the frontier reaches them.

## Out of scope

Individual developer scoring or ranking, which is a design principle and not a deferred feature. Replacing Azure Boards or dashboards. GitHub, GitLab or Jira connectors before v1.0. A hosted or commercial offering, except as an input to the license decision.

## Risks to the plan itself

- **The gap has narrowed.** DevLake now supports Server for pipelines and code, so DORA metrics on Server are no longer a differentiator on their own. Work item flow metrics and PI reporting carry more of the case than the brief assumed, which is an argument for the v0.1 scope as recommended and a reason the ticket "Does the differentiation hold" comes first.
- **One reference environment.** Designing against one team's Server risks baking in its conventions. The mapping configuration and the second-team criterion in v0.4 are the counterweight.
- **Definitions without data.** A definition written before any real data may miss the edge cases that matter. Expect the flow definitions to be revised once after v0.1 first runs, and plan for that revision instead of treating it as failure.
- **Demand is not quantified.** Nothing in the research shows how many organizations want this.

## Open questions for you

1. Which Azure DevOps Server version and update does your team run, and is Analytics enabled on it? This is the fastest single fact to unblock Stage 1.
2. Can you get a read-only service account or a personal access token for a test project, and can an older Server version be stood up for testing?
3. Does your deployment to production run through classic releases, YAML environments, or something outside Azure DevOps? And where is a production incident recorded today?
4. How is a Program Increment represented in your iteration tree?
5. Do builds stay in v0.1, or move to v0.2?
6. Is Azure DevOps Services the v1.0 second connector, or does the second connector leave v1.0?
7. Is a bundled UI required for v0.1, or would a Grafana-first mode be acceptable to your first users?
8. Do you want a zero-install database option for evaluation, or is SQL Server alone enough for v0.1?
9. Is a future hosted or commercial offering a real possibility? The license choice turns on this. The repository currently carries an MIT license file.
10. Should the map stay as markdown files in `.scratch/`, or move to GitHub Issues so the blocking edges show in the tracker?
