# 01: Azure DevOps Analytics and OData availability on Server and Services

Research date: 2026-11-10

## Question

Is the Azure DevOps Analytics service and its OData endpoint available, and does it behave consistently, on each of these: Azure DevOps Server 2019 (and its updates), Azure DevOps Server 2020, Azure DevOps Server 2022 (and updates), the current unversioned continuously updated Azure DevOps Server release (GA December 2025), and Azure DevOps Services (cloud)?

## Short answer

Analytics with an OData endpoint exists on every version in scope, but it is not uniform.

- **Server 2019 is the outlier.** Analytics is a preview there. On 2019 RTM it is a Marketplace extension an admin must install per collection. On 2019 Update 1 it is built in but an admin must enable it per collection for existing collections. Microsoft documents only work tracking entity sets for 2019 OData. Pipeline OData is listed as not available on 2019.
- **Server 2020, Server 2022 and the current Azure DevOps Server release** have Analytics generally available and enabled automatically for new collections. Collections upgraded from an earlier version may need an admin to enable it. A collection admin can pause or disable it on all Server versions, and disabling deletes the Analytics data. Pipeline and test entity sets are documented for these versions, but only under the preview OData versions (v3.0-preview, v4.0-preview).
- **Services** has Analytics always on. It cannot be paused or disabled. It uses a separate host (analytics.dev.azure.com).
- Across all versions the released OData versions are v1.0 and v2.0 and they cover work tracking only. Everything about pipelines and tests sits behind a "-preview" version that Microsoft says can have breaking changes.
- `CycleTimeDays`, `LeadTimeDays`, `InProgressDate`, `CompletedDateSK` and `StateCategory` are documented for the 2019 moniker and for current monikers with the same definitions.
- A tool that depends on Analytics must handle "not installed", "not enabled", "paused" and "disabled, history deleted" as real states on Server, and must handle the absence of pipeline entity sets on 2019.

Nothing here was tested against a live server. All claims come from Microsoft documentation and release notes. See "Unverified" for what the documentation does not settle.

## How to read the sources

Microsoft Learn pages are versioned by a `view=` moniker. The monikers seen during this research:

| Moniker | Page header it produced | Used here for |
| --- | --- | --- |
| `azure-devops-2019` | "Azure DevOps Server 2019" | Server 2019 and 2019 Update 1 |
| `azure-devops-2020` | includes "Azure DevOps Server 2020" | Server 2020 |
| `azure-devops-2022` | "Azure DevOps Server 2022" | Server 2022 and updates |
| `azure-devops-server` | "Azure DevOps Server" (no year) | the unversioned release, GA 2025-12-09 |
| `azure-devops` | "Azure DevOps Services" | cloud |

Several current pages carry one shared text block for `azure-devops-2022` and `azure-devops-server`, so claims for those two are usually the same sentence. The mapping of the `azure-devops-server` moniker to the December 2025 release is inferred from the header text and the release notes page. It is not stated in one sentence anywhere I found (confidence: medium).

Confidence levels used below: **High** means stated directly in a primary source for that version. **Medium** means stated in a primary source but in shared or loosely scoped text, or inferred from two sources. **Low** means the documentation is ambiguous or contradicts itself.

## Per-version table

