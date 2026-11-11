# 02. REST API surface per Azure DevOps Server version

Research date: 2026-11-10

## Question

Which REST API endpoints and api-versions that a delivery-metrics tool needs are available on each of: Azure DevOps Server 2019 (and updates), Server 2020, Server 2022 (and updates), the current unversioned continuously updated Azure DevOps Server release (GA December 2025), and Azure DevOps Services?

## How this was researched, and its limits

Read this before relying on the document. Coverage is partial.

- The only source that was reachable was Microsoft Learn, through the Microsoft Learn documentation fetch tool. Every claim below cites the exact Learn URL, including the `view=` moniker it was read under.
- Direct web fetch of learn.microsoft.com was blocked in this session (host not in the allowed network domains). GitHub was also not usable: requests for files in the MicrosoftDocs repositories returned 404 or 403 through the session proxy, so the REST API specification repository (vsts-rest-api-specs) could not be consulted. That repository would have been the most efficient way to diff endpoints per api-version.
- Reference pages are large, so only a subset of endpoints was read directly. Everything not read is listed under "Unverified" rather than filled in from memory.
- Monikers checked: `azure-devops-server-rest-5.0` (Server 2019), `azure-devops-server-rest-6.0` (Server 2020), `azure-devops-rest-7.1` and `azure-devops-rest-7.2` (Services). The 5.1, 7.0 and Server 7.x monikers were not read for any endpoint.
- "Present at 5.0" means the page exists under the Server 2019 moniker. It does not establish which earlier version introduced the endpoint. Introduction versions earlier than 5.0 were not researched.

## Short answer

- Microsoft documents one maximum REST api-version per Server release: Server 2019 is 5.0, Server 2020 is 6.0, Server 2022 is 7.0, Server 2022.1 is 7.1, and the new unversioned release is listed as "Azure DevOps Server vNext" with 7.2. Microsoft states that a REST API version works on the listed Server version and all newer ones.
- The core work item, Git and build endpoints that were checked all exist at api-version 5.0 on Server 2019 with the URL shape `https://{instance}/{collection}/{project}/_apis/...`. This includes the reporting work item revisions endpoint with continuation token, work item updates, team iterations, pull request list, pull request commits, and build list with minTime, maxTime and continuationToken.
- Three things differ by version in ways that matter to a metrics tool:
  1. Pull request list at 5.0 has no time-range filter and no `mergeStrategy`. Only the boolean `squashMerge` exists. At 7.1 the list has `searchCriteria.minTime`, `maxTime` and `queryTimeRangeType` (created or closed), and `completionOptions.mergeStrategy` (noFastForward, squash, rebase, rebaseMerge).
  2. Work item type states (which carries the state category) is preview-only at 5.0 (`5.0-preview.1`) and released at 7.1.
  3. Pipelines Runs and Environments are documented for Server 2020 only as `6.0-preview.1`. They were not found to be documented for Server 2019 in this research (not positively confirmed absent).
- Rate limiting (TSTUs, Retry-After, X-RateLimit headers) is documented for Azure DevOps Services only. No Server throttling documentation was found.
- Service hooks, including webhooks, are documented for Azure DevOps Server. Server has a smaller pipeline event set than Services.
- Runtime discovery of the supported api-version (OPTIONS or connectionData) could not be confirmed from a primary source.

## Server release to api-version table

Source for all rows unless noted: "API and TFS version mapping" table at https://learn.microsoft.com/en-us/rest/api/azure/devops/?view=azure-devops-rest-7.2

