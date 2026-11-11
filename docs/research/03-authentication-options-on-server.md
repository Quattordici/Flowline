# 03. Authentication options on Azure DevOps Server and Services

## Question

What authentication options can an unattended background service use to read from Azure DevOps Server (2019, 2020, 2022, and the current unversioned continuously updated release that went GA in December 2025) and from Azure DevOps Services, and what is the least-privilege read-only setup for each?

## Research date

2026-11-10 (pages fetched 2026-11-10 and 2026-11-11).

## How the research was done, and what was blocked

- learn.microsoft.com pages were read through the Microsoft Learn documentation search and fetch tools. The `view=` moniker used is given with each source.
- `devblogs.microsoft.com` could not be reached from this session (DNS resolution failed, the host is outside the allowed network domains). No Azure DevOps blog post was read. Blog URLs appear below only as links that the Learn pages point to. Every claim that rests on a blog post is listed under "Unverified".
- `raw.githubusercontent.com` was reachable. Two README files of the Microsoft auth samples repository were read.
- No live Azure DevOps Server or Services instance was available. Nothing below was tested against a running system.

Monikers seen on Learn: `azure-devops` (Services), `azure-devops-server` (the unversioned Server release, shown on pages as "Azure DevOps Server"), `azure-devops-2022`, `azure-devops-2020`, `azure-devops-2019`. Many current pages list only "Azure DevOps Services | Azure DevOps Server | Azure DevOps Server 2022". Pages requested with `view=azure-devops-2020` or `view=azure-devops-2019` returned older page versions, which are cited as such.

## Short answer

1. On every Server version in scope, the documented options for a non-interactive client are a personal access token (PAT) and Windows authentication (NTLM or Kerberos through Negotiate). Microsoft's guidance for on-premises is "use .NET client libraries, Windows authentication, or personal access tokens".
2. Microsoft Entra ID tokens, service principals, managed identities and OAuth 2.0 apps are documented as available for Azure DevOps Services only, not for Azure DevOps Server. The docs state this for "Azure DevOps Server" in general, not version by version. The release notes of the December 2025 Server release do not add Entra-based authentication.
3. On Services, Microsoft tells new integrations to use Entra tokens (managed identity or service principal for background services) and describes PATs as the highest-risk choice. PATs still work on Services. Global PATs are being retired (the docs give two different dates, 2026-12-01 and 2027-01-27). Azure DevOps OAuth (the older, non-Entra OAuth) stopped accepting new registrations in April 2025 and is "scheduled for removal in 2026".
4. A PAT never grants more than its owner has, so least privilege has two layers: a dedicated identity in the project Readers group with Basic access, and a PAT limited to read scopes. No Microsoft page defines a minimal permission set for reporting tools.
5. There is no read-only PAT scope for pipeline environments in the published scope list: the Environments List API names only `vso.environment_manage`.
6. Windows authentication from a Linux container is not covered by the Azure DevOps docs. The only related Azure DevOps statement found is that the pipelines agent supports Integrated and Negotiate registration on "Windows agents only". .NET itself can do Negotiate and NTLM on Linux with extra setup.
7. SSH works for Git operations only, on both Services and Server. It is not a REST API credential. Alternate credentials (username and password basic auth) are no longer supported on Services. On Server, enabling IIS Basic Authentication breaks PATs.
8. Several items could not be confirmed: PAT maximum lifetime on Server and on Services, which PAT scopes each Server version actually offers, whether Graph endpoints exist on Server, gMSA use from Linux containers, and everything published only on the Azure DevOps blog.

## Options by version

Legend: Yes = documented as supported. No = documented as not available. Not documented = no statement found either way.

| Option | Server 2019 | Server 2020 | Server 2022 | Server (unversioned, GA 2025-12-09) | Services |
| --- | --- | --- | --- | --- | --- |
| PAT sent as HTTP Basic header | Yes | Yes | Yes | Yes | Yes, discouraged for production services |
| Windows integrated auth (NTLM, Kerberos, Negotiate) for REST | Yes at product level, not confirmed per version | Same | Yes, named in guidance | Yes, named in guidance | No |
| Entra service principal or managed identity | No (general Server statement) | No (general Server statement) | No | No | Yes, recommended |
| Entra ID OAuth (user delegated) | No | No | No | No | Yes |
| Azure DevOps OAuth 2.0 apps | No | No | No | No | Deprecated, no new registrations since April 2025 |
| SSH keys | Git only, not confirmed per version | Git only, not confirmed per version | Git only | Git only | Git only |
| Alternate credentials or IIS Basic auth with password | Discouraged, breaks PATs | Discouraged, breaks PATs | Discouraged, breaks PATs | Discouraged, breaks PATs | Not supported |
| PAT admin policies (restrict scope, lifetime, creation) | Not documented | Not documented | Not documented | Not documented | Yes, need an Entra tenant |
| Highest REST API major version | 5.0 | 6.0 | 7.0 | Not in the table read | Latest |
| Analytics (OData) | Preview | GA | GA | GA | GA |

Sources for the table rows are given in the details below.

## Details

Confidence scale: High = stated directly in a primary source read in this session. Medium = stated in a primary source but for a neighbouring scope (for example a general "Azure DevOps Server" statement applied to one version), or the sources disagree. Low = inferred.

### 1. Personal access tokens

