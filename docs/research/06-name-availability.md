# 06. Name availability: "Flowline"

Read dates: 2026-11-10 and 2026-11-11 (each row says which). This is a quick public-source check, not a trademark clearance and not legal advice.

## Short answer

**"Flowline" is not usable as it stands for a .NET project.** The exact NuGet package id `Flowline` is taken by an actively maintained .NET CLI tool in the Microsoft ecosystem (a Dataverse ALM CLI for Dynamics 365 and Power Apps, version 0.21.0 published 2026-11-09), and the same owner also publishes `Flowline.Attributes`. The GitHub organisation name `Flowline` is taken as well. PyPI `flowline` and the Docker Hub account `flowline` are taken. Secondary sources list live FLOWLINE trademark registrations in software classes, which I could not confirm on the registers themselves.

The clash on NuGet is the deciding point: same registry, same language, same Microsoft tooling audience, and a tool that is released frequently. A rename is recommended. Of the alternatives checked, **Flowgauge**, **Flowtally**, **Deliverylens** and **Flowcompass** came back clear on both the NuGet id and the GitHub account name.

## 1. NuGet

| Item | Result | Evidence | Read |
|---|---|---|---|
| Package id `Flowline` | **Taken.** Version 0.21.0, published 2026-11-09. Described as "a Dataverse ALM CLI for Dynamics 365 and Power Apps". 21 versions from 0.1.0 to 0.21.0. Source repo is RemyDuijkeren/Flowline (C#). | https://www.nuget.org/packages/Flowline and https://api.nuget.org/v3-flatcontainer/flowline/index.json | 2026-11-10 |
| `Flowline.Attributes` | **Taken**, same owner, 0.21.0, published 2026-11-09. Source-only package of plugin step registration attributes for the same CLI. | https://www.nuget.org/packages?q=flowline and https://api.nuget.org/v3/registration5-gz-semver2/flowline.attributes/index.json | 2026-11-10 and 2026-11-11 |
| `Flowline.Core`, `.Cli`, `.Abstractions`, `.Tool`, `.Sdk`, `.Client`, `.Api`, `.Web` | Not registered (HTTP 404 on each). | https://api.nuget.org/v3-flatcontainer/flowline.core/index.json (same pattern for the others) | 2026-11-10 |
| Prefix `Flowline.` reserved | **Unverified, probably not reserved.** The package page HTML contained no reserved-prefix marker, but I could not read the `verified` flag from the search API because that host was blocked. Even without a reservation, the existing owner already publishes under the prefix. | https://www.nuget.org/packages/Flowline | 2026-11-10 |
| NuGet search API | **Blocked.** The proxy refused the connection (HTTP 403 on CONNECT) for both azuresearch-usnc and azuresearch-ussc. The nuget.org web search was used instead and listed only `Flowline` and `Flowline.Attributes`. | https://azuresearch-usnc.nuget.org/query?q=flowline | 2026-11-10 |

## 2. GitHub

| Item | Result | Evidence | Read |
|---|---|---|---|
| Account `flowline` | **Taken.** An organisation named `Flowline` exists. A repository search scoped to it returned zero public repositories, so it looks dormant or private. The profile page itself could not be opened (HTTP 403 through the proxy). | https://api.github.com/search/users?q=flowline+in:login and https://github.com/Flowline | 2026-11-11 |
| Similar accounts | `flowline-io` (organisation, active), `flowlines-ai`, `Flowline-Health`, `flowlinejs` (organisations), plus several users. 48 logins contain "flowline". | https://api.github.com/search/users?q=flowline+in:login | 2026-11-11 |
| Repositories named flowline | 307 repositories match on name. | https://api.github.com/search/repositories?q=flowline+in:name&sort=stars | 2026-11-10 |

Notable repositories (stars as read on 2026-11-10):

| Repository | Stars | Language | What it is | Relevance |
|---|---|---|---|---|
| https://github.com/Dramwig/FlowLine | 396 | JavaScript | Automated tool for running Python programs in a streamlined manner | Developer tooling, most starred |
| https://github.com/Alezander9/Flowline | 29 | HTML | No description | Low |
| https://github.com/rochus-keller/FlowLine2 | 16 | C++ | Modelling tool for functional analysis and business process modelling | Low, last push 2022 |
| https://github.com/tsundata/flowline | 10 | Go | Workflow system for scheduling | Workflow tooling, last push 2024 |
| https://github.com/flowline-io/flowbot | 10 | Go | Homelab data hub under the flowline-io organisation, pushed 2026-11-10 | Active organisation with the name |
| https://github.com/RemyDuijkeren/Flowline | 4 | C# | Dataverse ALM CLI for Dynamics 365 and Power Apps, pushed 2026-11-09 | **High: .NET, Microsoft ALM tooling, owns the NuGet id** |
| https://github.com/Livsy90/Flowline | 3 | Swift | Library for preparing dependent operations in parallel | Low |
| https://github.com/kashmithnisakya/flowline | 2 | Jac | "Factory flow line for shipping software", pushed 2026-11-10 | Software delivery theme, very small |

None of these is an engineering-analytics or DORA product. The remaining matches are mostly glacier flowline models and UI effects.

## 3. Other registries

| Registry | Result | Evidence | Read |
|---|---|---|---|
| npm `flowline` | **Free.** Registry returns "Not found". | https://registry.npmjs.org/flowline | 2026-11-10 |
| PyPI `flowline` | **Taken.** Version 0.1.2, "Extract flowlines from a velocity field" (glaciology, unrelated field). | https://pypi.org/pypi/flowline/json | 2026-11-10 |
| Docker Hub `flowline` | **Taken.** User account `flowline`, joined 2023-01-10, zero public repositories. There is no official image `library/flowline` (404). | https://hub.docker.com/v2/users/flowline/ and https://hub.docker.com/v2/repositories/flowline/ | 2026-11-10 |

Images for this project could still be published to GitHub Container Registry under a different owner, but `docker.io/flowline/...` is not available.

## 4. Products, companies and trademarks

| Item | Result | Evidence | Read |
|---|---|---|---|
| Engineering analytics or DevOps product called exactly "Flowline" | None found in one web search. This is weak evidence, a single query. | Search results only, no product page found | 2026-11-11 |
| Nearby names in the same market | "Flow" is crowded in engineering analytics: Pluralsight Flow and Appfire Flow (software development analytics), and Flowyzer (automated engineering metrics). Any "Flow..." name sits close to these. | https://www.pluralsight.com/product/flow/executives , https://www.g2.com/products/appfire-flow/reviews , https://99x.io/services/product-engineering/flowyzer-automated-engineering-metrics | 2026-11-11 |
| US trademark FLOWLINE, serial 79121250 | Listed by aggregator sites as registered 2013 in classes 9, 41 and 42, owner Netflix, Inc., goods described as software for simulation and visualisation of physical processes for visual effects. **Not confirmed on the USPTO register.** | https://trademarks.justia.com/791/21/flowline-79121250.html | 2026-11-11 |
| US trademark FLOWLINE, serial 88455995 | Listed by aggregators as a class 9 registration from 2020 held by an individual, shown as active. Goods not read. **Not confirmed on the register.** | https://trademarkdir.com/trademark/flowline-88455995 | 2026-11-11 |
| US trademarks FLOWLINE, serials 74159443 and 74281012 | Listed as class 9 marks of Flowline Inc. covering sensors, monitors, indicators and controllers (hardware, level measurement). | https://www.trademarkelite.com/trademark/trademark-detail/74159443/FLOWLINE | 2026-11-11 |
| EU, UK, Benelux and WIPO registers | **Not checked.** The search surfaced UK and Australian journal entries but I did not open them. | https://www.ipo.gov.uk/t-tmj/tm-journals/2025-024/UK00004181501.html | 2026-11-11 |

The trademark rows come from search result summaries of third-party aggregator sites. I did not query USPTO, EUIPO, BOIP or WIPO directly. Treat them as leads to check, not as findings.

## 5. Domains

| Domain | Result | Evidence | Read |
|---|---|---|---|
| flowline.dev | **Unverified.** Proxy refused the connection (HTTP 403 on CONNECT). | https://flowline.dev | 2026-11-10 |
| flowline.io | **Unverified.** Same block. | https://flowline.io | 2026-11-10 |
| flowline.app | **Unverified.** Same block. | https://flowline.app | 2026-11-10 |
| flowline.com | **Unverified.** Same block. | https://flowline.com | 2026-11-10 |

A web search on 2026-11-11 returned no product at any of the first three, which says little. The domains need to be opened from an unrestricted network.

## Alternatives

NuGet check: exact id lookup at `https://api.nuget.org/v3-flatcontainer/<lowercase-id>/index.json`, where 404 means the id is not registered. GitHub check: `https://api.github.com/search/users?q=<name>+in:login`, looking for an exact login match (user or organisation). Both read 2026-11-11. Prefix reservation, domains and trademarks were not checked for any alternative.

| Name | Idea | NuGet id | GitHub account | Verdict |
|---|---|---|---|---|
| Flowgauge | A gauge for flow metrics | Free (404) | No exact match, 0 similar logins | **Clear** |
| Flowtally | Counting what flows through | Free (404) | No exact match, 0 similar logins | **Clear** |
| Deliverylens | A lens on delivery | Free (404) | No exact match, 0 similar logins | **Clear** |
| Flowcompass | Direction from flow data | Free (404) | No exact match, 1 similar login | **Clear** |
| Throughline | The line running through delivery | Free (404) | Taken (user `throughline`) | Taken on GitHub |
| Leadline | Lead time, sounding line | Free (404) | Taken (user `leadline`) | Taken on GitHub |
| Cyclelens | Cycle time lens | Free (404) | Taken (organisation `cyclelens`) | Taken on GitHub |
| Paceline | Team riding in formation | Free (404) | Taken (user `paceline`) | Taken on GitHub |
| Cadence | Delivery rhythm | Taken (200) | Taken (organisation) | Taken |

Also checked and taken on GitHub while the NuGet id was free: Flowmark, Shipgauge, Tideline, Flowledger, Shiptrace, Flowstat.

"Clear" here means only that the exact NuGet id returned 404 and no exact GitHub login appeared in search. GitHub search can miss accounts, so confirm by opening `https://github.com/<name>` before committing to a name. Names starting with "Flow" also sit near Pluralsight Flow and Appfire Flow; Deliverylens avoids that.

## Unverified

- **NuGet search API** (azuresearch-usnc and azuresearch-ussc): blocked by the proxy. The list of `Flowline.*` ids therefore rests on the nuget.org web search page and on eight direct id lookups. Other ids under the prefix may exist.
- **NuGet prefix reservation** for `Flowline.`: not confirmed either way.
- **GitHub profile pages** (github.com/flowline, github.com/flowline-io): blocked (HTTP 403). The organisation's existence comes from the GitHub search API only. Its activity, owner and whether it has private repositories are unknown.
- **GitHub availability of the four clear alternatives**: based on search API only, not on opening the profile URL.
- **Domains** flowline.dev, flowline.io, flowline.app, flowline.com: all blocked. Nothing is known about what they host.
- **Trademark registers**: none queried directly. All trademark statements come from search summaries of aggregator sites and may be out of date or wrong. EU, UK, Benelux and WIPO were not checked at all.
- **Existing products named Flowline**: one web search only. Absence of a result is not evidence that no such product exists.
- **Alternatives**: no check of prefix reservation, npm, PyPI, Docker Hub, domains or trademarks.
- **Download counts** for the existing `Flowline` NuGet package: not read.

This document is not legal advice. Have the chosen name checked against the relevant trademark registers before public release.
