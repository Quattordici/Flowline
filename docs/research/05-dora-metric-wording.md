# 05. DORA metric wording: exact definitions, history, and open ambiguities

Research date: started 2026-11-10, all fetches completed 2026-11-11.

## Access note (read this first)

`dora.dev` itself was **not reachable** from the research environment (the network policy refused the host, both for direct HTTP and for the fetch tool). The same applied to `kanbanguides.org`, `prokanban.org`, `web.archive.org`, `github.com` and `services.google.com`.

What was reachable:

- `raw.githubusercontent.com/dora-team/dora.dev` (branch `main`). This is DORA's own public repository that the dora.dev site is built from (Hugo content, the Quick Check app, survey question data, and the 2023 and 2024 report PDFs). Everything quoted below as "dora.dev source" was read from there.
- `cloud.google.com` blog pages.

Consequence: quotes below are exact as read from the site **source files** on `main`, not from the rendered pages. The rendered wording at `https://dora.dev/guides/dora-metrics/` should be identical, but that was not confirmed, and is listed under Unverified. Someone with normal web access should do a two-minute visual check before the definitions are frozen.

## Short answer

- DORA currently defines **five** software delivery performance metrics, in two groups. Throughput: change lead time, deployment frequency, failed deployment recovery time. Instability: change fail rate, deployment rework rate. The guide source was last marked `updated: 2026-01-05`.
- Note the grouping: **failed deployment recovery time sits under throughput**, not stability. This is the reverse of the pre-2024 "stability = change failure rate + time to restore" model that most tools still implement.
- "Time to restore service" / MTTR was renamed and redefined as "failed deployment recovery time" in the **2023** report. Rework rate was added as a survey question and a fifth metric in the **2024** report. The 2024 report calls it "rework rate" and the factor "stability"; the current guide calls it "deployment rework rate" and the factor "instability".
- DORA uses "change fail rate" and "change failure rate" interchangeably, even within one report (2024). The current guide and Quick Check say "change fail rate".
- Elite/high/medium/low clusters were last published in the 2024 report (computed on the original four metrics only). For 2025, DORA's own announcement describes "seven common team profiles or archetypes" instead, and the Quick Check compares a 0 to 10 score against 2025 benchmarks by industry. The full 2025 report could not be downloaded, so its details are Unverified.
- DORA's definitions are survey definitions. They do not specify commit timestamp semantics, what a "deployment" is in a staged rollout, the time window for "immediate", the start of the recovery clock, or mean versus median. The tool has to decide all of these and document them.
- Flow metric (Kanban) primary sources could not be reached. That section contains no verified quotes.

## Sources actually read

| Id | What | URL read | Published equivalent (not reached) |
|---|---|---|---|
| S1 | Metrics guide source, `updated: 2026-01-05` | https://raw.githubusercontent.com/dora-team/dora.dev/main/hugo/content/guides/dora-metrics/_index.md | https://dora.dev/guides/dora-metrics/ |
| S2 | "A history of DORA's software delivery metrics", dated 2026-01-02 | https://raw.githubusercontent.com/dora-team/dora.dev/main/hugo/content/insights/dora-metrics-history/index.md | https://dora.dev/insights/dora-metrics-history/ |
| S3 | Quick Check question text | https://raw.githubusercontent.com/dora-team/dora.dev/main/svelte/quick-check/src/lib/data/metrics_details.json | https://dora.dev/quickcheck/ |
| S4 | Quick Check answer options | https://raw.githubusercontent.com/dora-team/dora.dev/main/svelte/quick-check/src/lib/data/metrics_question_responses.json | https://dora.dev/quickcheck/ |
| S5 | 2025 survey questions data | https://raw.githubusercontent.com/dora-team/dora.dev/main/hugo/data/survey_questions/2025-state-of-ai-assisted-software-development.json | https://dora.dev/research/2025/questions/ |
| S6 | Accelerate State of DevOps 2023 report PDF (footer `v. 2023-12`) | https://raw.githubusercontent.com/dora-team/dora.dev/main/hugo/content/research/2023/dora-report/2023-dora-accelerate-state-of-devops-report.pdf | https://dora.dev/research/2023/dora-report/ |
| S7 | Accelerate State of DevOps 2024 report PDF (footer `v. 2024.3`) | https://raw.githubusercontent.com/dora-team/dora.dev/main/hugo/content/research/2024/dora-report/2024-dora-accelerate-state-of-devops-report.pdf | https://dora.dev/research/2024/dora-report/ |
| S8 | FAQ source, `updated: 2025-10-06` | https://raw.githubusercontent.com/dora-team/dora.dev/main/hugo/content/faq/_index.md | https://dora.dev/faq/ |
| S9 | Google Cloud blog, "Announcing the 2025 DORA Report", 2025-09-23 | https://cloud.google.com/blog/products/ai-machine-learning/announcing-the-2025-dora-report | same |
| S10 | Google Cloud blog post on the Four Keys open source project (title not recorded), dated 2020-09-22 | https://cloud.google.com/blog/products/devops-sre/using-the-four-keys-to-measure-your-devops-performance | same |
| S11 | Quick Check app source (metric list, version switch, results view) | https://raw.githubusercontent.com/dora-team/dora.dev/main/svelte/quick-check/src/App.svelte and `.../src/lib/YourPerformance.svelte` | https://dora.dev/quickcheck/ |

