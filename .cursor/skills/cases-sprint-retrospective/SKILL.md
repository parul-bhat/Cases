---
name: cases-sprint-retrospective
description: >-
  Generate biweekly Cases team sprint retrospective Confluence content from Jira
  worklogs. Use when the user asks for a Cases sprint retro, sprint retrospective
  data, hours/utilization tables, unplanned-sprint analysis, defect load by
  person, estimate-vs-actual variances, or content for the Retrospective:
  sprint-Cases-* Confluence page after a sprint ends.
---

# Cases Sprint Retrospective (biweekly)

Generate **data-only** Confluence content for the Cases team sprint retrospective.
The team fills discussion takeaways, Start/Stop/Keep, and action items **during**
the retro — do not invent those.

## When to use

- End of a Cases sprint (typically biweekly)
- User mentions: sprint retro, retrospective, hours logged, unplanned sprint,
  defect hours, estimate variance, spill into next sprint, utilization
- Confluence pages like `Retrospective: sprint-Cases-*` (example space: EMO)

## Defaults (override only if user specifies)

| Setting | Default |
|---|---|
| Sprint name pattern | `sprint-Cases-<Name>` (ask if unclear) |
| Worklog window | Sprint work period the user gives; if omitted, ask. Often sprint start→end inclusive |
| Jira browse base | `https://securly.atlassian.net/browse/` |
| People (worklog authors) | Amit Shirke, Gaurav Lonkar, Omkar Joshi, Omkarnath Panage, Goutham (Goutham Balashanmugam), Manshi Jain, Sujoy Nandi |
| Available hours (utilization) | Ask: e.g. `120 × 7 = 840` for a 3-week window, or next-sprint capacity like `55 × 7 = 385` |
| Unplanned label | exact label `unplanned-sprint` |
| Estimate variance threshold (main table) | absolute difference **≥ 3 hours** |
| Minute noise filter | ignore \|OE − TS\| &lt; 60 seconds |
| Exclude projects from sprint lists | **FILTER** (extension work often mis-tagged onto Cases sprints — exclude unless user asks to keep) |

### Defect set JQL (user’s board list — confirm/update reporters/assignee/version each sprint)

```jql
type = Defect
AND project IN (PRODUCT24, RESP, AWARE)
AND reporter IN (
  5d2f557bb7a11e0c7f31ebdc,
  712020:f22f3645-a2a5-4514-a7e2-70be82b49925,
  712020:7dfb46d9-22b4-43dd-905d-25b3c4ab4713
)
AND status != "Closed without action"
AND assignee = 61bb10f5e399d7007153af06
AND affectedVersion = FAOR-PKG-53
ORDER BY priority ASC, cf[11727] ASC, status ASC
```

Resolve account IDs to display names when presenting. Update `affectedVersion`
(and reporters/assignee) when the user says the package/version changed.

Reporters (as of skill creation): Goutham Balashanmugam, Manshi Jain, Sujoy Nandi.  
Assignee: Parul Bhat.

## Tooling

Prefer **Gandalf** (`Query` / `resume`) for Securly Jira reads when Atlassian MCP
is unavailable. If Atlassian MCP is authenticated, Jira/Confluence tools are fine.

Always produce **clickable** issue links: `[KEY](https://securly.atlassian.net/browse/KEY)`.

## Workflow

### 1. Confirm inputs

Ask only for what is missing:

1. Sprint name (e.g. `sprint-Cases-Rene`)
2. Worklog date range (inclusive)
3. Available hours basis for utilization (and leave days per person if known)
4. Affected version / package for the Defect JQL (if not FAOR-PKG-53)
5. Optional: Confluence page URL to update, or “markdown only for paste”
6. Optional: next-sprint capacity hours and spill/new/leave numbers (user often supplies these)

Do **not** block on: takeaways, action items, Start/Stop/Keep, burndown image —
those are team-owned during the meeting.

### 2. Pull data (parallelize Gandalf/Jira queries where possible)

Credit hours by **worklog author** and **worklog `started` date** in the window.
For parent Defect/Escape/Story metrics, include worklogs on the issue **or its
direct children/sub-tasks** unless a section says otherwise.

#### A. Hours by person × issue type

For the 7 people, sum hours on sprint-related worklogs in the window, split by
issue type where time was logged (Dev, Review, QA, Automation, Test Case
Creation, Task, Debug, Code Review, …).

Output columns used on the page:

| Team member | Dev hours | Review hours | Total | Leaves |
|---|---:|---:|---:|---:|

**Column convention (match existing Confluence):**

- For engineers who log mostly Dev/Review: split Dev vs Review as today.
- For QA/Automation (Goutham, Manshi, Sujoy): put their **total logged hours** in
  the Dev-hours column of that table if that is how the live page is structured,
  **or** prefer clearer labels (“Logged hours”) if creating a fresh page — ask
  once if ambiguous; default to matching the last retro page layout.
- Compute team totals and **Utilization %** = total logged ÷ available hours.