| Product release | Max REST api-version (documented) | Build version (documented) | Confidence |
| --- | --- | --- | --- |
| Azure DevOps Server 2019 | 5.0 | >= 17.143.28621.4 | High |
| Azure DevOps Server 2019 Update 1 (2019.1) | Not listed in the table | Not listed | Unverified |
| Azure DevOps Server 2020 (and 2020.1) | 6.0 (2020.1 not listed separately) | >= 18.170.30525.1 | High for 2020, unverified for 2020.1 |
| Azure DevOps Server 2022 | 7.0 | >= 19.205.33122.1 | High |
| Azure DevOps Server 2022.1 | 7.1 | >= 19.225.34309.2 | High |
| Azure DevOps Server 2022.2 | Not listed in the table | 19.235.35102.1 (RTW, from the build number list) | Unverified for api-version |
| Azure DevOps Server (unversioned, RTW 2025-12-09) | 7.2, listed as "Azure DevOps Server vNext" with no build number in the mapping table | 20.0.36719.1 (from the build number list) | Medium, see note |
| Azure DevOps Services | "the most recent Rest API version (including preview versions), as well as previous versions" | n/a | High |

Notes:

- The mapping table labels the 7.2 row "Azure DevOps Server vNext" and leaves the build column empty. It does not use the name of the released product. That the December 2025 release is the "vNext" row is an inference. Source for the release date and name: https://learn.microsoft.com/azure/devops/server/release-notes/azuredevopsserver?view=azure-devops (section "Azure DevOps Server Release Date: December 9, 2025"). Source for build 20.0.36719.1: https://learn.microsoft.com/azure/devops/release-notes/features-timeline-released (section "Azure DevOps Server Build Numbers").
- The same page states: "REST API versions are compatible with the Server version listed, as well as Server versions that are newer than the Server version listed." Confidence: high (direct quote).
- A second Microsoft table exists at https://learn.microsoft.com/en-us/azure/devops/integrate/concepts/rest-api-versioning?view=azure-devops . It lists major versions only (1.0 to 7.0) and stops at Server 2022 with 7.0. It does not mention 7.1, 7.2 or the unversioned release. It agrees with the table above for 2019 (5.0), 2020 (6.0) and 2022 (7.0). It also states "Major API version releases align with Team Foundation Server RTM releases."
- The new release can be installed directly or upgraded to from Server 2022, 2020, 2019 or TFS 2015 or newer. Source: release notes URL above. Confidence: high.

### How the api-version is specified

- The api-version must be specified with every request, as a query parameter or in the Accept header (`Accept: application/json;api-version=1.0`). Format is `{major}.{minor}[-{stage}[.{resource-version}]]`. Source: https://learn.microsoft.com/en-us/azure/devops/integrate/concepts/rest-api-versioning?view=azure-devops . Confidence: high.
- After an API is released, its preview version is deprecated and "can be deactivated after 12 weeks". Once deactivated, requests that specify a `-preview` version get rejected. Same source. Confidence: high. The page does not say whether this deactivation policy applies to on-premises Server, where the binaries do not change between updates. That point is unverified.

### Runtime discovery of the supported version

- Not confirmed. No primary source was found in this session for an OPTIONS request on `_apis` or for `_apis/connectionData` as a way to discover the maximum supported api-version. See "Unverified".
- What is documented: the Resource Areas API (`{organization or collection URL}/_apis/resourceAreas/{areaId}?api-version=5.0-preview.1`) returns a `locationUrl` that is the base URL for a given API area (for example release). The page lists fixed resource area ids for build, git, wit, work, release, pipelines, distributedtask and others. Source: https://learn.microsoft.com/en-us/azure/devops/extend/develop/work-with-urls?view=azure-devops-server . The page header lists Azure DevOps Services, Azure DevOps Server and Azure DevOps Server 2022 as applicable, but every example on it uses dev.azure.com. Confidence that the API exists: high. Confidence that it behaves the same on Server: medium (applicability banner only, no Server example).

## Endpoint availability matrix

Legend: "Yes (5.0)" means the reference page was read under the Server 2019 moniker and shows a released 5.0 version. "Inferred" means not read directly for that release, and derived only from Microsoft's statement that an api-version works on newer Server releases. "Not checked" means no page was read.

