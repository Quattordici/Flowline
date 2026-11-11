# Analytics and OData on Azure DevOps Server

Type: research (AFK)
Status: resolved
Label: wayfinder:research

## Question

Is the Analytics service and its OData endpoint available, and does it behave consistently, on Azure DevOps Server 2019, 2020, 2022, the current continuously updated release, and Services? Which entity sets and endpoint versions exist per version, is it on by default, and what are its limits?

Findings file: [docs/research/01-analytics-odata-on-server.md](../../../docs/research/01-analytics-odata-on-server.md)

## Answer

2026-11-10. From Microsoft documentation only; nothing was tested against a live server. Detail and sources are in the findings file above.

- Analytics OData exists on every version in scope, but not uniformly.
- Server 2019: preview. The first release needs the Analytics extension installed per collection; 2019.1 has it built in but an administrator must enable it. Only work tracking entity sets are documented, with no pipeline data.
- Server 2020, Server 2022 and the current continuously updated release: generally available and on automatically for new collections. Upgraded collections may need it enabled by hand. An administrator can pause or disable it, and disabling deletes the Analytics history.
- Services: always on, on a separate host.
- The released OData versions (v1.0, v2.0) cover work tracking only. Pipeline and test entity sets exist only in preview versions.
- CycleTimeDays, LeadTimeDays and StateCategory are documented from 2019 onward with the same definitions.

Still unverified: whether 2019 serves the preview versions, whether the metadata is identical across 2020, 2022 and the current release, whether classic pipelines populate pipeline runs, and timeout, throttling and latency figures on Server.