Report quotes were extracted from the PDFs with `pdftotext`; line breaks inside quotes have been joined, wording is unchanged.

## Grouping and scope (S1)

The guide's title is "DORA's software delivery performance metrics". It states:

> "DORA has identified five software delivery performance metrics that provide an effective way of measuring the outcomes of the software delivery process."

> "They can be divided into metrics that show the throughput of software changes, and metrics that show instability of software changes."

> "Throughput is a measure of how many changes can move through the system over a period of time. Higher throughput means that the system can move more changes through to the production environment. DORA uses three factors to measure software delivery throughput:"

> "Instability is a measure of how well the software deployments go. When deployments go well, teams can confidently push more changes into production and users are less likely to experience issues with the application immediately following a deployment. DORA uses two factors to measure software delivery instability:"

On the level of measurement:

> "These factors can be used to measure any application or service, regardless of the technology stack, the complexity of the deployment processes, or its end users."

> "These metrics work for any type of technology your organization is delivering, but are best suited for measuring one application or service at a time."

> "Context matters. Apply the metrics in the context of the application or service your team is delivering. [...] While it may be tempting to blend metrics across multiple teams–or entire organizations–these differences in context mean that doing so can be problematic."

The survey frames every question the same way (S5): "For the primary application or service you work on..."

## Metric 1: Change lead time (throughput)

Current guide definition (S1):

> "**Change lead time**: The amount of time it takes for a change to go from committed to version control to deployed in production."

Quick Check question, labelled "Lead time" (S3):

> "For the primary application or service you work on, what is your <b>lead time for changes</b> (that is, how long does it take to go from code committed to code successfully running in production)?"

2025 survey question (S5):

> "What is your lead time for changes (i.e., how long does it take to go from code committed to code successfully running in production)?"

Earlier wording:

- 2023 report (S6, p. 11), for "Change lead time": "how long it takes a change to go from committed to deployed."
- 2024 report (S7, p. 10): "Change lead time: the time it takes for a code commit or change to be successfully deployed to production."
- History article summary (S2): "Change lead time: Time from code commit to successful production deployment."
- 2020 Four Keys blog (S10), under the older name "Lead Time for Changes": "The amount of time it takes a commit to get into production".

## Metric 2: Deployment frequency (throughput)

Current guide definition (S1):

> "**Deployment frequency**: The number of deployments over a given period or the time between deployments."

Quick Check question, labelled "Deploy frequency" (S3):

> "For the primary application or service you work on, <b>how often does your organization deploy code</b> to production or release it to end users?"

2025 survey question (S5): "How often does your organization deploy code to production or release it to end users?"

Earlier wording:

- 2023 report (S6, p. 11): "how frequently changes are pushed to production."
- 2024 report (S7, p. 10): "Deployment frequency: how often application changes are deployed to production."
- 2020 Four Keys blog (S10): "How often an organization successfully releases to production".

