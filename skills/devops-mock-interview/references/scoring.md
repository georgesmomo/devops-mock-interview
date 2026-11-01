# Scoring

## Per domain (0 to 10)

Score each domain from the plan on the evidence in the answers. One weak answer is not a 2
if a follow-up recovered it; judge the best evidence and the overall pattern. The incident
scenario and the design question are scored as their own lines.

| Score | Meaning |
|---|---|
| 9–10 | Exceptional for the level. Mechanisms, failure modes, real numbers. Rare. |
| 7–8 | Clearly meets the bar. Would pass this round. |
| 5–6 | Borderline. Knows the tools, shaky on the "why" or on failure cases. |
| 3–4 | Below the bar. Lists tools, vague, or partly wrong. |
| 0–2 | No signal, wrong, or not answered. |

## Overall score

1. Compute the weighted average of the scores (weights from the plan, adding up to 100%).
2. Adjust it by at most ±0.5 for things that cut across domains (below).
3. Round **down** to one decimal (6.48 → 6.4). Never round up.

Adjustments:
- − technical errors stated with confidence, guessing instead of saying "I don't know",
  rambling, blaming others;
- + clear reasoning out loud, honest about limits, good questions at the end.

## Calibration by level

Calibrate to the level the user gave. Use the J / M / S / St tags in
[domains.md](domains.md) as the bar.

- **Junior:** fundamentals and method. Not knowing a tool is fine; not knowing how to look is not.
- **Mid:** autonomous in production. Debugs alone, knows the failure modes of their stack,
  mitigates before root-causing.
- **Senior:** trade-offs, design, reliability, mentoring. Leads the incident, not just the debugging.
- **Staff/lead:** organization-wide impact, migrations, cost, standards adopted by other teams.

Rules:
- 5.0 is "maybe", not "average". Most real candidates land between 4 and 7.
- Do not give 8+ unless you would defend the hire in a debrief.
- Never round up to be kind.

## Verdict

| Overall | Verdict |
|---|---|
| 8.0+ | Strong hire |
| 7.0–7.9 | Hire |
| 5.5–6.9 | Borderline: depends on the other candidates |
| < 5.5 | No hire |

Caps. In these cases the verdict label is at most "Borderline", whatever the score. Keep the
score as computed and add one line in the report explaining the cap:
- a scored line with a weight of 20% or more is at 3 or below;
- any red flag marked **(cap)** below.

## Red flags (always reported, whatever the score)

- **(cap)** A destructive action in production without a safety check: deleting database
  files, `terraform apply` without reading the plan, `kubectl delete` on a namespace to "fix" it.
- **(cap)** Cleaning Git history before revoking a leaked credential.
- Debugging for 5 exchanges or more with no mitigation while users are impacted (junior: report
  it, no penalty beyond the incident score).
- Blaming people in the postmortem.
- Admin-everywhere credentials presented as normal practice.
- No questions at all for the interviewer. (Questions only about logistics are not a red
  flag but count as weak in the report.)
