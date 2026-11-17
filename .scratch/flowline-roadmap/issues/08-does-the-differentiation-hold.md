# Does the differentiation hold

Type: grilling (HITL)
Status: resolved
Blocked by: 04
Label: wayfinder:grilling

## Question

Given the verified landscape, does the claimed gap (open source, on-premises Server, SAFe and PI reporting, lightweight .NET self-hosting) still justify building Flowline as scoped? The answer is one of: go as planned, narrow the scope, contribute to an existing project instead, or stop. Record the honest positioning statement the README will carry.

## Answer

2026-11-17. **Shelve the project for now.** Decided by Mattias in a grilling session.

How the answer was reached:

- The landscape research showed Apache DevLake added Azure DevOps Server support in August 2026, for pipelines, repositories and code review (beta, full refresh, no Boards work items). DORA metrics on Server are no longer a differentiator on their own.
- What remained open was flow and Program Increment reporting from Azure Boards on Server.
- Mattias's organization already uses Nooga Scale, a paid Azure DevOps extension with PI planning, flow metrics and objectives. A paid extension is therefore acceptable there, and PI reporting is already covered for the reference environment.
- With the first audience being Mattias's own release train, nothing uncovered was strong enough to justify building now.

Not decided, because the session stopped at the shelve decision:

- What, if anything, is missing or wrong in Nooga Scale.
- Whether the remaining gap is the link from Boards work items through pull requests to production deployments (end-to-end lead time, with DORA and flow metrics from one model). This was the most promising narrowed scope and is the first question to ask if the project is picked up again.
- The questions on building standalone versus on DevLake, and on swapping v0.2 and v0.3, were set aside.

Reasons to reopen: Nooga Scale is dropped or becomes too expensive; a concrete gap appears that it cannot fill, such as end-to-end lead time from work item to production; or interest from other organizations on Server shows up.