| Topic | Server 2019 RTM (2019.0.x) | Server 2019 Update 1 (2019.1.x) | Server 2020 | Server 2022 (and .1, .2) | Azure DevOps Server (unversioned, GA 2025-12-09) | Services (cloud) |
| --- | --- | --- | --- | --- | --- | --- |
| Status | Public preview | Preview | Generally available | Generally available | Generally available | Generally available |
| How it gets turned on | Admin installs the Analytics extension from the local Marketplace, per collection | Built in. Admin selects "Enable Analytics" in collection settings. On by default for new collections and for upgraded collections that had the extension | Automatic for new collections. Upgraded collections may need manual enable | Same as 2020 | Same as 2020 | Always on |
| Can admin pause or disable | Disable or uninstall the extension. Uninstall deletes Analytics data | Pause, disable, resume. Disable deletes Analytics data | Pause, disable, resume. Disable deletes data | Pause, disable, resume. Disable deletes data | Pause and disable still exist (release notes mention both states) | No. Cannot be paused or disabled |
| OData versions listed in docs | v1.0, v2.0, v3.0-preview, v4.0-preview (see Unverified) | Same listing | v1.0, v2.0, v3.0-preview, v4.0-preview | Same | Same | Same |
| URL shape | `https://{server}:{port}/tfs/{Collection}/{Project}/_odata/{version}/` | Same | Same | Same | Same | `https://analytics.dev.azure.com/{Org}/{Project}/_odata/{version}/` |
| Work tracking entity sets (WorkItems, WorkItemRevisions, WorkItemSnapshot, WorkItemBoardSnapshot, Iterations, Areas, Teams) | Documented, all OData versions | Documented | Documented | Documented | Documented | Documented |
| Pipeline entity sets (PipelineRuns, PipelineRunActivityResults and others) | Not documented. Data table says no Pipelines OData | Not documented | Documented, v3.0-preview and v4.0-preview only | Same as 2020 | Same as 2020 | Same as 2020 |
| Test entity sets (TestRuns, TestResults, TestResultsDaily, TestPoints and others) | Not in the 2019 data model page (see Unverified) | Not in the 2019 data model page | Documented, v3.0-preview and v4.0-preview only | Same | Same | Same |
| CycleTimeDays, LeadTimeDays, StateCategory | Documented | Documented | Documented | Documented | Documented | Documented |
| ParentWorkItemId | Not available (docs say 2020 and later) | Not available | Available | Available | Available | Available |
| Auth | Windows auth, PAT | Windows auth, PAT | Windows auth, PAT | Windows auth, PAT | Windows auth, PAT | Microsoft Entra ID, PAT |
| Legacy SSRS and warehouse reporting | Supported alongside Analytics | Supported | Supported, final version | Removed. Analytics only | Analytics only | Never supported. Analytics only |

## Details

### 1. Installation, enablement, pause and disable

**Server 2019 RTM**

- Analytics shipped as a Marketplace extension in public preview. "The Analytics Marketplace extension is now available for Azure DevOps Server... Analytics is in Public Preview. It currently contains data for Boards and Automated Test results via Pipelines." Source: Server 2019 release notes, https://learn.microsoft.com/en-us/azure/devops/server/release-notes/azuredevops2019 . Confidence: High.
- "For Azure DevOps Server 2019, you install the Analytics extension from your local marketplace for each project collection you want to use Analytics." The extension ships inside the on-premises installer, so no internet access is needed. Source (moniker azure-devops-2019): https://learn.microsoft.com/en-us/azure/devops/report/dashboards/analytics-extension?view=azure-devops-2019 . Confidence: High.
- Disabling the extension stops jobs and keeps data. Uninstalling it deletes the data from the Analytics tables, and reinstalling repopulates from scratch. Same source. Confidence: High.
- Known issues listed for 2019.0.1: "Analytics reports may show incomplete data because the model is marked as 'ready' before it is actually complete" and "A hold may be placed on Analytics data ingestion while doing maintenance which can cause stale reports." Source: https://learn.microsoft.com/en-us/azure/devops/server/release-notes/azuredevops2019 . Confidence: High.
- "The Analytics Marketplace extension and Analytics are not supported for TFS 2018 or earlier versions." Source: https://learn.microsoft.com/en-us/azure/devops/report/dashboards/analytics-extension?view=azure-devops-2019 . Confidence: High.

**Server 2019 Update 1**

- "Customers no longer need the Analytics extension to use Analytics. Customers can now enable Analytics underneath Project Collection Settings... New collections created in Update 1 and Azure DevOps Server 2019 collections with the Analytics extension installed that were upgraded will have Analytics enabled by default." Source: Server 2019 Update 1 release notes (Update 1 released 2019-08-20), https://learn.microsoft.com/en-us/azure/devops/server/release-notes/azuredevops2019u1 . Confidence: High.
- Pause, disable and resume exist from 2019.1. "Disabling Analytics in Azure DevOps Server 2019.1 is the same as uninstalling the Analytics extension for Azure DevOps Server 2019." Requires membership of Project Collection Administrators. Source (azure-devops-2019): https://learn.microsoft.com/en-us/azure/devops/report/dashboards/analytics-extension?view=azure-devops-2019 . Confidence: High.
- Still described as preview: "The Analytics service is in preview for Azure DevOps Server 2019... Power BI integration and access to the OData feed of the Analytics Service are in Preview." Source (azure-devops-2019): https://learn.microsoft.com/en-us/azure/devops/report/powerbi/what-is-analytics?view=azure-devops-2019 . Confidence: High.

**Server 2020, Server 2022, Azure DevOps Server (unversioned)**

