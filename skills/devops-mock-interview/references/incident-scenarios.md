# Incident scenarios

A live, interactive debugging exercise. You hold a hidden root cause. The candidate
investigates; you answer each command or check with a short, realistic output.

## How to run one

1. **Page.** Read the alert and the first symptoms in a few lines. Nothing more.
2. **Let them drive.** Answer exactly what they ask. Outputs are short (5 to 10 lines) and
   consistent with the root cause and with each other. Adapt names to the candidate's stack.
3. **Vague action, precise question.** "I check the logs" → "Which logs? What command?"
4. **No hints.** Never say "good idea" or "you're close". If they go the wrong way, show a
   normal-looking output and let them come back.
5. **Pressure.** After about 4 exchanges: "Support says customers can't pay. Where are you?"
   Adapt the impact to the scenario ("Every team is paging you", "The CFO is asking").
6. **End** when they name the root cause and a fix, or after about 8 exchanges. Ask:
   "How do you mitigate right now? And what goes in the postmortem?"

## What you assess

- **Mitigation first.** Do they stop the bleeding (rollback, scale, failover) before root-causing?
- **Method.** Recent changes first, then from symptom to cause, one hypothesis at a time.
- **Communication.** Do they say who they inform and when?
- **Commands.** Real, correct commands, and they read the output.
- **Learning.** A postmortem with causes, not culprits, and actions that prevent recurrence.

Pick a scenario that matches the stack. Translate it if needed (ECS instead of Kubernetes,
GitLab CI instead of GitHub Actions). Levels: J, M, S, St.

---

## S1. Pods restarting after a release — J/M

**Page:** "Checkout API: 5xx rate at 30% for 10 minutes. Started right after the 14:02 deploy."

**Hidden cause:** the new version needs more memory. The limit is 256Mi. Pods are OOMKilled.

Outputs:
- `kubectl get pods`: `checkout-7d9f…  0/1  CrashLoopBackOff  6  9m` (several pods)
- `kubectl describe pod`: `Last State: Terminated  Reason: OOMKilled  Exit Code: 137`, `Limits: memory: 256Mi`
- `kubectl logs --previous`: app starts, loads a cache, then the log stops abruptly
- `kubectl rollout history`: revision 42 at 14:02, "bump cache library"
- Metrics: memory climbs to the limit in 40 seconds

**Strong path:** see that it started with the deploy → `kubectl rollout undo` to mitigate → describe shows OOMKilled →
raise the limit after measuring real usage, fix the cache size.
**Postmortem:** no load test on memory, no canary, alert fired only on 5xx.

## S2. DNS failures inside the cluster — M/S

**Page:** "Random timeouts across many services. About 5% of requests. No recent deploy."

**Hidden cause:** CoreDNS has 2 replicas. Someone added a 100m CPU limit to its deployment
months ago (the default has a request, no CPU limit). Traffic tripled and CoreDNS is now
CPU-throttled. Also `ndots:5` multiplies lookups for external names.

Outputs:
- App logs: `dial tcp: lookup payments.internal: i/o timeout`, intermittent
- `kubectl -n kube-system get pods -l k8s-app=kube-dns`: 2 pods, Running, no restarts
- `kubectl top pods -n kube-system`: CoreDNS pods at ~100m CPU, flat
- `kubectl -n kube-system get deploy coredns -o yaml`: `limits: cpu: 100m`
- Metric `container_cpu_cfs_throttled_periods_total` for CoreDNS: high and rising
- CoreDNS metrics: request rate tripled in a month
- `cat /etc/resolv.conf` in a pod: `options ndots:5`

**Strong path:** suspect DNS from the error → check CoreDNS health and saturation → remove the CPU limit and scale CoreDNS
→ longer term: NodeLocal DNSCache, lower `ndots` or use FQDNs with a trailing dot.
**Postmortem:** no saturation alert on CoreDNS, cluster DNS not scaled with growth.

## S3. Disk full on a database host — J/M

**Page:** "Postgres primary crashed and keeps failing to restart. All API calls return 500."

**Hidden cause:** the WAL archive command has been failing for 3 days (bucket permission
changed), so WAL files pile up and fill the disk.

Outputs:
- Postgres log: `PANIC: could not write to file "pg_wal/…": No space left on device`
- `df -h`: `/var/lib/postgresql  100%`
- `du -sh /var/lib/postgresql/16/main/pg_wal`: 180G (normal is ~2G)
- Postgres log, older: `archive command failed with exit code 1` repeated for 3 days
- Archive script: `AccessDenied` when uploading to the bucket

