# 04 Landscape gaps: verification pass

Research date: 2026-11-10. Every source below was read on that date.

## Short answer

The claimed gap (open source, plus on-premises Azure DevOps Server, plus SAFe/PI reporting, plus lightweight) holds only as a combination. One leg of it has weakened materially since the earlier landscape pass.

- The claim "no open source tool supports Azure DevOps Server" is no longer true. Apache DevLake merged on-premises Azure DevOps Server support on 2026-08-16 (PR #9014, closing issue #8985) and shipped it in tag v1.0.3-beta16 on 27 Aug 2026. The project must not cite issue #8985 as evidence of a gap.
- What still appears open: nothing found combines open source licensing, Azure DevOps Server support, work item (Boards) based flow metrics, and SAFe Program Increment reporting in a small deployment. DevLake's Azure DevOps plugin, per its docs source and README, covers CI/CD, repositories and code review, not Boards work items, and is listed as Beta with full refresh only.
- Commercial pressure is real. Oobeya documents Azure DevOps Server compatibility and an on-premises deployment. Several Marketplace extensions offer DORA or SAFe/PI features inside Azure DevOps, though Server compatibility for most could not be confirmed.
- Microsoft offers no native DORA dashboard and none is on the published roadmap. Built-in widgets cover lead time, cycle time, velocity, CFD and burndown, on Server as well as Services.

Honest positioning: "lightweight, open source, Server-first, Boards-aware flow and PI reporting" rather than "the only open source option for Server".

## Access limits that affected this pass

The session's network policy blocked these hosts (DNS resolution refused): devlake.apache.org, linearb.helpdocs.io, www.swarmia.com, help.swarmia.com, jellyfish.co, www.oobeya.io. The `gh` API was refused for apache/devlake. Reachable: github.com web pages, learn.microsoft.com (via the Microsoft Learn connector), and a web search tool.

Consequences:

- Items 1, 6, 7, 8 and the GitHub parts of item 9 rest on primary pages read directly.
- Items 2, 3, 4, 5 and the Marketplace parts of item 9 rest on web search summaries of the vendors' own pages. The URLs are primary, but the pages were not read in full by me. Confidence is marked down accordingly.
- GitHub pages were read through a fetch tool that summarises page content. Quoted strings are as reported by that tool.

## 1. Apache DevLake and Azure DevOps Server

Status: on-premises Azure DevOps Server support was merged and tagged. Published docs appear not to have caught up.

Issue #8985

- Title: "[Feature][azuredevops] Support Azure DevOps Server (On-Premises)".
- Created 2026-07-13. Closed as completed 2026-08-16 by its author.
- Label: type/feature-request. No milestone. No comments. A maintainer (klesh) assigned the issue to the author on 2026-07-15.
- The issue page's Development section shows no linked PR, but PR #9014 states "Closes #8985".
- Source: https://github.com/apache/devlake/issues/8985

PR #9014

- Title: "Azure DevOps On-Premises Support" (commit title "feat(azuredevops): add Azure DevOps Server (On-Premises) support (#9014)").
- Opened 2026-07-27, approved and merged 2026-08-16 by klesh into main, 11 checks passing.
- Scope per the description: adds `Endpoint` and `Username` to the `azuredevops_go` connection, base URL normalisation (custom domains, IPs, subpaths such as a collection path), a database migration, Basic auth header injection for `git clone` to avoid IIS NTLM challenges with PATs, and Config UI fields. Author states backward compatibility with cloud connections and verification against a live Azure DevOps Server instance, plus 12 unit tests.
- Not stated in the PR: which Azure DevOps Server versions are supported, which REST API version is used, known limitations.
- Source: https://github.com/apache/devlake/pull/9014

Release

- Tag v1.0.3-beta16, released 27 Aug (year not shown on the page; 2026 inferred from the merge date), lists "Azure DevOps On-Premises Support by ... in #9014".
- All recent tags are betas (latest seen: v1.0.3-beta19, 02 Nov). I did not find a non-beta release containing the change.
- Sources: https://github.com/apache/devlake/releases and https://github.com/apache/devlake/releases/tag/v1.0.3-beta16

Plugin source

- Go plugin, `backend/plugins/azuredevops_go/models/connection.go` on main: has an `Endpoint` field with the comment "Endpoint is the base URL for On-Premises Azure DevOps Server (optional, empty for Cloud)"; `GetEndpoint()` falls back to `https://dev.azure.com/`. Source: https://github.com/apache/devlake/blob/main/backend/plugins/azuredevops_go/models/connection.go
- Older Python plugin, `backend/python/plugins/azuredevops/azuredevops/api.py` on main: base URL still hardcoded to `https://dev.azure.com/`, with two hardcoded `app.vssps.visualstudio.com` URLs. Source: https://github.com/apache/devlake/blob/main/backend/python/plugins/azuredevops/azuredevops/api.py
- The Python plugin framework was deprecated (#9097) and that deprecation was reverted on 27 Oct 2026 (PR #9259), so both plugins exist on main. Source: https://github.com/apache/devlake/pulls (PR list)
- The `azuredevops_go` README lists PAT read scopes Build, Code, Graph and Release. It does not list the Work Items scope and does not mention boards or work items. Source: https://github.com/apache/devlake/blob/main/backend/plugins/azuredevops_go/README.md

Published docs

- devlake.apache.org was blocked. I read the docs source in the website repository instead. This is the source for the site, not proof of what is currently rendered.
- Supported Data Sources: Azure DevOps is marked Beta, domains "CI/CD, Source Code Management, Code Review", versions "Cloud", collection mode "Full Refresh"; the listed subtasks (builds, jobs, pull requests, pull request commits) are marked as not incremental. Source: https://github.com/apache/incubator-devlake-website/blob/main/docs/Overview/SupportedDataSources.md
- Azure DevOps configuration page: describes a connection name and token, no endpoint field, no mention of Server or on-prem. Source: https://github.com/apache/incubator-devlake-website/blob/main/docs/Configuration/AzureDevOps.md
- DORA page: four metrics (deployment frequency, lead time for changes, change failure rate, median time to restore service), plus a note that Failed Deployment Recovery Time replaces MTTR per the 2023 DORA report. So four at a time, with two alternatives for the fourth. No rework rate or reliability metric. Source: https://github.com/apache/incubator-devlake-website/blob/main/docs/DORA.md

Confidence: high for issue, PR, tag and source code. Medium for the docs statements (source repo read, live site not reachable; docs main branch may differ from the versioned docs shown on the site).

## 2. LinearB

Status: no explicit statement of Azure DevOps Server support found. Some features are documented as unsupported on Azure DevOps on-prem.

- LinearB's Git integration hub lists GitHub Enterprise Server, GitLab self-managed and Bitbucket Server explicitly, while Azure DevOps appears as "Azure DevOps Repos" without a deployment qualifier. Source: https://linearb.helpdocs.io/article/j2fsc8hphl-git-integration-hub
- Azure Repos and Azure Boards guides describe PAT based setup with no mention of on-prem. Sources: https://linearb.helpdocs.io/article/27ncmkugv7-azure-integration and https://linearb.helpdocs.io/article/ocpyz8m71d-connect-azure-dev-ops-using-a-personal-access-token-ikz
- The pricing FAQ is reported to say gitStream workflow automations are not supported on Azure DevOps and Bitbucket on-prem. Source: https://linearb.io/pricing
- A WorkerB page is reported to mark Azure DevOps on-prem as not supported for inline PR approval. Source: https://docs.linearb.io/workerb-inline-approve-pr/

The docs not mentioning Server is not evidence that Server is unsupported. The mentions of "Azure DevOps on-prem" in feature matrices suggest the vendor at least recognises such customers.

Confidence: low to medium. Search summaries only; help centre host blocked.

## 3. Swarmia

Status: Azure DevOps is supported in part. Server support not confirmed.

- Azure Boards integration announced in beta, changelog entry dated 2026-08-03; connects by PAT and organisation name; work items feed cycle time and flow efficiency. Sources: https://www.swarmia.com/changelog/2026-08-03-azure-boards/ and https://help.swarmia.com/settings/integrations/issue-trackers/azure-boards
- The integrations page is reported to list Azure DevOps as a code hosting platform (Azure Repos pull requests) and Azure Boards as Beta. Source: https://www.swarmia.com/integrations/
- Azure Pipelines: not mentioned in the results. Swarmia has a deployment API, which is a different thing. Source: https://help.swarmia.com/settings/organization/configuring-deployments-in-swarmia/generate-deployments-via-the-deployment-api
- Azure DevOps Server: not addressed. Swarmia's enterprise page is reported to mention an on-premise proxy agent for network restricted environments, without saying it covers Azure DevOps Server. Source: https://www.swarmia.com/enterprise/

This contradicts any earlier assumption that Swarmia is GitHub only. Setup by "organization name" suggests the cloud service, but that is an inference, not a documented exclusion of Server.

Confidence: medium for "Azure Boards beta exists", low for everything about Server. Search summaries only. Note: the search tool reported that the Swarmia help page contained embedded text instructing automated readers to make an HTTP request; it was ignored.

## 4. Jellyfish

Status: Azure DevOps supported. Server not mentioned.

- Jellyfish announced Azure DevOps support on 2025-10-21. Source: https://jellyfish.co/blog/ado/
- The integration pages are reported to cover Repos, Pipelines and Boards, feeding DORA metrics and delivery views. Sources: https://jellyfish.co/integration/azure-devops/ and https://jellyfish.co/integrations/
- No Jellyfish page found mentions Azure DevOps Server, on-premises or an agent. That is absence of a statement, not a statement of non-support.

Confidence: medium for cloud support, low for the Server question. Search summaries only; host blocked.

## 5. Oobeya

Status: the strongest commercial overlap found. Documents Server compatibility and on-premises deployment. Pricing is not public.

- Docs are reported to state the Azure DevOps addon is "compatible with both Azure DevOps Services (cloud) and Azure DevOps Server (TFS2017, TFS2018, TFS2019)". Azure DevOps Server 2020, 2022 and later are not named. The page appears to be old. Source: https://docs.oobeya.io/integrations/scm-addons/azure-devops-integration
- On-premises offering exists, with Docker and OpenShift installation guides (OpenShift minimum reported as 16 GB RAM, 8 CPU, 100 GB disk; Docker images pulled from a private registry with credentials supplied by the vendor). Sources: https://oobeya.io/on-premise, https://docs.oobeya.io/installations/installation-video, https://docs.oobeya.io/installations/oobeya-installation-tutorial-openshift
- Pricing page is reported to say pricing is tailored per organisation, with on-premise deployment listed on Starter, Enterprise and Enterprise AI tiers. No public price found. No free tier found. Source: https://www.oobeya.io/pricing
- Also sold through the Microsoft marketplace. Source: https://www.oobeya.io/microsoft-partner
- Proprietary, not open source. The reported hardware footprint is not lightweight.

Confidence: medium. Search summaries of vendor pages; hosts blocked. Whether a free tier or trial exists is unverified.

## 6. MiddlewareHQ (middlewarehq/middleware)

Status: active open source, no Azure DevOps support found.

- Licence Apache-2.0. Not archived. About 1.6k stars, 171 forks, 1,571 commits.
- Most recent commit on main: 2026-08-03 (a merge of PR #699). Before that 2026-07-08 and 2026-06-08. Activity is low but not zero. Source: https://github.com/middlewarehq/middleware/commits/main
- README covers the four DORA metrics from pull requests, deployments and incidents. It names no specific code hosts in the captured text and does not mention Azure DevOps. Source: https://github.com/middlewarehq/middleware
- An issue search for "azure devops" returned one unrelated closed issue (#628, Jira integration, 2025-04-08). No open request for Azure DevOps found. Source: https://github.com/middlewarehq/middleware/issues?q=azure+devops

Confidence: high for licence and activity, medium for "no Azure DevOps" (README and issue search only; the integrations code was not read).

## 7. Google Four Keys (dora-team/fourkeys)

Status: archived.

- "This repository was archived by the owner on Jan 23, 2024. It is now read-only." README adds that it is not currently maintained and suggests forking or using it as inspiration. No successor named.
- Licence Apache-2.0.
- Sources supported: Cloud Build and GitHub events, with GitLab referenced; other sources by extending `AUTHORIZED_SOURCES`. Azure DevOps not mentioned. Runs on Google Cloud.
- Source: https://github.com/dora-team/fourkeys

Confidence: high.

## 8. Microsoft's own offering

Status: flow widgets are built in on Server; no native DORA dashboard, no official DORA Power BI template found, nothing on the roadmap.

Built in, documented as applying to Azure DevOps Services, Azure DevOps Server and Azure DevOps Server 2022:

- Boards widgets: Lead Time, Cycle Time, Velocity, Cumulative Flow Diagram, Burndown, Burnup, Sprint Burndown. Lead and cycle time are work item based (created or started to completed), not commit to production. Sources: https://learn.microsoft.com/azure/devops/report/dashboards/widget-catalog?view=azure-devops and https://learn.microsoft.com/azure/devops/report/dashboards/cycle-time-and-lead-time?view=azure-devops
- Pipelines widgets: Build History, Deployment status, Release Pipeline Overview, Requirements quality. None computes deployment frequency, change failure rate or time to restore. Source: https://learn.microsoft.com/azure/devops/pipelines/reports/pipeline-widgets?view=azure-devops
- Pipeline reports: pass rate and duration, in product and as Power BI sample reports over the Analytics OData feed (`PipelineRuns`). Sources: https://learn.microsoft.com/azure/devops/report/powerbi/sample-pipelines-pass-rate-trend?view=azure-devops and https://learn.microsoft.com/azure/devops/server/release-notes/azuredevops2020?view=azure-devops
- Power BI: sample OData reports for lead and cycle time, CFD, rollup, burndown. Source: https://learn.microsoft.com/azure/devops/report/extend-analytics/quick-ref?view=azure-devops
- Limitation stated by Microsoft: "Analytics views only support Azure Boards data (work items)", not pipelines, and "there are no plans to update the connector to support other types of data". Source: https://learn.microsoft.com/azure/devops/report/powerbi/data-connector-connect?view=azure-devops
- SAFe: Microsoft publishes a guide to implementing SAFe with Azure Boards (PIs as iteration paths, rollup columns, velocity, CFD, lead and cycle time, Delivery Plans, Feature Timeline). No PI predictability, PI objectives or ART level flow report is described. Delivery Plans are built in from Server 2022. Sources: https://learn.microsoft.com/azure/devops/boards/plans/safe-configure-boards?view=azure-devops and https://learn.microsoft.com/azure/devops/boards/plans/?view=azure-devops

DORA guidance: Microsoft Learn mentions DORA metrics conceptually (platform engineering guide, Well-Architected Framework) but I found no Azure DevOps how-to, widget or Power BI template for DORA. Sources: https://learn.microsoft.com/platform-engineering/plan and https://learn.microsoft.com/azure/well-architected/operational-excellence/tools-processes

Roadmap: the Azure DevOps roadmap page lists no DORA or delivery metrics item for Services or Server. The Server section has only "Future" items; the Reporting items are "Dashboard Global Parameter" and "Dashboard Template". Source: https://learn.microsoft.com/en-us/azure/devops/release-notes/features-timeline

Confidence: high for what is documented. Medium for "no official DORA template exists" (two documentation searches and the roadmap page; a blog post or sample repository could exist outside Learn).

## 9. Other tools targeting Azure DevOps (Server) delivery metrics

Open source or source available, on GitHub:

| Tool | Licence | Activity | Server support | Notes | Source |
|---|---|---|---|---|---|
| Apache DevLake | Apache-2.0 | Very active (merges on 2026-11-10) | Yes since Aug 2026, beta tags | See item 1. Heavy stack; no Boards data for Azure DevOps | https://github.com/apache/devlake |
| DeveloperMetrics/DevOpsMetrics | MIT | 274 stars, updated within a day of reading | Not stated | Four DORA metrics from GitHub and Azure DevOps; deploys to Azure (web apps, functions, Key Vault, Storage); time to restore depends on Azure Monitor. No self-host path described | https://github.com/DeveloperMetrics/DevOpsMetrics |
| nbrown02/FlowViz | MIT | 246 stars, last commit date not shown | Yes, description names Azure DevOps Server and TFS | Power BI template for flow metrics and Monte Carlo forecasting. No DORA. Needs Power BI | https://github.com/nbrown02/FlowViz |
| LetPeopleWork/Lighthouse | Source available (custom licence); releases up to v26.9.9.9 were MIT | 18 stars, 12,425 commits | Not stated ("Azure DevOps" only) | Self-hosted flow metrics and forecasting, Docker or binaries. Free community edition with limits on projects and teams. No DORA, no SAFe terms | https://github.com/LetPeopleWork/Lighthouse |
| Codpal-Limited/deckgauge | FSL-1.1-ALv2 (converts to Apache-2.0 after two years) | 32 stars, updated within a week | Not stated | Self-hosted via Docker Compose; Jira, GitHub, GitLab, Bitbucket, Azure DevOps; DORA and flow widgets. Not OSI open source today | https://github.com/Codpal-Limited/deckgauge |
| adam-wheater/azure-devops-delivery-insights | Proprietary, source visible | 0 stars, 1 commit | Not stated | Azure DevOps extension, four DORA metrics, uses retention leases to detect production deploys | https://github.com/adam-wheater/azure-devops-delivery-insights |

A GitHub repository search for "azure devops" dora metrics returned 14 results, the rest being zero star personal projects. Source: https://github.com/search?q=%22azure+devops%22+dora+metrics&type=repositories&s=updated

Visual Studio Marketplace extensions (from search summaries of the listings; listings not read directly, licences are proprietary unless a listing says otherwise):

| Extension | What | Server support as reported | Source |
|---|---|---|---|
| Velo (Ingenious Labs) | DORA tile widget, five metrics, free, listed about three months before reading | Not stated | https://marketplace.visualstudio.com/items?itemName=IngeniousLabs.velo |
| DORA Metrics Dashboard (QuantumEdge Solutions) | DORA hub under Pipelines, some views behind a Pro tier | Not stated | https://marketplace.visualstudio.com/items?itemName=QuantumEdgeSolutions.dora-metrics-dashboard |
| Devops Performance Metrics (Falck) | Widget, three of four DORA metrics from a release definition, about 1,969 installs, listing roughly six years old | Not stated | https://marketplace.visualstudio.com/items?itemName=FalckDevOps.DevopsPerformanceMetrics |
| Agile Analytics (Baytek) | Flow metrics, DORA view, sprint and exec reports | Listing reported to say Azure DevOps Server is not supported | https://marketplace.visualstudio.com/items?itemName=Baytek.agile-analytics |
| Nooga Scale | SAFe: PI planning, roadmaps, flow metrics, dependencies, objectives, WSJF. Paid with trial, about 2,171 installs | Not stated | https://marketplace.visualstudio.com/items?itemName=nooga.nooga-scale |
| Great Gadgets (StonikByte) | Throughput, cycle time, WIP aging, CFD; mentions SAFe | Not verified | https://marketplace.visualstudio.com/items?itemName=StonikByte.stonikbyte-great-gadgets-extension |
| Honest Cheetah (Benjamin Day Consulting) | Throughput and Monte Carlo forecasting | Listing reported to require Azure DevOps Server 2019 or later; a second search did not reconfirm | https://marketplace.visualstudio.com/items?itemName=bendayconsulting.honest-cheetah |
| ActionableAgile Analytics (55 Degrees) | Paid flow analytics | Listing reported as for Azure DevOps Services | https://marketplace.visualstudio.com/items?itemName=actionableagile.aa-analytics-paid |
| DORA Metrics (Burhan Ocut), DevOps Metrics (Solidify) | DORA widgets | Reported as unpublished from the Marketplace | https://marketplace.visualstudio.com/items?itemName=BurhanOcut.dora-metrics and https://marketplace.visualstudio.com/items?itemName=solidify.devops-metrics |

Other commercial products seen in passing with Azure DevOps DORA features, Server support not checked: Harness SEI, CodeAnt AI, SquaredUp, Faros AI.

Confidence: high for the GitHub rows, low for the Marketplace rows.

## Still unverified

1. Whether devlake.apache.org as rendered today documents the on-prem endpoint for Azure DevOps (site blocked; the docs source main branch still says "Cloud").
2. Which Azure DevOps Server versions DevLake's on-prem support works with, and whether it has been used beyond the contributor's own instance. No stable (non-beta) release containing it was found.
3. Whether DevLake collects Azure Boards work items in any form. The README scopes and data source table suggest not, but the collector code was not read.
4. LinearB, Swarmia, Jellyfish: whether Azure DevOps Server is supported. None of their pages, as summarised, says yes or no. Requires reading the help centres directly or asking the vendors.
5. Oobeya: current supported Server versions (docs name only TFS 2017 to 2019), whether any free tier or trial exists, and actual pricing.
6. Server compatibility of every Marketplace extension in item 9. The Marketplace "Works with" field was not read for any of them. Nooga Scale is the most important one to check, since it is the closest SAFe/PI competitor.
7. MiddlewareHQ's supported code hosts (README text captured did not name them).
8. Faros Community Edition: the repository URL tried returned 404; status unknown.
9. Whether Microsoft or a Microsoft employee has published an unofficial DORA Power BI template outside Learn.

To close items 1, 4, 5 and 6, the hosts devlake.apache.org, linearb.helpdocs.io, help.swarmia.com, www.swarmia.com, jellyfish.co, www.oobeya.io, docs.oobeya.io and marketplace.visualstudio.com need to be allowed in the session's network settings.

## What the evidence does not show

- It does not show that no competitor supports Azure DevOps Server. For LinearB, Swarmia and Jellyfish the evidence is silence in summarised pages, which is not the same as non-support.
- It does not show that DevLake's Server support is production quality, or that it is poor. It shows a merged PR, one contributor's testing claim, unit tests and a beta tag.
- It does not show that DevLake cannot do flow or PI reporting on Azure DevOps. It shows the plugin's documented domains exclude issue tracking; DevLake's generic webhook or customize plugins could be used to load work items, and that was not investigated.
- It does not show demand. Issue #8985 had no comments or reactions reported, from one requester who then implemented it. That is weak evidence of how many Server users want this.
- It does not show that the built-in Azure DevOps widgets are insufficient for SAFe reporting in practice, only that Microsoft's guide describes no PI predictability or ART level report.
- "Lightweight" was not measured for any tool. Oobeya's reported minimum (16 GB RAM) and DevLake's multi-service stack are indications, not benchmarks.
- Marketplace install counts and listing ages come from search summaries and may be stale.
