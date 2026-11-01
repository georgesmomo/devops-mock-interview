# System design prompts

For mid level and above. About 10 to 15 minutes. Give the prompt in two sentences, then let
the candidate ask questions. Answer clarifying questions with realistic numbers.

## How to run one

1. Give the prompt. Do not list requirements unless asked.
2. Answer questions with concrete numbers (traffic, team size, budget, SLO).
3. Let them draw it in words. Then push on failure: "This zone goes down. What happens?"
4. Push on one trade-off: cost, complexity, or speed of delivery.
5. Stop after about 6 exchanges.

## What you assess

- Asks about requirements and constraints before designing.
- Starts simple, then scales. Does not over-engineer for the stated numbers.
- Names failure modes and how the design survives them.
- Considers operations: deploys, monitoring, on-call, cost.
- Explains trade-offs, not just choices.

Expectations by level:
- **M:** a correct, simple design. Handles one failure mode.
- **S:** clear trade-offs, failure modes, observability, rollout plan.
- **St:** organizational impact, migration path, cost, how teams will use it.

---

## D1. CI/CD platform for 40 teams — M/S/St
"We have 40 product teams, each with its own pipeline scripts. Design a shared CI/CD platform."
Hidden numbers: 600 builds/day, 3 cloud accounts, mixed Java/Node/Go, deploys to Kubernetes.
Push on: secrets, build cache, how a team customizes without forking, rollout to teams.

## D2. Zero-downtime deployments for a stateful app — S
"Our monolith with a Postgres database goes down 10 minutes at every release. Fix it."
Hidden numbers: 2 releases a week, 300 requests/s, a 400 GB database, sessions stored in memory.
Push on: database migrations (expand and contract), session handling, rollback when the
schema changed.

## D3. Observability stack from scratch — M/S
"A 30-service platform has only application logs in files. Design monitoring and alerting."
Hidden numbers: 50k requests/s at peak, 2 SREs, small budget.
Push on: what to alert on, retention, cost, who gets paged.

## D4. Multi-region active-passive — S/St
"Our payment API runs in one region. Business wants it to survive a region outage. RTO 15 min, RPO 1 min."
Hidden numbers: 2,000 payments/min at peak, 1 TB Postgres, budget +40% at most.
Push on: data replication, DNS failover, testing the failover, cost of the passive side.

## D5. Self-service environments — M/S
"Developers wait 3 days for a test environment. Give them one in 10 minutes."
Hidden numbers: 60 developers, 25 services, one Postgres per service, about 40 open merge requests.
Push on: IaC modules versus Kubernetes namespaces, cleanup of idle environments, cost control, data.

## D6. Secrets management migration — S/St
"Secrets are in CI variables, `.env` files and a wiki page. Design the target and the migration."
Hidden numbers: about 400 secrets, 3 environments, 70 services, no rotation ever done.
Push on: rotation, bootstrapping (secret zero), access audit, migrating without an outage.