Also keep a separate **role-mix** summary (% of total hours): Dev cluster / QA /
Automation / Review for narrative (data insight, not a “filled takeaway”).

#### B. Defect load (user’s Defect JQL set)

Run the Defect JQL above (updated version). For each of the 7 people:

| Person | Defect Count | Hours logged |

- **Defect Count** = distinct parent Defects with **positive** time by that person
  (via defect or children) in the window.
- **Hours** = sum of those worklogs.
- Grand total: unique Defects touched by anyone of the 7 + total hours.
- Show each person’s defect-hours **% of their own total**.

List all Defect keys (clickable), priority, and primary Dev assignee if useful
for the empty-ish table the team annotates — **leave Takeaway blank**.

#### C. Escape Defects

In the same sprint (and/or labeled `unplanned-sprint`), list Escape Defects with
priority and reporter/logger. Keys clickable; **Reason/Takeaway blank**.

#### D. Unplanned non-Defect work

Issues in the sprint with label `unplanned-sprint` and type **≠ Defect**.

Exclude FILTER project by default.

For each with **any** child/sub-task hours in the worklog window, output:

| JIRA | Summary | Issue Type | Date added to sprint | Hours on sub-tasks (window) |

- **Date added** = most recent changelog transition that added the issue to this
  sprint (re-adds: use latest add; mention prior remove/re-add only if notable).
- Hours = worklogs on **direct children only**, started in the window (not parent
  self-logs unless user asks).

Optionally note the fuller list (including 0-hour rows) only if asked.

#### E. Estimate vs actual (sub-tasks)

Child issues in the sprint (non-empty parent) with ≥1 worklog in the window where
`|Original Estimate − Time Spent| ≥ 60 seconds` at issue-level fields.

Default published table: keep only **\|diff\| ≥ 3 hours**.

| JIRA | Summary | Issue Type | Parent | Original Estimate | Time Spent | Difference | Takeaway |

Leave **Takeaway** column empty for the team.

Mention count of smaller variances only if useful (“N other sub-tasks with 1min–&lt;3h drift — omitted”).

#### F. Spills (if user asks or data is available)

If the user/Jira provides spill list into next sprint: Person, JIRA (link), Reason
— leave Reason blank when unknown. Do not invent reasons.

#### G. Optional insights block (facts only)

Short bullets the facilitator can use — **not** action items:

- Unplanned-labeled hours vs total (%); defect-set hours vs total (%)
- Who carried disproportionate defect/unplanned load
- Where ≥3h estimate misses cluster (e.g. Automation vs Dev; by author)
- Review hours as % of total if thin
- Label hygiene: Defects in sprint missing `unplanned-sprint` (count + keys)

Do **not** write Start/Stop/Keep, Keep doing, or Action items unless the user
explicitly asks for draft suggestions.

### 3. Format for Confluence

Produce paste-ready markdown matching `references/confluence-template.md`.

Rules:

1. Every Jira key is a markdown link.
2. Leave blank: Start/Stop/Keep (unless user already drafted), Takeaways, Action
   items, Spill reasons the team owns, Burndown (image — note “embed burndown
   chart manually”).
3. Exclude FILTER issues unless the user overrides.
4. State methodology in one short footnote: worklog author, started date, child
   inclusion, issue-level OE vs TS for variance.
5. Round hours to 2 decimals from seconds; keep person totals consistent with
   the grand total.

### 4. Publish

- If user wants Confluence update and Atlassian tools work: update/create the page
  under the usual EMO retro parent; preserve any existing images (burndown) and
  team-edited sections.
- Otherwise: return markdown sections ready to paste, plus the Defect JQL link
  pattern they use on the board.

### 5. After delivery

Offer follow-ups only if useful: deeper drill on one person, re-run with
different variance threshold, include FILTER, or next-sprint capacity table.

## Anti-patterns

- Do not fill Takeaways / Action items / Start-Stop-Keep for the team.
- Do not treat missing burndown image or “blank” Confluence cells as errors when
  keys are Jira macros/links the fetcher cannot see.
- Do not mix Escape Defect into the Defect-only hours table without labeling.
- Do not credit hours by issue assignee; use **worklog author**.
- Do not use issue Updated date instead of worklog Started date.
- Do not include FILTER mis-tagged sprint members by default.

## Quick checklist

```
- [ ] Sprint name + worklog window confirmed
- [ ] Available hours / leaves confirmed
- [ ] Hours × person (+ Dev/Review split) + utilization
- [ ] Defect JQL set: Person | Count | Hours | % of own total
- [ ] Escape Defects listed (keys + priority + logger)
- [ ] Unplanned non-Defect (non-FILTER) with sprint-add date + child hours > 0
- [ ] Sub-task OE vs TS, |diff| ≥ 3h, Takeaway blank
- [ ] Optional facts-only insights
- [ ] Clickable links; methodology footnote
- [ ] Team-owned sections left blank
```