## Metric 3: Failed deployment recovery time (throughput)

Current guide definition (S1):

> "**Failed deployment recovery time**: The time it takes to recover from a deployment that fails and requires immediate intervention."

Quick Check question, labelled "Failure recovery" (S3):

> "For the primary application or service you work on, <b>how long does it generally take to restore service</b> after a change to production or release to users results in degraded service (for example, lead to service impairment or service outage) and subsequently requires remediation (for example, require a hotfix, rollback, fix forward, or patch)?"

2025 survey question (S5):

> "How long does it generally take to restore service after a change to production or release to users results in degraded service (for example, lead to service impairment or service outage) and subsequently require remediation (for example, requires a hotfix, rollback, fix forward, or patch)?"

Earlier wording:

- 2023 report (S6, p. 11): "how long it takes to recover from a failed deployment."
- 2024 report (S7, p. 10): "Failed deployment recovery time: the time it takes to recover from a failed deployment."
- Pre-2023 question, as quoted in the 2023 appendix (S6, p. 92): "For the primary application or service you work on, how long does it generally take to restore service when a service incident or a defect that impacts users occurs (for example, unplanned outage, service impairment)?"
- 2020 Four Keys blog (S10), as "Time to Restore Service": "How long it takes an organization to recover from a failure in production".

Why it is grouped under throughput (S7, p. 12):

> "Change lead time, deployment frequency, and failed deployment recovery time are used when we describe software delivery throughput. This factor measures the speed of making updates of any kind, normal changes and changes in response to a failure."

## Metric 4: Change fail rate (instability)

Current guide definition (S1):

> "**Change fail rate**: The ratio of deployments that require immediate intervention following a deployment. Likely resulting in a rollback of the changes or a "hotfix" to quickly remediate any issues."

Quick Check question, labelled "Change fail rate" (S3), identical to the 2025 survey question (S5):

> "Approximately what percentage of changes to production or releases to users result in degraded service (for example, leads to service impairment or service outage) and subsequently requires remediation (for example, requires a hotfix, rollback, fix forward or patch), if at all?"

Earlier wording:

- 2023 report (S6, p. 11), as "Change failure rate": "how frequently a deployment introduces a failure that requires immediate intervention."
- 2024 report (S7, p. 10), as "Change fail rate": "the percentage of deployments that cause failures in production,1 requiring hotfixes or rollbacks." Footnote 1: "We consider a deployment to be a change failure only if it causes an issue after landing in production, where it can be experienced by end users. In contrast, a change that is stopped on its way to production is a successful demonstration of the deployment process's ability to detect errors."
- History article summary (S2): "Change fail rate: Percentage of deployments causing failures in production."
- 2020 Four Keys blog (S10): "The percentage of deployments causing a failure in production".

## Metric 5: Deployment rework rate (instability)

Current guide definition (S1):

> "**Deployment rework rate**: The ratio of deployments that are unplanned but happen as a result of an incident in production."

Quick Check question, labelled "Deployment rework rate" (S3), identical to the 2025 survey question (S5):

> "Approximately what percentage of deployments in the last 6 months were not planned but were performed to address a user-facing bug in the application?"

Earlier wording:

- 2024 report (S7, p. 11), as "rework rate": "For the primary application or service you work on, approximately how many deployments in the last six months were not planned but were performed to address a user-facing bug in the application?"
- History article summary (S2): "Deployment rework rate: Percentage of deployments that are unplanned work to fix bugs."

Note the three slightly different triggers in DORA's own text: "an incident in production" (guide), "a user-facing bug in the application" (survey), "unplanned work to fix bugs" (history article).

## History

All of this is from S2 unless another source is given.

**2014 to 2015.** "They began with four variables: deployment frequency, lead time for changes, mean time to recover (MTTR), and change fail rate." In 2014 change fail rate did not load onto the construct, so "the 2014 definition of IT performance relied on three metrics". By 2015: "Throughput measures: Deployment frequency and deployment lead time. Stability measures: Mean time to recover (MTTR) and change fail rate."