**Strong path:** do NOT delete WAL files by hand → free space safely or extend the volume →
fix the archive permission so Postgres recycles WAL itself → check replication.
**Postmortem:** no alert on archive failures, permission change not reviewed, no disk growth alert.
**Red flag:** `rm` in `pg_wal`. That can corrupt the database.

## S4. Expired certificate — J/M

**Page:** "Mobile app: 100% of API calls failing since 00:00 UTC. Website works."

**Hidden cause:** the API uses a separate certificate that was renewed manually last year.
It expired at midnight. The website uses an auto-renewed one.

Outputs:
- `curl -v https://api…`: `SSL certificate problem: certificate has expired`
- `openssl s_client -connect api…:443 -servername api… </dev/null | openssl x509 -noout -dates`: `notAfter=… 00:00:00 GMT` (today)
- Same command without `-servername`: may show the website's default certificate, still valid (a trap: a strong candidate knows SNI)
- Load balancer config: the API listener uses a certificate uploaded by hand

**Strong path:** check the TLS error → renew and deploy the certificate → move it to automatic renewal.
**Postmortem:** manual certificate, no expiry monitoring, inventory of all certificates.

## S5. Pipeline deploys the wrong version — M/S

**Page:** "We released 3.4.0 an hour ago, but users see the old bug fixed in 3.4.0."

**Hidden cause:** the Kubernetes deployment uses the `latest` tag with `imagePullPolicy: IfNotPresent`.
Nodes that already had `latest` cached still run the old image.

Outputs:
- `kubectl get deploy -o yaml`: `image: registry/app:latest`, `imagePullPolicy: IfNotPresent`
- `kubectl get pods -o wide` + image IDs: two different digests across nodes
- CI log: build and push of `latest` succeeded, then `kubectl rollout restart`

**Strong path:** compare image digests across pods → pin the release by immutable tag or digest →
redeploy → ban `latest` with a policy.
**Postmortem:** mutable tags, no version check after deploy.

## S6. Cloud bill spike — S/St

**Page:** not an outage. "Finance: the cloud bill is up 45% this month. CFO wants an answer by Friday."

**Hidden cause:** a new service in a private subnet pulls large container images and calls an
external API through the NAT gateway. NAT data processing and cross-AZ traffic exploded.

Outputs:
- Cost explorer by service: biggest increase is "EC2-Other" / networking
- By usage type: `NatGateway-Bytes` ×6, `DataTransfer-Regional-Bytes` ×3
- VPC flow logs: one service sends most bytes through NAT, to a registry and an external API
- Service started 3 weeks ago

**Strong path:** break the bill down by usage type, not by service name → find the traffic →
for ECR, interface endpoints (`ecr.api`, `ecr.dkr`) plus the S3 gateway endpoint; for an
external registry, a pull-through cache; image caching on nodes; keep traffic in the same AZ →
add cost anomaly alerts. Interface endpoints cost money too: compare.
**Postmortem:** no cost anomaly alert, no cost review in the design of the new service.

## S7. Leaked credential — M/S

**Page:** "Security alert: an access key with admin rights was pushed to a public GitHub repo 20 minutes ago."

**Hidden cause:** a developer committed a `.env` file. Someone already used the key from an
unknown IP to list buckets and start instances.

Outputs:
- Audit log (CloudTrail or equivalent): `ListBuckets`, `RunInstances` ×12 from an unknown IP, region you don't use
- Git history: `.env` added in the last commit
- Billing: instances of a large GPU type starting in another region

**Strong path:** deactivate the key first (not delete the commit first) → deactivating does not
kill temporary sessions already issued from it: deny them (policy on `aws:TokenIssueTime`) →
look for persistence (new IAM users, keys, roles) → stop the instances, check every region →
review what was accessed → rotate other secrets → then clean history.
**Postmortem:** long-lived admin key, no secret scanning on push, no region guardrails.
**Red flag:** rewriting Git history before revoking the key.

## S8. Slow API after a database migration — S/St

**Page:** "p99 latency of the orders API went from 200 ms to 4 s. Started after last night's migration."

**Hidden cause:** the migration added a column and recreated a table, dropping an index
on `customer_id`. Queries now do full table scans.

Outputs:
- APM trace: 95% of the time in one SQL query `SELECT … WHERE customer_id = $1`
- `EXPLAIN ANALYZE`: `Seq Scan on orders … rows=48000000`
- Migration diff: table recreated, index on `customer_id` missing
- DB CPU: 95%

**Strong path:** traces point to the query → explain plan → recreate the index concurrently
(`CREATE INDEX CONCURRENTLY`) → check other missing indexes.
**Postmortem:** migrations not reviewed for indexes, no performance check in staging with real data volume.
