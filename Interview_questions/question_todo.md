# Interview Practice Backlog & Questions To-Do

> Master guide on how to portray answers in real-time interviews: sequence of steps, choice of words, spoken scripts, and fast revision subpoints.

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

> **Question:**  
> *A new firmware build pipeline starts failing during artifact uploads. S3 shows no error, Jenkins logs are incomplete. Where do you start?*

#### 🗣️ How to Portray Your Answer in the Interview:
- **The Opening Anchor (First 15s):**  
  *"When a build agent logs terminate abruptly without an S3 error code or standard stack trace, it almost always means the build process died outside of the JVM/runtime control—typically via an OS kernel kill (OOM), an exhausted local disk, or an expired STS token mid-flight."*
- **Step 1: Check Host Kernel & Process Death (OOM Killer):**  
  *"I immediately check `dmesg -T` on the Jenkins runner host to see if the Linux Out-Of-Memory killer terminated the upload process when buffering the firmware into RAM."*
- **Step 2: Check Local Disk & Ephemeral Workspace Saturation:**  
  *"Next, I inspect the agent workspace disk (`df -h /var/lib/jenkins`). Multi-gigabyte firmware builds often exhaust ephemeral storage during compression, causing the agent process to abruptly crash."*
- **Step 3: Audit IAM STS Token Lifetime:**  
  *"If the build takes over 1 hour, I check whether the temporary IAM role credentials (`AWS_SESSION_TOKEN`) expired right before the upload started."*
- **Step 4: Check S3 Multipart Upload & MTU Fragmentation:**  
  *"For files over 100MB, S3 switches to multipart upload. I check if packet fragmentation across intermediate NAT gateways or proxy buffer limits (`client_max_body_size`) dropped chunks silently."*

#### ⚡ Fast Revision Subpoints:
```bash
# Check OOM Killer invocations
dmesg -T | grep -i "killed process"

# Check disk space on runner
df -h /tmp /var/lib/jenkins

# Test upload directly from runner with verbose AWS CLI debug
aws s3 cp firmware.bin s3://firmware-bucket/ --debug
```

---

### Scenario 2: Cilium CNI Update Breaks Inter-Pod Communication

> **Question:**  
> *You deployed a Cilium update, and suddenly inter-pod communication breaks for one namespace. What’s your triage approach to confirm root cause?*

#### 🗣️ How to Portray Your Answer in the Interview:
- **The Opening Anchor (First 15s):**  
  *"Since the failure is isolated to a single namespace following a CNI update, the underlying node network and overlay tunnels are healthy. This immediately points to namespace-scoped policy enforcement changes or eBPF BPF map capacity exhaustion for those specific endpoints."*
- **Step 1: Inspect Cilium Endpoint Health & Drop Monitor:**  
  *"My first command is `cilium-dbg monitor --type drop` to capture real-time packet drops at the kernel eBPF probe level."*
- **Step 2: Check Stricter NetworkPolicy Parsing:**  
  *"Cilium minor updates frequently tighten syntax validation for `CiliumNetworkPolicy` or CIDR rules. An existing malformed policy that previous versions silently bypassed may now be strictly enforced as a default deny."*
- **Step 3: Check eBPF Map Limits on Local Nodes:**  
  *"I run `cilium status` to check if nodes hosting this namespace exhausted eBPF map limits (`cilium_lxc` or `cilium_ipcache`)."*

#### ⚡ Fast Revision Subpoints:
```bash
# Check Cilium real-time packet drops
cilium-dbg monitor --type drop

# Inspect endpoint status and policy enforcement
cilium-dbg endpoint list

# Validate active Cilium network policies in the namespace
kubectl get cnp,ccnp -n <failing-namespace>
```

---

### Scenario 3: European Supercharger Fleet "Backend Unavailable"

> **Question:**  
> *30% of Tesla Superchargers in Europe report "backend unavailable", but cloud dashboards show all green. How would you isolate whether it’s DNS, TLS, or routing drift?*

#### 🗣️ How to Portray Your Answer in the Interview:
- **The Opening Anchor (First 15s):**  
  *"When 30% of geographically dispersed hardware nodes fail while cloud metrics remain 100% green, it proves the failure is happening before requests reach our central application ingress.  
  I isolate this by systematically separating DNS resolution, TLS termination, and BGP Anycast routing."*
- **Step 1: Test Regional Geo-DNS Resolution:**  
  *"First, I probe DNS from multiple European ISP vantage points (`dig @resolver-ip supercharger.tesla.com`). I verify if regional ISP resolvers are serving stale records, experiencing split-horizon drift, or returning expired CDN IP addresses."*
- **Step 2: Verify TLS Certificate Chain & OCSP Stapling:**  
  *"Next, I verify the TLS handshake. Edge hardware firmware often has baked-in root certificates. If an intermediate CA rotated or if OCSP stapling servers are unreachable over satellite/cellular uplinks, the embedded client drops the connection before sending HTTP bytes."*
- **Step 3: Analyze BGP Anycast & ISP Peering Routing:**  
  *"Finally, I inspect BGP routing table announcements. If a major European telecommunication transit provider has a route flapping or a misconfigured AS-path, packets from 30% of charging stations are blackholed at the transit peering boundary."*