1.1 PATs are documented for every Server version in scope. The "Use personal access tokens" page exists for the monikers `azure-devops-2019`, `azure-devops-2020`, `azure-devops-2022` and `azure-devops-server`. The 2022 and unversioned Server view has its own creation steps ("Sign in to your Azure DevOps Server web portal", "select the collection"). Confidence: High for 2022 and the unversioned release. Medium for 2019 and 2020, because those page versions show Services screenshots and Services wording under a Server title.
- https://learn.microsoft.com/en-us/azure/devops/organizations/accounts/use-personal-access-tokens-to-authenticate?view=azure-devops-2022
- https://learn.microsoft.com/en-us/azure/devops/organizations/accounts/use-personal-access-tokens-to-authenticate?view=azure-devops-2020
- https://learn.microsoft.com/en-us/azure/devops/organizations/accounts/use-personal-access-tokens-to-authenticate?view=azure-devops-2019

1.2 The comparison table in the integration authentication guidance lists PATs as working with "Azure DevOps Services and Azure DevOps Server". Confidence: High. Monikers: Services, Server, Server 2022.
- https://learn.microsoft.com/en-us/azure/devops/integrate/get-started/authentication/authentication-guidance?view=azure-devops

1.3 Wire format: a PAT is sent as HTTP Basic, `Authorization: Basic base64(":" + PAT)`, username may be empty. Confidence: High. Monikers: Services, Server, Server 2022.
- https://learn.microsoft.com/en-us/azure/devops/organizations/accounts/use-personal-access-tokens-to-authenticate?view=azure-devops-2022

1.4 A PAT is always tied to the user who created it and "can't grant more permissions than its owner has". Applications cannot create PATs. Confidence: High.
- https://learn.microsoft.com/en-us/azure/devops/organizations/accounts/use-personal-access-tokens-to-authenticate?view=azure-devops-2022

1.5 On Server, IIS Basic Authentication must stay disabled: "For Azure DevOps Server, IIS Basic Authentication prevents PAT authentication." The 2019 and 2020 page versions say the same ("Enabling IIS Basic Authentication invalidates using PATs for Azure DevOps Server"). Confidence: High for 2019, 2020, 2022 and the unversioned release.
- Same three URLs as 1.1.

1.6 The 2019 and 2020 page versions also contain this sentence in a warning about Git with IIS Basic Authentication: "The extra header must be used for all Azure DevOps Server installations, as Windows Auth also prevents using PATs." The sentence is about Git requests and is not explained further. It is not present in the 2022 view. What it means for REST calls is not stated. Confidence that the sentence exists: High. Confidence in any interpretation: Low.
- https://learn.microsoft.com/en-us/azure/devops/organizations/accounts/use-personal-access-tokens-to-authenticate?view=azure-devops-2019

1.7 Pipelines agent documentation says PAT registration against "Azure DevOps Server & TFS" is "Supported when server is configured with HTTPS". This statement is about agent registration, not about REST calls in general. Confidence: High for the agent, Low as a general rule.
- https://learn.microsoft.com/en-us/azure/devops/pipelines/agents/agent-authentication-options?view=azure-devops-2022

1.8 Scope list. The only published scope table is on the OAuth 2.0 page, which carries the Services moniker only. The Server PAT pages link to that same table. No Server-specific scope list was found. Confidence that the table below is correct for Services: High. Confidence that each scope exists in each Server version: not established, see Unverified.
- https://learn.microsoft.com/en-us/azure/devops/integrate/get-started/authentication/oauth?view=azure-devops

Read scopes relevant to a read-only metrics reader, from that table:

| Area | Scope | Display name | Note from the table |
| --- | --- | --- | --- |
| Work items | `vso.work` | Work items (read) | Read work items, queries, boards, area and iteration paths, execute queries. Inherits `vso.hooks_write`. |
| Code, repos, pull requests | `vso.code` | Code (read) | Read source code and metadata about commits, branches and other version control artifacts. Inherits `vso.hooks_write`. |
| Build | `vso.build` | Build (read) | Build results, definitions, requests. Inherits `vso.hooks_write`. |
| Release | `vso.release` | Release (read) | Releases, release definitions, release environments. Inherits `vso.profile`. |
| Environments | `vso.environment_manage` | Environment (read and manage) | Marked high privilege. No read-only environment scope is listed. |
| Project and team | `vso.project` | Project and team (read) | Read projects and teams. |
| Analytics | `vso.analytics` | Analytics (read) | Query analytics data. |
| Graph | `vso.graph` | Graph (read) | Read user, group, scope and group membership information. |
| Identity | `vso.identity` | Identity (read) | Read identities and groups. |
| Agent pools | `vso.agentpools` | Agent pools (read) | View pools, queues, agents and jobs. |
| Test | `vso.test` | Test management (read) | Inherits `vso.profile`. |
| Profile | `vso.profile` | User profile (read) | Profile, accounts, collections, projects, teams. |

1.9 Three "read" scopes (`vso.work`, `vso.code`, `vso.build`) are listed as inheriting `vso.hooks_write`, described as the ability to create and update service hook subscriptions. A token with those scopes is therefore not strictly read-only at the scope level. Whether the token can actually create a subscription still depends on the owner's permissions (see 1.4). Confidence: High that the table says this. Medium on practical effect.
- https://learn.microsoft.com/en-us/azure/devops/integrate/get-started/authentication/oauth?view=azure-devops

1.10 Environments. The REST reference for Environments List (api-version 7.1) lists exactly one scope, `vso.environment_manage`. Confidence: High for Services at 7.1. Not checked for Server API versions.
- https://learn.microsoft.com/en-us/rest/api/azure/devops/distributedtask/environments/list?view=azure-devops-rest-7.1

1.11 PATs do not work with every API: "Some APIs, including the Organizations, Profiles, and PAT Lifecycle Management APIs, require Microsoft Entra tokens." Confidence: High. This is a Services concern.
- https://learn.microsoft.com/en-us/azure/devops/organizations/accounts/use-personal-access-tokens-to-authenticate?view=azure-devops-2022