- "For Azure DevOps Server 2020 and later versions, the Analytics service is generally available and automatically enabled for all new project collections added to your server. For project collections upgraded from a previous version, you might need to manually enable it." Sources: moniker azure-devops-2020, https://learn.microsoft.com/en-us/azure/devops/report/dashboards/analytics-extension?view=azure-devops-2020 ; the same sentence under monikers azure-devops-2022 and azure-devops-server, https://learn.microsoft.com/en-us/azure/devops/report/dashboards/analytics-extension?view=azure-devops-server . Confidence: High.
- "You can pause and restart the service. When paused, no new data is added to Analytics." Source (azure-devops-2022, azure-devops-server): https://learn.microsoft.com/en-us/azure/devops/report/analytics/analytics-permissions-prerequisites?view=azure-devops-server . Confidence: High.
- Effects of pause or disable: Analytics widgets, in-context reports, Analytics views and "OData endpoint queries" are all disabled. Pause keeps data and stops staging jobs. Disable "turns off all Analytics staging jobs and deletes the Analytics data... All Analytics data that captures historical trends is lost... You can't undo this action." Source: https://learn.microsoft.com/en-us/azure/devops/report/dashboards/analytics-extension?view=azure-devops-server (same text at view=azure-devops-2020 and view=azure-devops-2019). Confidence: High.
- After enabling, "Analytics staging jobs begin to populate the Analytics data tables. This process could take some time depending on the size of your collection." Same source. Confidence: High.
- The unversioned release still has the paused and disabled states. The 2025-12-09 RTW notes list a fix for the Agent Pool page failing "when Analytics was paused or disabled". Source: https://learn.microsoft.com/en-us/azure/devops/server/release-notes/azuredevopsserver . Confidence: High that the states exist. Medium that the admin UI steps are unchanged.
- Server 2022 Update 1: "Analytics Views are now available... to all project collections." This concerns Analytics views for Power BI, not the OData endpoint. Source: https://learn.microsoft.com/en-us/azure/devops/server/release-notes/azuredevops2022u1 . Confidence: High.

**Services**

- "For Azure DevOps Services, Analytics is always on. You can't disable it or pause it." Source (moniker azure-devops zone of the page): https://learn.microsoft.com/en-us/azure/devops/report/analytics/analytics-permissions-prerequisites?view=azure-devops . Confidence: High.

### 2. OData endpoint versions and URL shape

- Supported version strings on every moniker: "The API version follows the `_odata` element in the request path and can be one of the supported versions: v1.0, v2.0, v3.0-preview, or v4.0-preview." Sources: azure-devops-2019, https://learn.microsoft.com/en-us/azure/devops/report/extend-analytics/odata-api-version?view=azure-devops-2019 ; azure-devops, azure-devops-2022, azure-devops-server, https://learn.microsoft.com/en-us/azure/devops/report/extend-analytics/odata-api-version?view=azure-devops . Confidence: High for 2022, unversioned Server and Services. Low for 2019, see Unverified.
- "The latest supported version of the OData API is v2.0, and the latest preview version is v4.0-preview." Same sources. Confidence: High.
- v1.0 and v2.0 are "Released" and "don't include breaking changes". v3.0-preview and v4.0-preview "might include breaking changes and aren't guaranteed to have the same features in the final release". Released versions are supported "for a minimum of 12 months". A deprecated version returns HTTP 410. Same sources. Confidence: High.
- Custom fields are not versioned: "Removing or changing the types of work items or custom fields can cause breaking changes to your model." Same sources. Confidence: High.
- What each version covers: v1.0 "Azure Boards only excluding Process data", v2.0 "Azure Boards only including Process data", v3.0-preview "Azure Boards, Test Plans, and Azure Pipelines", v4.0-preview "v3.0-preview plus snapshots related to pipeline jobs, task agent pool size, and task agent requests". Source: https://learn.microsoft.com/en-us/azure/devops/report/powerbi/data-available-in-analytics?view=azure-devops-2019 . Confidence: High.
- Server URL shape, version page: `https://{servername}:{port}/tfs/{CollectionName}/{ProjectName}/_odata/{version}/$metadata`. Sources: azure-devops-2019 and azure-devops-2022/azure-devops-server zones of the odata-api-version page above. Confidence: High.
- Server URL shape, query tutorial: `https://{ServerName}/{CollectionName}/{ProjectName}/_odata/version/$metadata`, with the example `https://fabrikam-devops/DefaultCollection/_odata/v4.0-preview/$metadata`. This form has no `/tfs` segment and no port. The two pages differ, so the virtual directory and port depend on the deployment. Source: https://learn.microsoft.com/en-us/azure/devops/report/analytics/analytics-query-parts?view=azure-devops-2019 . Confidence: High that both forms are documented. Medium on which a given server uses.
- Services URL shape: `https://analytics.dev.azure.com/{OrganizationName}/{ProjectName}/_odata/{version}/$metadata`. Sources: both pages above. Confidence: High.
- The project segment is optional on both platforms. Omitting it gives a collection-scoped (Server) or organization-scoped (Services) endpoint. Source: https://learn.microsoft.com/en-us/azure/devops/report/analytics/analytics-query-parts?view=azure-devops-2019 . Confidence: High.
- Entity sets "vary by project" and the authoritative list for an instance is its own `$metadata`. Source: https://learn.microsoft.com/en-us/azure/devops/report/extend-analytics/data-model-analytics-service?view=azure-devops . Confidence: High.

