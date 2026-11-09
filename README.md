<h1 align="center">devops-mock-interview</h1>

<p align="center"><strong>A realistic DevOps / SRE / platform engineer interview, run by your coding agent.<br>
Live incident scenarios, system design, follow-up questions, then an honest score out of 10.</strong></p>

<p align="center">
  <img src="https://img.shields.io/badge/agent%20skill-SKILL.md-8a63d2" alt="Agent skill">
  <img src="https://img.shields.io/badge/works%20with-Claude%20Code%20·%20Codex%20·%20Cursor%20·%20Gemini%20CLI%20·%20Copilot-555" alt="Works with Claude Code, Codex, Cursor, Gemini CLI, Copilot">
  <img src="https://img.shields.io/badge/roles-DevOps%20·%20SRE%20·%20Platform%20·%20Cloud%20·%20DevSecOps-2b7489" alt="Roles">
  <a href="./LICENSE"><img src="https://img.shields.io/badge/license-MIT-blue" alt="MIT license"></a>
</p>

Reading lists of DevOps interview questions is not the same as answering them under pressure.
This skill turns your agent into a Staff SRE who researches the job, asks one question at a
time, digs into vague answers, pages you with a live production incident, and then tells you
honestly whether they would hire you.

## How it works

| Step | What happens |
|---|---|
| **1. Intake** | One message, 3 required answers: role, level, interview type. Stack, company, job posting and CV are optional. |
| **2. Research** | The agent reads the posting, maps the stack, checks the company's engineering blog and public postmortems. You see each step live. |
| **3. Briefing** | You meet your interviewer: a name, a title, a style. |
| **4. Interview** | Domain questions, a **live incident** you debug by typing real commands, a **system design** question, a behavioral one. No hints until the end. |
| **5. Report** | Score /10, hire or no-hire verdict, incident replay, technical corrections, your two weakest answers rewritten, 3 priorities. |

## The live incident

The interviewer holds a hidden root cause. You type commands, it answers with realistic output.

```text
Karim: [6/10] You're on call. PagerDuty fires:
       "Checkout API: 5xx rate at 30% for 10 minutes. Started right after the 14:02 deploy."
       Go.

You:   kubectl get pods -n checkout

Karim: NAME                        READY   STATUS             RESTARTS   AGE
       checkout-7d9f8c6b5-2xkqp    0/1     CrashLoopBackOff   6          9m
       checkout-7d9f8c6b5-9wzlt    0/1     CrashLoopBackOff   6          9m
       checkout-7d9f8c6b5-hmv4r    0/1     CrashLoopBackOff   5          9m

You:   I check the logs.

Karim: Which logs? What command?
```

8 scenarios ship with v1: OOMKilled after a release, CoreDNS saturation, a disk filled by WAL
files, an expired certificate, the `latest` tag trap, a NAT gateway cost spike, a leaked cloud
key, a missing index after a migration.

It grades what real on-call teams care about: **did you mitigate first?** Did you communicate?
Did you run a dangerous command in production?

## What you get at the end

```text
## Verdict: 6.2/10 — Borderline

Your Kubernetes and Terraform basics are solid. But during the incident you spent six
exchanges debugging before thinking of a rollback, and on-call teams will notice that first.

## Technical corrections
- You said: "A liveness probe failure removes the pod from the service."
  In fact: that is the readiness probe. A failing liveness probe restarts the container.
```

[See a full sample report](examples/sample-report.md).

## What it covers

12 domains, with questions calibrated for junior, mid, senior and staff:

Linux · Networking · Containers · Kubernetes · CI/CD · Infrastructure as Code (Terraform) ·
Cloud (AWS / GCP / Azure) · Observability (SLOs, alerting) · Security and DevSecOps ·
Reliability and incident response · Scripting and Git · Collaboration and culture

Plus 6 system design prompts: a CI/CD platform for 40 teams, zero-downtime deploys,
observability from scratch, multi-region failover, self-service environments, secrets migration.

## Install

**Any agent** (Claude Code, Codex, Cursor, Gemini CLI, GitHub Copilot, OpenCode and more),
with the [skills CLI](https://github.com/vercel-labs/skills):

```bash
npx skills add georgesmomo/devops-mock-interview
```

**Claude Code plugin:**

```text
/plugin marketplace add georgesmomo/devops-mock-interview
/plugin install devops-mock-interview@devops-mock-interview
```

**Manual:** copy `skills/devops-mock-interview/` into your agent's skills folder
(for Claude Code: `~/.claude/skills/devops-mock-interview/`).

## Use

Just ask:

```text
Interview me for a senior SRE role. Stack: GCP, GKE, Terraform, ArgoCD.
```

```text
Mock DevOps interview, mid level, incident scenarios only.
```

```text
Prepare me for my platform engineer interview at Datadog. Here's the posting: https://…
```

In Claude Code you can also type `/devops-mock-interview`.

During the interview: `pause` to pause, `stop` to end early and get your report.

## Why this one

- **Real conditions.** One question at a time, follow-ups, pressure, no feedback until the end.
- **Live incidents, not quiz questions.** You debug with real commands against a hidden root cause.
- **Calibrated.** Junior is not judged like staff. 5 means "maybe", 8+ means "I'd defend this hire".
- **Technically strict.** Wrong answers get corrected in the report, with the right answer.
- **Short report.** Plain sentences. Evidence from your own answers. No corporate filler.
- **Any language.** It interviews you in the language you write in; commands stay in English.
- **Any agent.** A plain `SKILL.md`, no code, no API key, nothing to run on your machine.

## Files

```text
skills/devops-mock-interview/
├── SKILL.md                       # the 5-phase flow
└── references/
    ├── domains.md                 # 12 domains, questions by level, signals and red flags
    ├── incident-scenarios.md      # 8 live incidents with hidden causes and outputs
    ├── system-design.md           # 6 design prompts
    ├── scoring.md                 # calibrated rubric, verdict rules and caps
    └── report-template.md         # exact report structure
```

## Roadmap

- **Hands-on lab mode:** the agent breaks a real `kind` cluster and you fix it live.
- More scenarios: Kafka lag, etcd quota, Terraform state lock, failing node drain.
- Company-specific loops built from public interview reports.

Ideas and new scenarios are welcome: open an issue or a pull request.

## Related

Preparing for a non-DevOps role? Use [**mock-interview**](https://github.com/georgesmomo/mock-interview),
the same realistic interview for any job title.

Want to explain your architecture in a video? [**tech-to-motion**](https://github.com/georgesmomo/tech-to-motion)
turns a docker-compose, Terraform or k8s manifest into a motion-design explainer.
For any other topic, [**idea-to-motion**](https://github.com/georgesmomo/idea-to-motion) turns one line into a video.

## License

[MIT](./LICENSE) © Georges MOMO

If it helped you land a job, a ⭐ helps others find it.