1.12 Admin policies. The page "Manage personal access tokens using policies" carries the Services moniker only and lists "Your organization is linked to a Microsoft Entra tenant" as a prerequisite. It describes: restrict global PAT creation (tenant), restrict full-scoped PAT creation (tenant), enforce maximum PAT lifespan in days (tenant, off by default), restrict PAT creation with an allowlist (organization, Entra-backed organizations only), automatic revocation of leaked PATs (tenant, on by default). In the PAT page, the policy, audit and lifecycle API sections are inside `moniker range="azure-devops"` blocks, which means they are shown for Services only. Confidence: High that these policies are Services features. Medium that Server has no equivalent, because that is an absence in the docs, not a statement.
- https://learn.microsoft.com/en-us/azure/devops/organizations/accounts/manage-pats-with-policies-for-administrators?view=azure-devops
- https://learn.microsoft.com/en-us/azure/devops/organizations/accounts/use-personal-access-tokens-to-authenticate?view=azure-devops-2022

1.13 On Services with the "Restrict personal access token creation" policy on, users not on the allowlist can only create packaging-scope PATs (if that sub-policy is ticked) or none. A read-only reporting PAT then needs an allowlist entry. Confidence: High.
- https://learn.microsoft.com/en-us/azure/devops/organizations/accounts/manage-pats-with-policies-for-administrators?view=azure-devops

1.14 Services only: "For organizations backed by Microsoft Entra ID, sign in with a new PAT within 90 days or it becomes inactive", and PATs can stop working until the owner completes an interactive sign-in, depending on Conditional Access. The 2020 page version says signing in "once every 30 days is sufficient for many users". Confidence: High for Services. Whether anything similar applies on Server with Active Directory is not stated.
- https://learn.microsoft.com/en-us/azure/devops/organizations/accounts/use-personal-access-tokens-to-authenticate?view=azure-devops-2022
- https://learn.microsoft.com/en-us/azure/devops/organizations/accounts/use-personal-access-tokens-to-authenticate?view=azure-devops-2020

1.15 PAT format: 84 characters, fixed `AZDO` signature at positions 76 to 80. Older page versions say the format changed in July 2024 and that 52-character tokens remain valid. Whether Server issues the new format is not stated per version. Confidence: High for Services.
- https://learn.microsoft.com/en-us/azure/devops/organizations/accounts/use-personal-access-tokens-to-authenticate?view=azure-devops-2022

### 2. Windows integrated authentication on Server

2.1 Product level: "Security for Azure DevOps Server is integrated with and relies on Windows integrated authentication and the security features of the Windows operating system." Kerberos can be configured after installation. Confidence: High. Moniker: `azure-devops` on a Server architecture page.
- https://learn.microsoft.com/azure/devops/server/architecture/key-concepts?view=azure-devops#authentication

2.2 The application tier's network authentication provider is set with `TFSConfig Authentication /provider:NTLM|Negotiate`. Confidence: High. Moniker: `azure-devops-server`.
- https://learn.microsoft.com/azure/devops/server/command-line/tfsconfig-cmd?view=azure-devops-server#authentication

2.3 Microsoft's integration guidance names Windows authentication as a supported choice for on-premises integrations: "For on-premises scenarios, use .NET client libraries, Windows authentication, or personal access tokens", the scenario table row "Azure DevOps Server apps ... .NET client libraries or Windows Auth", and "Use .NET client libraries with Windows Authentication when possible." The OAuth page repeats the first sentence. Confidence: High that it is named as supported. Monikers: Services, Server, Server 2022.
- https://learn.microsoft.com/en-us/azure/devops/integrate/get-started/authentication/authentication-guidance?view=azure-devops
- https://learn.microsoft.com/en-us/azure/devops/integrate/get-started/authentication/oauth?view=azure-devops

2.4 Documentation depth for REST is thin. The current "Get started with the REST APIs" page lists only Entra ID, PATs, and service principals or managed identities under Authentication, and does not mention Windows authentication, although it gives Server URL forms. The current ".NET client library samples" page (2022 and 2020 views) has no Windows credential example for a non-interactive service. The only Windows-flavoured sample there is `VssClientCredentials` for interactive use, marked ".NET Framework only - not supported in .NET Core/.NET 5+". Confidence: High that no REST-with-Windows-auth walkthrough exists on the pages read.
- https://learn.microsoft.com/en-us/azure/devops/integrate/how-to/call-rest-api?view=azure-devops-2022
- https://learn.microsoft.com/en-us/azure/devops/integrate/get-started/client-libraries/samples?view=azure-devops-2022
- https://learn.microsoft.com/en-us/azure/devops/integrate/get-started/client-libraries/samples?view=azure-devops-2020

2.5 The Microsoft auth samples repository states its samples cover "Azure DevOps and Azure DevOps Server" and includes a "Dual Support (Azure DevOps/TFS) Client Sample" whose README describes "seemless Windows credential authentication for TFS users" and its audience as "windows native applications". Confidence: High that the sample exists. It was written for Windows desktop apps, and the code file itself could not be fetched (HTTP 404 on the path tried).
- https://raw.githubusercontent.com/microsoft/azure-devops-auth-samples/master/README.md
- https://raw.githubusercontent.com/microsoft/azure-devops-auth-samples/master/DualSupportClientSample/README.md