#### ⚡ Fast Revision Subpoints:
```bash
# Query specific regional DNS resolvers
dig +trace @8.8.8.8 supercharger.backend.tesla.com

# Test TLS handshake, cipher support, and certificate expiration
openssl s_client -connect supercharger.backend.tesla.com:443 -servername supercharger.backend.tesla.com -showcerts

# Check BGP route convergence and packet reachability
traceroute -T -p 443 supercharger.backend.tesla.com
```

---

## Round 3: Leadership, Reliability Culture & Innovation DNA

### Scenario 4: Defining SLOs Across Diverse Systems

> **Question:**  
> *Tesla infra spans manufacturing, energy, and vehicle telemetry, how would you define SLOs that make sense across such different systems?*

#### 🗣️ How to Portray Your Answer in the Interview:
- **The Core Framing:**  
  *"I never define SLOs based on generic infrastructure metrics like `CPU < 80%`. Infrastructure utilization doesn't measure customer pain. I define SLOs around the **cost of failure** and **critical user journeys** specific to each business domain:  
  1. **Manufacturing (Gigafactory Lines):** Availability is measured in physical seconds. If a conveyor or stamping press API stops responding for 50ms, physical robots halt. SLO: `99.99%` of press telemetry acknowledged in `< 20ms`.  
  2. **Energy Grid (Megapack/Powerwall):** Focuses on data durability and regulatory dispatch commands. SLO: `99.9%` of grid dispatch signals received within 1 second.  
  3. **Vehicle Fleet Telemetry:** Highly tolerant of intermittent drops because cars buffer data locally on flash memory. SLO: Focuses on ingestion pipeline freshness—`99.5%` of telemetry queryable within 5 minutes of vehicle upload."*

---

### Scenario 5: Enforcing Daily Release Confidence Without Sacrificing Speed

> **Question:**  
> *When engineers ship daily, how do you enforce release confidence without slowing down innovation?*

#### 🗣️ How to Portray Your Answer in the Interview:
- **The Core Framing:**  
  *"The wrong answer is adding more manual approval gates or change review boards. That slows innovation without improving safety.  
  Instead, I build automated guardrails that decouple deployment from release:  
  1. **Automated Canary Analysis (ACA):** Every merge automatically deploys to a 5% canary. Prometheus monitors error rates and p99 latency against the baseline. If error budgets dip, the system rolls back automatically without human intervention.  
  2. **Contract & Regression Testing in CI:** Fast, ephemeral PR preview environments spun up via lightweight clusters.  
  3. **Feature Flags:** Code ships dark behind feature flags, allowing product teams to decouple technical deployment from business launch."*

---

### Scenario 6: Chaos Engineering for Edge Clusters

> **Question:**  
> *You’re tasked with introducing chaos testing for edge clusters, how do you scope risk boundaries?*

#### 🗣️ How to Portray Your Answer in the Interview:
- **The Core Framing:**  
  *"In physical edge environments—like factory assembly lines or vehicle charging hubs—chaos testing cannot follow the traditional Netflix 'Chaos Monkey in Production' model because physical equipment can get damaged.  
  I enforce 3 non-negotiable risk boundaries:  
  1. **Ring 0 / Hardware-In-The-Loop (HIL) Lab First:** Chaos experiments must run against identical physical hardware test benches in the lab before touching a live production site.  
  2. **Automated Dead-Man Kill Switches:** If safety sensor latency spikes or factory line throughput dips by 1%, the chaos orchestrator immediately aborts and restores baseline state.  
  3. **Single-Variable Failure Testing:** Test one failure domain at a time (e.g., pulling WAN connection, simulating local disk corruption, introducing NTP clock drift) rather than chaotic multi-failure cascades."*

---

### Scenario 7: Mentoring DevOps Engineers to Think Systemically

> **Question:**  
> *What’s your approach to mentoring DevOps engineers to think systemically, not syntactically?*

#### 🗣️ How to Portray Your Answer in the Interview:
- **The Core Framing:**  
  *"Many junior engineers think syntactically: 'Which Terraform flag or Helm parameter do I paste to fix this?'  
  My approach to shifting them to systemic thinking follows three pillars:  
  1. **First-Principles Deep Dives:** Instead of treating Kubernetes as a black box, we trace requests through the Linux kernel: cgroups, namespaces, eBPF maps, and TCP state machines.  
  2. **Blameless Post-Mortems as Learning Labs:** When an outage occurs, we never blame human error. We trace the systemic ergonomic flaws in the tooling, the missing observability signals, and the blast radius boundaries.  
  3. **RFC & Design Reviews:** I require engineers to write 1-page Design RFCs evaluating alternative trade-offs, disaster recovery steps, and operational cost 2 years into the future before writing a line of code."*

---

## Key Takeaway

> [!TIP]
> **Summary Insight for Interviews:**  
> Senior engineers answer questions in a predictable, comforting rhythm:  
> 1. **Immediate Composure & Mental Model:** State your high-level hypothesis.  
> 2. **Sequential Triage Path:** Move from client to edge, then to runtime, then to storage/OS.  
> 3. **Concrete CLI Subpoints:** Mention the exact flags and commands without getting bogged down in syntax.  
> 4. **Decisive Action / Escalation:** State the clear remediation or hand-off artifact.