**2018.** Availability added as an operational measure; term "Software Delivery and Operational (SDO) performance" introduced.

**2021.** Availability broadened to reliability. S2: "The 2021 report inaccurately called the "reliability" metric the "fifth metric" when, in reality, "reliability" is more a measure of operational performance than a measure of software delivery performance." So "the fifth metric" has meant two different things: reliability (2021, since disowned) and rework rate (2024 onward).

**2023: time to restore becomes failed deployment recovery time.** S2: "The metric historically known as "mean time to recover (MTTR)" or "time to restore service" was renamed and redefined as failed deployment recovery time." The primary statement is the 2023 report appendix "Refining how we measure software delivery performance" (S6, pp. 92 to 93):

> "The previous way of asking about recovery times did not allow for a distinction between a failure initiated by a software change and a failure initiated by something like an earthquake interrupting service at a data center."

> "We are now using the term "Failed deployment recovery time" to distinguish our measure from the more generic "time-to-restore" that we've used in the past and sometimes abbreviated to "MTTR." MTTR has caused some confusion in the community: is that "M" for mean or median?"

**2023: change failure rate answer format changed.** Same appendix (S6, p. 92): "In previous years, respondents were presented with six options (0-15%, 16-30%, etc.). This year we presented respondents with a slider so that they could select any value between 0% and 100%." The question itself gained the same "degraded service ... subsequently requires remediation" qualifiers as the recovery question.

**"Change failure rate" versus "change fail rate".** There was no single dated rename that could be found. The 2023 report uses "Change failure rate" throughout. The 2024 report uses "Change fail rate" in the definition box and the performance level table (pp. 10, 13) and "change failure rate" in the surrounding prose (pp. 11, 12). The current guide, Quick Check and history article use "change fail rate"; the history article also applies that name retroactively to 2014. Treat them as synonyms; "change fail rate" is the current DORA spelling.

**2024: rework rate introduced, five metrics.** S7, p. 11:

> "We have a longstanding hypothesis that the change failure rate metric works as a proxy for the amount of rework a team is asked to do. When a delivery fails, this requires the team to fix the change, likely by introducing another change. To test this theory, we added another question this year about the rework rate for an application"

> "Our data analysis confirmed our hypothesis that rework rate and change failure rate are related. Together, these two metrics create a reliable factor of software delivery stability."

> "All five metrics are considered for describing software delivery performance." (p. 12)

S2 dates the structural change the same way: "The most significant structural change to the metrics occurred in 2024. [...] they introduced a fifth metric: deployment rework rate. With this addition, DORA now utilizes five metrics".

**2024 to now: naming drift.** In the 2024 report the factor is "Software delivery stability" and the metric is "Rework rate". In the current guide (updated 2026-01-05) and the history article (2026-01-02) the factor is "instability" and the metric is "deployment rework rate". The 2025 survey data file already uses the headings "Software delivery throughput" and "Software delivery instability" (S5). The Quick Check results view still labels the second group "Software delivery stability" (S11). The exact date the guide page moved from four to five metrics was not established (see Unverified).

**The older four-keys guide page.** No source file for a separate `guides/dora-metrics-four-keys` page was found at the paths tried in the repository, and the live URL could not be tested. The current guide still uses the hero image `hero-fourkeys.png`, and the FAQ (S8) still links the phrase "the "Four Key" metrics" to `/guides/dora-metrics/`. Whether the old URL redirects is Unverified.

## Answer buckets, performance levels, and guidance on comparison

### Current answer options

Quick Check options (S4), lowest to highest:

- Lead time and failure recovery: "More than six months", "One to six months", "One week to one month", "One day to one week", "Less than one day", "Less than one hour".
- Deploy frequency: "Less than once per six months", "Between once per month and once every six months", "Between once per week and once per month", "Between once per day and once per week", "Between once per hour and once per day", "On demand (multiple deploys per day)".
- Change fail rate and rework rate: a 0 to 100 percent slider (S11 component `MetricsQuestion.svelte`).