2.6 The .NET client library page states Server compatibility by package version: 16.205.x for Server 2022 and later, 16.170.x for Server 2020 and later, and says the NetStandard 2.0 packages "correlate with Azure DevOps Server". It adds that interactive-dialog authentication paths are not available in the .NET Standard version. Confidence: High. Server 2019 is not in the mapping table of the view read.
- https://learn.microsoft.com/en-us/azure/devops/integrate/concepts/dotnet-client-libraries?view=azure-devops-2022

2.7 Pipelines agent, as evidence of what the Server accepts: registration methods "Integrated" and "Negotiate" are "Not supported" on Services and "Windows agents only" on Azure DevOps Server and TFS. Both need Windows Authentication enabled in IIS on the Server with NTLM, Kerberos or Negotiate providers. Confidence: High. This is the only Azure DevOps statement found that ties Windows authentication to the client operating system, and it is about the agent, not about arbitrary REST clients.
- https://learn.microsoft.com/en-us/azure/devops/pipelines/agents/agent-authentication-options?view=azure-devops-2022

2.8 The same page notes that IIS anonymous authentication is required on Server for the OAuth tokens that agents and pipelines use, and that the application enforces authentication behind the IIS front door. Confidence: High.
- Same URL as 2.7.

2.9 Generic .NET behaviour, not specific to Azure DevOps: `HttpClientHandler.UseDefaultCredentials = true` (or `CredentialCache.DefaultCredentials`) authenticates as the account the process runs under when the server asks. Negotiate picks Kerberos when possible and otherwise NTLM. Confidence: High.
- https://learn.microsoft.com/dotnet/api/system.net.http.httpclienthandler.usedefaultcredentials?view=net-10.0
- https://learn.microsoft.com/dotnet/framework/network-programming/ntlm-and-kerberos-authentication
- https://learn.microsoft.com/windows/win32/secauthn/microsoft-negotiate

2.10 Linux difference in .NET: "On Unix platforms, use of NTLM authentication may require installing the NTLM plugin for the GSSAPI layer ... On Ubuntu distributions, the package is called `gss-ntlmssp`. Alternatively, applications may opt in for a managed implementation of NTLM by setting the `_UseManagedNtlm` .csproj property (.NET 9+ only), or setting the `System.Net.Security.UseManagedNtlm` AppContext switch." Kerberos on Linux depends on the host or container being set up for the domain. The ASP.NET Core Windows Authentication article points to instructions for joining a Linux machine to a Windows domain, but that article covers the server side of Negotiate, not an outbound client. Confidence: High for the NTLM note. Medium for Kerberos client setup, which was not read in a dedicated client article.
- https://learn.microsoft.com/dotnet/api/system.net.networkcredential?view=net-10.0
- https://learn.microsoft.com/aspnet/core/security/authentication/windowsauth?view=aspnetcore-10.0

2.11 Microsoft's own .NET guidance calls NTLM "a legacy authentication protocol" to be used "only when required for compatibility with existing systems". Confidence: High.
- https://learn.microsoft.com/dotnet/framework/network-programming/ntlm-and-kerberos-authentication

2.12 Service accounts and access level on Server: "Add Azure DevOps service accounts to the default access level. If you make Stakeholder the default access level, you must add the service accounts to Basic, Basic + Test Plans, or VS Enterprise access. Service accounts don't require a CAL or other purchase." This section is about Azure DevOps Server's own service accounts. Whether a third-party tool's domain account needs a CAL is a licensing question that was not researched. Confidence: High for the quoted text. Monikers: Server 2022 and unversioned Server.
- https://learn.microsoft.com/en-us/azure/devops/organizations/security/access-levels?view=azure-devops-2022

### 3. What is not available on Server, and the Services direction

3.1 "OAuth 2.0 and Microsoft Entra ID authentication are available for Azure DevOps Services only, not Azure DevOps Server." Confidence: High as a general Server statement. Monikers on the page: Services, Server, Server 2022. The docs do not break this down by Server version.
- https://learn.microsoft.com/en-us/azure/devops/integrate/get-started/authentication/authentication-guidance?view=azure-devops

3.2 "OAuth 2.0 is available only for Azure DevOps Services, not Azure DevOps Server." Confidence: High. Moniker: Services.
- https://learn.microsoft.com/en-us/azure/devops/integrate/get-started/authentication/oauth?view=azure-devops

3.3 The "Use service principals and managed identities" page carries the Services moniker only, and requires "An Azure DevOps organization connected to a Microsoft Entra tenant". Confidence: High.
- https://learn.microsoft.com/en-us/azure/devops/integrate/get-started/authentication/service-principal-managed-identity?view=azure-devops

3.4 Pipelines agent table: Service Principal and Device code flow (Entra ID) are "Currently not supported" on "Azure DevOps Server & TFS". Confidence: High.
- https://learn.microsoft.com/en-us/azure/devops/pipelines/agents/agent-authentication-options?view=azure-devops-2022

3.5 The unversioned Server release (RC 2025-10-07, RTW 2025-12-09, patches 1 to 9 up to 2026-10-13): the release notes were read in full and contain no entry that adds Entra ID, service principal, managed identity or OAuth app authentication for callers of the Server. The only Entra item is pipelines accessing Azure Service Bus. Confidence: Medium (absence of a release note, combined with 3.1).
- https://learn.microsoft.com/en-us/azure/devops/server/release-notes/azuredevopsserver

3.6 Per-version confirmation for 2019 and 2020: the 2019 and 2020 views of the PAT and samples pages recommend Entra tokens and service principals in their boilerplate, without restricting that advice to Services. Those sentences are inherited from the Services text and conflict with 3.1 and 3.2. No statement was found that Entra authentication works on Server 2019 or 2020. Confidence that it is not available on 2019 and 2020: Medium.
- https://learn.microsoft.com/en-us/azure/devops/organizations/accounts/use-personal-access-tokens-to-authenticate?view=azure-devops-2019
- https://learn.microsoft.com/en-us/azure/devops/integrate/get-started/client-libraries/samples?view=azure-devops-2020