| Capability | Server 2019 (5.0) | Server 2020 (6.0) | Server 2022 / 2022.1 (7.0 / 7.1) | Unversioned Server (7.2) | Services |
| --- | --- | --- | --- | --- | --- |
| Reporting work item revisions (continuation token) | Yes (5.0) | Inferred | Inferred | Inferred | Yes (7.1 read) |
| Work item updates list | Yes (5.0) | Inferred | Inferred | Inferred | Not checked |
| Work items get / list / batch | Not checked | Not checked | Not checked | Not checked | Not checked |
| Work item revisions list | Not checked | Not checked | Not checked | Not checked | Not checked |
| WIQL | Not checked | Not checked | Not checked | Not checked | Not checked |
| Classification nodes | Not checked | Not checked | Not checked | Not checked | Not checked |
| Team iterations list | Yes (5.0) | Inferred | Inferred | Inferred | Not checked |
| Work item type states with category | Preview only (5.0-preview.1) | Not checked | Not checked | Not checked | Yes, released (7.1 read) |
| Pull requests list by status | Yes (5.0) | Inferred | Inferred | Inferred | Yes (7.1 read) |
| Pull requests list by time range (minTime, maxTime, queryTimeRangeType) | No (not in 5.0 page) | Not checked | Not checked | Not checked | Yes (7.1 read) |
| PR closedDate, lastMergeCommit | Yes (5.0) | Inferred | Inferred | Inferred | Yes (7.1 read) |
| PR completionOptions.squashMerge | Yes (5.0) | Inferred | Inferred | Inferred | Yes, marked deprecated (7.1 read) |
| PR completionOptions.mergeStrategy | No (not in 5.0 page) | Not checked | Not checked | Not checked | Yes (7.1 read) |
| Pull request commits | Yes (5.0) | Inferred | Inferred | Inferred | Not checked |
| PR iterations, commits list, pushes | Not checked | Not checked | Not checked | Not checked | Not checked |
| Builds list (minTime, maxTime, continuationToken, queryOrder) | Yes (5.0) | Inferred | Inferred | Inferred | Not checked |
| Build timeline | Not checked | Not checked | Not checked | Not checked | Not checked |
| Pipelines runs list | Not checked | Preview only (6.0-preview.1) | Not checked | Not checked | Not checked |
| Classic release API (release area) | Yes (5.0), collection URL | Inferred | Inferred | Inferred | Separate vsrm host |
| Release deployments list | Not checked | Not checked | Not checked | Not checked | Not checked |
| Environments list | Not checked | Preview only (6.0-preview.1) | Not checked | Not checked | Not checked |
| Environment deployment records | Not checked | Not checked | Not checked | Not checked | Not checked |
| Rate limit headers | No documentation found | No documentation found | No documentation found | No documentation found | Yes |
| Service hooks (webhooks) | Not checked for 2019 | Not checked for 2020 | Yes (2022) | Yes | Yes |

## Details

### 2. Work item history