### 3. Entity sets

**Work tracking (all versions in scope)**

- Areas, Iterations, BoardLocations, Dates, Projects, Tags, Teams, Users, WorkItemBoardSnapshot, WorkItemLinks, WorkItemRevisions, WorkItemSnapshot, WorkItems, WorkItemTypeFields are listed for v1.0, v2.0, v3.0-preview and v4.0-preview. Processes is listed from v2.0. Sources: azure-devops-2019, https://learn.microsoft.com/en-us/azure/devops/report/extend-analytics/data-model-analytics-service?view=azure-devops-2019 ; azure-devops-2020, https://learn.microsoft.com/en-us/azure/devops/report/extend-analytics/data-model-analytics-service?view=azure-devops-2020 ; azure-devops, azure-devops-2022, azure-devops-server, https://learn.microsoft.com/en-us/azure/devops/report/extend-analytics/data-model-analytics-service?view=azure-devops . Confidence: High.
- WorkItems is "the current state of work items". WorkItemRevisions is "all historic work item revisions, including the current revision. Does not include deleted work items." WorkItemSnapshot and WorkItemBoardSnapshot are composite daily snapshots. WorkItemLinks "includes only the latest revision of links, no history". Same sources. Confidence: High.
- Analytics does not store HTML or rich text fields (Description, History) and does not store link or attachment counts. Sources: https://learn.microsoft.com/en-us/azure/devops/report/analytics/entity-reference-boards?view=azure-devops-2019 and https://learn.microsoft.com/en-us/azure/devops/report/analytics/entity-reference-boards?view=azure-devops-server . Confidence: High.
- `ParentWorkItemId`: "The Parent field is available from Azure DevOps Server 2020 and later versions." Source (azure-devops-2019): https://learn.microsoft.com/en-us/azure/devops/report/analytics/entity-reference-boards?view=azure-devops-2019 . Confidence: High.
- Custom fields appear automatically as properties prefixed `Custom_`. Same sources. Confidence: High.

**Pipelines**

- Server 2020, Server 2022, unversioned Server, Services: Branches, Pipelines, PipelineJobs, PipelineRuns, PipelineRunActivityResults, PipelineTasks are listed for v3.0-preview and v4.0-preview. ParallelPipelineJobsSnapshot, TaskAgentPoolSizeSnapshots, TaskAgentRequestSnapshots are v4.0-preview only. None are in v1.0 or v2.0. Sources: the azure-devops-2020 and azure-devops data model pages above. Confidence: High.
- Server 2019: the azure-devops-2019 rendering of the data model page contains only the work tracking table. The pipeline and test sections are absent. Source: https://learn.microsoft.com/en-us/azure/devops/report/extend-analytics/data-model-analytics-service?view=azure-devops-2019 . Confidence: High that they are undocumented for 2019.
- Server 2019: the platform table lists Pipelines on 2019 as "Test Analytics" only, while Services, 2020 and 2022 get "Test Analytics, Pipeline Analytics, OData". Test Plans on 2019 is "None". Source: https://learn.microsoft.com/en-us/azure/devops/report/powerbi/data-available-in-analytics?view=azure-devops-2019 (table dated August 2022). Confidence: High.
- Pipeline failure and duration reports (the in-product pipeline Analytics) arrived in Server 2020. Source: https://learn.microsoft.com/en-us/azure/devops/server/release-notes/azuredevops2020 . Confidence: High.
- PipelineRun properties include `PipelineRunId`, `QueuedDate`, `StartedDate`, `CompletedDate`, `QueueDurationSeconds`, `RunDurationSeconds`, `TotalDurationSeconds`, `RunOutcome` (Succeed, PartiallySucceeded, Failed, Canceled), `RunReason`, with navigation to Branch and Pipeline. PipelineRunActivityResults is a "merged log of all the stages, steps, jobs, and tasks within a specific pipeline execution". Source (azure-devops-server): https://learn.microsoft.com/en-us/azure/devops/report/analytics/entity-reference-pipelines?view=azure-devops-server . Confidence: High.
- The prerequisites page says pipeline reporting needs teams to "Define pipelines using YAML". Whether classic build and classic release pipelines populate PipelineRuns is not stated. Source: https://learn.microsoft.com/en-us/azure/devops/report/analytics/analytics-permissions-prerequisites?view=azure-devops-server . Confidence: Medium. See Unverified.
- Repos and Artifacts have no Analytics data on any version ("None", "Under investigation"). There is no pull request entity set. Source: https://learn.microsoft.com/en-us/azure/devops/report/powerbi/data-available-in-analytics?view=azure-devops-2019 . Confidence: High as of the table date (August 2022). Medium for today.