3.7 Services direction, from the authentication guidance: "Use Microsoft Entra ID authentication for new applications that integrate with Azure DevOps Services. Use personal access tokens sparingly, and only when Microsoft Entra ID isn't available." For "Service/background apps" the recommended method is "Service principals and managed identities". PATs are described as "Highest risk of the common choices" and to be avoided when "The integration is a production service, shared automation". Confidence: High.
- https://learn.microsoft.com/en-us/azure/devops/integrate/get-started/authentication/authentication-guidance?view=azure-devops

3.8 Same page, for Server: "Choose a PAT only for personal, temporary, legacy, or Azure DevOps Server scenarios where the more secure options don't apply", and for a tool that targets both: "Azure DevOps Services: Use Microsoft Entra ID OAuth. Azure DevOps Server: Use .NET client libraries with Windows Authentication or personal access tokens." Confidence: High.
- Same URL as 3.7.

3.9 Service principals and managed identities on Services: must be added explicitly to each organization as a user, get an access level and group memberships like any member, cannot create PATs or SSH keys, cannot sign in interactively. Token resource for client credentials: `https://app.vssps.visualstudio.com/.default`, sent as a Bearer token. "Azure Repos access requires Basic or a higher functional access level." They have the same rate limits as users. Confidence: High.
- https://learn.microsoft.com/en-us/azure/devops/integrate/get-started/authentication/service-principal-managed-identity?view=azure-devops

3.10 Global PAT retirement on Services. Sprint 270 release notes: creation and regeneration block on 2026-03-15 is struck through in the text, and "December 1, 2026, all existing global PATs will be fully decommissioned and stop working." The PAT page banner says "All existing global PATs stop working on January 27, 2027", while the same page's best-practices section says "Global PATs stop working on December 1, 2026." The two dates conflict. Organization-scoped PATs are the stated replacement together with Entra authentication. Confidence: High that global PATs are being retired and that organization-scoped PATs remain. Medium on the date.
- https://learn.microsoft.com/en-us/azure/devops/release-notes/2026/sprint-270-update
- https://learn.microsoft.com/en-us/azure/devops/organizations/accounts/use-personal-access-tokens-to-authenticate?view=azure-devops-2022

3.11 No statement was found on Learn that organization-scoped PATs on Services have a retirement date. One older page version (samples page, `view=azure-devops-2020`) says "Personal access tokens are being deprecated." The newer view of the same page (`view=azure-devops-2022`) no longer says that and instead says "Consider using the more secure Microsoft Entra tokens over higher-risk personal access tokens." Confidence: Medium that there is no announced end date for organization-scoped PATs, since the blog could not be read.
- https://learn.microsoft.com/en-us/azure/devops/integrate/get-started/client-libraries/samples?view=azure-devops-2020
- https://learn.microsoft.com/en-us/azure/devops/integrate/get-started/client-libraries/samples?view=azure-devops-2022

3.12 Azure DevOps OAuth (the non-Entra one): "deprecated and scheduled for removal in 2026", "New app registrations are no longer accepted as of April 2025", existing apps "stop working when the service is fully deprecated in 2026". No exact day is given on Learn. The linked blog post slug says "beginning-february-2025", which differs from the "April 2025" in the page text. Confidence: High on deprecation, Medium on dates.
- https://learn.microsoft.com/en-us/azure/devops/integrate/get-started/authentication/azure-devops-oauth?view=azure-devops
- https://learn.microsoft.com/en-us/azure/devops/integrate/get-started/authentication/oauth?view=azure-devops

3.13 Tokens must be treated as opaque: "Starting summer 2025, Azure DevOps is further encrypting authentication tokens", so clients that decode token claims break. Confidence: High.
- https://learn.microsoft.com/en-us/azure/devops/integrate/get-started/authentication/authentication-guidance?view=azure-devops

### 4. Permissions for a read-only identity

4.1 No page was found that defines a minimal permission set for reporting or analytics tools. The closest material is the default permissions quick reference, which shows what the built-in Readers group can do. Confidence: High that nothing was found in the pages read. This is not proof that no such page exists.

4.2 "To access project-level resources, the View project-level information permission must be set to Allow. This permission gates all other project-level permissions." Valid Users groups get read permissions such as View project-level information and View collection-level information by default, and Microsoft advises not to change them. Confidence: High. Moniker: `azure-devops` (page applies to Server too).
- https://learn.microsoft.com/azure/devops/organizations/security/permissions?view=azure-devops#project-level-permissions
- https://learn.microsoft.com/azure/devops/organizations/security/about-permissions?view=azure-devops#security-groups-and-membership

4.3 Readers group defaults, from the quick reference (view `azure-devops-2022`, which also renders the Services and unversioned Server variants):
- Work items: "View work items in this node" (an Area Path permission) is granted. Edit is not.
- Git: "Read (clone, fetch, and explore the contents of a repository)" is granted. The same row also includes "can create, comment on, vote, and Contribute to pull requests", so Readers is not purely read-only for pull requests.
- TFVC: "Read only".
- Build (Server 2022 table): "View builds" and "View build pipeline" granted, nothing else.
- Release (Server table): "View releases" and "View release pipeline" granted.
- Services table: "View release pipelines" and "Can view library items such as variable groups" granted.
- Test: "View test runs" granted.
- Dashboards: view granted.
- Analytics: the table marks "View Analytics" for Readers.
Confidence: High that the table says this. "Members of the Readers, Contributors, or Project Administrators group are assumed to have at least Basic access."
- https://learn.microsoft.com/en-us/azure/devops/organizations/security/permissions-access?view=azure-devops-2022