- Reporting Work Item Revisions, Read Reporting Revisions Get. On Server 2019 at api-version 5.0: `GET https://{instance}/{collection}/{project}/_apis/wit/reporting/workitemrevisions?api-version=5.0`. Parameters at 5.0 include `continuationToken` ("Specifies the watermark to start the batch from"), `startDateTime`, `fields`, `types`, `includeDeleted`, `includeIdentityRef`, `includeTagRef`, `includeLatestOnly`, `$expand`, `includeDiscussionChangesOnly` and `$maxPageSize`. The response type `ReportingWorkItemRevisionsBatch` has `values`, `continuationToken`, `nextLink` and `isLastBatch`. Source (Server 2019 moniker): https://learn.microsoft.com/en-us/rest/api/azure/devops/wit/reporting-work-item-revisions/read-reporting-revisions-get?view=azure-devops-server-rest-5.0 . Confidence: high.
- The same endpoint at 7.1 on Services has the same parameter list as 5.0. No parameter was added or removed between the 5.0 and 7.1 pages. Source (Services 7.1 moniker): https://learn.microsoft.com/en-us/rest/api/azure/devops/wit/reporting-work-item-revisions/read-reporting-revisions-get?view=azure-devops-rest-7.1 . Confidence: high.
- `startDateTime` is documented as "Cannot be used in conjunction with 'watermark' parameter." Both pages above. Confidence: high.
- In the documented sample response for an unfinished revision, `System.RevisedDate` is `9999-01-01T00:00:00Z`. Both pages above. Confidence: high that the sample shows this. It is a sample, not a stated contract.
- The sample `nextLink` values in both pages carry `api-version=2.0`. This suggests the endpoint predates 5.0, but it is a sample value and not a statement of when the endpoint was introduced. Confidence: low.
- Work item Updates, List. On Server 2019 at 5.0: `GET https://{instance}/{collection}/{project}/_apis/wit/workItems/{id}/updates?api-version=5.0`, with optional `$top` and `$skip`. Described as returning "the deltas between work item revisions". Source (Server 2019 moniker): https://learn.microsoft.com/en-us/rest/api/azure/devops/wit/updates/list?view=azure-devops-server-rest-5.0 . Only the top of this page (URL shapes and the start of the parameter table) was read. Confidence: high for existence and URL shape.
- Team iterations, List. On Server 2019 at 5.0: `GET https://{instance}/{collection}/{project}/{team}/_apis/work/teamsettings/iterations?api-version=5.0`, with optional `$timeframe` ("Only Current is supported currently"). The response items have `id`, `name`, `path`, `url` and `attributes` with `startDate`, `finishDate` and `timeFrame` (past, current, future). Source (Server 2019 moniker): https://learn.microsoft.com/en-us/rest/api/azure/devops/work/iterations/list?view=azure-devops-server-rest-5.0 . Confidence: high.
- Work Item Type States, List. On Server 2019 this is preview-only: `GET https://{instance}/{collection}/{project}/_apis/wit/workitemtypes/{type}/states?api-version=5.0-preview.1`. The response item `WorkItemStateColor` has `name`, `color` and `category`. The sample shows categories Proposed, InProgress, Resolved and Completed. Source (Server 2019 moniker): https://learn.microsoft.com/en-us/rest/api/azure/devops/wit/work-item-type-states/list?view=azure-devops-server-rest-5.0 . Confidence: high.
- The same endpoint is a released version at 7.1 on Services (`api-version=7.1`), with the same response shape. Source (Services 7.1 moniker): https://learn.microsoft.com/en-us/rest/api/azure/devops/wit/work-item-type-states/list?view=azure-devops-rest-7.1 . Confidence: high. The version at which it left preview (5.1, 6.0 or 7.0) was not checked.

### 3. Git