2025 survey options (S5) differ slightly in wording: "More than six months, Between one month and six months, Between one week and one month, Between one day and one week, Less than one day, Less than one hour, I don't know or NA" for both time questions; "Fewer than once per six months, Between once per month and once every 6 months, Between once per week and once per month, Between once per day and once per week, Between once per hour and once per day, On demand (multiple deploys per day), I don't know or NA" for frequency; "(value between 0% and 100%), I don't know or NA" for both rates.

### Performance levels

The last elite/high/medium/low table that was read is in the 2024 report (S7, p. 13). It was computed on four metrics only: "We performed the cluster analysis on the original four software delivery metrics to remain consistent with previous years' cluster analyses."

| Level | Change lead time | Deployment frequency | Change fail rate | Failed deployment recovery time | Respondents |
|---|---|---|---|---|---|
| Elite | Less than one day | On demand (multiple deploys per day) | 5% | Less than one hour | 19% |
| High | Between one day and one week | Between once per day and once per week | 20% | Less than one day | 22% |
| Medium | Between one week and one month | Between once per week and once per month | 10% | Less than one day | 35% |
| Low | Between one month and six months | Between once per month and once every six months | 40% | Between one week and one month | 25% |

(The high cluster really does have a worse change fail rate than the medium cluster in 2024; the report discusses this on p. 14.) The levels are emergent, not fixed thresholds: "We do not set these levels in advance, rather we let them emerge from the survey responses." The FAQ (S8) confirms the count of clusters has varied: three before 2018, four from 2018 to 2021, three in 2022, four in 2023 and 2024.

For 2025, what could be confirmed:

- The announcement (S9) describes a different analysis: "we conducted a cluster analysis that reveals seven common team profiles or archetypes, each with a unique interplay of performance, stability, and well-being."
- The FAQ (S8) says Quick Check results are "an overall software delivery performance score, on a scale from 0 to 10. This blended metric is derived from the five individual metrics", plus "Throughput & stability scores", compared with "benchmarks derived from DORA's 2025 research program". The app offers industry and organization size benchmarks (S11).
- The 2025 errata page refers to "Figure 12: Distribution of deployment frequency responses from the 2025 DORA Survey", which suggests per-metric distributions are published.

Whether the 2025 report contains no elite/high/medium/low table at all could not be confirmed because the report PDF was blocked. A tool should not hard-code elite/high/medium/low thresholds as "the DORA levels"; at most it can cite the 2024 table by year.

### Guidance against comparison and targets

From the guide's "Common pitfalls" (S1):

> "**Setting metrics as a goal.** Ignoring Goodhart's law and making broad statements like, "Every application must deploy multiple times per day by year's end," increases the likelihood that teams will try to game the metrics."

> "**Making disparate comparisons.** These metrics are meant to be applied at the application or service level. Comparing metrics between vastly different applications (for example, a mobile app and a mainframe system) can be misleading."

> "**Competing.** The goal is to improve your team's performance over time, not to compete against other teams or organizations. Use the metrics as a guide for identifying areas for growth and celebrating progress."

> "**Having siloed ownership.** Sharing all five metrics across development, operations, and release teams fosters collaboration and shared ownership of the delivery process. Isolating teams with specific metrics can lead to friction and finger-pointing."

From the reports:

> "The best comparisons are those performed over time on the same applications rather than between different applications, which will always have different contexts." (S6, p. 11)

> "Improving should be more important to a team than reaching a particular performance level. The best teams are those that achieve elite improvement, not necessarily elite performance." (S7, p. 14)

None of the sources read contains a sentence that names **individuals** explicitly. The guidance read is about applications, services, teams and organizations. An explicit DORA statement against measuring individuals is Unverified.

## Ambiguities DORA leaves open

For each item: what DORA says, then where it is silent.

### 1. Start and end of change lead time

DORA says: "from committed to version control to deployed in production" (S1); "from code committed to code successfully running in production" (S3, S5); "a code commit or change" (S7).

DORA is silent on:

- Which commit timestamp: author date, committer date, push time, or merge to the main branch. "Committed to version control" is the whole specification. PR merged is not mentioned anywhere in the sources read, so a tool that starts the clock at PR merge is measuring a narrower interval than DORA describes and should say so.
- Which commit when a change or deployment contains several: first commit, last commit, or each commit separately.
- The aggregate: mean, median or percentile. The survey asks for one bucket.
- Whether "successfully running in production" means deployment finished, full rollout, or feature enabled.

