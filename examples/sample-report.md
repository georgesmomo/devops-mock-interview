# DevOps interview report — SRE, mid
Fictional example · full loop · AWS, EKS, Terraform, GitHub Actions · 2026-11-01 · 10 questions

## Verdict: 6.2/10 — Borderline: depends on the other candidates

Your Kubernetes and Terraform basics are solid. But during the incident you spent six
exchanges debugging before thinking of a rollback, and on-call teams will notice that first.

## Scores

| Domain | Weight | Score | In one line |
|---|---|---|---|
| Kubernetes | 25% | 7/10 | Read `describe` and events well, knew OOMKilled vs throttling. |
| Incident scenario | 25% | 5/10 | Found the cause, but no mitigation and no communication. |
| Infrastructure as Code | 20% | 7/10 | Clear on state and locking, good `moved` block answer. |
| Observability | 15% | 5/10 | Knew the golden signals, could not define an error budget. |
| CI/CD | 15% | 6/10 | Good pipeline walk-through, weak on rollback. |

## Incident replay
- **Scenario:** pods in CrashLoopBackOff right after a deploy (OOMKilled).
- **Time to mitigation:** never. Rollback mentioned only when asked.
- **Root cause found:** yes, at exchange 6.
- **Best move:** `kubectl logs --previous` to read the crashed container.
- **Worst move:** raising the memory limit to 2Gi in production without measuring usage.

## What worked
- **Kubernetes reflexes.** Your first command was `kubectl describe pod`, and you read `Exit Code: 137` correctly.
- **Terraform state.** "Remote state in S3 with DynamoDB locking, never apply from a laptop" is exactly the bar.
- **Honesty.** "I've never run a cluster upgrade alone" was better than bluffing.

## What hurt you
- **Root cause before mitigation.** 30% of checkouts were failing while you debugged. → First reflex after a deploy: `kubectl rollout undo`, then investigate.
- **Silent incident.** You never said who you would inform. → Say it out loud: "I post in the incident channel and update the status page."
- **Error budgets.** "It's the time we can be down" is vague. → Learn it with a number: 99.9% over 30 days = 43 minutes.

## Technical corrections
- You said: "A liveness probe failure removes the pod from the service." In fact: that is the readiness probe. A failing liveness probe restarts the container.

## Red flags
None.

## Two answers to redo

### Q6: The incident scenario, first five minutes
**You said:** "I'd check the logs, then the metrics, then look at the code changes."
**Problem:** Users are impacted and the deploy is the obvious suspect. Nothing is mitigated.
**Stronger answer:** "It started right after the 14:02 deploy, so I roll back first: `kubectl rollout undo deployment/checkout`. I post in the incident channel. Once the error rate is back to normal, I look at why the new version crashed, starting with `kubectl describe` on a failed pod."

### Q8: "What is an SLO, and what do you do when the error budget is burned?"
**You said:** "It's an objective for uptime, like 99.9%. If we miss it we try to improve."
**Problem:** No SLI, no budget, no decision.
**Stronger answer:** "An SLO is a target on an SLI, for example 99.9% of checkout requests succeed in under 300 ms, over 30 days. The remaining 0.1% is the error budget. When it's burned, we freeze risky releases and spend the time on reliability until we're back in budget."

## Your 3 priorities before the real interview
1. Practice 3 incidents on a kind cluster and say "mitigation first" out loud each time.
2. Write a 5-line definition of SLI, SLO and error budget with a real number, and learn it.
3. Prepare one real incident story from your job: timeline, your role, the postmortem actions.