- Pull Requests, Get Pull Requests on Server 2019 at 5.0: `GET https://{instance}/{collection}/{project}/_apis/git/repositories/{repositoryId}/pullrequests?api-version=5.0`. Search criteria at 5.0: `status`, `creatorId`, `reviewerId`, `sourceRefName`, `targetRefName`, `repositoryId`, `sourceRepositoryId`, `includeLinks`, plus `$top` and `$skip`. `searchCriteria.status` "Defaults to Active if unset". Status values: notSet, active, abandoned, completed, all. There is no time filter at 5.0. Source (Server 2019 moniker): https://learn.microsoft.com/en-us/rest/api/azure/devops/git/pull-requests/get-pull-requests?view=azure-devops-server-rest-5.0 . Confidence: high.
- At 7.1 on Services the same endpoint adds `searchCriteria.minTime`, `searchCriteria.maxTime` and `searchCriteria.queryTimeRangeType` (values `created` and `closed`, "Defaults to Created if unset"). The 7.1 page also adds the note that the description field is truncated to 400 symbols in the result. Source (Services 7.1 moniker): https://learn.microsoft.com/en-us/rest/api/azure/devops/git/pull-requests/get-pull-requests?view=azure-devops-rest-7.1 . Confidence: high. The api-version that introduced the time filter (anywhere from 5.1 to 7.1) was not checked.
- Completion time. `closedDate` ("The date when the pull request was closed (completed, abandoned, or merged externally)") and `completionQueueTime` are on `GitPullRequest` at both 5.0 and 7.1. Both pages above. Confidence: high.
- Merge strategy. At 5.0 `GitPullRequestCompletionOptions` has `squashMerge` (boolean) and no `mergeStrategy`. At 7.1 it has `mergeStrategy` with values noFastForward, squash, rebase and rebaseMerge, and `squashMerge` is described as deprecated: "If MergeStrategy is not set, the merge strategy will be no-fast-forward if this flag is false, or squash if true." Both pages above. Confidence: high. Which api-version introduced `mergeStrategy` was not checked.
- Last merge commit. `lastMergeCommit` ("The commit of the most recent pull request merge. If empty, the most recent merge is in progress or was unsuccessful"), `lastMergeSourceCommit` and `lastMergeTargetCommit` are on `GitPullRequest` at both 5.0 and 7.1. Both pages above. Confidence: high.
- Pull Request Commits on Server 2019 at 5.0: `GET https://{instance}/{collection}/{project}/_apis/git/repositories/{repositoryId}/pullRequests/{pullRequestId}/commits?api-version=5.0`, returning `GitCommitRef[]`. Source (Server 2019 moniker): https://learn.microsoft.com/en-us/rest/api/azure/devops/git/pull-request-commits/get-pull-request-commits?view=azure-devops-server-rest-5.0 . Confidence: high.
- Linking a squash-merged pull request to its commits. The documented pieces are: the merge strategy "squash" is defined as "Put all changes from the pull request into a single-parent commit" (7.1 page); `lastMergeCommit` identifies the commit of the most recent PR merge; `lastMergeSourceCommit` is the head of the source branch at the time of the last merge; and the pull request commits endpoint returns the commits of the pull request. Combining these, `lastMergeCommit.commitId` is the commit on the target branch and the pull request commits endpoint gives the original source commits. No Microsoft page was found that states this procedure explicitly, so the combination is an inference from the field descriptions. Confidence: medium.
- At 5.0, whether a completed PR was squashed can only be read from `completionOptions.squashMerge`. It was not verified whether list responses include `completionOptions` at all (the documented sample list responses at both 5.0 and 7.1 do not show it). Confidence: low, see "Unverified".

### 4. Pipelines and deployments

