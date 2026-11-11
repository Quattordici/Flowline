# Name

Type: grilling (HITL)
Status: resolved
Blocked by: 06
Label: wayfinder:grilling

## Question

Does the project keep the name Flowline? The name becomes the namespace and NuGet prefix of every project, so it is settled before the solution is scaffolded.

## Answer

2026-11-11. The project is named **Flowcompass**. Decided by Mattias after the name availability research showed the NuGet id `Flowline` is taken by an active .NET tool.

- Namespace and NuGet prefix: `Flowcompass.*` (for example `Flowcompass.Core`).
- At the time of the check the NuGet id was free and no GitHub account had that exact name. One similar GitHub login exists.
- Not checked: trademarks, domains, NuGet prefix reservation. Check these before the first public package.
- The repository and these planning documents still say Flowline. Renaming the repository is a separate step for the owner; documents are renamed after that so links do not break twice.
