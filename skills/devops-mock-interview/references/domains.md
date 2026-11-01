# Interview domains

For each domain: sample questions by level, what a strong answer contains, and red flags.
Adapt every question to the candidate's stack. Rephrase them in your persona's voice.

Levels: **J** junior, **M** mid, **S** senior, **St** staff/lead. A level must know everything
from the levels below.

---

## 1. Linux and systems

- **J** What happens, step by step, when you type `ls` in a shell and press Enter?
- **J** A disk is full. How do you find what is using it? (`df -h`, `du`, deleted files still open: `lsof +L1`)
- **M** The load average is 40 on a 4-core box, but CPU usage is low. What is going on? (I/O wait, D-state processes)
- **M** Difference between a process and a thread. What is a zombie process?
- **S** A server is slow only at 2 a.m. How do you investigate without being awake? (sar, cron, logs, metrics history)
- **S** What happens when the kernel runs out of memory? How do you know a process was OOM-killed? (`dmesg`)
- **St** How would you roll out a kernel or OS upgrade to 2,000 hosts safely?

Strong: explains mechanisms, names the exact command and what to read in its output.
Red flags: "I reboot it", no idea how to read `top`/`free`, confuses memory used and cache.

## 2. Networking

- **J** What happens when you type a URL in a browser? (DNS, TCP, TLS, HTTP)
- **J** Difference between TCP and UDP. Give a use for each.
- **M** A service can't reach another one. How do you debug, layer by layer? (DNS, routing, security groups/firewall, port, TLS, app)
- **M** What is a 502 versus a 504 at a load balancer?
- **S** L4 versus L7 load balancing. When do you need each?
- **S** How does TLS termination work, and where would you terminate it in your setup? Why?
- **St** Design the network for a multi-region service: DNS, failover, latency, cost.

Strong: debugs bottom-up with tools (`dig`, `curl -v`, `ss`, `traceroute`, `tcpdump`).
Red flags: no DNS reflex, "it's a network problem" with no method.

## 3. Containers

- **J** Difference between a container and a virtual machine.
- **J** What is a Docker image layer? Why does instruction order in a Dockerfile matter?
- **M** How do you make an image small and safe? (multi-stage, slim/distroless base, non-root user, no secrets in layers)
- **M** A container works locally and crashes in production. Where do you look?
- **S** What are namespaces and cgroups? What does a container *not* isolate?
- **S** How do you handle image provenance and vulnerabilities in CI? (scanning, SBOM, signing)

Strong: knows what is in the image and why. Red flags: secrets in `ENV`, `latest` tag in production, runs as root by default without knowing it.

## 4. Kubernetes

- **J** What is a pod? A deployment? A service?
- **J** A pod is in `CrashLoopBackOff`. What do you run first? (`kubectl describe`, `kubectl logs --previous`)
- **M** Readiness versus liveness probe. What goes wrong if you mix them up?
- **M** Requests versus limits. What happens when a container exceeds its memory limit? Its CPU limit? (OOMKilled versus throttling)
- **M** How does a request reach a pod from outside the cluster? (Ingress/Gateway, Service, kube-proxy, endpoints)
- **S** How do you upgrade a production cluster with no downtime? (PodDisruptionBudgets, surge, node drain, API deprecations)
- **S** How does the scheduler decide where a pod goes? Affinity, taints, tolerations.
- **S** Secrets in Kubernetes: how are they stored, and what do you use instead or on top?
- **St** Multi-tenant cluster for 30 teams: isolation, quotas, RBAC, network policies, cost allocation.

Strong: reasons from the control loop (desired versus actual state), reads events.
Red flags: `kubectl delete pod` as the only fix, no idea what a probe does, edits live resources by hand in production.

## 5. CI/CD

- **J** What is the difference between continuous integration, delivery and deployment?
- **M** Walk me through the pipeline you work with today, from commit to production.
- **M** How do you keep a pipeline fast? (cache, parallelism, test selection, artifacts built once)
- **M** Blue-green versus canary versus rolling. When do you choose each?
- **S** A bad release is in production. How does your pipeline let you roll back in under 5 minutes?
- **S** How do you secure a CI system? (least-privilege tokens, OIDC to cloud instead of static keys, pinned actions, protected branches, no secrets in logs)
- **St** 50 teams, one platform: how do you standardize pipelines without blocking teams? (golden paths, templates, policy as code)

Strong: build once, promote the same artifact; can explain a rollback. Red flags: builds a different artifact per environment, long-lived cloud keys in CI variables.

## 6. Infrastructure as Code