4.4 Conflict on View analytics. The Analytics security page says: "By default, the View analytics permission is set for all Contributors with Basic access", and "By default, all members of the Contributors group are granted access to ... view Analytics data." The quick reference in 4.3 marks View Analytics for Readers as well. The two pages disagree about Readers. Confidence: Medium. This needs a check on a real instance.
- https://learn.microsoft.com/en-us/azure/devops/report/powerbi/analytics-security?view=azure-devops-2022
- https://learn.microsoft.com/en-us/azure/devops/organizations/security/permissions-access?view=azure-devops-2022

4.5 Analytics behaviour that affects a reader:
- "Analytics does not support security at the area path level." A user with project access and View analytics sees all work item data in the project through Analytics, even for areas they cannot read through the work item APIs.
- Analytics does not trim results by security. A query that touches a project the caller cannot access returns a "Project access denied" error and no data. Microsoft recommends always using a project-level filter.
- Users with Stakeholder access "don't have permissions to view or edit Analytics views".
Confidence: High. Monikers: Services, Server, Server 2022.
- https://learn.microsoft.com/en-us/azure/devops/report/powerbi/analytics-security?view=azure-devops-2022

4.6 Analytics availability: "generally available for Azure DevOps Services and Azure DevOps Server 2020 and later versions. It is in preview for Azure DevOps Server 2019." On Server 2020 and later it is enabled automatically for new collections. Upgraded collections may need it enabled manually by a Project Collection Administrator, and it can be paused or disabled. Confidence: High.
- https://learn.microsoft.com/azure/devops/report/powerbi/what-is-analytics?view=azure-devops
- https://learn.microsoft.com/azure/devops/report/dashboards/analytics-extension?view=azure-devops-server

4.7 Stakeholder access breaks reads that a delivery-metrics tool needs:
- "Stakeholders don't have access to code repositories (Azure Repos)."
- Access level table: the Code feature and the Build feature are not marked for Stakeholder. Text: "Stakeholders can view builds but can't create, edit, or manage build pipelines."
- On Server, Stakeholder has no Azure Artifacts access ("Azure DevOps Server requires Basic or a higher access level").
- No Analytics views (4.5).
- Release (Server table): Stakeholders can view releases but "View release pipeline" is not marked for them.
Confidence: High. Monikers: Services, Server, Server 2022.
- https://learn.microsoft.com/en-us/azure/devops/organizations/security/stakeholder-access?view=azure-devops-2022
- https://learn.microsoft.com/en-us/azure/devops/organizations/security/access-levels?view=azure-devops-2022
- https://learn.microsoft.com/en-us/azure/devops/organizations/security/permissions-access?view=azure-devops-2022

4.8 Project-Scoped Users. The section is inside a Services-only moniker block. It is a preview feature ("Limit user visibility and collaboration to specific projects"). Members see only projects they are explicitly added to and get limited organization settings views. Microsoft's own warning: "The limited visibility features described in this section apply only to interactions through the web portal. By using the REST APIs or az devops CLI commands, project members can access the restricted data." Confidence: High that it is a Services feature and that Microsoft says it does not restrict REST. Its effect on specific collection-level REST calls made by a scoped identity was not tested.
- https://learn.microsoft.com/en-us/azure/devops/user-guide/manage-organization-collection?view=azure-devops#project-scoped-user-group
- https://learn.microsoft.com/azure/devops/organizations/security/permissions?view=azure-devops#collection-level-groups

4.9 Server-level and collection-level "View instance-level information" is assigned to the Valid Users groups by default. Confidence: High.
- https://learn.microsoft.com/azure/devops/organizations/security/permissions?view=azure-devops#server-level-permissions

4.10 On Server, users are added through access levels and "must also be added to a project to connect and access features through a client or the web portal". Confidence: High. Monikers: Server 2022 and unversioned Server.
- https://learn.microsoft.com/en-us/azure/devops/user-guide/manage-organization-collection?view=azure-devops

### 5. SSH and basic authentication

5.1 SSH is for Git: "securely authenticate to Azure Repos Git repositories". It works on Server: "Can I use SSH with Azure DevOps Server? Yes." Keys are tied to a user profile. Only RSA keys are supported, with RSA-SHA2 signatures. The SHA-1 `ssh-rsa` signature is deprecated or unsupported on Services, and may still be the only host key offer on "older self-hosted Azure DevOps Server configurations". Git LFS over SSH is not supported on Services. Confidence: High. Monikers: Services, Server, Server 2022. No statement was found that SSH can authenticate REST calls. The page scope is Git only.
- https://learn.microsoft.com/en-us/azure/devops/repos/git/use-ssh-keys-to-authenticate?view=azure-devops-2022

5.2 On Services, SSH can be disabled by the organization policy "SSH authentication", and "Validate SSH key expiration" is on by default. SSH keys also stop working if the owner has not completed an interactive sign-in recently (same pattern as PATs). Confidence: High.
- https://learn.microsoft.com/en-us/azure/devops/organizations/accounts/change-application-access-policies?view=azure-devops
- https://learn.microsoft.com/en-us/azure/devops/repos/git/use-ssh-keys-to-authenticate?view=azure-devops-2022

5.3 Alternate credentials on Services: "Azure DevOps doesn't support Alternate Credentials authentication." Confidence: High on the statement. The date and details are in a blog post that could not be read.
- https://learn.microsoft.com/en-us/azure/devops/organizations/accounts/change-application-access-policies?view=azure-devops