**Tests**

- Server 2020, Server 2022, unversioned Server, Services: TestConfigurations, TestResults, TestResultsDaily, TestRuns, Tests, TestPoints, TestPointHistorySnapshots, TestSuites, for v3.0-preview and v4.0-preview only. Sources: the azure-devops-2020 and azure-devops data model pages above. Confidence: High.
- Server 2019: not listed in the 2019 data model page. The 2019 release notes say Analytics "contains data for Boards and Automated Test results via Pipelines", which is why the in-product Test Analytics tab works, but no 2019 page documents a test OData entity set. Confidence: Low on whether they are queryable. See Unverified.

### 4. CycleTimeDays, LeadTimeDays and state categories

- The properties are documented for the entity types WorkItem, WorkItemRevision, WorkItemSnapshot and WorkItemBoardSnapshot on both the 2019 moniker and current monikers. Sources: https://learn.microsoft.com/en-us/azure/devops/report/analytics/entity-reference-boards?view=azure-devops-2019 and https://learn.microsoft.com/en-us/azure/devops/report/analytics/entity-reference-boards?view=azure-devops-server . Confidence: High.
- `CycleTimeDays` (Double): "Cycle time calculated from first entering an In Progress or Resolved state category to entering a Completed state category." Same sources. Confidence: High.
- `LeadTimeDays` (Double): "Lead time calculated from work item creation or entering a Proposed state category to entering a Completed state category." Same sources. Confidence: High.
- `InProgressDate` and `InProgressDateSK`: when the item "was moved into a state that belongs to the In Progress state category". `CompletedDateSK` and `CompletedOn`: when the item "entered a workflow state associated with the Completed state category". A `CompletedDate` property appears in the WorkItem metadata sample. Sources: the entity reference pages above and https://learn.microsoft.com/en-us/azure/devops/report/analytics/analytics-query-parts?view=azure-devops-2019 . Confidence: High.
- `StateCategory` (String): values Proposed, In Progress, Resolved, Removed, Completed. Both the 2019 and current pages say it is "Valid only for the WorkItemRevision entity type". Same entity reference sources. Confidence: High that the docs say this. Low that it is accurate for WorkItems, see Unverified.
- Reactivated items: "cycle time starts from the first time the work item enters an In Progress or Resolved state category and ends the final time it enters a Completed state category. Cycle time includes the entire active work period, including any time after reactivation." Lead time "covers the entire period from work item creation to final completion, regardless of any intermediate completed states". Source (azure-devops, azure-devops-2022, azure-devops-server): https://learn.microsoft.com/en-us/azure/devops/report/dashboards/cycle-time-and-lead-time?view=azure-devops-server . Confidence: High for current versions. Medium for 2019 and 2020, where I did not find the same paragraph under those monikers.
- The widget page defines lead time simply as creation to Completed, while the entity reference says "creation or entering a Proposed state category". The two wordings are not reconciled by Microsoft. Confidence: High that the wording differs.
- Time zone: date surrogate keys are "expressed as YYYYMMDD in the time zone defined for the organization". Source: the entity reference pages above. Confidence: High.

### 5. Limits

**Latency and refresh**

- "Analytics is not a real-time store. Copying the data introduces up to a 30-second delay before the data associated with any one change shows up in Analytics." Source (page header lists Services, Server, Server 2022; the section is not version gated): https://learn.microsoft.com/en-us/azure/devops/report/powerbi/what-is-analytics?view=azure-devops-server . Also in the reporting roadmap, https://learn.microsoft.com/en-us/azure/devops/report/powerbi/reporting-roadmap?view=azure-devops . Confidence: High that it is the documented figure. Medium that it holds on self-hosted hardware, since nothing separates Server from Services.
- Snapshots: "A snapshot writes to Analytics at the same time each day". Source: https://learn.microsoft.com/en-us/azure/devops/report/analytics/entity-reference-boards?view=azure-devops-server . Confidence: High.
- The unversioned Server release notes say of backlog rollup columns that "some delay in Analytics processing is normal" and that a last refreshed timestamp was added. Source: https://learn.microsoft.com/en-us/azure/devops/server/release-notes/azuredevopsserver . Confidence: High.
- Query response target: "Analytics responds to any aggregation or non-aggregated query within 3 to 5 seconds" when recommended patterns are used. Source: the what-is-analytics page above. Confidence: Medium (a design goal, not a guarantee).

