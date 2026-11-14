# DORA metric wording

Type: research (AFK)
Status: open
Label: wayfinder:research

## Question

What is the exact current wording at dora.dev of the five software delivery metrics, how has it changed, and which choices does DORA leave to the implementer (start of change lead time, what a deployment is, what a failure is, recovery without a detection time, rework versus change fail)? Include primary definitions of the flow metrics.

Findings file: [docs/research/05-dora-metric-wording.md](../../../docs/research/05-dora-metric-wording.md)

## Comments

2026-11-11, partial answer, ticket stays open. The DORA part is answered; the flow metric part is not. Detail and sources are in the findings file above.

- DORA publishes five metrics. Throughput: change lead time, deployment frequency, failed deployment recovery time. Instability: change fail rate, deployment rework rate. Recovery time now sits under throughput, not stability.
- Time to restore was renamed in the 2023 report. Rework rate became the fifth metric in the 2024 report.
- "Change fail rate" and "change failure rate" are used interchangeably.
- The last elite, high, medium and low table read is from 2024. The 2025 report announces seven team archetypes instead.
- DORA is silent on which commit timestamp starts change lead time, what counts as a deployment, the window for attributing a failure, when the recovery clock starts, and mean versus median. Flowline must decide and document each.

The quotes were read from the source of DORA's own site in its public repository, not from the rendered pages, because dora.dev was blocked from the research environment.

Still unverified: the full 2025 report, explicit DORA guidance about individuals, and every flow metric definition (cycle time, lead time, throughput, WIP, work item age). The Kanban Guide and ProKanban sites were blocked and nothing was quoted from them.
