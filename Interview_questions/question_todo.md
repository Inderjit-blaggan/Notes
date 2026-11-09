# Interview Practice Backlog & Questions To-Do

> Comprehensive practice scenarios covering Incident Simulation, Root Cause Analysis (RCA), Systems Architecture, and Reliability Engineering Leadership.

---

## Practice Tracker

| Status | Question / Scenario | Topic Area |
|:---:|:---|:---|
| ⏳ **Pending** | [Firmware Build Pipeline S3 Upload Failure](#scenario-1-firmware-build-pipeline-s3-upload-failure) | CI/CD, Storage & Networking |
| ⏳ **Pending** | [Cilium CNI Update Breaks Inter-Pod Communication](#scenario-2-cilium-cni-update-breaks-inter-pod-communication) | Kubernetes CNI, eBPF & Network Policies |
| ⏳ **Pending** | [European Supercharger Fleet "Backend Unavailable"](#scenario-3-european-supercharger-fleet-backend-unavailable) | Edge Telemetry, Geo-DNS, TLS & Routing |
| ⏳ **Pending** | [Defining SLOs Across Diverse Systems](#scenario-4-defining-slos-across-diverse-systems) | SRE, Observability & Platform Strategy |
| ⏳ **Pending** | [Enforcing Daily Release Confidence](#scenario-5-enforcing-daily-release-confidence-without-sacrificing-speed) | Delivery Engineering, Guardrails & Quality |
| ⏳ **Pending** | [Chaos Engineering for Edge Clusters](#scenario-6-chaos-engineering-for-edge-clusters) | Resilience, Edge Computing & Blast Radius |
| ⏳ **Pending** | [Mentoring Engineers to Think Systemically](#scenario-7-mentoring-devops-engineers-to-think-systemically) | Engineering Culture, Mentorship & Leadership |

---

## Round 2: Incident Simulation, RCA & Energy Grid Fire Drills

### Scenario 1: Firmware Build Pipeline S3 Upload Failure

- [ ] **Question:**  
  *A new firmware build pipeline starts failing during artifact uploads. Amazon S3 shows no service errors or permission rejections, and Jenkins logs terminate abruptly without a clear error message. Where do you start?*

#### Key Investigation Angles to Prepare:
- **Ephemeral Runner Storage & Disk Fill:** Check if the local agent workspace ran out of disk space (`/var/lib/jenkins` or `/tmp`) midway through multi-gigabyte firmware artifact compression.
- **IAM Session / STS Token Expiration:** If the build pipeline takes 45+ minutes, check whether the assumed IAM role temporary credentials (STS token lifetime) expired before the upload phase initiated.
- **Multipart Upload & Network Buffer Drift:** S3 CLI switches to multipart upload for files >100MB. Check for MTU packet fragmentation, intermediate NAT gateway connection resets, or proxy body size limits (`client_max_body_size`).
- **OOM Killer on Runner Agent:** Inspect `dmesg -T` or `journalctl` on the Jenkins runner host to verify if the Java runtime or upload process was terminated by the Linux Out-Of-Memory (OOM) killer.

---

### Scenario 2: Cilium CNI Update Breaks Inter-Pod Communication

- [ ] **Question:**  
  *You deployed an update to Cilium CNI, and suddenly inter-pod communication breaks exclusively for one specific namespace while other namespaces remain healthy. What is your systematic triage approach to confirm the root cause?*

#### Key Investigation Angles to Prepare:
- **Cilium eBPF Map Limits & Exhaustion:** Run `cilium status` and check if BPF map capacities (e.g., `cilium_lxc`, `cilium_ipcache`, or conntrack maps) reached saturation on nodes hosting that namespace.
- **Stricter NetworkPolicy Enforcement:** Cilium updates often introduce stricter default validation or bug fixes around `CiliumNetworkPolicy` or standard `NetworkPolicy` CIDR parsing. Check whether an existing policy in that namespace had invalid syntax that previous versions silently ignored.
- **Cilium Endpoints & Node Connectivity:** Run:
  ```bash
  cilium-dbg endpoint list
  cilium-dbg monitor --type drop
  ```
  Look for drop reasons such as `Policy denied`, `Invalid source IP`, or `Encapsulation failure`.
- **IPAM & Subnet Overlay Routing:** Verify whether the pods in that namespace were assigned an IP range affected by an invalid Geneve/VXLAN overlay route or MTU mismatch.

---

### Scenario 3: European Supercharger Fleet "Backend Unavailable"

- [ ] **Question:**  
  *30% of Tesla Superchargers across Europe report "backend unavailable", yet all central cloud dashboards show 100% green and healthy. How would you isolate whether the root cause is DNS, TLS certificate validation, or BGP routing drift?*

#### Key Investigation Angles to Prepare:
- **Anycast BGP Routing & Regional Peering:** Inspect whether local ISP peering points or Anycast routes are blackholing traffic from specific European telco providers before it hits the central cloud edge.
- **Geo-DNS / Split-Horizon DNS Resolution:** Probe external DNS resolvers across European regions (e.g., `dig @resolver-ip supercharger.backend.tesla.com`) to check for stale records, TTL caching anomalies, or regional CDN resolution failures.
- **TLS Handshake & Intermediate Certificate Chain:** Verify if an intermediate CA expired or if edge hardware firmware lacks newly rotated root certificates. Check OCSP stapling response times.
- **Telemetry Ingestion Queue Backpressure:** Verify if the devices are connecting to an edge gateway that terminates TCP cleanly (showing green on dashboards) but drops payloads downstream before entering Kafka/MQTT brokers.

---

## Round 3: Leadership, Reliability Culture & Innovation DNA

### Scenario 4: Defining SLOs Across Diverse Systems

- [ ] **Question:**  
  *Tesla infrastructure spans manufacturing factories, energy grid storage, and high-frequency vehicle telemetry. How would you define Service Level Objectives (SLOs) that make sense across such radically different operating environments?*

#### Core Principles to Articulate:
- **Contextual Failure Costs:** In manufacturing (Gigafactory assembly lines), downtime directly halts physical robots (cost measured in minutes of halted production). In vehicle telemetry, loss of real-time packets is tolerable if buffered locally on-car.
- **User-Centric SLIs over Generic Server Metrics:** Replace superficial indicators (e.g., `CPU < 80%`) with business-critical user journeys (e.g., *“Time to unlock car via app < 2.5s”* or *“Factory stamping press telemetry acknowledged < 50ms”*).
- **Error Budget Policies:** Tie error budgets directly to release velocity. If an energy grid system burns 50% of its monthly error budget in week 1, deployment gates automatically block feature releases in favor of reliability sprints.

---

### Scenario 5: Enforcing Daily Release Confidence Without Sacrificing Speed

- [ ] **Question:**  
  *When engineering teams ship code daily to production, how do you enforce release confidence and safety without slowing down developer velocity?*

#### Core Principles to Articulate:
- **Automated Canary Analysis (ACA):** Decouple deployments from releases using progressive rollouts (e.g., Argo Rollouts with Prometheus metric analysis) that automatically advance or roll back based on real-time error budgets.
- **Ephemeral Preview Environments:** Provide developers with on-demand isolated test environments spun up automatically per pull request.
- **Shift-Left Automated Guardrails:** Embed static analysis, container scanning, and contract tests directly into CI so security and compliance validation happens asynchronously before human code review.

---

### Scenario 6: Chaos Engineering for Edge Clusters

- [ ] **Question:**  
  *You are tasked with introducing chaos engineering to factory and edge clusters. How do you scope risk boundaries to prevent impacting physical operations?*

#### Core Principles to Articulate:
- **Strict Blast Radius Containment:** Start with "Shadow Mode" or "Hardware-In-The-Loop" (HIL) lab environments before touching live production edge clusters.
- **Automated Kill Switches & Abort Triggers:** Implement dead-man switches that instantly halt the chaos experiment if physical factory telemetry latency exceeds safety thresholds.
- **Hypothesis-Driven Testing:** Every chaos experiment must start with a written hypothesis and expected recovery time, focusing on single failure domains (e.g., WAN link severance, single-node power drop, NTP time drift).

---

### Scenario 7: Mentoring DevOps Engineers to Think Systemically

- [ ] **Question:**  
  *What is your approach to mentoring DevOps and platform engineers to think systemically about architecture rather than syntactically about tooling?*

#### Core Principles to Articulate:
- **First-Principles Problem Solving:** Encourage understanding the underlying OS and network primitives (e.g., Linux cgroups, TCP state machines, eBPF) rather than merely copying YAML syntax or Helm values.
- **Blameless Post-Mortems with Systems Mapping:** Guide engineers to analyze systemic contributing factors (ergonomics of tools, missing alerts, blast radius boundaries) rather than attributing root cause to "human error".
- **Architecture Reviews & Design RFCs:** Involve engineers early in cross-team design proposals, asking them to evaluate edge cases, disaster recovery paths, and maintenance overhead 2 years down the road.

---

## Key Takeaway

> [!TIP]
> **Summary Insight:**  
> High-scale infrastructure interviews evaluate more than memorized commands or tool names. They test your **judgment under ambiguity**, your **instinctive triage path under pressure**, and your ability to design resilient systems that fail gracefully without human intervention.