- Builds, List on Server 2019 at 5.0: `GET https://{instance}/{collection}/{project}/_apis/build/builds?api-version=5.0`. Parameters at 5.0 include `minTime` and `maxTime` ("filters to builds that finished/started/queued after this date based on the queryOrder specified"), `queryOrder` (finishTime, queueTime or startTime, ascending or descending), `continuationToken`, `$top`, `definitions`, `statusFilter`, `resultFilter`, `reasonFilter`, `branchName`, `repositoryId`, `repositoryType`, `buildIds` and `deletedFilter`. The `Build` object has `queueTime`, `startTime`, `finishTime`, `sourceVersion`, `sourceBranch`, `result`, `status`, `reason` and `triggerInfo`. Source (Server 2019 moniker): https://learn.microsoft.com/en-us/rest/api/azure/devops/build/builds/list?view=azure-devops-server-rest-5.0 . Confidence: high. The page does not say how the continuation token is returned (header or body).
- Pipelines, Runs, List on Server 2020 is preview-only: `GET https://{instance}/{collection}/{project}/_apis/pipelines/{pipelineId}/runs?api-version=6.0-preview.1`. It is described as "Gets top 10000 runs for a particular pipeline" and has no filter, time or continuation parameters at this version. The `Run` object has `createdDate`, `finishedDate`, `state`, `result`, `pipeline` and `resources`. Source (Server 2020 moniker): https://learn.microsoft.com/en-us/rest/api/azure/devops/pipelines/runs/list?view=azure-devops-server-rest-6.0 . Confidence: high.
- Classic Release on Server. The release API on Server 2019 is addressed on the collection URL, with no separate host: `GET https://{instance}/{collection}/{project}/_apis/release/releases/{releaseId}?...&api-version=5.0`. Source (Server 2019 moniker): https://learn.microsoft.com/en-us/rest/api/azure/devops/release/releases/get-release-revision?view=azure-devops-server-rest-5.0 . Confidence: high for the URL shape. This was established from one small release endpoint. The Releases List and Deployments List pages themselves were not read.
- Classic Release on Services uses a different base URL. Microsoft's example: build APIs at `https://dev.azure.com/Fabrikam`, release management APIs at `https://vsrm.dev.azure.com/Fabrikam`, and the recommended way to find it is the Resource Areas API with the release area id. Source: https://learn.microsoft.com/en-us/azure/devops/extend/develop/work-with-urls?view=azure-devops-server . Confidence: high.
- Environments, List on Server 2020 is preview-only: `GET https://{instance}/{collection}/{project}/_apis/distributedtask/environments?api-version=6.0-preview.1`, with `name`, `continuationToken` and `$top`. Source (Server 2020 moniker): https://learn.microsoft.com/en-us/rest/api/azure/devops/distributedtask/environments/list?view=azure-devops-server-rest-6.0 . Confidence: high.
- Preview-only items found so far: Work Item Type States on Server 2019 (5.0-preview.1), Pipelines Runs on Server 2020 (6.0-preview.1), Environments on Server 2020 (6.0-preview.1). The Resource Areas API is also shown only with `5.0-preview.1`.

### 5. Rate limiting and throttling

- The rate limits article is scoped to Azure DevOps Services only. Its applicability line reads "Azure DevOps Services", and requesting it with the Server view returns the same Services-only content. Sources: https://learn.microsoft.com/en-us/azure/devops/integrate/concepts/rate-limits?view=azure-devops and https://learn.microsoft.com/en-us/azure/devops/integrate/concepts/rate-limits?view=azure-devops-server . Confidence: high.
- Services, global consumption limit: 200 Azure DevOps throughput units (TSTUs) per user within a sliding five-minute window. Requests are first delayed (from milliseconds up to 30 seconds per request). When blocked, the response is HTTP 429 with a TF400733 message. Same source. Confidence: high.
- Services, headers: `Retry-After` (seconds), `X-RateLimit-Resource`, `X-RateLimit-Delay`, `X-RateLimit-Limit`, `X-RateLimit-Remaining`, `X-RateLimit-Reset` (Unix epoch) and `X-RateLimit-Cost`. "Except for X-RateLimit-Delay, all these headers are sent before requests start getting delayed." A response carrying `Retry-After` can still be HTTP 200. Same source. Confidence: high.
- Services: each pipeline also has its own 200 TSTU limit per sliding five-minute window. Same source. Confidence: high.
- Server: no Microsoft documentation of throttling, TSTUs or rate limit headers on Azure DevOps Server was found. One documentation search for Server throttling returned only the Services article. Absence of documentation is not proof of absence of behaviour. Confidence: medium that nothing is documented, low on actual Server behaviour.
- Server, a related documented limit: the release notes for the unversioned Server release state that the REST API allows at most 1,000 comments per work item. Source: https://learn.microsoft.com/azure/devops/server/release-notes/azuredevopsserver?view=azure-devops (RC section, October 7, 2025). Confidence: high.

### 6. Service hooks