**Query size and shape**

- Server-driven paging: "Analytics forces paging when query results exceed 10000 records" and returns `@odata.nextLink`. Sources: https://learn.microsoft.com/en-us/azure/devops/report/analytics/analytics-query-parts?view=azure-devops-2019 and https://learn.microsoft.com/en-us/azure/devops/report/analytics/analytics-query-parts?view=azure-devops . Confidence: High.
- "DON'T use `$top` and `$skip` query options to implement client-driven paging." Follow `@odata.nextLink` instead. Source: https://learn.microsoft.com/en-us/azure/devops/report/extend-analytics/odata-query-guidelines?view=azure-devops-server . Confidence: High.
- GET length: "The current limit of OData queries sent with HTTP GET is 3,000 characters. If you exceed it, you get back a 404 Not Found response." Longer queries go through the OData `$batch` endpoint as a POST, and a batch may hold only one query. Same source. Confidence: High.
- Column limit: queries returning more than 800 columns are blocked with error VS403670. Same source. Confidence: High.
- Snapshot entity sets are blocked for anything other than aggregation: "The Snapshot tables in Analytics are intended to be used only in an aggregation." Same source. Confidence: High.
- Blocked or disallowed: entity keys in resource paths, `$expand` of `Revisions` on WorkItem, `countdistinct`, grouping on distinct columns. Same source. Confidence: High.
- Historical queries on WorkItemRevisions or WorkItemSnapshot should filter on `RevisedDateSK`. Same source. Confidence: High.
- Omitting `$select` or `$apply` returns warning VS403507. Source: https://learn.microsoft.com/en-us/azure/devops/report/analytics/analytics-query-parts?view=azure-devops-2019 . Confidence: High.
- Timeouts surface as "TF400733: The request has been canceled: The request has exceeded request timeout, try again." No numeric timeout value is given. Source: the query guidelines page above. Confidence: High on the error, see Unverified for the number.
- Usage limits: the guidelines say heavy use can get a client "temporarily blocked" and link to the rate limits page. That page is headed "Azure DevOps Services" only and describes the 200 TSTU per five minutes limit for the cloud. Source: https://learn.microsoft.com/en-us/azure/devops/integrate/concepts/rate-limits?view=azure-devops . Confidence: High for Services. No documented equivalent found for Server.

**Authentication**

- Server (azure-devops-2022, azure-devops-server): Windows authentication is the documented default for OData clients. A PAT can be used with Basic auth, with the PAT as the password and scope "Analytics (read)" (`vso.analytics`). Microsoft labels the PAT option "not recommended". Source: https://learn.microsoft.com/en-us/azure/devops/report/powerbi/client-authentication-options?view=azure-devops-server . Confidence: High. The page is written for Power BI and Excel, so applying it to a custom HTTP client is an inference (Medium).
- "OAuth 2.0 and Microsoft Entra ID authentication are available for Azure DevOps Services only, not Azure DevOps Server. For on-premises scenarios, use .NET client libraries, Windows authentication, or personal access tokens." Source: https://learn.microsoft.com/en-us/azure/devops/integrate/get-started/authentication/authentication-guidance?view=azure-devops . Confidence: High.
- Services: Microsoft Entra ID is recommended, PAT with `vso.analytics` is the fallback. Source: the client-authentication-options page, azure-devops zone. Confidence: High.

**Permissions**

- "To use Power BI for Azure DevOps or to exercise an OData query for Analytics, have the View analytics permission." It is a project-level permission, granted by default to Readers, Contributors and Project Administrators. Sources (azure-devops, azure-devops-2022, azure-devops-server): https://learn.microsoft.com/en-us/azure/devops/report/powerbi/analytics-security?view=azure-devops-server and https://learn.microsoft.com/en-us/azure/devops/report/analytics/analytics-permissions-prerequisites?view=azure-devops-server . Confidence: High.
- Access level: project member with at least Basic access. Stakeholders have no access to Analytics views. Sources: same, plus https://learn.microsoft.com/en-us/azure/devops/report/analytics/analytics-query-parts?view=azure-devops-2019 for the 2019 moniker. Confidence: High.
- "Analytics does not support security at the area path level." A user with project access can see data for areas they cannot open as work items. Source: https://learn.microsoft.com/en-us/azure/devops/report/powerbi/analytics-security?view=azure-devops-server . Confidence: High.
- A collection-scoped or organization-scoped query that touches a project the caller cannot read returns a "Project access denied" error and no data, not a trimmed result. Microsoft recommends a project filter or the project-scoped endpoint. Same source. Confidence: High.
- Enabling, pausing, disabling or resuming Analytics on Server needs Project Collection Administrators membership. Source: https://learn.microsoft.com/en-us/azure/devops/report/dashboards/analytics-extension?view=azure-devops-server . Confidence: High.

