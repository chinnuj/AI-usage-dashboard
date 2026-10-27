# BRD: Claude Usage and Value Dashboard

Sep 26, 2026 · @CJ

## 1. Purpose and background

This document defines the business requirements for a dashboard that tells the Analytics team whether its Claude licences are used, what they cost, and whether Claude's output is accepted.

The Analytics team is made up of business analysts, data analysts, data engineers and project managers. Every member holds a Claude licence and the team uses Claude Code. No consolidated view of usage, spend or output quality exists today.

The dashboard will be built twice, in Power BI and in Tableau, on one shared data model. It is a portfolio project on dummy data, designed so the same model can be pointed at a live Claude organisation.

## 2. Problem statement

The team pays for a Claude licence for every member, plus usage on top, and cannot currently say:

- how many of those licences are used, and how deeply;

- what each active user costs, and how much is spent on licences nobody uses;

- whether Claude's output is accepted as-is, or is frequently rejected and adds review overhead.

Without these answers, the renewal decision and the question of who needs a licence rest on opinion.

## 3. Objectives and success criteria

The dashboard must let the head of the team decide which licences to keep, reassign or drop, and see where Claude is and isn't delivering usable work.

| \#  | Objective             | Question it answers                                                   |
|-----|-----------------------|-----------------------------------------------------------------------|
| O1  | Measure adoption      | Is the team using Claude, how often, and how deeply?                  |
| O2  | Measure cost          | What does usage cost per person, and what is wasted on idle licences? |
| O3  | Measure effectiveness | Are Claude Code suggestions accepted or rejected, and where?          |
| O4  | Support a decision    | Which licences should be kept, reassigned or dropped?                 |

The project succeeds when:

- each of the three questions can be answered from the dashboard without opening another tool;

- every KPI traces to a documented source field and a written formula;

- Power BI and Tableau show the same value for every KPI on the same data;

- the dashboard produces a named list of idle licences and their cost.

## 4. Stakeholders and users

These roles are assumed for the scenario.

| Role                        | What they need from the dashboard                                    |
|-----------------------------|----------------------------------------------------------------------|
| Head of Analytics (sponsor) | Renewal and licence decisions; overall adoption, cost and acceptance |
| Project manager             | Adoption trend by team and role; who has not started                 |
| Team leads (BA, DA, DE)     | Their team's engagement and acceptance rates; user-level detail      |
| Finance partner             | Monthly licence and usage cost; idle-licence cost                    |
| Claude administrator        | Provides the analytics API key and enables Claude Code telemetry     |
| Data engineer (builder)     | Source specifications, refresh rules and data quality checks         |

## 5. Scope

The project covers Claude only, across three themes: adoption, cost and effectiveness.

**In scope**

- Claude usage across chat, Claude Code, Cowork and the Office add-ins

- Licence and usage cost per user

- Accept and reject decisions on Claude Code edit suggestions

- One dashboard, built in both Power BI and Tableau

- A dummy dataset shaped to match Claude's real API and telemetry fields

**Out of scope**

- Gemini, GitHub Copilot and any other AI tool

- Governance and unapproved-tool monitoring

- Task or ticket tracking (no Jira data)

- Survey feedback and satisfaction scores

- Prompt or response content of any kind

- Productivity or time-saved estimates

## 6. Business questions and KPI definitions

Every KPI below is computed from one of three fact tables, and all can be sliced by date, team and role.

### 6.1 Adoption

| KPI                  | Definition                                                                                                                                                                                           |
|----------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Licensed users       | Users holding a licence on any day of the period                                                                                                                                                     |
| Active users         | A monthly measure: distinct users with at least 5 active days in the month, where an active day has at least one interaction. There is no weekly or daily active-user figure                         |
| Adoption rate        | Active users ÷ licensed users in the month × 100                                                                                                                                                     |
| Usage frequency      | Active days per active user per month                                                                                                                                                                |
| Engagement score     | Sum of weighted, capped activity points per user per day, using the scoring rules below                                                                                                              |
| Engagement benchmark | One per role: the median engagement score per active user-day over the three most recent complete months, recalculated at each quarterly refresh. Weekly benchmark = daily × 5; monthly = daily × 20 |
| Meeting benchmark    | Share of active users whose monthly engagement score reached their role's monthly benchmark                                                                                                          |
| User segment         | Against the user's role benchmark: Idle = 0, Light = below 50%, Regular = 50% to 150%, Power = above 150%                                                                                            |
| Feature mix          | Share of engagement points by feature                                                                                                                                                       |