Nearest implementation guidance, old and not normative (S10, 2020): "Lead Time to Changes metric requires two important pieces of data: when the commit happened, and when the deployment happened. This means that for every deployment, you need to maintain a list of all the changes included in the deployment." and "then calculate the median lead time". That is per commit, measured to the deployment that contains it, aggregated by median.

### 2. What counts as a deployment

DORA says: "deploy code to production or release it to end users" (S3, S5); the metrics cover "changes of any kind, including changes to configuration and changes to code" (S7, p. 10); frequency is "The number of deployments over a given period or the time between deployments" (S1).

DORA is silent on: partial or canary rollouts, multi-region or multi-environment rollouts of one version, feature flag flips, infrastructure-only changes, failed deployment attempts, redeploys of an unchanged version, and how to treat one pipeline run that deploys many services. It also gives two formulations (count per period, or time between) without choosing one.

Nearest implementation guidance (S10): "Next you have to consider what constitutes a successful deployment to production. Do you include deployments that are only to 5% traffic? 80%? Ultimately, this depends on your team's individual business requirements." and, on bucketing, "to qualify for "deploy daily," you must deploy on most working days."

### 3. What counts as a failure

DORA says: a deployment that requires "immediate intervention following a deployment. Likely resulting in a rollback of the changes or a "hotfix"" (S1); a change that results "in degraded service (for example, leads to service impairment or service outage) and subsequently requires remediation (for example, requires a hotfix, rollback, fix forward or patch)" (S5). The 2024 footnote adds that it counts "only if it causes an issue after landing in production, where it can be experienced by end users", and that a change stopped before production is not a failure (S7).

DORA is silent on: how long after a deployment "immediate" lasts; how a failure is attributed to a specific deployment; whether a failed deployment that is automatically rolled back before users notice counts; severity thresholds; whether one incident caused by several deployments counts once or several times; and whether the denominator includes the remediation deployments themselves. Note also the unit mismatch inside DORA: the guide says "ratio of deployments", the survey says "percentage of changes to production or releases to users".

### 4. Recovery time when detection time is unknown

DORA says: "how long does it generally take to restore service after a change to production or release to users results in degraded service" (S5); "The time it takes to recover from a deployment that fails and requires immediate intervention" (S1).