### 6. Microsoft's position on Analytics as the reporting path

- "Analytics is the reporting platform for Azure DevOps, replacing the previous platform based on SQL Server Reporting Services." Sources: https://learn.microsoft.com/en-us/azure/devops/report/powerbi/what-is-analytics?view=azure-devops-2019 and the same page at view=azure-devops-server. Confidence: High.
- "Analytics is the future of reporting for Azure DevOps and Azure DevOps Server." Source: Server 2019 release notes, https://learn.microsoft.com/en-us/azure/devops/server/release-notes/azuredevops2019 . Confidence: High.
- "Azure DevOps Server 2020 is the final version to support SQL Server Reporting Services. Starting with Azure DevOps Server 2022 and all future versions, only Analytics is supported for reporting and data insights." Services "continues to use only Analytics for all reporting capabilities". Source: https://learn.microsoft.com/en-us/azure/devops/report/powerbi/reporting-roadmap?view=azure-devops . Confidence: High.
- "The Warehouse and Analysis Service was deprecated in the previous version of Azure DevOps Server (2020). In Azure DevOps Server 2022 the Warehouse and Analysis Service has been removed from the product. Analytics now provides the in-product reporting experience." Source: https://learn.microsoft.com/en-us/azure/devops/server/release-notes/azuredevops2022 . Confidence: High.
- Server 2019 with the Inheritance process model: "you can only use Analytics for reporting, the project collection must not be configured to support reporting." Source: https://learn.microsoft.com/en-us/azure/devops/report/powerbi/data-available-in-analytics?view=azure-devops-2019 . Confidence: High.
- Built-in features depend on it. Velocity, Burndown, Cumulative Flow, Lead Time and Cycle Time widgets, in-context reports, and backlog rollup columns all read from Analytics, and stop working when it is paused or disabled. Sources: https://learn.microsoft.com/en-us/azure/devops/report/powerbi/what-is-analytics?view=azure-devops-server and https://learn.microsoft.com/en-us/azure/devops/report/dashboards/analytics-extension?view=azure-devops-server . Confidence: High.
- All of these statements compare Analytics with SSRS and the warehouse. I found no Microsoft statement that compares Analytics OData with the REST API for a custom metrics tool, and none that calls OData "required". Confidence: Medium (absence of evidence after two documentation searches).

## Unverified

1. **Whether Server 2019 actually answers `v3.0-preview` and `v4.0-preview`, and with which entity sets.** The 2019 moniker pages list all four version strings, but that text block is shared with newer versions. The same 2019 pages omit pipeline and test entity sets, and the platform table says there is no Pipelines OData on 2019. I could not find which OData versions 2019 RTM or 2019.1 really serve. Tried: the 2019 monikers of odata-api-version, data-model-analytics-service, data-available-in-analytics, analytics-query-parts, and the 2019 and 2019 Update 1 release notes. Only a `$metadata` request against a real 2019 server settles it.
2. **Test entity sets on Server 2019.** Release notes say 2019 Analytics holds automated test results. No 2019 page lists TestRuns or TestResultsDaily as queryable. Not confirmed either way.
3. **Whether the entity sets and properties in `$metadata` are identical across 2020, 2022 and the unversioned release.** The docs use one shared table for all three. I found no per-version changelog of the Analytics data model. Not tested against live servers.
4. **`StateCategory` on entity types other than WorkItemRevision.** The docs say "Valid only for the WorkItemRevision entity type". This needs a `$metadata` check per version before relying on it for WorkItems or WorkItemSnapshot.
5. **Whether classic build pipelines and classic release pipelines populate PipelineRuns.** The docs only say to define pipelines using YAML. No explicit statement found. There is no documented release or deployment entity set.
6. **Numeric query timeout and any throttling on Server.** The timeout error text is documented without a number. The rate limits page covers Services only. Tried: odata-query-guidelines (azure-devops-server) and rate-limits.
7. **Latency on Server.** The "up to 30 seconds" figure sits in text shared between Services and Server. No on-premises measurement or statement was found. The 2019 moniker of the page has no latency section at all (the old performance-latency URL now lands on the what-is-analytics page).
8. **PAT and Windows auth against the OData endpoint on Server 2019 and 2020 specifically.** The page I read is scoped to Server 2022 and the unversioned release, and is written for Power BI. I did not fetch the 2019 or 2020 monikers of client-authentication-options. Whether IIS Basic authentication must be enabled for PATs on a given server was not researched.
9. **Mapping of the `azure-devops-server` moniker to the December 2025 release.** Inferred from the page header "Azure DevOps Server" and the release notes page. Not found stated outright.
10. **Whether there is still no pull request or Repos entity set in 2026.** The "None, under investigation" table is dated August 2022.
11. **The `VSTS.Analytics.MaxSize` preference header.** The guidelines mention it and a 1000 record failure when it is set. I did not read the full section, so the exact behaviour is not recorded here.
12. **Cross-check against the docs source repository.** Not done. See "Tool and access problems".