#### 6.1.1 Engagement scoring rules

The engagement score turns everything a person did with Claude in a day into one number. It rewards deeper use, and it stops any single high-volume activity from dominating. Each activity goes through three steps, and the day’s points are added up.

**The three steps**

1.  Count units. The raw count from the Analytics API is divided by “count per unit” and rounded up. Most activities are one unit each. Chat and Office add-in messages take 2 messages per unit, because a prompt and its follow-up are roughly one exchange. Rounding up means a single message still counts as one unit.

2.  Apply the weight. Units × weight, where the weight reflects depth: 1 for a basic prompt up to 4 for agentic work.

3.  Apply the cap. Points from that activity can’t exceed its daily cap, if it has one.

As a formula: ***points = MIN( ROUNDUP(count ÷ count per unit) × weight, daily cap )***

| Activity               | Category            | Weight | Counted as            | Daily cap (points) |
|------------------------|---------------------|--------|-----------------------|--------------------|
| Chat messages          | Basic prompt        | 1      | 1 unit per 2 messages | 10                 |
| Files uploaded         | Summarise / extract | 2      | Each file             | None               |
| Web searches           | Summarise / extract | 2      | Each search           | 10                 |
| Artifacts created      | Generate            | 3      | Each artifact         | None               |
| Office add-in messages | Generate            | 3      | 1 unit per 2 messages | 9                  |
| Projects used          | Advanced / agentic  | 4      | Each distinct project | None               |
| Connector calls        | Advanced / agentic  | 4      | Each call             | 12                 |
| Claude Code sessions   | Advanced / agentic  | 4      | Each session          | None               |
| Cowork sessions        | Advanced / agentic  | 4      | Each session          | None               |

Expected daily benchmark per role. These are the values from the dummy data for July to September 2026, the three months before the October refresh.

| Role             | Typical active day                                           | Daily | Weekly | Monthly |
|------------------|--------------------------------------------------------------|-------|--------|---------|
| Project manager  | 8 messages, 1 file; those who build add Claude Code sessions | 8     | 40     | 160     |
| Business analyst | 12 messages, 1 file, 1 artifact                              | 12    | 60     | 240     |
| Data analyst     | 10 messages, 1 file, 1 artifact, 1 project                   | 14    | 70     | 280     |
| Data engineer    | 6 messages, 2 Claude Code sessions, 2 connector calls        | 19    | 95     | 380     |

**Worked example**

| Activity in a day | Units | × Weight | Before cap | After cap |
|---|---|---|---|---|
| 13 chat messages | 7 | × 1 | 7 | 7 |
| 40 chat messages | 20 | × 1 | 20 | 10 |
| 3 web searches | 3 | × 2 | 6 | 6 |
| 5 connector calls | 5 | × 4 | 20 | 12 |
| 3 Claude Code sessions | 3 | × 4 | 12 | 12 (no cap) |

**Why the caps matter**

Compare two days:

- Chat-heavy analyst: 60 chat messages and 1 file upload.

- Data engineer: 6 chat messages, 2 Claude Code sessions and 2 connector calls.

| | Analyst | Engineer |
|---|---|---|
| Without caps | 30 + 2 = 32 | 3 + 8 + 8 = 19 |
| With caps | 10 + 2 = 12 | 3 + 8 + 8 = 19 |

Without caps, a long chat outscores two coding sessions, which is the opposite of what the weights are meant to say. With caps, the deeper work scores higher, and the analyst still earns credit for a busy day.

