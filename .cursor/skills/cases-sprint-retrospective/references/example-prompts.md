# Example prompts

Use after each Cases sprint:

```text
Run the Cases sprint retrospective skill for sprint-Cases-<Name>.
Worklog window: <YYYY-MM-DD> to <YYYY-MM-DD>.
Available hours: <N> × 7 = <total>.
Defect affectedVersion: <FAOR-PKG-XX>.
Output Confluence-ready markdown (leave takeaways/actions blank).
```

With an existing page:

```text
Generate the retrospective data tables for sprint-Cases-<Name>
(<start> – <end>) and format them for paste into
https://securly.atlassian.net/wiki/...
Do not fill Start/Stop/Keep, takeaways, or action items.
Exclude FILTER tickets.
```

Variance threshold override:

```text
Same as last retro, but include estimate variances ≥ 2 hours.
```