## Tool and access problems

- The general web fetch tool could not resolve learn.microsoft.com (host not allowed in this session). All Learn pages were read through the Microsoft Learn documentation fetch and search tools instead, which worked.
- The Azure DevOps docs GitHub repository (MicrosoftDocs/azure-devops-docs) could not be read. The GitHub tool is restricted to this project's own repository, and raw.githubusercontent.com returned 404 for the guessed file path on two branch names. Moniker ranges were therefore read from rendered pages, not from the Markdown source.
- General web search was not used. No blog or forum posts were used as sources.
- Four release notes pages were too large to read whole. They were searched for every line mentioning Analytics, OData, Reporting Services or warehouse, and only those passages were read. Other parts of those pages were not read.
- No live Azure DevOps Server or Services instance was queried.

## Implications for the data access path decision

Facts only. No decision is made here.

- Analytics OData is present on every in-scope version, but on Server it is an optional, per-collection service with four possible states (not installed or not enabled, enabled, paused, disabled). Only Services guarantees it is on.
- On Server 2019 RTM an admin must install an extension. On 2019.1 and on collections upgraded into 2020 or later, an admin may have to enable it by hand. That action needs Project Collection Administrators rights.
- Pausing stops updates and makes OData queries unavailable. Disabling deletes Analytics data, and history is rebuilt from scratch on re-enable. First population after enabling takes time that scales with collection size.
- Server 2019 has Analytics in preview status, with documented known issues about incomplete data and stalled ingestion.
- The released OData versions (v1.0, v2.0) contain work tracking only. Pipeline and test entity sets exist only under v3.0-preview and v4.0-preview, which Microsoft says may change in breaking ways. A released version is supported for at least 12 months and a removed version returns HTTP 410.
- Pipeline entity sets are documented for Server 2020 and later and for Services. They are not documented for Server 2019.
- There is no Analytics data for Repos (commits, pull requests) or Artifacts on any version, per the August 2022 table. No release or deployment entity set is documented.
- `CycleTimeDays` and `LeadTimeDays` are precomputed by Microsoft with fixed definitions tied to state categories. Cycle time starts at the first In Progress or Resolved state and ends at the last Completed state, so time spent closed before a reactivation is included. A tool that wants a different definition has to compute from WorkItemRevisions or another source.
- `StateCategory` is documented only on WorkItemRevision. `ParentWorkItemId` is not available before Server 2020.
- Analytics excludes rich text fields, history comments, link counts, attachment counts and deleted work items. WorkItemLinks carries no link history.
- Snapshot entity sets can only be queried through aggregation. Row-level history has to come from WorkItemRevisions, with paging at 10,000 records, an 800 column cap, and a 3,000 character GET limit (POST via `$batch` for longer queries, one query per batch).
- The URL differs by platform: a separate host on Services, the collection URL on Server, with an optional `/tfs` virtual directory and port that depend on the installation.
- Server accepts Windows authentication and PATs (Basic, scope `vso.analytics`). Entra ID and OAuth are Services only.
- The caller needs the project-level "View analytics" permission and at least Basic access. Collection-wide queries fail outright if the caller lacks access to any included project. Area path security is not enforced by Analytics.
- Microsoft's stated direction is that Analytics is the only supported reporting platform from Server 2022 onward. That statement is about replacing SSRS and the warehouse. No Microsoft statement was found that ranks Analytics OData against the REST API for third-party tools.
- Data freshness is documented as up to 30 seconds behind, with no Server-specific figure and no documented Server throttling numbers.
- The only authoritative description of what a given server exposes is that server's own `/_odata/{version}/$metadata` response.