- Service hooks are documented as applying to "Azure DevOps Services | Azure DevOps Server | Azure DevOps Server 2022". Sources: https://learn.microsoft.com/en-us/azure/devops/service-hooks/overview?view=azure-devops-server and https://learn.microsoft.com/en-us/azure/devops/service-hooks/events?view=azure-devops . Confidence: high for Server 2022 and the unversioned Server. Server 2019 and 2020 are not named on the current page (those doc versions are no longer in the applicability banner), so they are unverified.
- Webhooks are an available target on Server 2022 with "All" events supported, managed in Azure DevOps. The Server 2022 services table also lists Azure Service Bus, Azure Storage, Jenkins, Slack, Teams and others. Source: overview URL above (section marked for azure-devops-2022). Confidence: high. Note that the page shows a service table for Services and one for Server 2022 only. No table is shown for the unversioned Server moniker.
- Events available on all listed products, with their event ids. Source: events URL above. Confidence: high.
  - Build and release: build completed (`build.complete`), release created (`ms.azure-devops-release.release-created-event`), release abandoned, release deployment started (`ms.azure-devops-release.deployment-started-event`), release deployment completed (`ms.azure-devops-release.deployment-completed-event`), release deployment approval pending and completed.
  - Code: code pushed (`git.push`), pull request created (`git.pullrequest.created`), pull request updated (`git.pullrequest.updated`), pull request merge attempted (`git.pullrequest.merged`), pull request commented on, TFVC check-in, repository created, deleted, forked, renamed, status changed.
  - Work item: created (`workitem.created`), updated (`workitem.updated`), deleted (`workitem.deleted`), restored (`workitem.restored`), commented on (`workitem.commented`).
  - Service connection: created, updated.
- Pipeline events differ. For Server 2022 and the unversioned Server the page lists four: run state changed (`ms.vss-pipelines.run-state-changed-event`), run stage state changed (`ms.vss-pipelines.stage-state-changed-event`), run stage waiting for approval, run stage approval completed. Services additionally lists run job state changed (`ms.vss-pipelines.job-state-changed-event`), check updated, manual intervention pending, elastic agent pool resized and project-level agent pool created and updated. Advanced security events are Services only. Source: events URL above, "Available event types" section. Confidence: high.
- Subscriptions can be created programmatically. Source: overview URL above, FAQ. Confidence: high. The REST endpoint and api-version for that were not read.

## Unverified

Each item says what was tried.

