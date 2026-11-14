# REST API surface per Server version

Type: research (AFK)
Status: open
Label: wayfinder:research

## Question

Which REST endpoints and api-versions that Flowline needs (work item history, Git pull requests and commits, builds, releases, environments and deployments) exist on Server 2019, 2020, 2022, the current release, and Services? How does a client discover the supported version, and what throttling applies?

Findings file: [docs/research/02-rest-api-surface-per-server-version.md](../../../docs/research/02-rest-api-surface-per-server-version.md)

## Comments

2026-11-10, partial answer, ticket stays open. Findings so far are in the findings file above.

- Microsoft's mapping: Server 2019 supports api-version 5.0, Server 2020 6.0, Server 2022 7.0, Server 2022.1 7.1, and "Server vNext" 7.2. A version works on the listed Server and newer ones.
- Present at 5.0 on Server 2019: reporting work item revisions with a continuation token, work item updates, team iterations, pull request list by status with closed date and last merge commit, pull request commits, builds list with a minimum time and continuation.
- Gaps at 5.0: no time-range filter on pull requests, no merge strategy field (only a squash flag), state categories only in a preview version.
- Pipelines runs and Environments are preview on Server 2020.
- Rate limiting is documented for Services only.

Still unread, so still unverified: runtime version discovery, WIQL, classification nodes, commits and pushes, pull request iterations, build timeline, release deployments, environment deployment records, Server throttling, and whether "vNext" is the December 2025 release. Direct access to learn.microsoft.com and the MicrosoftDocs repositories was blocked from the research environment; only the Microsoft Learn documentation tool worked.