**Why only some activities are capped**

Caps apply only to activities measured as raw volume: messages, searches and connector calls. Someone can generate dozens of those in a day without doing more meaningful work. Files, artifacts, projects and sessions are distinct pieces of work, so their counts stay naturally small and each one is worth its full points.

### 6.2 Cost

| KPI                    | Definition                                                                                                        |
|------------------------|-------------------------------------------------------------------------------------------------------------------|
| Total cost             | Licence cost + usage cost for the period                                                                          |
| Cost per active user   | Total cost ÷ monthly active users                                                                                 |
| Idle-licence cost      | Licence cost of users who hold a licence and had no activity in the last two months                               |
| Cost per accepted edit | Total cost of users with edit decisions ÷ accepted edits. Approximate, because usage cost is not split by surface |

### 6.3 Effectiveness

| KPI                     | Definition                                                                                           |
|-------------------------|------------------------------------------------------------------------------------------------------|
| User decisions          | Accept and reject decisions made by a person at a prompt, excluding automatic approvals              |
| Acceptance rate         | Accepted user decisions ÷ user decisions × 100                                                       |
| Explicit rejection rate | Decisions where the user chose "No" ÷ user decisions × 100. Proxy for error and correction frequency |
| Auto-approval share     | Decisions approved by settings or hooks ÷ all decisions × 100. Shows how much output goes unreviewed |

Effectiveness covers Claude Code only. Acceptance shows behaviour, not correctness: a rejection doesn't prove the output was wrong, and an accepted edit that is later fixed by hand isn't visible.

## 7. Functional requirements

The dashboard is a single page with a KPI header and six visuals, two per theme.

| ID    | Requirement                                                                                                                                                                                                                                                 |
|-------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| FR-1  | KPI header showing adoption rate, active users vs licensed users, share of active users meeting their role benchmark, total cost, cost per active user, and acceptance rate, for the last month of the selected range, each with change vs the month before |
| FR-2  | Adoption: monthly active users against licensed users over time                                                                                                                                                                                             |
| FR-3  | Adoption: user segments (Idle, Light, Regular, Power) by role                                                                                                                                                                                               |
| FR-4  | Cost: monthly cost split into licence and usage, with cost per active user                                                                                                                                                                                  |
| FR-5  | Cost: idle-licence table listing user, team, role, last active date and days idle for every licence with no activity in the last two months, longest idle first                                                                                             |
| FR-6  | Effectiveness: acceptance rate over time by role                                                                                                                                                                                                            |
| FR-7  | Effectiveness: acceptance rate by language and edit tool                                                                                                                                                                                                    |
| FR-8  | Filters for date range, team and role that apply to every visual                                                                                                                                                                                            |
| FR-9  | Drill from any team or role to a user-level table with one row per user per month, showing segment, engagement score, cost and acceptance rate                                                                                                              |
| FR-10 | A visible "data as of" date, and a note beside the effectiveness visuals stating that they cover Claude Code only                                                                                                                                           |
| FR-11 | Benchmarks are read from the role benchmark table. Engagement weights and caps, segment thresholds and the idle window are fixed assumptions (section 10), documented in the data dictionary                                         |
| FR-12 | The dashboard is delivered in Power BI and in Tableau from the same data model, with every KPI defined once and implemented as a DAX measure and as a Tableau calculated field                                                                              |

## 8. Data requirements

The model is a star schema of one dimension (users), one role benchmark table and three facts. The facts carry the calendar date directly in dd-mm-yyyy format, and the usage fact names each activity directly, so no date or feature table is needed. Month and quarter are derived from the date in the report. The column-level schema is in the data dictionary.

