# High-Scale Systems & SRE Interview Preparation Guide

A curated, production-tested collection of system design scenarios, live incident simulations, Kubernetes root cause analysis (RCA), and site reliability engineering (SRE) interview questions.

---

## 💡 How These Guides Are Structured

Every scenario is formatted specifically to help you succeed in real-time interviews:
1. **🗣️ The Spoken Script (Choice of Words):** The exact phrasing, senior vocabulary, and narrative structure to deliver aloud with composure.
2. **🪜 Sequential Steps:** Clear step-by-step triage paths so you never freeze or jump randomly between layers.
3. **⚡ Fast Revision Subpoints:** Exact CLI commands, flags, Prometheus queries, and sysctl tunables for a 30-second review before your interview.
4. **📊 Visual Flowcharts:** Compact Mermaid diagrams visualizing the troubleshooting decision tree or architecture.

---

## Navigation & Study Modules

| Module | Primary Topics | Core Focus | Link |
|:---|:---|:---|:---:|
| **Kubernetes Troubleshooting** | • Pod stuck in `ContainerCreating`<br/>• Kubelet events & CRI/containerd diagnostics<br/>• PVC/VolumeAttachment locks<br/>• CNI/IPAM exhaustion | Node-level triage, storage locks, IP exhaustion | [Explore Guide](kubernetes_interview_question.md) |
| **System Design Architecture** | • Secure Hybrid CI/CD Pipeline (Cloud + Edge)<br/>• "Push to Cloud, Pull to Edge" philosophy<br/>• SLSA compliance, OIDC, Cosign & SBOM<br/>• GitOps with ArgoCD & Flagger canary rollouts | Zero long-lived secrets, outbound-only edge pull | [Explore Guide](system_design_question.md) |
| **Real Production Incidents** | • Kafka + Redis intermittent failure triage<br/>• NGINX Ingress gradual latency creep (CPU flat)<br/>• TCP socket backlogs, `somaxconn`, conntrack<br/>• Redis `SLOWLOG` & Kafka queue delays | High-concurrency triage, I/O wait vs CPU bottlenecks | [Explore Guide](real_usecase_interview_question.md) |
| **Practice Backlog & Fire Drills** | • Firmware build pipeline S3 upload failure<br/>• Cilium CNI inter-pod communication break<br/>• Supercharger fleet geo-DNS & BGP drift<br/>• Defining real SLOs across manufacturing & telemetry | Incident simulation, chaos testing & SRE culture | [Explore Guide](question_todo.md) |

---

## Triage Quick Reference

```text
Symptom: Pod stuck in ContainerCreating
  └─► Command 1: kubectl describe pod <pod> -n <ns> (Check FailedMount, FailedCreatePodSandBox)
  └─► Command 2: kubectl get pvc,pv,volumeattachment OR kubectl describe node <node>
  └─► Command 3: journalctl -u kubelet / containerd (Check CRI hangs, disk space)

Symptom: NGINX Ingress Latency Creep (CPU/RAM Flat)
  └─► Step 1: Compare request_time vs upstream_response_time (Proxy vs App)
  └─► Step 2: Check worker_connections, ss -s, and upstream-keepalive-connections
  └─► Step 3: Inspect proxy disk buffer spills (grep "buffered to a temporary file")
  └─► Step 4: Check kernel socket overflow: netstat -s | grep listen and /proc/sys/net/core/somaxconn

Symptom: Redis + Kafka Service Intermittent Drops
  └─► Step 1: Check Ingress codes (504 = latency, 503 = pool exhaustion)
  └─► Step 2: redis-cli SLOWLOG GET 50 & redis-cli --latency (Check single-thread locks)
  └─► Step 3: Check Kafka max.block.ms buffer timeouts & RequestQueueTimeMs
  └─► Step 4: Inspect kernel CFS throttling: rate(container_cpu_cfs_throttled_periods_total[5m])
```

---

## Recommended Study Path

1. **Step 1:** Review [Kubernetes Troubleshooting](kubernetes_interview_question.md) to master low-level pod lifecycle and CRI/CSI triage.
2. **Step 2:** Study [Real-World Incidents](real_usecase_interview_question.md) to practice distinguishing platform/network issues from application bottlenecks.
3. **Step 3:** Deepen architectural knowledge with [System Design](system_design_question.md) on hybrid cloud/edge topologies.
4. **Step 4:** Work through the open scenarios in the [Practice Backlog](question_todo.md) using the provided investigation angles.
