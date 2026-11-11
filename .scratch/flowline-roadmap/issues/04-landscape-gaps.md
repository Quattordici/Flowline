# Landscape gaps

Type: research (AFK)
Status: resolved
Label: wayfinder:research

## Question

Close the unverified items from the landscape research: the current status and date of Apache DevLake issue 8985 and what the published DevLake docs say about Server; Azure DevOps Server support in LinearB, Swarmia, Jellyfish and Oobeya; MiddlewareHQ's main repository; Google Four Keys; Microsoft's own Power BI templates and DORA guidance; any other open source tool aimed at Server.

Findings file: [docs/research/04-landscape-gaps.md](../../../docs/research/04-landscape-gaps.md)

## Answer

2026-11-10. Detail, sources and confidence are in the findings file above.

- Apache DevLake merged on-premises Azure DevOps Server support on 2026-08-16 (pull request 9014, which closes issue 8985) and shipped it in tag v1.0.3-beta16. Issue 8985 can no longer be cited as evidence of a gap.
- DevLake's Azure DevOps plugin is documented as Beta, full refresh only, covering pipelines, repositories and code review. It does not cover Azure Boards work items.
- Oobeya, a commercial product, documents Server compatibility and on-premises deployment.
- Microsoft has lead time, cycle time, velocity and burndown widgets on Server, no native DORA dashboard, and none on its published roadmap.
- Google Four Keys is archived. MiddlewareHQ has no Azure DevOps support.
- The gap that still appears open is the combination: open source, Server, work item based flow metrics, SAFe and PI reporting, in a small deployment.

Still unverified, because the vendor sites were blocked from the research environment: Server support in LinearB, Swarmia and Jellyfish; Oobeya pricing and supported Server versions; Server compatibility of Marketplace extensions, including one that offers SAFe and PI features; the live DevLake documentation and which Server versions DevLake supports.