DORA is silent on: when the clock starts (the failed deployment's timestamp, the moment degradation began, detection, or incident creation); when it stops (remediating deployment finished, or service confirmed healthy); and the statistic. On the statistic, the only relevant text is the 2023 appendix observing that "MTTR has caused some confusion in the community: is that "M" for mean or median?" without resolving it for the new metric, and the survey's word "generally".

Nothing in the sources read addresses unknown detection time at all. A literal reading of "after a change to production ... results in degraded service" supports starting at the failed deployment and ending at the deployment that remediates it, which needs only deployment data. The 2020 Four Keys post (S10), written for the older metric, instead used incident records: "you need to know when the incident was created and when it was resolved."

### 5. Rework rate versus change fail rate

DORA says the two are related but distinct and together form one factor (S7, p. 11). Change fail rate: share of deployments that "result in degraded service ... and subsequently requires remediation" (S5). Rework rate: share of deployments "in the last 6 months [that] were not planned but were performed to address a user-facing bug in the application" (S5). The factor "measures the likelihood deployments unintentionally lead to immediate, additional work" (S7, p. 12).

Plain reading: change fail rate counts the deployments that **caused** a problem; rework rate counts the deployments that **were the fix**. Rework rate is also broader in one respect, because a "user-facing bug" need not be degraded service or traceable to one recent deployment.

DORA is silent on: how to tell planned from unplanned deployments in tooling data; whether a hotfix deployment for a failed deployment counts in the rework numerator (the 2024 rationale implies yes); whether a rollback is itself a deployment for either metric; and which trigger applies, given the guide says "an incident in production" and the survey says "a user-facing bug". The six-month window appears only in the survey question, not in the guide definition.

### 6. Other points an implementer should note

- Aggregation level. DORA says one application or service at a time and warns against blending across teams (S1). It gives no rule for rolling up to a team or portfolio.
- Time window. Only the rework question names a window (six months).
- Precision. The guide itself warns that "Building integrations to multiple systems to get precise data about your software delivery performance might not be worth the initial investment" (S1). DORA's own numbers are self-reported survey buckets, so tool-computed values are not directly comparable with report benchmarks.

## Flow metric definitions (Kanban)

**No primary source could be reached for this section.** `kanbanguides.org` and `prokanban.org` were refused by the network policy, and after two refused hosts no further sites were tried. There are therefore **no verified quotes** here, and per the rules for this ticket nothing is asserted from memory as a definition.

Sources to read once access is available:

- The Kanban Guide (Vacanti and Coleman), https://kanbanguides.org/ . It is expected to define four mandatory flow measures (WIP, throughput, work item age, cycle time) in terms of team-chosen "started" and "finished" points. Verify the exact text and the guide version date.
- ProKanban.org, https://prokanban.org/ , for the same measures and for its position on "lead time" versus "cycle time".
- Daniel Vacanti, "Actionable Agile Metrics for Predictability", for the published definitions of cycle time, throughput, WIP and work item age.
- Flow Framework (Mik Kersten, "Project to Product", https://flowframework.org/ ), which uses a different vocabulary (flow velocity, flow time, flow load, flow efficiency, flow distribution).

Points to check when reading them, stated as questions, not findings:

- Is cycle time defined as elapsed time between started and finished, and does the guide state a rounding or "plus one day" convention?
- Is throughput a count of items finished per unit of time (not story points)?
- Is WIP the count of items started but not finished?
- Is work item age the elapsed time since an unfinished item started?
- Does the Kanban Guide define "lead time" at all, or leave it as a customer-facing synonym chosen by the team?
- How do Flow Framework "flow time" and "flow velocity" map onto cycle time and throughput?

A naming collision to plan for regardless of the answers: DORA's "throughput" is a factor made of three metrics about deployments, and DORA's "change lead time" runs from commit to production. Kanban throughput and lead time are about work items. The two vocabularies should be kept visibly separate in the tool's definitions.

## Unverified

1. **Rendered dora.dev pages.** `https://dora.dev/guides/dora-metrics/`, `/insights/dora-metrics-history/`, `/quickcheck/`, `/faq/` and `/research/2025/questions/` were not read as published. Quotes come from the site's source on `main` in `dora-team/dora.dev`. The repository could differ from what is deployed.
2. **Older four-keys guide URL.** Whether `https://dora.dev/guides/dora-metrics-four-keys/` still exists or redirects was not determined.
3. **Full 2025 report** (State of AI-assisted Software Development). The PDF at `https://services.google.com/fh/files/misc/2025_state_of_ai_assisted_software_development.pdf` was blocked. Not verified: its wording of the five metric definitions, whether it publishes any elite/high/medium/low table, the per-metric distributions, and the details of the seven archetypes.
4. **Exact date the guide changed** from four metrics to five, and from "stability" and "rework rate" to "instability" and "deployment rework rate". Only the `updated: 2026-01-05` front matter and the 2024 report were seen; the repository history was not accessible.
5. **Exact date of the "change failure rate" to "change fail rate" spelling change** on dora.dev. Evidence only shows both in use in 2024 and "change fail rate" in current sources.
6. **Reports before 2023** (2014 to 2022). Not read. Their content is reported here only as described by DORA's own history article (S2) and the 2023 appendix (S6).
7. **Explicit DORA guidance against measuring individuals.** Not found in the sources read. The guidance found covers applications, teams and organizations.
8. **All flow metric definitions** (Kanban Guide, ProKanban, Vacanti, Flow Framework). No source reached, nothing quoted.
9. **PDF versions.** The report PDFs in the repository carry footers `v. 2023-12` and `v. 2024.3`. Whether these are the latest published versions was not checked against the errata pages for 2023 and 2024.