5.4 Basic authentication with a domain password on Server: Microsoft recommends keeping IIS Basic Authentication off, and enabling it stops PATs from working (1.5). The agent documentation still lists an "Alternate (ALT)" basic-auth registration method for Server over HTTPS that requires configuring basic authentication on the server. The SOAP samples page labels username and password authentication "deprecated and insecure". Confidence: High that it is discouraged. No formal deprecation notice for IIS Basic on Server was found.
- https://learn.microsoft.com/en-us/azure/devops/organizations/accounts/use-personal-access-tokens-to-authenticate?view=azure-devops-2022
- https://learn.microsoft.com/en-us/azure/devops/pipelines/agents/agent-authentication-options?view=azure-devops-2022
- https://learn.microsoft.com/azure/devops/integrate/get-started/client-libraries/soapsamples?view=azure-devops#legacy-authentication-methods

### 6. Identity data returned by the APIs

6.1 `IdentityRef` is the object embedded in most REST responses for people (created by, assigned to, reviewers, requested for). Fields in the 7.1 reference: `displayName` ("non-unique display name"), `uniqueName` ("Deprecated - use Domain+PrincipalName instead"), `id`, `descriptor` ("the primary way to reference the graph subject"), `url`, `_links` (includes `avatar`), `imageUrl` (deprecated), `directoryAlias` (deprecated), `profileUrl` (deprecated), `inactive`, `isAadIdentity`, `isContainer`, `isDeletedInOrigin`. Confidence: High for Services at API 7.1.
- https://learn.microsoft.com/en-us/rest/api/azure/devops/distributedtask/environments/list?view=azure-devops-rest-7.1

6.2 Identifiers also appear inside URLs. The Graph sample response shows `url`, `_links.self`, `_links.avatar` and others each containing the user's descriptor. A tool that pseudonymizes people has to treat these link fields as identifying. Confidence: High.
- https://learn.microsoft.com/en-us/rest/api/azure/devops/graph/users/list?view=azure-devops-rest-7.1

6.3 `GraphUser` (Graph Users List, 7.1-preview.1, scope `vso.graph`) adds: `principalName`, `mailAddress` ("may be different than the principal name"), `domain` ("for AD the name of the domain, for AAD the tenantID"), `origin` ("AD, AAD, MSA"), `originId` ("Typically a sid, object id or Guid", can change when a user is relinked), `directoryAlias`, `subjectKind`, `metaType`. `principalName` "is not guaranteed to be immutable". The sample response includes service identities such as "Project Collection Build Service", which shows that not every identity is a person. Confidence: High for Services. The documented host is `vssps.dev.azure.com`. Availability on Server is not established.
- https://learn.microsoft.com/en-us/rest/api/azure/devops/graph/users/list?view=azure-devops-rest-7.1

6.4 Work item identity fields: "In Azure DevOps Services REST API version 5.0 and later, the service returns identities as IdentityRef objects instead of strings." The work item field metadata has an `isIdentity` attribute that marks identity fields, which includes custom identity fields. Confidence: High for Services. Medium for Server, where the same API versions exist but the statement is worded for Services.
- https://learn.microsoft.com/azure/devops/extend/develop/public-project?view=azure-devops#work-item-tracking-considerations
- https://learn.microsoft.com/azure/devops/boards/work-items/work-item-fields?view=azure-devops#field-attributes

6.5 Stability of identifiers, from the field descriptions: `displayName` is non-unique and changes at the source provider, `principalName` can change, `originId` can change on relink, and `descriptor` is described as the primary reference that "will uniquely identify the same graph subject across both Accounts and Organizations". Confidence: High that the docs say this. Behaviour on Server with Active Directory was not verified.
- https://learn.microsoft.com/en-us/rest/api/azure/devops/graph/users/list?view=azure-devops-rest-7.1

### 7. Version facts used above

7.1 REST API major versions: Server 2019 supports up to 5.0, Server 2020 up to 6.0, Server 2022 up to 7.0 according to the supported versions table, which ends at 7.0 and does not list the unversioned release. Confidence: High for what the table says.
- https://learn.microsoft.com/en-us/azure/devops/integrate/concepts/rest-api-versioning?view=azure-devops-2022

7.2 The unversioned "Azure DevOps Server" RTW release date is 2025-12-09. It upgrades directly from Server 2022, 2020, 2019 or TFS 2015 and newer. Patch 9 is dated 2026-10-13. Patch 6 (2026-07-21) "Added validation to ensure Azure DevOps Server only connects to trusted Active Directory domains when resolving identities." Patch 1 (2026-02-10) "Disabled the ability to use a PAT with the EndpointProxy API when an internal EndpointId is specified." Confidence: High.
- https://learn.microsoft.com/en-us/azure/devops/server/release-notes/azuredevopsserver

7.3 Support end dates under the Fixed Lifecycle Policy: Server 2019 on 2029-04-10, Server 2020 on 2030-10-08 (mainstream ended 2025-10-14), Server 2022 on 2033-01-11 (mainstream ends 2028-01-11). Confidence: High. The lifecycle policy of the unversioned release was not read.
- https://learn.microsoft.com/lifecycle/end-of-support/end-of-support-2029
- https://learn.microsoft.com/lifecycle/end-of-support/end-of-support-2030
- https://learn.microsoft.com/lifecycle/end-of-support/end-of-support-2033

## Least-privilege read-only setup, as far as the sources support it

These are assemblies of the facts above, not Microsoft-published recipes (see 4.1).