- **J** Why use IaC instead of clicking in the console?
- **M** What is the Terraform state? Where do you store it, and why does locking matter?
- **M** Someone changed a resource by hand. What happens at the next `terraform plan`? How do you handle drift?
- **M** How do you structure Terraform for several environments? (modules, workspaces versus directories, trade-offs)
- **S** How do you refactor a resource without destroying it? (`moved` blocks, `terraform state mv`, `import`)
- **S** How do you review and test IaC changes? (plan in CI, policy as code, `terraform test`, cost estimation)
- **St** How do you migrate a large hand-built estate into IaC without downtime?

Strong: knows `plan` is a contract, treats state as critical data. Red flags: state committed to Git, `apply` from a laptop on production, never read a plan.

## 7. Cloud

Adapt to AWS, GCP or Azure; ask the equivalent service.
- **J** What is a region versus an availability zone?
- **M** How does an instance get permission to read a bucket without a key? (instance roles / workload identity)
- **M** Public versus private subnet. How does a private instance reach the internet? (NAT)
- **S** Your cloud bill jumped 40% this month. How do you find out why, and what are the usual suspects? (egress, NAT data, idle resources, logs volume)
- **S** Design IAM for a company with 20 teams and three environments.
- **St** Multi-account / multi-project strategy, landing zone, guardrails.

Strong: least privilege by default, knows where the money goes. Red flags: `*:*` policies, root/owner credentials in use, no idea of data transfer costs.

## 8. Observability

- **J** Difference between logs, metrics and traces.
- **M** What would you put on the dashboard of a web API? (latency percentiles, traffic, errors, saturation)
- **M** Why is average latency a bad metric? (percentiles, p99)
- **S** What makes a good alert? Why should most alerts be on symptoms, not causes?
- **S** What is an SLI, an SLO and an error budget? Give real examples. What do you do when the budget is burned?
- **S** High-cardinality labels in Prometheus: what is the problem?
- **St** Observability costs too much. How do you cut it by half without losing signal? (sampling, retention tiers, dropping unused metrics)

Strong: alerts tied to user impact, knows the golden signals. Red flags: alerts on CPU > 80%, no runbooks, dashboards nobody reads.

## 9. Security and DevSecOps

- **J** Where should secrets live? Where should they never be?
- **M** How do you manage secrets for apps in production? (Vault, cloud secret managers, external-secrets, rotation)
- **M** What do you scan in a pipeline, and at which step? (SAST, dependencies/SCA, image scan, IaC scan, secrets scan)
- **S** A critical CVE lands in a base image used by 200 services. What is your plan for the next 48 hours?
- **S** Supply-chain security: what are SBOMs and signatures for? (SLSA, cosign, provenance)
- **St** How do you make security a default without becoming the team that blocks every release?

Strong: thinks in blast radius and least privilege. Red flags: security = "we run a scanner", no rotation plan, ignores findings to ship.

## 10. Reliability and incident response (SRE)

- **J** What do you do in the first 5 minutes of an incident?
- **M** What is in a good postmortem? Why blameless?
- **M** Describe an incident you handled. Your role, the timeline, the fix, what changed after.
- **S** How do you run an incident with 5 people on the call? (incident commander, communication, roles)
- **S** What is toil? Give an example you removed, with numbers.
- **S** How do you decide between shipping features and fixing reliability? (error budgets)
- **St** Design an on-call rotation that doesn't burn people out.

Strong: mitigate first, root-cause later; communicates; learns. Red flags: blames people, debugs for an hour before mitigating, no postmortems.

## 11. Scripting, automation and Git

- **J** Write, in words or code, a script that deletes log files older than 7 days.
- **M** Bash versus Python for automation: where is the line?
- **M** Git: merge versus rebase. How do you fix a commit that leaked a secret? (rotate first, then rewrite history)
- **S** How do you make an automation script safe to run twice? (idempotency)
- **S** GitOps: what changes when Git is the source of truth for deployments?

Strong: idempotent, handles errors (`set -euo pipefail`), rotates leaked secrets before anything else.
Red flags: thinks deleting the commit is enough after a leak.

## 12. Collaboration and culture

- **M** Developers say your platform slows them down. What do you do?
- **M** Tell me about a production mistake you made. What happened next?
- **S** How do you get a team to adopt a new tool or standard they didn't ask for?
- **St** How do you measure the success of a platform team? (DORA metrics, developer satisfaction, adoption)

Strong: owns mistakes, treats developers as customers. Red flags: "devs don't understand ops", hides mistakes.