1. Runtime discovery of the supported api-version through an OPTIONS request on `_apis` or through `_apis/connectionData`. Not found in any page read. No dedicated search was run for it because the remaining research budget was spent on reference pages. GitHub sources that might document it were unreachable.
2. api-version 5.1 and Azure DevOps Server 2019 Update 1. The mapping table has no row for 2019.1 or for 5.1. The Server 2019 Update 1 release notes (https://learn.microsoft.com/en-us/azure/devops/server/release-notes/azuredevops2019u1?view=azure-devops) were fetched and searched for "5.1", "api-version" and "REST API". No relevant match.
3. Maximum api-version for Server 2020.1 and Server 2022.2. Not in the mapping table. Their release notes were not read.
4. Whether "Azure DevOps Server vNext" at 7.2 in the mapping table is exactly the December 2025 release, and whether 7.2 is released or still preview on that release. The table gives no build number for the row.
5. Not read at all for any version: Work Items Get, List and Get Work Items Batch; Revisions List; WIQL (Query By Wiql); Classification Nodes; team settings other than iterations; Work Item Types list; Git Commits; Git Pushes; Pull Request Iterations; Build Timeline; Release Releases List; Release Deployments List.
6. Environment deployment records. Two guessed reference URLs returned "could not be retrieved": `.../rest/api/azure/devops/environments/environmentdeployment-records/list` under both `azure-devops-server-rest-6.0` and `azure-devops-rest-7.1`. This does not show the endpoint is missing. The URL path was probably wrong. Availability on any version is unverified.
7. Whether Pipelines Runs and Environments exist on Server 2019. The Server 2019 moniker was not requested for these pages, so absence at 5.0 is not confirmed.
8. Whether Pipelines Runs, Environments and Work Item Type States are released (non-preview) at 7.0, 7.1 or 7.2 on Server. Only the Services 7.1 page for Work Item Type States was read.
9. Which api-version introduced `searchCriteria.minTime`, `maxTime`, `queryTimeRangeType` and `completionOptions.mergeStrategy` on pull requests. Known only as absent at 5.0 and present at 7.1. Server 2020 (6.0) and Server 2022 (7.0) were not checked.
10. Whether `completionOptions` is populated in pull request list responses or only on the single pull request Get. Sample list responses do not show it.
11. How the Builds List continuation token is returned, and the maximum `$top`. Not stated on the page read.
12. Reporting work item revisions on Server: whether behaviour differs from Services (page size limits, how deleted work items appear, retention of the watermark). Pages for 5.0 and 7.1 are identical in parameters. Nothing more was found.
13. Throttling on Azure DevOps Server. No documentation found. One search was run.
14. Service hooks on Server 2019 and Server 2020, and the pipeline events available on them. The current pages name only Server 2022 and the unversioned Server.
15. Whether the 12-week preview deactivation policy applies to on-premises Server.
16. Introduction version of any endpoint earlier than 5.0. Not researched.

Blocked tools and sites: direct web fetch of learn.microsoft.com (DNS not resolvable from the session, outside allowed network domains); raw.githubusercontent.com, api.github.com and codeload.github.com for the MicrosoftDocs repositories (404 or 403 through the session proxy); the GitHub connector failed to connect.

## Implications for minimum supported version and sync design

Facts only. No decision is made here.

- Version floor facts. Server 2019 accepts at most api-version 5.0 by Microsoft's table. Server 2020 accepts 6.0. Server 2022 accepts 7.0, 2022.1 accepts 7.1. A request sent with an api-version higher than the server's maximum is outside the documented compatibility statement.
- A client that pins every call to 5.0 is inside the documented compatibility statement for all five targets, because Microsoft says an api-version works on the listed Server and all newer ones, and Services supports previous versions.
- At 5.0 the following verified capabilities exist: incremental work item sync through reporting work item revisions with a continuation token; per-item updates; team iterations with dates; pull request list by status with `closedDate` and `lastMergeCommit`; pull request commits; build list filtered by time with continuation.
- At 5.0 the following verified gaps exist: no server-side time-range filter on pull request list, so an incremental pull request sync at 5.0 has to page by `$top` and `$skip` and filter client side or by status; no `mergeStrategy`, only `squashMerge`; state categories only through a preview api-version (`5.0-preview.1`).
- State category data depends on a preview API on Server 2019. Preview versions carry a documented deprecation rule after release.
- YAML pipeline data through the Pipelines Runs and Environments APIs is documented for Server 2020 only as `6.0-preview.1`, and the 6.0 runs list has no time filter and returns the top 10000 runs per pipeline. The Builds List endpoint at 5.0 has time filters and continuation and exists on Server 2019.
- Classic release data lives on the collection URL on Server and on a separate `vsrm.dev.azure.com` host on Services. Microsoft's guidance is not to assume a fixed domain and to resolve the base URL through the Resource Areas API.
- Server URL shape is `https://{instance}/{collection}/{project}/_apis/...`. Services URL shape is `https://dev.azure.com/{organization}/{project}/_apis/...`.
- Rate limit headers and HTTP 429 handling are documented for Services only. A response with `Retry-After` on Services can be HTTP 200. No documented signal exists for Server.
- Service hooks with a webhook target are documented for Server 2022 and the unversioned Server and cover work item created, updated, deleted and restored, code pushed, pull request created, updated and merge attempted, build completed, classic release deployment started and completed, and run and stage state changes. Job state changed is documented for Services only.
- Several items a design would depend on remain unverified in this document, most importantly runtime version discovery, the build timeline, classic release deployments list, environment deployment records, WIQL and classification nodes. They need a second pass with access to the REST specification repository or direct access to learn.microsoft.com.