Azure DevOps Server (all versions in scope):
- Identity: a dedicated Active Directory account, added to the collection with Basic access (4.7, 4.10) and to the Readers group of each project to be read (4.3). View analytics must be confirmed or granted explicitly because the docs conflict (4.4).
- Credential, option A: a PAT created by that account, scoped to one collection, with read scopes from 1.8. IIS Basic Authentication must be off (1.5).
- Credential, option B: Windows authentication as that account (2.1 to 2.3). The docs do not describe scoping for this path. A Windows-authenticated session is not limited by PAT scopes, so the identity's permissions are the only limit.
- Environments need `vso.environment_manage` if read with a PAT (1.10).

Azure DevOps Services:
- Identity: a service principal or managed identity added to the organization with Basic access and Readers membership (3.9, 4.3), or a user account if a PAT is used.
- Credential: Entra token for the service principal or managed identity (3.7, 3.9). A PAT remains possible: organization-scoped, read scopes only, subject to tenant and organization PAT policies (1.12, 1.13, 3.10).

## Unverified

1. Maximum PAT lifetime on Server, and on Services when no tenant policy is set. Tried: both PAT pages in four monikers, the policy page, two Learn searches. No number is stated anywhere that was read.
2. Which PAT scopes each Server version offers in its token dialog (in particular `vso.analytics`, `vso.graph`, `vso.environment_manage` on 2019 and 2020). The only scope table is Services-scoped. No Server instance was available.
3. Whether PATs on Server work over plain HTTP for REST. The only HTTPS statement found is about agent registration (1.7).
4. The meaning of "Windows Auth also prevents using PATs" (1.6) for REST clients on Server.
5. A working, documented example of a non-interactive .NET (modern, cross-platform) client calling Server REST APIs with Windows authentication. The guidance names the option (2.3) but no walkthrough was found. The sample code file in the Microsoft repository returned 404 at the path tried.
6. Windows authentication to Server from a Linux container, including Kerberos with a keytab and group managed service accounts on Linux. Tried: Learn searches for Kerberos, gMSA and Linux containers. Only Windows-container gMSA pages and the generic .NET NTLM-on-Unix note came back. Nothing Azure DevOps specific.
7. Whether a group managed service account can own a PAT on Server. A PAT has to be created in the web portal by the signed-in user, and no page discusses doing that for an account that cannot sign in interactively.
8. Whether Graph REST endpoints (`_apis/graph/...`) and the `descriptor` field exist on each Server version. The reference pages show only the `vssps.dev.azure.com` host.
9. Whether Readers have View analytics by default (4.4). The two Learn pages disagree.
10. Environment-level permissions (the Reader role on pipeline environments) and whether project Readers can list environments. Not researched beyond the PAT scope.
11. Exact dates and wording of the Azure DevOps blog announcements: "Reducing PAT usage across Azure DevOps", "Retirement of Global Personal Access Tokens in Azure DevOps", "No new Azure DevOps OAuth apps", alternate credentials retirement, and the Server GA post. `devblogs.microsoft.com` was not reachable from this session. The host needs to be allowed in the session network settings to read them.
12. The global PAT shutdown date: 2026-12-01 or 2027-01-27 (3.10).
13. Whether Microsoft has announced any end date for organization-scoped PATs on Services (3.11).
14. Whether Entra authentication is planned for a future Server release. No roadmap page was read.
15. Per-version confirmation for Server 2019 and 2020 that SSH and Windows authentication behave as on 2022. The pages that state these facts carry only the Services, Server and Server 2022 monikers.
16. Identity fields returned by Git itself (commit author and committer name and email). These come from Git data, not from the Azure DevOps identity service. No source was fetched for them.
17. Whether a non-Microsoft tool's read-only domain account needs a client access licence on Server (2.12).
18. The highest REST API version of the unversioned Server release (7.1).

## Implications for the authentication model decision

Facts only.

- There is no single credential type that Microsoft documents for both Server and Services background services. Server: PAT or Windows authentication. Services: Entra token recommended, PAT still accepted.
- Both Server credentials are bound to a user identity. Server has no documented non-user (application) identity.
- PAT handling is the one mechanism documented on all five targets, with the same wire format (HTTP Basic).
- A PAT expires and has to be replaced by its owner. No PAT lifecycle API or admin policy is documented for Server. On Services the lifecycle APIs need an Entra user token.
- PAT scopes limit a token below its owner's permissions. Windows authentication has no documented equivalent, so the identity's group membership is the only limit on that path.
- Published "read" scopes for work, code and build include service hook write, and environments have no read-only scope.
- Windows authentication is documented by Microsoft for Windows clients. For Linux hosts the only relevant sources are generic .NET notes, and the agent, Microsoft's own cross-platform client, restricts Integrated and Negotiate to Windows.
- The Readers group can comment on and vote on pull requests by default. Readers is the least-privileged built-in project group but is not strictly read-only.
- Stakeholder access cannot read code or Analytics, so the reading identity needs Basic access on every target.
- Analytics is not security-trimmed at area path level and fails a whole query on any inaccessible project. Analytics is preview on Server 2019 and may be disabled per collection on later versions.
- Project-Scoped Users exists on Services only, and Microsoft states it does not restrict REST access.
- Identity data arrives in several shapes: display name, unique or principal name (often an email or DOMAIN\user), mail address, GUID id, descriptor, origin id (possibly a SID), and URLs that embed the descriptor. Service identities appear alongside people.
- Services policies can block or shorten PATs per tenant or organization, so a PAT-based setup on Services depends on the customer's policy settings.
- Server 2019 tops out at REST API 5.0 and Server 2020 at 6.0, which bounds which documented scopes and fields can be assumed there.