| Table                    | Grain                                                                                                        | Source                                                                                                                        | Refresh                           |
|--------------------------|--------------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------|-----------------------------------|
| dim_user                 | One row per team member                                                                                      | HR system or identity provider                                                                                                | Daily                             |
| dim_role_benchmark       | One row per role per quarter, with daily, weekly and monthly benchmark and the months it was calculated from | Calculated from the last three complete months of usage                                                                       | Quarterly                         |
| fact_usage_daily         | User, feature, day                                                                                           | Claude Enterprise Analytics API, user activity endpoint                                                                       | Daily                             |
| fact_cost_daily          | User, day                                                                                                    | Licence assignment from the identity provider; licence price from the contract; usage cost from the Analytics API cost report | Daily, reloading the last 30 days |
| fact_edit_decision_daily | User, edit tool, language, decision source, day                                                              | Claude Code OpenTelemetry export                                                                                              | Daily                             |

Data rules:

- Users are matched across all sources on work email.

- Usage data is requested one day at a time and can arrive up to three days late, so loads must handle late-arriving days.

- Cost values can be revised for up to 30 days, so the trailing 30 days are reloaded on every run.

- Raw API responses and telemetry events are stored unchanged before any transformation, so the model can be rebuilt if definitions change.

- If telemetry is unavailable, edit decisions fall back to the Analytics API accept and reject counts, with language and decision source marked unknown.

Field references: [Claude Enterprise Analytics API reference](https://support.claude.com/en/articles/13703965), [Analytics APIs overview](https://platform.claude.com/docs/en/manage-claude/analytics-api), [Claude Code monitoring](https://code.claude.com/docs/en/monitoring-usage).

## 9. Non-functional requirements

| ID    | Area            | Requirement                                                                                                                    |
|-------|-----------------|--------------------------------------------------------------------------------------------------------------------------------|
| NFR-1 | Freshness       | Data refreshes once a day; the dashboard shows the latest complete day                                                         |
| NFR-2 | Privacy         | The default view is aggregated by team and role. User-level detail is restricted to the sponsor and team leads                 |
| NFR-3 | Privacy         | No prompt or response content is collected or stored                                                                           |
| NFR-4 | Consistency     | Power BI and Tableau return identical KPI values on the same data                                                              |
| NFR-5 | Traceability    | Every KPI has a written definition, formula and source field in a data dictionary                                              |
| NFR-6 | Maintainability | Benchmarks change through the data model. Weights, caps and thresholds are documented assumptions applied in the load and the report measures |
| NFR-7 | Portability     | The model loads from flat files for the dummy build and from a database for a live build, with no change to the reports        |

## 10. Assumptions, constraints and risks

**Assumptions**

- The team is on a Claude Enterprise plan, which the Analytics API requires.

- The organisation’s primary owner can issue an analytics API key.

- Claude Code telemetry can be enabled for all users and sent to a collector the team runs.

- An HR or identity roster with work email, team and role is available.

Fixed values applied in the report measures, not stored in the data model:

- User segments against the role benchmark: Idle = no activity, Light = below 50%, Regular = 50% to 150%, Power = above 150%.

- Idle licence: no activity in the last two months.

- Benchmark multipliers: weekly = daily × 5, monthly = daily × 20.

- Licence price: USD 20 per user per month, plus usage billed per token.

- Claude Code is used across roles, including project managers who build prototypes and small tools with it.

**Constraints**

- Analytics API data starts on 1 January 2026.

- Chat usage has no quality signal, so effectiveness covers Claude Code only.

- Usage data lags by one to three days.

**Risks**

| Risk                                             | Impact                                 | Mitigation                                                                 |
|--------------------------------------------------|----------------------------------------|----------------------------------------------------------------------------|
| Acceptance rate is read as correctness           | Wrong conclusions about output quality | Label the measure as acceptance behaviour; state the limit on the dashboard |
| Automatic approvals inflate acceptance           | Overstated effectiveness               | Count only decisions made by a person; show auto-approval share separately |
| Staff see the dashboard as individual monitoring | Resistance, changed behaviour           | Aggregate by default; restrict user-level access; state the purpose openly |
| Telemetry is not enabled for everyone            | Gaps in effectiveness data             | Report telemetry coverage; fall back to Analytics API counts               |
| The engagement benchmark is arbitrary            | Segments lose credibility              | Derive it from the team's own data and review it each quarter              |

