# Map: Flowline roadmap, v0.1 to v1.0

Label: wayfinder:map

Tickets live in [`issues/`](issues/). A readable summary with the recommendation and milestone gates is in [`docs/delivery-plan.md`](../../docs/delivery-plan.md).

## Destination

A build-ready plan for Flowline: every decision that gates code is made and written down, so that building v0.1 can start with nothing left to decide, and each later milestone (v0.2, v0.3, v0.4, v1.0) has its gating decisions either made or named. Concretely that means verified facts about Azure DevOps Server, a written definition for every metric in the next milestone, and settled answers for data access path, minimum Server version, name, license, UI, storage and authentication.

This map plans. It does not build. Reaching for code is the signal that the map's edge has been reached for that milestone.

## Notes

- Domain: delivery metrics (DORA, flow, SAFe Program Increment reporting) read from Azure DevOps, with on-premises Azure DevOps Server as the first-class target. The reference environment is a Windows-based .NET 10 backend team on Azure DevOps Server (Git, pipelines, Azure Boards), SAFe with four Program Increments per year, two-week sprints, PR before merge, squash merge.
- Source material: the brief, the landscape research and the pitch supplied on 2026-11-10. Their unverified items are the research tickets here.
- Standing rules for every session on this map:
  1. Metric definitions come before code. A definition has: definition, inputs, assumptions, edge cases, what would make it wrong or misleading, how to test it, and a worked example on a synthetic event timeline.
  2. Verify, do not recall. Say which Server version (2019, 2020, 2022, the current continuously updated release) or Services a claim applies to. Mark anything unverified as unverified.
  3. Team-level metrics only. No ticket may resolve into a feature that ranks or scores individuals.
  4. Every feature names its milestone. Challenge scope creep.
  5. Be honest about the competition. Do not claim that nothing open source exists.
  6. Privacy by default: self-hosted, least-privilege read-only access, pseudonymization option, no telemetry.
  7. Documentation deliverables are documentation only. No scripts or pipeline definitions.
- Skills to consult: `grilling` and `domain-modeling` for grilling tickets (the event model ticket should produce the glossary in `CONTEXT.md` and ADRs under `docs/adr/`), `research` for research tickets, `prototype` if the UI ticket needs something to react to.
- Research findings are kept in [`docs/research/`](../../docs/research/), one file per research ticket.
- When building starts, reuse the GapWatch conventions: `Directory.Build.props`, central package management, nullable enabled, warnings as errors, xunit v3 with Shouldly, a pure core library with I/O at the edges, stable public ids, golden-file tests.
- Tracker: local markdown under `.scratch/`, because the repo has no tracker configured yet. The map can move to GitHub Issues later without changing its content.
- The repo was created with an MIT `LICENSE` file. Treat that as provisional until the License ticket is resolved.

## Decisions so far

<!-- one line per resolved ticket: gist, then the link -->

- [Analytics and OData on Azure DevOps Server](issues/01-analytics-odata-on-server.md): present on every version but optional or switchable on Server, and pipeline data is preview only, so it cannot be the only data path.
- [Landscape gaps](issues/04-landscape-gaps.md): DevLake added Server support in August 2026, so the gap is now the combination of open source, Server, work item flow metrics and PI reporting, not Server support alone.
- [Authentication options on Server](issues/03-authentication-options-on-server.md): Server offers personal access tokens and Windows authentication only; Entra identities are Services only; no read-only scope is published for pipeline environments.
- [Name availability](issues/06-name-availability.md): the NuGet id Flowline belongs to an active .NET tool in the Microsoft ecosystem, so the working name cannot be the package prefix; four alternatives looked clear.

## Not yet specified

- **Program Increment and iteration model (v0.3).** How PIs are represented when Azure DevOps only knows iteration paths, how team membership by period is captured, and whether PI reporting belongs in the core or as a layer on top of a general flow-metrics core. Waits on the event model and on seeing how the reference environment actually structures its iteration tree.
- **PI metric definitions (v0.3).** PI burn-up, planned versus delivered, predictability, feature and epic roll-ups. Each needs a written definition. Cannot be phrased sharply until the PI model exists.
- **Mapping configuration (v0.2 and v0.4).** The shape of the configuration that maps a team's states, work item types, environments and "production" to the model, the presets shipped, and how per-team overrides work.
- **Metric definition versioning (v1.0).** How a definition is versioned, how a stored result records which version produced it, and what "stable" means at v1.0. Waits on the first definitions being written.
- **Sync design.** Watermarks, full versus incremental sync, backfill of history, throttling, and how derived tables are recalculated when a mapping or definition changes. Waits on the data access path and the REST surface research.
- **Hosting and packaging.** Container, Windows service, IIS, or all of these, given that the reference environment is Windows on-premises. Waits on the storage and UI decisions.
- **Export (v0.4).** CSV and OpenMetrics shapes, and whether a Prometheus and Grafana first mode changes them. Waits on the UI decision.
- **Data quality reporting.** How the product shows that a metric is unreliable because of work item hygiene or pipeline conventions, rather than showing a confident wrong number. Waits on the metric definitions.
- **Demand.** The research shows the platform is supported, not that demand is large. Whether and how to test demand before v1.0 is not yet a sharp question.

## Out of scope

- Individual developer scoring, ranking or leaderboards. This is a design principle, not a deferred feature.
- Replacing Azure Boards or Azure DevOps dashboards.
- GitHub, GitLab or Jira connectors before v1.0. The connector abstraction stays clean so they can follow.
- A hosted or commercial offering. It only matters here as an input to the License ticket.
- Scripts, pipeline definitions and any calculator code. This map ends where building starts.
