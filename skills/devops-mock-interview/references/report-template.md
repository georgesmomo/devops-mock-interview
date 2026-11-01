# Report template

Use this structure exactly. Write it in the interview language, headings and verdict labels
included, with technical terms in English. Replace everything in angle brackets. Keep each bullet to one or two lines.

```markdown
# DevOps interview report — <Role>, <level>
<Company or "No company given"> · <interview type> · <stack> · <date, if known> · <n> main questions

## Verdict: <score>/10 — <Strong hire | Hire | Borderline | No hire>

<Two sentences max. The real reason for the decision, as you would say it in the debrief.>

## Scores

| Domain | Weight | Score | In one line |
|---|---|---|---|
| <Kubernetes> | <25%> | <6/10> | <why, concretely> |
| <Incident scenario> | <20%> | <5/10> | <why> |
| … | | | |

## Incident replay
- **Scenario:** <name, one line>
- **Time to mitigation:** <exchange where they mitigated, or "never">
- **Root cause found:** <yes / partly / no>
- **Best move:** <command or decision>
- **Worst move:** <command or decision, and why it matters in real life>

## What worked
- **<Strength>.** <Evidence: short quote or the command they ran.>
- …(2 to 4 items)

## What hurt you
- **<Weakness>.** <Evidence.> → <What to do instead, in one line.>
- …(2 to 4 items, worst first)

## Technical corrections
- You said: "<claim>". In fact: <correct statement, one or two lines>.
- …(or "None.")

## Red flags
<List them, or write "None.">

## Two answers to redo

### Q<n>: "<question>"
**You said:** "<short quote or summary>"
**Problem:** <one line>
**Stronger answer:** <4 to 6 lines, at the candidate's level, built from their real background. Use [placeholders] for facts the user didn't give.>

### Q<n>: "<question>"
…

## Your 3 priorities before the real interview
1. <Concrete, doable action. E.g. "Break a pod on a kind cluster with a low memory limit and read describe/events until OOMKilled is a reflex.">
2. …
3. …
```

## Style check before sending

- The verdict and the score are at the top.
- Every strength and weakness has evidence.
- Every technical correction is accurate. If unsure, leave it out.
- No sentence starts with "Overall", "It's important to", "Great job".
- No paragraph longer than three lines, except the stronger answers.
- Nothing invented: no experience, numbers or company facts the user did not give or you did not find.
