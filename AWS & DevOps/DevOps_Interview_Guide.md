# DevOps — Senior Software Engineer Interview Guide

Organized in the order code flows: **Foundations → Plan → Code → Build → Test → Secure → CI/CD → IaC → Runtime (Kubernetes) → GitOps → Secrets → Policy → Observability → SRE → Platform Engineering → FinOps → AI in DevOps**, ending with an end-to-end "connect the dots" section.

Each section covers **what it is, why it exists, key concepts and uses**, followed by **5+ frequently asked interview questions with senior-level answers**.

> Tools evolve quickly (licenses change, projects get archived or forked). Facts reflect the ecosystem as of 2025–2026 — verify the latest status before quoting specifics in an interview.

## Table of Contents

**Layer 0 — Foundations**

- [1. Linux](#1-linux)
- [2. Networking](#2-networking)
- [3. Git](#3-git)
- [4. Scripting (Bash, Python, jq/yq)](#4-scripting-bash-python-jqyq)
- [5. Data Formats (YAML, JSON, HCL)](#5-data-formats-yaml-json-hcl)
- [6. Cloud Basics (AWS / GCP / Azure)](#6-cloud-basics-aws--gcp--azure)

**Stage 1 — Plan & Collaborate**

- [7. DevOps Culture, Agile & Planning Tools](#7-devops-culture-agile--planning-tools)
- [8. ADRs, RFCs & Design Reviews](#8-adrs-rfcs--design-reviews)
- [9. Branching Strategies](#9-branching-strategies)

**Stage 2 — Code & Version Control**

- [10. Git Platforms & Pull Request Practices](#10-git-platforms--pull-request-practices)
- [11. Monorepo vs Polyrepo (Bazel, Nx, Turborepo, Pants)](#11-monorepo-vs-polyrepo-bazel-nx-turborepo-pants)
- [12. Local Quality Gates (pre-commit, Linters, Formatters, EditorConfig)](#12-local-quality-gates-pre-commit-linters-formatters-editorconfig)
- [13. Developer Environments (Dev Containers, Codespaces, Gitpod, Tilt/Skaffold)](#13-developer-environments-dev-containers-codespaces-gitpod-tiltskaffold)

**Stage 3 — Build & Package**

- [14. Build Tools (Maven/Gradle, npm/pnpm, Go modules, Poetry/uv, Make)](#14-build-tools-mavengradle-npmpnpm-go-modules-poetryuv-make)
- [15. Docker Fundamentals](#15-docker-fundamentals)
- [16. Image Hardening & Build Tooling (BuildKit/buildx, Podman, Buildah, Kaniko, Buildpacks)](#16-image-hardening--build-tooling-buildkitbuildx-podman-buildah-kaniko-buildpacks)
- [17. Container Standards & Runtimes (OCI, containerd, CRI-O)](#17-container-standards--runtimes-oci-containerd-cri-o)
- [18. Artifact Registries, Versioning & "Build Once, Deploy Many"](#18-artifact-registries-versioning--build-once-deploy-many)

**Stage 4 — Test & Quality**

- [19. Test Pyramid & Testing Tools (Testcontainers, Pact, Playwright, Cypress)](#19-test-pyramid--testing-tools-testcontainers-pact-playwright-cypress)
- [20. Performance & Load Testing (k6, JMeter, Gatling, Locust)](#20-performance--load-testing-k6-jmeter-gatling-locust)
- [21. Code Quality (SonarQube, Coverage Gates, Mutation Testing)](#21-code-quality-sonarqube-coverage-gates-mutation-testing)
- [22. Test Strategy in CI (Parallelization, Flaky Tests, Test Impact Analysis, Preview Environments)](#22-test-strategy-in-ci-parallelization-flaky-tests-test-impact-analysis-preview-environments)

**Stage 5 — Security (DevSecOps, "Shift Left")**

- [23. SAST — Static Application Security Testing (Semgrep, CodeQL, SonarQube, Checkmarx)](#23-sast--static-application-security-testing-semgrep-codeql-sonarqube-checkmarx)
- [24. SCA — Software Composition Analysis (Snyk, Dependabot, Renovate, OWASP Dependency-Check, Grype)](#24-sca--software-composition-analysis-snyk-dependabot-renovate-owasp-dependency-check-grype)
- [25. Secret Scanning (Gitleaks, TruffleHog, GitHub Secret Scanning & Push Protection)](#25-secret-scanning-gitleaks-trufflehog-github-secret-scanning--push-protection)
- [26. Container Image Scanning & IaC Scanning (Trivy, Grype, Docker Scout, Inspector, Checkov, KICS, Terrascan)](#26-container-image-scanning--iac-scanning-trivy-grype-docker-scout-inspector-checkov-kics-terrascan)
- [27. DAST — Dynamic Application Security Testing (OWASP ZAP, Burp Suite)](#27-dast--dynamic-application-security-testing-owasp-zap-burp-suite)
- [28. Software Supply Chain Security (SBOM, SLSA, Sigstore/cosign, in-toto, Provenance)](#28-software-supply-chain-security-sbom-slsa-sigstorecosign-in-toto-provenance)
- [29. Runtime Security (Falco, Tetragon, GuardDuty, Microsoft Defender)](#29-runtime-security-falco-tetragon-guardduty-microsoft-defender)
- [30. Identity in Pipelines (OIDC Federation, Least-Privilege Deploy Roles)](#30-identity-in-pipelines-oidc-federation-least-privilege-deploy-roles)

**Stage 6 — CI/CD Pipelines**

- [31. GitHub Actions](#31-github-actions)
- [32. GitLab CI/CD](#32-gitlab-cicd)
- [33. Jenkins](#33-jenkins)
- [34. Other CI/CD Platforms (CircleCI, Azure DevOps, Buildkite, Tekton, AWS CodePipeline)](#34-other-cicd-platforms-circleci-azure-devops-buildkite-tekton-aws-codepipeline)
- [35. Pipeline Design & Concepts](#35-pipeline-design--concepts)
- [36. Deployment Strategies](#36-deployment-strategies)
- [37. Feature Flags (LaunchDarkly, Unleash, Flagsmith, OpenFeature, AWS AppConfig)](#37-feature-flags-launchdarkly-unleash-flagsmith-openfeature-aws-appconfig)
- [38. Database Changes in CI/CD (Flyway, Liquibase, Expand/Contract)](#38-database-changes-in-cicd-flyway-liquibase-expandcontract)

**Stage 7 — Infrastructure as Code (IaC) & Configuration**

- [39. Terraform / OpenTofu](#39-terraform--opentofu)
- [40. Pulumi, CloudFormation/CDK & Azure Bicep (Other Declarative IaC)](#40-pulumi-cloudformationcdk--azure-bicep-other-declarative-iac)
- [41. Crossplane (Kubernetes-Native IaC)](#41-crossplane-kubernetes-native-iac)
- [42. Ansible (and Chef/Puppet)](#42-ansible-and-chefpuppet)
- [43. Image Building with Packer & EC2 Image Builder (Immutable Infrastructure)](#43-image-building-with-packer--ec2-image-builder-immutable-infrastructure)
- [44. IaC Workflow Automation (HCP Terraform, Atlantis, Spacelift, env0, Terragrunt)](#44-iac-workflow-automation-hcp-terraform-atlantis-spacelift-env0-terragrunt)
- [45. IaC Testing, Policy & Cost (Terratest, `terraform test`, Checkov, Conftest/OPA, Sentinel, Infracost)](#45-iac-testing-policy--cost-terratest-terraform-test-checkov-conftestopa-sentinel-infracost)

**Stage 8 — Runtime Platform: Kubernetes & Ecosystem**

- [46. Kubernetes Architecture](#46-kubernetes-architecture)
- [47. Kubernetes Workloads (Pod, Deployment, ReplicaSet, StatefulSet, DaemonSet, Job, CronJob)](#47-kubernetes-workloads-pod-deployment-replicaset-statefulset-daemonset-job-cronjob)
- [48. Kubernetes Networking (Services, Ingress, Gateway API, CoreDNS, CNI)](#48-kubernetes-networking-services-ingress-gateway-api-coredns-cni)
- [49. Kubernetes Configuration & Storage (ConfigMaps, Secrets, PV/PVC, StorageClass, CSI)](#49-kubernetes-configuration--storage-configmaps-secrets-pvpvc-storageclass-csi)
- [50. Kubernetes Health & Resources (Probes, Requests/Limits, QoS, PDBs)](#50-kubernetes-health--resources-probes-requestslimits-qos-pdbs)
- [51. Kubernetes Scheduling (Selectors, Affinity, Taints/Tolerations, Topology Spread, Priority)](#51-kubernetes-scheduling-selectors-affinity-taintstolerations-topology-spread-priority)
- [52. Kubernetes Security (RBAC, ServiceAccounts, NetworkPolicies, Pod Security Standards, Admission Controllers)](#52-kubernetes-security-rbac-serviceaccounts-networkpolicies-pod-security-standards-admission-controllers)
- [53. Kubernetes Extensibility (CRDs, Operators, Controllers & the Reconcile Loop)](#53-kubernetes-extensibility-crds-operators-controllers--the-reconcile-loop)
- [54. Kubernetes Troubleshooting](#54-kubernetes-troubleshooting)
- [55. Helm & Kustomize (Packaging)](#55-helm--kustomize-packaging)
- [56. Kubernetes Autoscaling (HPA, VPA, Cluster Autoscaler, Karpenter, KEDA)](#56-kubernetes-autoscaling-hpa-vpa-cluster-autoscaler-karpenter-keda)
- [57. Cilium & eBPF Networking](#57-cilium--ebpf-networking)
- [58. Service Mesh (Istio, Linkerd, Envoy) & Ingress Controllers](#58-service-mesh-istio-linkerd-envoy--ingress-controllers)
- [59. Managed Kubernetes & Alternatives (EKS, GKE, AKS, k3s/kind/minikube, ECS, Cloud Run)](#59-managed-kubernetes--alternatives-eks-gke-aks-k3skindminikube-ecs-cloud-run)
- [60. Multi-Cluster Management & Backup (Cluster API, Karmada, Velero)](#60-multi-cluster-management--backup-cluster-api-karmada-velero)

**Stage 9 — GitOps & Progressive Delivery**

- [61. GitOps Principles (and Push vs Pull Deployments)](#61-gitops-principles-and-push-vs-pull-deployments)
- [62. Argo CD](#62-argo-cd)
- [63. Flux](#63-flux)
- [64. Progressive Delivery (Argo Rollouts, Flagger)](#64-progressive-delivery-argo-rollouts-flagger)

**Stage 10 — Configuration & Secrets Management**

- [65. HashiCorp Vault & OpenBao](#65-hashicorp-vault--openbao)
- [66. Kubernetes Secrets Integration & Cloud Secret Stores (External Secrets Operator, Secrets Store CSI, Sealed Secrets, SOPS)](#66-kubernetes-secrets-integration--cloud-secret-stores-external-secrets-operator-secrets-store-csi-sealed-secrets-sops)
- [67. Certificates (cert-manager, Let's Encrypt, ACM)](#67-certificates-cert-manager-lets-encrypt-acm)
- [68. Workload Identity (SPIFFE/SPIRE, IRSA/Pod Identity, GCP & Azure Workload Identity)](#68-workload-identity-spiffespire-irsapod-identity-gcp--azure-workload-identity)

**Stage 11 — Policy as Code & Governance**

- [69. Policy as Code (OPA/Gatekeeper, Kyverno, Conftest, ValidatingAdmissionPolicy)](#69-policy-as-code-opagatekeeper-kyverno-conftest-validatingadmissionpolicy)
- [70. Compliance (CIS Benchmarks, kube-bench, Audit Logging, SOC 2/ISO Evidence Automation)](#70-compliance-cis-benchmarks-kube-bench-audit-logging-soc-2iso-evidence-automation)

**Stage 12 — Observability**

- [71. Observability Concepts (Golden Signals, RED/USE, Cardinality, Sampling, Correlation, Exemplars, Synthetics vs RUM)](#71-observability-concepts-golden-signals-reduse-cardinality-sampling-correlation-exemplars-synthetics-vs-rum)
- [72. OpenTelemetry (OTel)](#72-opentelemetry-otel)
- [73. Prometheus (& Thanos/Mimir/VictoriaMetrics, Alertmanager)](#73-prometheus--thanosmimirvictoriametrics-alertmanager)
- [74. Logging (Loki, ELK/EFK, Fluentd/Fluent Bit, Vector, Splunk, Datadog Logs)](#74-logging-loki-elkefk-fluentdfluent-bit-vector-splunk-datadog-logs)
- [75. Distributed Tracing Backends (Jaeger, Grafana Tempo, Zipkin, Datadog APM, X-Ray, Honeycomb)](#75-distributed-tracing-backends-jaeger-grafana-tempo-zipkin-datadog-apm-x-ray-honeycomb)
- [76. Grafana & Observability Platforms (Datadog, New Relic, Dynatrace)](#76-grafana--observability-platforms-datadog-new-relic-dynatrace)
- [77. Profiling & eBPF-Based Observability (Pyroscope, Parca, Datadog Profiler, Pixie, Hubble, Beyla)](#77-profiling--ebpf-based-observability-pyroscope-parca-datadog-profiler-pixie-hubble-beyla)

**Stage 13 — Reliability (SRE) & Incident Management**

- [78. SLI, SLO, SLA, Error Budgets & Burn-Rate Alerting](#78-sli-slo-sla-error-budgets--burn-rate-alerting)
- [79. DORA Metrics](#79-dora-metrics)
- [80. On-Call & Incident Management (PagerDuty, incident.io, Rootly, FireHydrant, Grafana OnCall, Postmortems, Runbooks)](#80-on-call--incident-management-pagerduty-incidentio-rootly-firehydrant-grafana-oncall-postmortems-runbooks)
- [81. Chaos Engineering (Chaos Mesh, LitmusChaos, Gremlin, AWS FIS, Game Days)](#81-chaos-engineering-chaos-mesh-litmuschaos-gremlin-aws-fis-game-days)
- [82. Resilience, Capacity Planning & Disaster Recovery](#82-resilience-capacity-planning--disaster-recovery)
- [83. Toil Reduction & Automation](#83-toil-reduction--automation)

**Stage 14 — Platform Engineering & Developer Experience**

- [84. Internal Developer Platforms (IDP), Golden Paths & Platform-as-a-Product](#84-internal-developer-platforms-idp-golden-paths--platform-as-a-product)
- [85. Developer Portals (Backstage, Port, Cortex)](#85-developer-portals-backstage-port-cortex)
- [86. Platform Orchestration & Ephemeral Environments (Humanitec, Score, Kratix, Crossplane Compositions, vCluster, Okteto)](#86-platform-orchestration--ephemeral-environments-humanitec-score-kratix-crossplane-compositions-vcluster-okteto)
- [87. Developer Experience Measurement & Team Topologies (DORA, SPACE, DevEx, Cognitive Load)](#87-developer-experience-measurement--team-topologies-dora-space-devex-cognitive-load)

**Stage 15 — FinOps (Cost)**

- [88. FinOps & Kubernetes Cost Management (OpenCost, Kubecost, Infracost, Commitments)](#88-finops--kubernetes-cost-management-opencost-kubecost-infracost-commitments)

**The AI Layer — AI in DevOps (2025–2026)**

- [89. AI Coding Agents (GitHub Copilot, Claude Code, Cursor, Codex, Gemini CLI/Jules, Windsurf, Q Developer/Kiro, Devin)](#89-ai-coding-agents-github-copilot-claude-code-cursor-codex-gemini-clijules-windsurf-q-developerkiro-devin)
- [90. Context Engineering (AGENTS.md / CLAUDE.md, Spec-Driven Development, Rules & Steering Files)](#90-context-engineering-agentsmd--claudemd-spec-driven-development-rules--steering-files)
- [91. Model Context Protocol (MCP)](#91-model-context-protocol-mcp)
- [92. Agent-to-Agent (A2A) Protocol](#92-agent-to-agent-a2a-protocol)
- [93. AI-Generated Infrastructure Code (Terraform, Helm, Pipelines)](#93-ai-generated-infrastructure-code-terraform-helm-pipelines)
- [94. AI in Code Review, Testing & Security Autofix (Copilot Code Review, CodeRabbit, Graphite, Qodo, Claude Code GitHub Actions, Copilot Autofix, Snyk)](#94-ai-in-code-review-testing--security-autofix-copilot-code-review-coderabbit-graphite-qodo-claude-code-github-actions-copilot-autofix-snyk)
- [95. Running Agents in Pipelines & Governing AI-Generated Code](#95-running-agents-in-pipelines--governing-ai-generated-code)
- [96. AIOps for Kubernetes (K8sGPT, kagent, HolmesGPT, Kubernetes MCP Servers)](#96-aiops-for-kubernetes-k8sgpt-kagent-holmesgpt-kubernetes-mcp-servers)
- [97. AI in Observability, Incident Platforms & Cloud Provider Agents (Datadog Bits AI, Dynatrace Davis, New Relic AI, Grafana Assistant, Elastic, Splunk, PagerDuty, incident.io, Rootly, AWS DevOps Agent, Azure SRE Agent, Gemini Cloud Assist)](#97-ai-in-observability-incident-platforms--cloud-provider-agents-datadog-bits-ai-dynatrace-davis-new-relic-ai-grafana-assistant-elastic-splunk-pagerduty-incidentio-rootly-aws-devops-agent-azure-sre-agent-gemini-cloud-assist)
- [98. AIOps Patterns (Anomaly Detection, Alert Correlation & Noise Reduction, Automated RCA, Auto-Remediation)](#98-aiops-patterns-anomaly-detection-alert-correlation--noise-reduction-automated-rca-auto-remediation)
- [99. GPUs on Kubernetes (NVIDIA GPU Operator, DRA, MIG/Time-Slicing, Kueue, Volcano)](#99-gpus-on-kubernetes-nvidia-gpu-operator-dra-migtime-slicing-kueue-volcano)
- [100. Model Serving (vLLM, KServe, llm-d, NVIDIA Triton/TensorRT-LLM/Dynamo, Ray Serve, SGLang, TGI, Ollama)](#100-model-serving-vllm-kserve-llm-d-nvidia-tritontensorrt-llmdynamo-ray-serve-sglang-tgi-ollama)
- [101. ML Pipelines & Experiment Tracking (Kubeflow, MLflow, Airflow, Argo Workflows, Weights & Biases)](#101-ml-pipelines--experiment-tracking-kubeflow-mlflow-airflow-argo-workflows-weights--biases)
- [102. AI Gateways (LiteLLM, Envoy AI Gateway, Kong AI Gateway, Portkey)](#102-ai-gateways-litellm-envoy-ai-gateway-kong-ai-gateway-portkey)
- [103. LLM Observability (Langfuse, LangSmith, Arize Phoenix, Helicone, OpenTelemetry GenAI Semantic Conventions)](#103-llm-observability-langfuse-langsmith-arize-phoenix-helicone-opentelemetry-genai-semantic-conventions)
- [104. Evaluation in CI (promptfoo, DeepEval, Ragas, LLM-as-Judge Regression Suites)](#104-evaluation-in-ci-promptfoo-deepeval-ragas-llm-as-judge-regression-suites)
- [105. Agent Frameworks to Deploy & Operate (LangGraph, CrewAI, LlamaIndex, OpenAI Agents SDK, Strands, Google ADK)](#105-agent-frameworks-to-deploy--operate-langgraph-crewai-llamaindex-openai-agents-sdk-strands-google-adk)
- [106. Vector Databases (pgvector, Qdrant, Weaviate, Milvus, Pinecone, OpenSearch)](#106-vector-databases-pgvector-qdrant-weaviate-milvus-pinecone-opensearch)
- [107. AI Security (OWASP Top 10 for LLMs, Prompt Injection, Guardrails, Model & Dataset Supply Chain)](#107-ai-security-owasp-top-10-for-llms-prompt-injection-guardrails-model--dataset-supply-chain)

**Connecting the Dots**

- [108. End-to-End Scenario: One Commit's Journey (and Senior System-Design Questions)](#108-end-to-end-scenario-one-commits-journey-and-senior-system-design-questions)

---

# LAYER 0 — FOUNDATIONS

---

## 1. Linux

**What it is:** The operating system running almost every server, container and Kubernetes node. DevOps work is ultimately Linux work — containers are just Linux processes with isolation.

**Purpose:** Understanding processes, resources, files and networking at the OS level lets you debug what higher-level tools hide.

**Key concepts**
- **Processes:** PID, parent/child, `fork`/`exec`, process states (R, S, D — uninterruptible I/O wait, Z — zombie), PID 1 responsibilities (reaping zombies, handling signals).
- **Signals:** `SIGTERM` (15, graceful — can be caught), `SIGKILL` (9, cannot be caught), `SIGINT` (Ctrl+C), `SIGHUP` (reload config), `SIGCHLD`.
- **File descriptors:** Everything is a file — sockets, pipes, files. stdin/stdout/stderr = 0/1/2. Limits via `ulimit -n`.
- **Permissions:** `rwx` for user/group/other, octal (`chmod 755`), setuid/setgid/sticky bit, ownership (`chown`), capabilities (`CAP_NET_BIND_SERVICE`) instead of full root.
- **systemd:** Units, services, `systemctl start/enable/status`, `journalctl -u svc`, restart policies, timers (cron replacement).
- **cgroups & namespaces (the basis of containers):** *Namespaces* isolate what a process can **see** (pid, net, mnt, uts, ipc, user, cgroup). *cgroups* limit what it can **use** (CPU, memory, I/O, PIDs). cgroup v2 is now standard.
- **Memory:** RSS vs virtual memory, page cache, swap, OOM killer (`oom_score`), `free -m` ("available" matters, not "free").
- **Load average:** Number of runnable + uninterruptible tasks averaged over 1/5/15 min — compare to CPU count.

**Troubleshooting toolkit**
| Tool | Use |
|---|---|
| `top` / `htop` | CPU, memory, load, per-process usage |
| `ps aux`, `pstree` | Process list and hierarchy |
| `strace -p PID` | System calls a process makes (hangs, file/permission errors) |
| `lsof -p PID`, `lsof -i :8080` | Open files/sockets, who holds a port |
| `ss -tulpn` / `netstat` | Listening ports, connections |
| `df -h`, `du -sh *`, `df -i` | Disk space, inode exhaustion |
| `journalctl`, `dmesg` | Service logs, kernel messages (OOM kills, disk errors) |
| `vmstat`, `iostat`, `sar` | CPU/memory/I/O over time |

### Interview Questions

**Q1. What's the difference between SIGTERM and SIGKILL, and why does it matter for containers?**
SIGTERM asks a process to exit gracefully — it can catch it, finish in-flight requests, close connections and flush data. SIGKILL is handled by the kernel; the process dies immediately without cleanup. Docker and Kubernetes send SIGTERM first, then SIGKILL after a grace period (`terminationGracePeriodSeconds`, default 30 s). If your app doesn't handle SIGTERM — or runs under a shell wrapper (`sh -c`) that doesn't forward signals as PID 1 — it gets killed abruptly, causing dropped requests during deploys. Fix: use exec form `ENTRYPOINT ["node","app.js"]`, handle SIGTERM in code, or use an init like `tini`.

**Q2. How do containers work at the Linux level?**
A container is a normal Linux process with: **namespaces** giving it its own view of PIDs, network stack, mount points, hostname and users; **cgroups** limiting CPU, memory and I/O; a **layered root filesystem** (overlayfs) from the image; plus security layers (capabilities dropped, seccomp syscall filters, AppArmor/SELinux). There's no separate kernel — all containers share the host kernel, which is why kernel vulnerabilities and privileged containers are dangerous and why VMs/microVMs (Firecracker, gVisor, Kata) are used for stronger isolation.

**Q3. A server is slow. How do you troubleshoot it?**
Use a structured method (Brendan Gregg's **USE**: Utilization, Saturation, Errors for each resource). `uptime` (load vs CPU count) → `top` (CPU hogs, `wa` = I/O wait, `st` = steal on VMs) → `free -m` (memory pressure, swap) → `vmstat 1` (run queue, swapping) → `iostat -x 1` (disk utilization/latency) → `df -h`/`df -i` (disk full) → `ss -s` (connection counts) → `dmesg -T` (OOM kills, hardware errors) → app logs. Then dig into the specific process with `strace`, `lsof`, or profilers.

**Q4. `df` shows the disk is full but `du` doesn't find large files. Why?**
A process still holds a deleted file open (commonly a log file deleted while the app is writing to it) — space isn't released until the file descriptor closes. Find it with `lsof | grep deleted` (or `lsof +L1`), then restart the process or truncate via `/proc/PID/fd/N`. Other causes: inode exhaustion (`df -i`) from millions of small files, or files hidden under a mount point.

**Q5. What is a zombie process and an orphan process?**
A **zombie** has finished executing but its parent hasn't called `wait()` to read its exit status — it holds only a process table entry. Many zombies indicate a buggy parent; kill/fix the parent. An **orphan** is a running process whose parent died — it gets re-parented to PID 1 (or a subreaper). In containers your app may be PID 1 and must reap children, otherwise zombies accumulate — another reason to use `tini` or `--init`.

**Q6. What happens when a Linux system runs out of memory?**
The kernel first reclaims page cache and swaps (if configured). If still insufficient, the **OOM killer** picks a process with the highest `oom_score` (mostly based on memory usage, adjustable via `oom_score_adj`) and kills it, logging to `dmesg`. In containers, exceeding the cgroup memory limit triggers an OOM kill of a process in that cgroup — in Kubernetes this shows as `OOMKilled` with exit code 137 (128 + 9).

**Q7. How do you find what's listening on port 8080 and which process owns it?**
`ss -tulpn | grep 8080` or `lsof -i :8080`. If "address already in use" occurs after restart, check for lingering processes or sockets in `TIME_WAIT` (use `SO_REUSEADDR` in the app).

---

## 2. Networking

**What it is:** How data moves between machines — the core of distributed systems, load balancers, Kubernetes networking and most production incidents ("it's always DNS").

**Key concepts**
- **OSI / TCP-IP model:** L2 (Ethernet/MAC), L3 (IP, routing), L4 (TCP/UDP, ports), L7 (HTTP, gRPC, DNS).
- **TCP:** 3-way handshake (SYN, SYN-ACK, ACK), reliable ordered delivery, flow/congestion control, connection teardown, `TIME_WAIT`. **UDP:** connectionless, no guarantees — DNS, video, QUIC.
- **DNS:** Recursive resolution (stub resolver → recursive resolver → root → TLD → authoritative), record types (A, AAAA, CNAME, MX, TXT, NS, SRV), TTLs and caching, `/etc/resolv.conf`, `ndots` in Kubernetes.
- **HTTP versions:** HTTP/1.1 (keep-alive, head-of-line blocking per connection), **HTTP/2** (multiplexing many streams over one TCP connection, header compression, used by gRPC), **HTTP/3** (over **QUIC/UDP** — no TCP head-of-line blocking, faster handshakes, connection migration).
- **TLS:** Handshake (TLS 1.3 = 1 round trip), certificates, chain of trust, CA, SNI, mTLS (both sides present certs), cert expiry.
- **Load balancing:** L4 (TCP connections, fast, no content awareness) vs L7 (HTTP routing by path/header, TLS termination, retries). Algorithms: round robin, least connections, consistent hashing, weighted.
- **NAT:** Translate private IPs to public (SNAT for outbound, DNAT/port forwarding for inbound).
- **CIDR & subnetting:** `10.0.0.0/16` = 65,536 addresses; `/24` = 256. Private ranges: 10/8, 172.16/12, 192.168/16.
- **Proxies:** **Forward proxy** acts for clients (corporate egress, filtering); **reverse proxy** acts for servers (NGINX, Envoy — TLS termination, routing, caching).

**Debugging tools:** `curl -v` (HTTP/TLS details), `dig +trace` / `nslookup` (DNS), `tcpdump` / Wireshark (packets), `traceroute`/`mtr` (path), `nc` (port reachability), `openssl s_client` (certificates), `ip addr`/`ip route`.

### Interview Questions

**Q1. What happens when you type `https://example.com` into a browser?**
URL parsing → browser/OS DNS cache → recursive resolver → root → `.com` TLD → authoritative server returns IP (possibly via CDN CNAME) → TCP 3-way handshake (or QUIC for HTTP/3) → TLS handshake (ClientHello with SNI, server certificate validated against trusted CAs, key exchange) → HTTP request with headers/cookies → load balancer/CDN → reverse proxy → app server → database/cache → response (status, headers, body, compressed) → browser parses HTML, fetches CSS/JS/images (more requests, multiplexed on HTTP/2), renders. Senior candidates mention caching layers, CDN, keep-alive and where latency is spent.

**Q2. HTTP/1.1 vs HTTP/2 vs HTTP/3?**
HTTP/1.1: text protocol, one request at a time per connection (browsers open ~6 connections), head-of-line blocking. HTTP/2: binary framing, **multiplexing** many concurrent streams over one TCP connection, header compression (HPACK), server push (deprecated) — but a lost TCP packet still blocks all streams. HTTP/3: runs on **QUIC over UDP** — streams are independent at the transport level (no TCP HOL blocking), 0-1 RTT handshakes with built-in TLS 1.3, and connection migration when the client changes networks (mobile).

**Q3. A service can't reach another service. How do you debug?**
Layer by layer: DNS (`dig service.name` — does it resolve, to the right IP?), network reachability (`nc -zv host port`, `traceroute`), firewalls/security groups/NetworkPolicies, is the target listening (`ss -tlnp` on target), TLS (`openssl s_client -connect host:443` — cert valid, SNI, mTLS?), HTTP layer (`curl -v` — status codes, redirects, auth), application logs. In Kubernetes also check Service selectors match pod labels and Endpoints/EndpointSlices are populated.

**Q4. Explain the TLS handshake and what mTLS adds.**
TLS 1.3: client sends ClientHello (supported ciphers, key share, SNI) → server replies with ServerHello (chosen cipher, its key share), its certificate and a signature proving private-key ownership → both derive session keys (ECDHE — forward secrecy) → encrypted application data. Client verifies the certificate chain to a trusted CA and the hostname. **mTLS** adds client certificate authentication: the server also requests and verifies the client's certificate — used for service-to-service zero-trust (service meshes, SPIFFE identities).

**Q5. L4 vs L7 load balancing?**
L4 balances TCP/UDP connections by IP/port — very fast, protocol-agnostic, preserves end-to-end TLS, but can't route by URL or understand requests; long-lived connections (gRPC/HTTP2) can cause imbalance because all requests on one connection go to the same backend. L7 understands HTTP: path/host/header routing, TLS termination, retries, per-request balancing, auth, rate limiting, observability — at higher CPU cost. Use L7 for HTTP microservices and gRPC (per-request balancing), L4 for raw TCP, extreme throughput, or TLS passthrough.

**Q6. Why is "it's always DNS" a common saying? Give DNS failure examples.**
DNS sits on the path of nearly every connection and is cached at many layers. Examples: stale records after failover due to long TTLs or apps caching DNS forever (JVM), CoreDNS overloaded in Kubernetes (the `ndots:5` setting causes several search-domain lookups per external name), misconfigured search domains, split-horizon DNS returning private vs public IPs, expired domains, DNSSEC misconfiguration, and UDP packet drops in conntrack races.

**Q7. How many usable IPs are in a /26 and how would you split a /16 for 3 AZs?**
/26 = 64 addresses (cloud providers reserve some — AWS reserves 5 per subnet → 59 usable). For a /16 across 3 AZs with public, private, and data tiers: e.g. private app subnets /19 each (8,192 IPs — large for Kubernetes pod IPs), public /22 each, data /22 each, leaving room for future expansion. Always plan non-overlapping CIDRs across VPCs and on-prem.

---

## 3. Git

**What it is:** Distributed version control system — every clone has full history. The source of truth for code, infrastructure (IaC) and deployments (GitOps).

**Key concepts**
- **Objects:** blobs (content), trees (directories), commits (snapshot + parents + metadata), tags. Branches are just movable pointers to commits; `HEAD` points to the current branch.
- **Three areas:** working directory → staging (index) → repository.
- **Merge vs rebase:** Merge preserves history with a merge commit; rebase rewrites commits on top of another base for linear history.
- **Cherry-pick:** Apply a specific commit onto another branch (hotfix backports).
- **Bisect:** Binary search through history to find the commit that introduced a bug (`git bisect start/good/bad`, `git bisect run test.sh`).
- **Reflog:** Local log of where `HEAD` and branches pointed — recover "lost" commits after bad reset/rebase.
- **Conflict resolution:** Understand both sides, `git mergetool`, `rerere` to reuse resolutions.
- **Signed commits:** GPG/SSH/Sigstore (gitsign) signatures prove authorship; branch protection can require them.
- **Hooks:** Client-side (`pre-commit`, `commit-msg`, `pre-push`) and server-side (`pre-receive`) scripts.
- Others: `stash`, `reset` (`--soft/--mixed/--hard`), `revert` (new inverse commit — safe on shared branches), `submodules`, `worktree`, `sparse-checkout`, shallow clones (`--depth 1` in CI), Git LFS for large files.

### Interview Questions

**Q1. Merge vs rebase — when do you use each?**
Rebase your **local/feature** branch onto main to keep a clean linear history and resolve conflicts commit by commit; never rebase **shared/public** branches (rewrites history others depend on, forcing force-pushes). Merge (or squash-merge) when integrating into main — it preserves context. Many teams use "rebase locally, squash-merge PRs" for a readable main history with one commit per PR. Use `git push --force-with-lease` (not `--force`) if you must update a rebased remote branch.

**Q2. `git reset` vs `git revert`?**
`reset` moves the branch pointer backward — `--soft` keeps changes staged, `--mixed` (default) keeps them unstaged, `--hard` discards them. It rewrites history, so use only on local commits. `revert` creates a new commit that undoes a previous one — safe on shared branches like main because history isn't rewritten. In production, a bad merge to main is reverted, not reset.

**Q3. You accidentally ran `git reset --hard` and lost commits. How do you recover?**
`git reflog` shows previous positions of HEAD (e.g. `HEAD@{2}`); `git reset --hard HEAD@{2}` or `git branch recovered <sha>` restores them. Reflog is local and entries expire (default 90 days), but unreachable objects persist until garbage collection. Uncommitted changes that were never staged are not recoverable this way.

**Q4. How do you find which commit introduced a bug?**
`git bisect`: mark a known bad commit and a known good commit, Git checks out the midpoint, you test and mark good/bad, repeating — O(log n) steps. Automate with `git bisect run ./test.sh` where the script exits 0 for good and non-zero for bad. Works best with small, buildable commits.

**Q5. How do you apply a hotfix to a release branch and main?**
Fix on main (or a hotfix branch) via PR, then `git cherry-pick <sha>` onto the release branch (or vice versa), resolving conflicts and tagging a patch release. Use `-x` to record the original SHA. In trunk-based development, fix forward on main and cherry-pick to the maintained release branch only if needed.

**Q6. How do you handle large repositories or files in CI efficiently?**
Shallow clones (`fetch-depth: 1`), partial clone (`--filter=blob:none`), sparse-checkout for monorepos, Git LFS for binaries (or better, keep binaries in artifact storage), caching the `.git` directory on self-hosted runners, and avoiding committing generated files.

---

## 4. Scripting (Bash, Python, jq/yq)

**What it is:** Automation glue — CI steps, deployment scripts, ops tasks, data munging.

**Key concepts**
- **Bash:** Pipes (`|`), redirection (`>`, `>>`, `2>&1`), exit codes (`$?`, 0 = success), `&&`/`||`, variables and quoting (`"$var"` always), arrays, functions, loops, `[[ ]]` tests, `trap` for cleanup, `xargs`, `find`, `grep`, `sed`, `awk`, `cut`, `sort | uniq -c`.
- **Strict mode:** `set -euo pipefail` — `-e` exit on error, `-u` error on unset variables, `-o pipefail` a pipeline fails if any command fails. Plus `IFS=$'\n\t'` sometimes.
- **ShellCheck** for linting scripts.
- **Python for automation:** When logic grows (APIs, JSON, error handling, tests) — `boto3`, `requests`, `subprocess`, `argparse`/`typer`, `pathlib`; virtual environments; packaging with `uv`/Poetry.
- **jq:** Query/transform JSON (`jq '.items[] | select(.status=="Running") | .name'`). **yq:** Same for YAML (Kubernetes manifests, CI configs).

### Interview Questions

**Q1. What does `set -euo pipefail` do and what are its pitfalls?**
`-e` exits on any command failure, `-u` fails on unset variables (catches typos like `rm -rf "$DIR/"*` with empty DIR), `-o pipefail` makes a pipeline return the first failing command's status (otherwise `curl ... | jq` succeeds even if curl failed). Pitfalls: `-e` is ignored inside conditions (`if`, `&&`, `||`), commands expected to fail (like `grep` with no match) need `|| true`, and functions called in conditions lose `-e` behavior. Combine with `trap 'cleanup' EXIT` for cleanup.

**Q2. When would you switch from Bash to Python?**
When a script exceeds ~50–100 lines, needs complex data structures, JSON/API handling, robust error handling, retries, unit tests, or cross-platform support. Bash excels at orchestrating CLI tools and simple glue; Python is more maintainable for logic. Mention ShellCheck for Bash and type hints/tests for Python.

**Q3. Write a one-liner to find the top 10 IPs in an access log.**
`awk '{print $1}' access.log | sort | uniq -c | sort -rn | head -10`. Explain each stage: extract field 1, sort so duplicates are adjacent, count, sort numerically descending, take top 10.

**Q4. How do you use jq to extract data from kubectl output?**
`kubectl get pods -o json | jq -r '.items[] | select(.status.phase != "Running") | "\(.metadata.namespace)/\(.metadata.name) \(.status.phase)"'` — lists non-running pods. `-r` outputs raw strings. `yq` can patch YAML: `yq -i '.spec.replicas = 3' deploy.yaml`.

**Q5. How do you make scripts safe and idempotent?**
Strict mode, quote all variables, check preconditions, use `mkdir -p`, `ln -sf`, check-before-create (`if ! kubectl get ns x; then ...`), use declarative tools (`kubectl apply`, Terraform) instead of imperative create commands, `trap` for cleanup of temp files (`mktemp`), dry-run flags, logging with timestamps, and non-zero exit codes on failure so CI notices.

---

## 5. Data Formats (YAML, JSON, HCL)

**What it is:** Configuration languages used everywhere in DevOps — Kubernetes and CI configs (YAML), APIs and policies (JSON), Terraform (HCL).

**Key concepts**
- **YAML:** Indentation-based (spaces only), maps, lists, multi-document (`---`), anchors/aliases (`&default`, `*default`, `<<: *default`), multi-line strings (`|` keeps newlines, `>` folds).
- **YAML pitfalls:** The "Norway problem" (`NO` → boolean false in YAML 1.1), unquoted `on`/`yes`/`off`, octal-looking numbers (`0755`), version numbers (`1.10` → float `1.1`), tabs, implicit typing, duplicate keys silently overwritten — quote strings when in doubt.
- **JSON:** Strict, no comments, used by APIs, IAM policies, `package.json`; JSON is (mostly) valid YAML.
- **HCL (HashiCorp Configuration Language):** Terraform/Packer/Vault — blocks, attributes, expressions, functions, `for_each`.
- **Templating vs overlays:** Helm (Go templates generating YAML) vs Kustomize (patching YAML) vs CUE/Jsonnet/KCL (typed config languages).
- **Validation:** JSON Schema, `kubeconform`, `yamllint`.

### Interview Questions

**Q1. Name common YAML pitfalls you've seen break deployments.**
Unquoted values interpreted as booleans (`enabled: yes`, country code `NO`), version strings parsed as floats (`version: 1.10` → 1.1), leading-zero numbers as octal, tabs vs spaces, wrong indentation silently moving a key to a different parent (e.g. `env` under the wrong container), duplicate keys overwriting earlier ones, and Helm template whitespace issues. Mitigate with quoting, `yamllint`, schema validation (`kubeconform`) in CI.

**Q2. YAML anchors — what are they and when would you use them?**
`&name` defines an anchor, `*name` references it, `<<: *name` merges a map — avoids repetition (e.g. shared job defaults in GitLab CI or docker-compose). Downsides: reduces readability, not supported by every tool (GitHub Actions added limited support only recently), and merge keys are a YAML 1.1 feature. Reusable workflows/templates are often clearer.

**Q3. JSON vs YAML for configuration?**
JSON: strict, unambiguous, great for machine-generated data and APIs, no comments. YAML: human-friendly, comments, multi-line strings, anchors — but whitespace-sensitive and has implicit typing gotchas. Many tools accept both; Kubernetes stores everything as JSON internally.

**Q4. Helm templates vs typed config languages (CUE/Jsonnet/KCL)?**
Helm uses text templating — easy to start, but generating YAML with string templates is error-prone (indentation, quoting) and has no type checking. Typed languages model config as data with schemas, validation and composition, catching errors before deploy; trade-off is a learning curve and smaller ecosystem.

**Q5. How do you validate configuration files in CI?**
Lint (`yamllint`, `hadolint` for Dockerfiles, `actionlint` for GitHub workflows), schema validation (`kubeconform` against Kubernetes schemas including CRDs, JSON Schema), render templates (`helm template`, `kustomize build`) and validate the output, policy checks (Conftest/Kyverno CLI), and `terraform validate`/`fmt -check` for HCL.

---

## 6. Cloud Basics (AWS / GCP / Azure)

**What it is:** The shared building blocks across cloud providers. Senior DevOps interviews expect you to map concepts across clouds and reason about them (AWS details are in the separate [AWS guide](AWS_and_AI_on_AWS_Interview_Guide.md)).

**Concept mapping**
| Concept | AWS | GCP | Azure |
|---|---|---|---|
| Virtual machines | EC2 | Compute Engine | Virtual Machines |
| Managed Kubernetes | EKS | GKE | AKS |
| Serverless functions | Lambda | Cloud Run functions | Azure Functions |
| Serverless containers | Fargate / App Runner | Cloud Run | Container Apps |
| Object storage | S3 | Cloud Storage | Blob Storage |
| Virtual network | VPC | VPC (global) | VNet |
| IAM | IAM roles/policies | IAM + service accounts | Entra ID + RBAC |
| Managed SQL | RDS/Aurora | Cloud SQL/AlloyDB/Spanner | Azure SQL/Flexible Server |
| NoSQL | DynamoDB | Firestore/Bigtable | Cosmos DB |
| Messaging | SQS/SNS/EventBridge | Pub/Sub | Service Bus/Event Grid |
| Monitoring | CloudWatch | Cloud Monitoring/Logging | Azure Monitor |
| IaC native | CloudFormation/CDK | Infrastructure Manager | ARM/Bicep |

**Core concepts:** Regions and availability zones, shared responsibility, IAM and least privilege, networking (private/public subnets, NAT, load balancers), storage types (block/file/object), pricing models (on-demand, commitments, spot/preemptible), managed vs self-managed trade-offs.

### Interview Questions

**Q1. Region vs availability zone — how do they affect design?**
A region is a geographic area with multiple isolated AZs (separate power, cooling, networking, low-latency links). Deploying across multiple AZs protects against data-center failures with minimal latency cost — standard for production. Multi-region protects against regional outages and serves global users but adds data replication and consistency complexity, plus cost.

**Q2. How does IAM differ conceptually between AWS, GCP and Azure?**
AWS: identity-based and resource-based JSON policies attached to users/roles; roles assumed via STS. GCP: role bindings (principal + role) attached on the resource hierarchy (org → folder → project → resource) and inherited downward; service accounts for workloads. Azure: Entra ID identities with RBAC role assignments at scopes (management group → subscription → resource group → resource); managed identities for workloads. All support workload identity federation (OIDC) to avoid static keys.

**Q3. When would you choose managed services over self-hosting (e.g. RDS vs Postgres on VMs)?**
Managed when you want backups, patching, HA failover, monitoring and scaling handled for you, so the team focuses on product — usually worth the premium. Self-host when you need unsupported extensions/versions, specific tuning, cost at very large scale, portability, or there's an on-prem requirement. Senior answer: quantify operational cost (people time, on-call) not just the bill.

**Q4. How do you avoid cloud vendor lock-in — and should you?**
Full avoidance is expensive (lowest-common-denominator architecture). Pragmatic approach: use portable layers where cheap (containers, Kubernetes, Terraform, OpenTelemetry, PostgreSQL-compatible databases, S3-compatible APIs), isolate provider-specific code behind interfaces, and accept lock-in for high-value managed services where the productivity gain outweighs migration risk. Multi-cloud should be driven by business needs (regulation, acquisitions, negotiation), not ideology.

**Q5. What are spot/preemptible instances and how do you use them safely?**
Spare capacity at up to ~60–90% discount that can be reclaimed with short notice (2 minutes on AWS, ~30 seconds on GCP). Use for stateless, fault-tolerant workloads: CI runners, batch jobs, Kubernetes worker nodes for stateless services. Diversify instance types/zones, handle termination notices (drain nodes), checkpoint long jobs, and keep an on-demand baseline for critical capacity.

---

# STAGE 1 — PLAN & COLLABORATE

---

## 7. DevOps Culture, Agile & Planning Tools

**What it is:** DevOps is primarily a **culture and set of practices** that break down silos between development and operations to deliver software faster and more reliably. Tools support it, but don't create it.

**Key concepts**
- **CALMS framework:** **C**ulture (shared ownership, blameless), **A**utomation (CI/CD, IaC), **L**ean (small batches, reduce waste/WIP), **M**easurement (DORA metrics, SLOs), **S**haring (knowledge, tooling, postmortems).
- **"You build it, you run it":** Teams own services in production, including on-call — creates feedback loops that improve quality.
- **The Three Ways (The Phoenix Project / DevOps Handbook):** Flow (left to right, small batches), Feedback (fast, from right to left), Continual learning and experimentation.
- **Agile methods:** **Scrum** (sprints, sprint planning, daily standup, review, retrospective; roles: Product Owner, Scrum Master, Developers), **Kanban** (continuous flow, WIP limits, visualize work, measure lead/cycle time), Scrumban.
- **Planning tools:** **Jira** (enterprise standard — epics, stories, workflows, boards), **Linear** (fast, opinionated, popular with startups), **GitHub Issues/Projects** (close to code), **Azure Boards**; documentation in **Confluence** or **Notion**.
- **Shift left / shift right:** Move testing and security earlier; also learn from production (observability, experiments).

### Interview Questions

**Q1. What does DevOps mean to you?**
A culture and set of practices where the people who build software also share responsibility for running it, supported by automation and fast feedback, so we deliver small changes frequently, safely and measurably. It's measured by outcomes — DORA metrics (deployment frequency, lead time, change failure rate, time to restore) and reliability (SLOs) — not by having Jenkins or Kubernetes. "DevOps engineer" as a separate silo can recreate the problem DevOps aims to solve.

**Q2. Explain the CALMS framework.**
Culture: shared ownership, psychological safety, blameless postmortems. Automation: CI/CD, IaC, testing, so humans don't do repetitive error-prone work. Lean: small batch sizes, limit work in progress, eliminate handoffs/waits. Measurement: metrics that drive improvement (DORA, SLOs, flow metrics). Sharing: knowledge, tools and responsibility across teams. It's used to assess DevOps maturity.

**Q3. Scrum vs Kanban — which suits a platform/ops team?**
Scrum suits product development with plannable work in fixed sprints. Platform/SRE teams receive interrupt-driven work (incidents, requests), so **Kanban** (continuous flow, WIP limits, explicit classes of service like "expedite" for incidents) often fits better, or a hybrid with a rotating "interrupt handler" protecting the rest of the team's focus.

**Q4. How do you introduce DevOps practices in a team with separate Dev and Ops silos?**
Start with a shared pain point and measure it (e.g. lead time, failed deploys). Form a cross-functional pilot team with shared goals and on-call, automate the deployment pipeline end-to-end, adopt blameless postmortems, make production metrics visible to developers, and expand based on demonstrated results. Get leadership support for incentives — if Dev is rewarded for speed and Ops for stability, conflict persists.

**Q5. What does "you build it, you run it" require to work well?**
Good observability and runbooks so developers can operate services, a sustainable on-call (rotation size, compensation, alert quality), a platform that abstracts infrastructure complexity, error budgets to balance features vs reliability, and time allocated to fix operational issues found on-call. Without these it leads to burnout.

---

## 8. ADRs, RFCs & Design Reviews

**What they are:**
- **ADR (Architecture Decision Record):** A short document capturing one significant decision — **context, decision, consequences (trade-offs), status** — stored in the repo (`docs/adr/0007-use-argo-cd.md`). Immutable; superseded by new ADRs.
- **RFC / Design document:** A longer proposal for discussion before building something significant — problem, goals/non-goals, proposed design, alternatives considered, risks, rollout plan, open questions.
- **Design review:** Structured peer review of the proposal (async comments + meeting), often with checklists (security, reliability, cost, operability).

### Interview Questions

**Q1. What's in a good ADR and why write them?**
Title, status (proposed/accepted/superseded), context (forces and constraints), the decision, consequences (positive and negative), and alternatives considered. They preserve the "why" behind decisions for future engineers, prevent re-litigating settled questions, make onboarding faster, and make it explicit when a decision should be revisited because its context changed.

**Q2. How do you run an effective design review as a senior engineer?**
Circulate the doc in advance, state goals/non-goals clearly, focus discussion on the riskiest decisions and alternatives, use a checklist (failure modes, scaling, security, data, observability, cost, migration/rollback), invite the right reviewers (security, SRE, dependent teams), timebox, record decisions and action items, and distinguish blocking concerns from suggestions.

**Q3. ADR vs RFC — when do you use which?**
RFC/design doc for exploring and getting feedback on a significant proposal before work starts (more detail, alternatives, discussion). ADR to record the final decision concisely after the discussion — one RFC can produce several ADRs. Small decisions may only need an ADR.

**Q4. How do you handle disagreement in a design review?**
Return to requirements and data: clarify the goals and constraints, list trade-offs explicitly, prototype or benchmark when facts are disputed, consider reversibility (two-way doors can be decided quickly; one-way doors need more care), and if consensus fails, escalate to a designated decision-maker — then "disagree and commit" and record it in the ADR.

**Q5. What should a design doc include for operational readiness?**
SLOs and how they're measured, dashboards/alerts, failure modes and mitigations, capacity estimates, deployment and rollback strategy, data migration plan, security/threat model, dependencies, runbooks, on-call ownership, and cost estimate.

---

## 9. Branching Strategies

**What they are:** Conventions for how teams use branches to integrate changes.
- **Trunk-based development (TBD):** Everyone integrates small changes into `main` (trunk) at least daily via short-lived branches (hours to 1–2 days). Incomplete work hidden behind **feature flags**. Releases cut from trunk (or deployed continuously). Recommended by DORA research for high performance.
- **GitFlow:** Long-lived `develop` and `main`, plus `feature/*`, `release/*`, `hotfix/*` branches. Suits versioned software with scheduled releases and multiple supported versions; heavy for continuous delivery.
- **GitHub Flow:** Branch from main → PR → review/CI → merge → deploy. Simple, works well with CD.
- **GitLab Flow / release branches:** GitHub Flow plus environment or release branches.

### Interview Questions

**Q1. Trunk-based development vs GitFlow — which do you recommend?**
For web services with continuous delivery: **trunk-based** — small, frequent merges reduce merge conflicts and integration risk, keep main always releasable, and enable fast feedback; feature flags decouple deploy from release. GitFlow fits products that ship versioned releases (mobile apps, on-prem software, libraries) with multiple maintained versions — but long-lived branches cause "merge hell" and slow feedback.

**Q2. How do you ship incomplete features with trunk-based development?**
Feature flags (off by default in production), branch by abstraction (introduce an interface, build the new implementation behind it, switch over), dark launches (deploy code paths that run without exposing results), and keep changes small and backward compatible. Remove flags after rollout to avoid flag debt.

**Q3. What prerequisites does trunk-based development need?**
Fast, reliable CI (tests run on every PR and on main in minutes), good automated test coverage, code review that is quick (small PRs), feature flags, monitoring and fast rollback, and a culture of not breaking main (and fixing it immediately if broken).

**Q4. How do you manage releases for a mobile app using trunk-based development?**
Develop on trunk; cut a **release branch** at a release point (since app stores need fixed builds and review time), only cherry-pick critical fixes into it, tag versions; continue development on trunk. Use remote feature flags to enable features after the app is published, and staged rollouts in app stores.

**Q5. What's the problem with long-lived feature branches?**
They diverge from main, causing large painful merges, late discovery of integration bugs, delayed feedback, big risky PRs that are hard to review, and blocked refactoring. DORA research associates them with lower delivery performance.

---

# STAGE 2 — CODE & VERSION CONTROL

---

## 10. Git Platforms & Pull Request Practices

**What it is:** Hosted Git platforms providing collaboration on top of Git — **GitHub** (largest; Actions, Copilot, Advanced Security), **GitLab** (all-in-one DevSecOps platform, strong self-managed option), **Bitbucket** (Atlassian/Jira integration), **Azure Repos** (Azure DevOps).

**Key practices**
- **Pull/Merge Requests:** Proposed changes with review, discussion, CI checks.
- **Code review:** Small PRs (<400 lines), clear descriptions, focus on correctness/design/security/readability; automation handles style.
- **Branch protection / rulesets:** Required reviews, required status checks, no force-push, signed commits, linear history, restrict who can push, merge queues.
- **CODEOWNERS:** File mapping paths to owning teams/people who are auto-requested and can be required to approve (`/infra/ @org/platform-team`).
- **Merge strategies:** Merge commit, squash, rebase.
- **Merge queue:** Serializes merges and tests each PR against the latest main + queued PRs — prevents "green PR, broken main".
- **Conventional Commits:** `feat:`, `fix:`, `chore:`, `docs:`, `refactor:`, `BREAKING CHANGE:` — enables automated changelogs and version bumps (semantic-release, release-please).
- **Semantic Versioning (SemVer):** `MAJOR.MINOR.PATCH` — major for breaking changes, minor for backward-compatible features, patch for fixes; pre-release tags (`1.2.0-rc.1`).

### Interview Questions

**Q1. What makes a good pull request and code review process?**
Small, focused PRs with a clear description (what/why/how tested, screenshots, risk, rollback), linked ticket, passing CI. Reviewers respond quickly (within hours), focus on correctness, design, security, tests and maintainability; nits labeled as optional; automation (linters, formatters, AI reviewers) handles style. Track review latency — slow reviews hurt lead time more than slow CI.

**Q2. What branch protection rules would you set on main?**
Require PRs with at least 1–2 approvals (CODEOWNERS for critical paths), required status checks (build, tests, security scans) passing on up-to-date branch or via merge queue, dismiss stale approvals on new commits, block force pushes and deletion, require signed commits (optional), require conversation resolution, restrict bypass to admins with audit. Use repository rulesets to apply org-wide.

**Q3. What is CODEOWNERS and how do you use it at scale?**
A file mapping path patterns to owners who are automatically requested for review and, with branch protection, must approve changes to those paths. At scale: map directories to teams (not individuals), keep it current, use it for sensitive areas (IaC, CI workflows, security configs), and avoid overly broad patterns that create review bottlenecks.

**Q4. Explain semantic versioning and how to automate releases.**
MAJOR.MINOR.PATCH: increment major for incompatible API changes, minor for backward-compatible features, patch for backward-compatible bug fixes. Automate with Conventional Commits + tools like **semantic-release** or **release-please**: they parse commit messages since the last tag, determine the version bump, generate a changelog, tag and publish. Note: for deployed services (not libraries), version by commit SHA/build number is often enough.

**Q5. Squash vs merge commit vs rebase merge?**
Squash: one commit per PR on main — clean history, easy revert of a whole feature, loses granular commits. Merge commit: preserves all commits and branch topology — full history but noisy. Rebase merge: linear history keeping individual commits — requires disciplined commits. Most teams with trunk-based development use squash.

**Q6. What is a merge queue and why use it?**
When many PRs merge into a busy main, each PR's CI ran against an older main; combining them can break main ("semantic conflicts"). A merge queue (GitHub merge queue, GitLab merge trains, Mergify, Bors) creates temporary branches combining main + queued PRs, runs CI on them, and merges only when green — keeping main always green with high throughput.

---

## 11. Monorepo vs Polyrepo (Bazel, Nx, Turborepo, Pants)

**What it is:**
- **Monorepo:** Many projects/services in one repository (Google, Meta, Uber). **Polyrepo:** One repository per service/library.
- **Monorepo build tools:** **Bazel** (Google; hermetic, reproducible builds, remote caching/execution, multi-language, steep learning curve), **Nx** (JS/TS-focused, also others; affected-project detection, computation caching, generators), **Turborepo** (JS/TS, simple task pipelines and remote caching, by Vercel), **Pants** (Python/Go/Java, fine-grained dependency inference).
- **Key concepts:** Dependency graph, **affected/changed detection** (build/test only what changed), local and **remote caching**, remote execution, hermeticity, sparse checkout, ownership via CODEOWNERS.

### Interview Questions

**Q1. Monorepo vs polyrepo — trade-offs?**
Monorepo pros: atomic cross-project changes, easy code sharing and refactoring, single source of dependency versions, unified tooling/CI standards, discoverability. Cons: needs specialized build tooling to scale CI, access control is coarser, large clone sizes, risk of tight coupling. Polyrepo pros: clear ownership boundaries, independent pipelines and permissions, simpler per-repo tooling. Cons: dependency/version drift, painful cross-repo changes, duplicated CI config, harder discovery. Choose based on team size, coupling and tooling investment.

**Q2. How do you keep CI fast in a large monorepo?**
Build/test only **affected** projects using the dependency graph (Nx affected, Bazel query, Turborepo filters), **remote caching** so unchanged targets reuse prior outputs across machines, remote execution for parallelism, sparse/shallow checkout, splitting test suites, path-based workflow triggers, and keeping the dependency graph clean (avoid everything depending on a giant "common" library).

**Q3. What does "hermetic build" mean in Bazel?**
The build's outputs depend only on declared inputs (sources, pinned toolchains, dependencies) — not on the machine environment, network, or undeclared files. This makes builds reproducible and cacheable: identical inputs → identical outputs → safe to reuse cached results remotely. Requires declaring all dependencies explicitly in BUILD files.

**Q4. When would you choose Nx/Turborepo over Bazel?**
For JavaScript/TypeScript-heavy repos where developer experience and quick adoption matter — Nx/Turborepo integrate with existing npm/pnpm tooling, have low setup cost and good caching. Bazel suits very large, polyglot repos needing strict hermeticity and remote execution, justifying its learning curve and maintenance effort (often a dedicated build team).

**Q5. How do you deploy services independently from a monorepo?**
Each service has its own build target, image and versioning (e.g. tags with service name + SHA); CI determines affected services and only builds/deploys those; separate deployment manifests per service (GitOps directory per service/environment); CODEOWNERS per service directory; contract tests to catch breaking shared-library changes.

---

## 12. Local Quality Gates (pre-commit, Linters, Formatters, EditorConfig)

**What it is:** Checks that run on the developer machine before code reaches CI — fastest feedback loop.
- **pre-commit framework / Husky + lint-staged / lefthook:** Run hooks on staged files at commit time (formatting, linting, secret scanning, YAML validation, Terraform fmt).
- **Linters:** ESLint, Pylint/**Ruff**, golangci-lint, ShellCheck, hadolint (Dockerfile), tflint, actionlint, markdownlint, yamllint.
- **Formatters:** Prettier, Black/Ruff format, gofmt, `terraform fmt`.
- **EditorConfig:** `.editorconfig` standardizes indentation, line endings, charset across editors/IDEs.
- Principle: **same checks in CI** — local hooks are convenience, CI is enforcement.

### Interview Questions

**Q1. Why use pre-commit hooks if CI runs the same checks?**
Faster feedback (seconds vs minutes), fewer failed CI runs and noisy fix-up commits, and some checks like secret scanning are best done *before* the secret ever enters Git history (once pushed, it must be rotated). CI still enforces because hooks can be skipped (`--no-verify`) or not installed.

**Q2. How do you prevent formatting debates in code review?**
Adopt an opinionated formatter (Prettier, Black/Ruff, gofmt) with shared config in the repo, run it automatically on save/commit and enforce in CI (`--check`). Style becomes non-negotiable and automated; reviews focus on logic.

**Q3. What linters would you add for a DevOps repository?**
ShellCheck (Bash), hadolint (Dockerfile best practices), yamllint, actionlint (GitHub Actions workflows), tflint + `terraform fmt/validate`, Checkov/Trivy (IaC security), kubeconform (Kubernetes manifests), markdownlint, gitleaks (secrets). Run them via pre-commit and CI.

**Q4. How do you keep hooks fast enough that developers don't disable them?**
Run only on staged/changed files, keep heavy checks (full test suite, deep scans) in CI, cache tool environments (pre-commit does), parallelize, and use fast tools (Ruff instead of multiple Python linters). Measure hook time — anything over a few seconds gets skipped.

**Q5. What is EditorConfig and why does it matter?**
A simple file defining whitespace conventions (indent style/size, final newline, trailing whitespace, line endings) understood by most editors. Prevents diffs caused by different editor settings — especially line-ending (CRLF vs LF) issues across Windows/macOS/Linux, which can break shell scripts in containers.

---

## 13. Developer Environments (Dev Containers, Codespaces, Gitpod, Tilt/Skaffold)

**What it is:** Reproducible, consistent development environments that eliminate "works on my machine".
- **Dev Containers:** `devcontainer.json` spec defining a containerized dev environment (image, tools, extensions, ports, post-create commands) — supported by VS Code, JetBrains, and CLIs.
- **GitHub Codespaces:** Cloud-hosted dev containers on GitHub; prebuilds for fast startup.
- **Gitpod / Coder / DevPod / Daytona:** Cloud or self-hosted remote development environments (Gitpod's open-source "Classic" was succeeded by Gitpod Flex/now "Ona"; Coder is popular self-hosted).
- **Local Kubernetes development:** **Tilt** (watches files, rebuilds/live-updates containers in a local or remote cluster, UI dashboard), **Skaffold** (build/push/deploy loop, also usable in CI), Telepresence/mirrord (run a service locally connected to a remote cluster), DevSpace, docker-compose for simpler stacks.

### Interview Questions

**Q1. How do you reduce onboarding time for new developers?**
Dev Containers/Codespaces with all tools pinned, a single `make dev` or `tilt up` command to start the stack, seeded test data, documentation in the repo, prebuilds to make environments start in seconds, and a developer portal (Backstage) with templates. Goal: first commit on day one.

**Q2. Dev Containers vs Codespaces — what's the difference?**
Dev Containers is the open specification (`devcontainer.json`) describing the environment; it runs locally with Docker. Codespaces is GitHub's managed cloud service that runs those dev containers on remote VMs — no local setup, powerful machines, consistent environment, but costs money and needs connectivity.

**Q3. How do you develop efficiently against microservices running on Kubernetes?**
Tilt or Skaffold for an inner loop that rebuilds and live-syncs code into pods on file save; or Telepresence/mirrord to run the service under development locally while it talks to the rest of the services in a shared dev cluster (intercepting traffic); ephemeral namespaces per developer (vCluster); mock or stub heavy dependencies.

**Q4. Tilt vs Skaffold?**
Both automate build → deploy → watch loops for Kubernetes. Tilt focuses on an interactive developer experience (Tiltfile in Starlark, web UI showing resource status/logs, live update). Skaffold (Google) is config-driven YAML, CLI-centric, and doubles as a CI/CD build-and-deploy tool with profiles. Choice depends on preferred workflow.

**Q5. What are the security considerations of cloud dev environments?**
They often hold source code and credentials: use SSO, short-lived credentials (OIDC), restrict network egress, prevent secrets in images/dotfiles, auto-stop idle environments, audit logs, approved base images, and policies on which repos can spawn environments.

---

# STAGE 3 — BUILD & PACKAGE

---

## 14. Build Tools (Maven/Gradle, npm/pnpm, Go modules, Poetry/uv, Make)

**What they are:** Language-specific tools that resolve dependencies, compile, test and package code.
| Ecosystem | Tools | Key files |
|---|---|---|
| Java/Kotlin | **Maven** (XML, convention, lifecycle phases), **Gradle** (Groovy/Kotlin DSL, incremental builds, build cache) | `pom.xml`, `build.gradle(.kts)` |
| JavaScript/TS | **npm**, **pnpm** (content-addressable store, strict, fast), Yarn | `package.json`, lockfiles |
| Go | **Go modules** | `go.mod`, `go.sum` |
| Python | pip + venv, **Poetry**, **uv** (very fast Rust-based installer/resolver/project manager) | `pyproject.toml`, `uv.lock`/`poetry.lock` |
| Generic | **Make** (task runner & dependency-based builds), Just, Task | `Makefile` |

**Key concepts:** Lockfiles for reproducibility, semantic version ranges, transitive dependencies, private registries/mirrors, build caching, reproducible builds.

### Interview Questions

**Q1. Why are lockfiles important and should they be committed?**
Lockfiles pin exact versions (and hashes) of all direct and transitive dependencies, ensuring every developer and CI build installs the same versions — reproducibility and supply-chain safety (prevents surprise upgrades or malicious new versions). Commit them for applications; for libraries, commit them for development reproducibility while declaring compatible ranges for consumers. In CI use strict install commands (`npm ci`, `pnpm install --frozen-lockfile`, `uv sync --locked`).

**Q2. Maven vs Gradle?**
Maven: declarative XML, strict lifecycle (validate → compile → test → package → verify → install → deploy), convention over configuration, very predictable. Gradle: programmable DSL, incremental builds, build cache and daemon → faster for large multi-module builds, more flexible but can become complex. Android and many large projects use Gradle; many enterprises keep Maven for simplicity.

**Q3. How do you speed up dependency installation in CI?**
Cache dependency directories keyed on the lockfile hash (`~/.m2`, `~/.gradle/caches`, pnpm store, `~/.cache/uv`), use a local mirror/proxy (Artifactory/Nexus/CodeArtifact) close to runners, faster tools (pnpm, uv), Docker layer caching that copies lockfiles before source code, and avoid reinstalling across jobs by sharing artifacts.

**Q4. Why use Make (or a task runner) in a modern repo?**
A single, discoverable entry point for common tasks (`make build`, `make test`, `make lint`, `make deploy`) that works the same locally and in CI, hiding tool-specific commands. Make also handles file-based dependencies. Alternatives like `just` or Task offer cleaner syntax without Make's quirks (tabs, phony targets).

**Q5. What is dependency confusion and how do you prevent it?**
An attacker publishes a public package with the same name as your internal private package and a higher version; misconfigured resolvers pull the public one. Prevent: use scoped packages (`@company/pkg`), configure registries so internal names resolve only from the private registry, use a single proxy registry with priority rules, pin versions with hashes in lockfiles, and reserve your names on public registries.

---

## 15. Docker Fundamentals

**What it is:** The tool that popularized containers — packaging an application with its dependencies into an **image** that runs identically anywhere as a **container**.

**Key concepts**
- **Image vs container:** Image = immutable, layered template (read-only). Container = running instance with a thin writable layer.
- **Layers & caching:** Each Dockerfile instruction creates a layer; unchanged layers are cached. Order instructions from least to most frequently changing (copy lockfile → install deps → copy source).
- **Multi-stage builds:** Build in a full SDK image, copy only the artifacts into a minimal runtime image → smaller, more secure images.
- **CMD vs ENTRYPOINT:** ENTRYPOINT = the executable; CMD = default arguments (overridable). Use **exec form** (`["app","--flag"]`) so the process is PID 1 and receives signals.
- **`.dockerignore`:** Exclude `.git`, `node_modules`, secrets, build outputs from the build context — faster builds, no leaked files.
- **Networking:** bridge (default, NAT), host (share host network), none, overlay (multi-host), user-defined networks with DNS by container name.
- **Volumes:** Named volumes (managed by Docker), bind mounts (host paths), tmpfs. Containers are ephemeral — persist data in volumes.
- **docker compose:** Define multi-container apps for local dev/testing.
- Other: `ARG` vs `ENV`, `HEALTHCHECK`, `USER`, `EXPOSE` (documentation only), image digests (`@sha256:`).

```dockerfile
# syntax=docker/dockerfile:1
FROM node:22-alpine AS build
WORKDIR /app
COPY package.json pnpm-lock.yaml ./
RUN corepack enable && pnpm install --frozen-lockfile
COPY . .
RUN pnpm build && pnpm prune --prod

FROM gcr.io/distroless/nodejs22-debian12
WORKDIR /app
COPY --from=build /app/dist ./dist
COPY --from=build /app/node_modules ./node_modules
USER nonroot
CMD ["dist/server.js"]
```

### Interview Questions

**Q1. How do you optimize a Dockerfile for size and build speed?**
Multi-stage builds (ship only runtime artifacts), minimal base images (distroless, alpine, slim, Chainguard), order layers for cache efficiency (dependency manifests before source), combine and clean up in the same `RUN` (`apt-get install ... && rm -rf /var/lib/apt/lists/*`), `.dockerignore`, BuildKit cache mounts (`RUN --mount=type=cache,target=/root/.cache`), avoid installing dev dependencies in the final image, and pin base image versions/digests.

**Q2. CMD vs ENTRYPOINT? Shell form vs exec form?**
ENTRYPOINT defines the command that always runs; CMD supplies default arguments (or the default command if no ENTRYPOINT) and is overridden by `docker run image <args>`. Exec form (`["node","server.js"]`) runs the process directly as PID 1, so it receives SIGTERM. Shell form (`node server.js`) runs under `/bin/sh -c`, which may not forward signals → slow, ungraceful shutdowns.

**Q3. Why should containers not run as root?**
If an attacker escapes the application or the container (kernel/runtime vulnerabilities), root inside the container may map to root on the host (without user namespaces). Running as a non-root user with a read-only root filesystem, dropped capabilities and no privilege escalation limits blast radius. Kubernetes Pod Security Standards (restricted) enforce `runAsNonRoot`.

**Q4. What is the difference between a volume and a bind mount?**
A named volume is managed by Docker (stored under Docker's directory), portable, and the preferred way to persist data. A bind mount maps a specific host path into the container — useful for local development (live code reload) but couples the container to host layout and can expose host files. In Kubernetes, the equivalents are PersistentVolumes and hostPath (avoid hostPath in production).

**Q5. Your image is 1.5 GB. Walk through how you'd reduce it.**
Inspect with `docker history` / `dive` to find large layers. Switch to multi-stage build and a slim/distroless base; remove build tools, caches and docs from the final stage; install production dependencies only; check for large files accidentally copied (missing `.dockerignore`); avoid separate `RUN rm` (deleted files still exist in earlier layers); for Python/Node, prune dev deps; for Go/Rust, a static binary on `scratch`/distroless can be <20 MB.

**Q6. How does container networking work between containers in docker compose?**
Compose creates a user-defined bridge network per project; containers are attached and can reach each other by service name via Docker's embedded DNS. Ports are only published to the host if mapped (`ports: "8080:80"`). `depends_on` controls start order but not readiness — use health checks with `condition: service_healthy`.

---

## 16. Image Hardening & Build Tooling (BuildKit/buildx, Podman, Buildah, Kaniko, Buildpacks)

**What it is:** Producing **secure, minimal, reproducible** images, and the tools that build them — especially in CI and Kubernetes where Docker daemons may not be available.

**Hardening**
- **Minimal base images:** **Distroless** (Google — only runtime and app, no shell/package manager), **Chainguard/Wolfi** images (minimal, frequently rebuilt, near-zero CVEs, SBOMs included), Alpine (musl — small, sometimes compatibility issues), `-slim` variants, `scratch` for static binaries.
- Non-root `USER`, read-only filesystem, no secrets in layers (use BuildKit `--mount=type=secret`), pin by digest, rebuild regularly to pick up patches, scan and sign.

**Build tools**
| Tool | What it does |
|---|---|
| **BuildKit** | Modern Docker build engine — parallel stages, cache mounts, secret mounts, SSH forwarding, remote cache export (`--cache-to/--cache-from`) |
| **buildx** | Docker CLI plugin for BuildKit — **multi-architecture builds** (`--platform linux/amd64,linux/arm64`), remote builders |
| **Podman** | Daemonless, rootless Docker-compatible container engine (Red Hat); supports pods |
| **Buildah** | Daemonless image builder (scriptable, OCI), used with Podman |
| **Kaniko** | Builds images inside a container/Kubernetes without a Docker daemon (note: Google archived the original project in 2025; community forks continue) |
| **Cloud Native Buildpacks** (Paketo, `pack`) | Build images from source without a Dockerfile — detects language, applies best practices, enables "rebasing" base layers for patches |
| Others | `ko` (Go apps), Jib (Java), BuildKit-based `buildctl`, Depot (hosted fast builders) |

### Interview Questions

**Q1. Why use distroless or Chainguard images, and what are the downsides?**
They contain only what the app needs — no shell, package manager or unnecessary libraries — dramatically reducing CVE count and attack surface (an attacker can't easily run tools). Downsides: harder debugging (no shell — use `kubectl debug` with ephemeral containers or `:debug` image variants), must build artifacts in a separate stage, and some apps need extra system libraries.

**Q2. How do you build multi-architecture images and why?**
`docker buildx build --platform linux/amd64,linux/arm64 -t repo/app:tag --push .` builds for each platform (natively on matching runners or via QEMU emulation, which is slower) and pushes a **manifest list** so clients pull the right architecture automatically. Needed for ARM (AWS Graviton, Apple Silicon dev machines) to gain price-performance. Faster approach: native ARM runners per platform, then merge manifests.

**Q3. How do you build container images in Kubernetes-based CI without Docker-in-Docker?**
Docker-in-Docker requires privileged containers — a security risk. Alternatives: BuildKit in rootless mode (buildkitd as a service), Buildah/Podman rootless, Kaniko (or its forks), or remote builders (e.g. a dedicated BuildKit cluster, Depot, cloud build services). Use registry-based caching to keep builds fast.

**Q4. How do you pass secrets to a build without leaking them into the image?**
BuildKit secret mounts: `RUN --mount=type=secret,id=npmrc,target=/root/.npmrc npm ci` with `docker build --secret id=npmrc,src=.npmrc` — the secret is available only during that step and never written to a layer. Never use `ARG`/`ENV` for secrets (they persist in image history). Use SSH forwarding (`--mount=type=ssh`) for private Git dependencies.

**Q5. What are Cloud Native Buildpacks and when would you use them?**
They transform source code into OCI images without Dockerfiles: detect the language, install the runtime and dependencies, configure the start command, and produce reproducible images with SBOMs. Platform teams like them because they enforce standard, secure images across hundreds of services and can **rebase** application layers onto a patched base image without rebuilding — fast CVE remediation at scale. Downside: less control for unusual builds.

**Q6. Docker vs Podman?**
Podman is daemonless (each container is a child process — no root daemon as a single point of failure/attack), runs rootless by default, has a Docker-compatible CLI (`alias docker=podman`), supports pods (Kubernetes-like groups) and can generate Kubernetes YAML/systemd units. Docker has the larger ecosystem (Docker Desktop, Compose, Hub). Both produce OCI images.

---

## 17. Container Standards & Runtimes (OCI, containerd, CRI-O)

**What it is:** The standards and lower-level software that actually run containers.
- **OCI (Open Container Initiative):** Standards for the **image format** (manifest, layers, config), **runtime spec** (how to run a container from a filesystem bundle) and **distribution spec** (registry API). Enables any compliant tool to build/push/run images interchangeably. OCI artifacts store non-image content (Helm charts, SBOMs, signatures) in registries.
- **Low-level runtimes:** **runc** (reference implementation — creates namespaces/cgroups), crun (faster, C), sandboxed runtimes **gVisor** (user-space kernel), **Kata Containers** (lightweight VMs), Firecracker (microVMs).
- **High-level runtimes:** **containerd** (CNCF graduated — image pull, storage, lifecycle; used by Docker and most Kubernetes distros), **CRI-O** (lightweight runtime built specifically for Kubernetes, used by OpenShift).
- **CRI (Container Runtime Interface):** The Kubernetes API between kubelet and runtime. **Dockershim** was removed in Kubernetes 1.24 — Kubernetes talks to containerd/CRI-O directly.
- Tools: `crictl` (debug CRI runtimes on nodes), `nerdctl` (Docker-like CLI for containerd), `ctr`.

### Interview Questions

**Q1. Kubernetes removed Docker support — does that mean Docker images don't work?**
No. Kubernetes removed **dockershim** (the adapter for the Docker Engine as runtime) in v1.24. Images built with Docker are standard **OCI images** and run fine on containerd or CRI-O. Only workflows depending on the Docker daemon on nodes (e.g. mounting `/var/run/docker.sock` for builds) broke and needed alternatives.

**Q2. What is the container runtime stack from kubelet to process?**
kubelet → (CRI gRPC) → containerd/CRI-O → pulls/unpacks image into a snapshot (overlayfs) → creates an OCI bundle → calls a low-level runtime via a shim (runc) → runc sets up namespaces, cgroups, seccomp, capabilities, mounts and executes the container process, then exits; the shim keeps the container's stdio and exit status.

**Q3. When would you use gVisor or Kata Containers?**
For running **untrusted or multi-tenant code** (customer workloads, CI jobs from external contributors, AI-agent-generated code) where shared-kernel isolation isn't enough. gVisor intercepts syscalls in a user-space kernel (lower overhead than VMs, some compatibility/performance costs); Kata runs each pod in a lightweight VM with its own kernel (stronger isolation, more overhead). Kubernetes selects them via `RuntimeClass`.

**Q4. What are OCI artifacts and why do they matter?**
The OCI distribution spec lets registries store arbitrary content types, not just images: Helm charts, WASM modules, SBOMs, signatures and attestations (cosign stores them alongside the image), policies, even ML models. One registry with one auth model and replication for all deployable artifacts, and supply-chain metadata attached to images via the referrers API.

**Q5. How do you debug containers directly on a Kubernetes node?**
`crictl ps`, `crictl logs`, `crictl inspect`, `crictl images` (CRI-level), `nerdctl` for containerd namespaces, check kubelet logs (`journalctl -u kubelet`), container runtime logs, and `nsenter` into a container's namespaces for network debugging. Prefer `kubectl debug node/<name>` to get a privileged debug pod.

---

## 18. Artifact Registries, Versioning & "Build Once, Deploy Many"

**What it is:** Storage and distribution of build outputs (container images, packages, binaries, Helm charts), plus conventions for versioning and promotion.

**Registries**
| Tool | Notes |
|---|---|
| **JFrog Artifactory** | Universal (Maven, npm, PyPI, Docker, Helm, Go...), remote proxies, virtual repos, enterprise features |
| **Sonatype Nexus** | Universal repository manager, popular self-hosted |
| **GitHub Packages / GHCR** | Close to source; GitHub permissions |
| **Docker Hub** | Public images; pull rate limits for anonymous/free users |
| **Amazon ECR / Google Artifact Registry / Azure ACR** | Cloud-native, IAM integrated, scanning, replication |
| **Harbor** | CNCF open-source registry with scanning, signing, replication, RBAC |
| Others | GitLab registry, Cloudsmith, AWS CodeArtifact (packages) |

**Versioning & promotion**
- **Immutable tags:** Never overwrite a tag once pushed; enforce in registry settings. Avoid `:latest` in deployments.
- **Tag with git SHA** (`app:3f9c2ab`) plus optionally SemVer (`app:1.4.2`) and build metadata; deploy by **digest** (`@sha256:...`) for absolute immutability.
- **Build once, deploy many:** Build the artifact once in CI and **promote the same artifact** through dev → staging → prod (by retagging or moving between repositories), injecting environment-specific config at deploy time. Never rebuild per environment.
- **Retention/cleanup policies**, pull-through caches/proxies, replication across regions.

### Interview Questions

**Q1. Why is "build once, deploy many" important?**
Rebuilding per environment can produce a different artifact (dependency updates, base image changes, build nondeterminism), so what you tested in staging isn't what runs in prod. Building once and promoting the identical artifact (same digest) guarantees that tested = deployed, speeds deployments and provides traceability. Configuration differs per environment through env vars, ConfigMaps or secrets — not in the image.

**Q2. Why shouldn't you use the `latest` tag in production?**
It's mutable — it points to different images over time, so you can't tell what's running, rollbacks are ambiguous, nodes may run different versions depending on when they pulled, and `imagePullPolicy` behavior gets confusing. Use immutable SHA/SemVer tags or digests, enforced by registry tag immutability and admission policies.

**Q3. How do you handle Docker Hub rate limits in CI and Kubernetes?**
Use a **pull-through cache/proxy** (ECR pull-through cache, Artifactory/Nexus/Harbor proxy), mirror base images into your private registry, authenticate pulls, and configure container runtimes with registry mirrors. This also improves availability and supply-chain control.

**Q4. How do you promote artifacts between environments securely?**
Separate repositories or registries per stage (e.g. `dev`, `prod`), CI pushes to dev; promotion copies the same digest to prod repo only after tests/approvals (via `crane copy`/registry APIs or Artifactory promotion), with signatures and attestations proving which pipeline built it. Production clusters only pull from the prod registry and verify signatures at admission.

**Q5. What's the difference between a tag and a digest?**
A tag is a human-readable, mutable pointer (`app:1.2.0`). A digest is the SHA-256 content hash of the image manifest (`app@sha256:ab12...`) — immutable and verifiable. Deploying by digest guarantees exactly which image runs; tools like Argo CD Image Updater or Kustomize can pin digests.

**Q6. How do you manage registry storage costs and hygiene?**
Lifecycle/retention rules (keep last N images per repo, delete untagged images after X days, keep images currently deployed), separate long-term retention for release artifacts, garbage collection, and avoid giant images. Make sure cleanup doesn't delete images still referenced by running workloads or needed for rollback.

---

# STAGE 4 — TEST & QUALITY

---

## 19. Test Pyramid & Testing Tools (Testcontainers, Pact, Playwright, Cypress)

**What it is:** A model for balancing test types: many fast, cheap **unit tests** at the bottom, fewer **integration** and **contract** tests in the middle, and few slow, brittle **end-to-end (E2E)** tests at the top. (Variants: "testing trophy" emphasizes integration tests.)

**Tools**
- **Unit:** JUnit, pytest, Jest/Vitest, Go test.
- **Integration — Testcontainers:** Spins up real dependencies (Postgres, Kafka, Redis, LocalStack) in throwaway Docker containers from test code — realistic tests without shared environments.
- **Contract testing — Pact:** **Consumer-driven contracts**: the consumer's tests generate a contract (expected requests/responses); the provider verifies it in its own CI; a **Pact Broker** stores contracts and answers "can I deploy?" (`can-i-deploy`). Alternative: schema-based contracts (OpenAPI, protobuf/buf breaking-change checks).
- **E2E/UI:** **Playwright** (Microsoft — multi-browser, auto-wait, parallel, tracing; now the most popular), **Cypress** (developer-friendly, in-browser runner), Selenium (legacy).
- **API testing:** Postman/Newman, REST Assured, Karate, Hurl.

### Interview Questions

**Q1. Explain the test pyramid and why it matters.**
Unit tests are fast, isolated and pinpoint failures — so have many. Integration tests verify components work together with real dependencies — fewer, slower. E2E tests verify full user journeys but are slow, flaky and expensive to maintain — keep only for critical paths. An inverted pyramid ("ice cream cone") of mostly manual/E2E tests yields slow, unreliable pipelines. The goal is fast feedback with high confidence.

**Q2. What is consumer-driven contract testing and why use it in microservices?**
Instead of spinning up all services for E2E tests, each consumer defines what it needs from a provider (a contract) through its tests. The provider runs those contracts against its implementation in its own pipeline. If a provider change breaks any consumer's contract, the provider's build fails before deployment. Pact Broker's `can-i-deploy` checks compatibility of versions across environments. Benefits: independent deployability, fast feedback, fewer brittle E2E environments.

**Q3. Why use Testcontainers instead of mocks or a shared test database?**
Mocks can hide real behavior (SQL dialect differences, transactions, serialization, Kafka semantics). Shared test databases cause flaky tests from shared state and contention. Testcontainers gives each test run real, isolated, disposable dependencies with the same versions as production, started from code. Trade-off: needs Docker on CI runners and adds startup time (mitigate with reusable containers and singletons).

**Q4. How do you reduce flaky E2E tests?**
Use frameworks with auto-waiting (Playwright) instead of sleeps, stable selectors (`data-testid`, roles), isolated test data per test, avoid dependencies on external systems (stub third parties), reset state, run in parallel safely, retries only as a temporary measure with flaky-test tracking/quarantine, and capture traces/videos for debugging. Keep E2E count small.

**Q5. How do you test a service that depends on many other services?**
Unit test business logic in isolation; integration test with Testcontainers for owned dependencies (DB, queue); contract tests (Pact/OpenAPI) for service-to-service APIs; service virtualization (WireMock, Mountebank) for third parties; a small number of E2E smoke tests in a staging environment; and progressive delivery with production monitoring as the final safety net.

---

## 20. Performance & Load Testing (k6, JMeter, Gatling, Locust)

**What it is:** Testing how a system behaves under load to verify capacity, latency SLOs and find bottlenecks before users do.

**Test types:** **Load** (expected traffic), **stress** (beyond capacity to find breaking point), **spike** (sudden bursts), **soak/endurance** (long duration — memory leaks, resource exhaustion), **scalability** (does throughput scale with resources), **breakpoint**.

**Tools**
| Tool | Notes |
|---|---|
| **k6** (Grafana) | Scripts in JavaScript/TypeScript, CLI-first, thresholds as pass/fail, great for CI, k6 Cloud/Grafana Cloud, browser module |
| **JMeter** | Long-standing Java GUI tool, many protocols, plugins; XML test plans |
| **Gatling** | Scala/Java/Kotlin DSL, efficient async engine, rich reports |
| **Locust** | Python-based, user behavior as code, distributed |
| Others | Artillery, Vegeta, wrk/hey (quick benchmarks), AWS Distributed Load Testing |

**Key metrics:** Throughput (RPS), latency percentiles (**p50, p95, p99**), error rate, saturation (CPU, memory, connection pools), concurrency.

### Interview Questions

**Q1. How would you plan a load test for a new service before launch?**
Define goals from business expectations (peak RPS, user journeys, SLO: p99 < 300 ms, error rate < 0.1%). Model realistic traffic (mix of endpoints, think time, data variety), use a production-like environment and data volume, warm up, ramp gradually, run load → stress → soak tests, monitor the whole system (app metrics, DB, caches, queues), identify the bottleneck, fix, and retest. Document capacity limits and autoscaling behavior.

**Q2. Why look at p99 instead of average latency?**
Averages hide tail latency: a mean of 100 ms could include 1% of requests at 5 s. With fan-out (one user request calls many services), the chance of hitting a slow tail grows quickly, so tail latency drives user experience. SLOs are typically defined on percentiles. Also beware **coordinated omission** — load generators that wait for slow responses under-report latency; k6/Gatling/wrk2 handle open-model load.

**Q3. How do you integrate performance tests into CI/CD?**
Short, smoke-level performance tests on every PR or nightly with **thresholds** (k6 `thresholds: { http_req_duration: ['p(95)<300'] }`) that fail the pipeline on regressions; full load/soak tests on a schedule or before major releases in a dedicated environment; track trends over time in dashboards.

**Q4. During a load test latency spikes at 2,000 RPS. How do you find the bottleneck?**
Correlate load with resource metrics (USE method) across tiers: app CPU/GC pauses/thread or event-loop saturation, connection pool exhaustion (DB pool waits), database CPU/locks/slow queries, cache hit ratio, network/LB limits, downstream dependency latency (tracing shows which span grows), autoscaling lag, and rate limits. Profile the hot path (CPU/async profilers). Then fix the narrowest constraint first.

**Q5. k6 vs JMeter?**
k6: code-based (JS/TS), lightweight, version-controlled tests, CI-friendly thresholds, modern developer workflow, efficient Go engine. JMeter: GUI-driven, broad protocol support (JDBC, JMS, FTP), large plugin ecosystem, but XML plans are hard to review and it's heavier. For new teams writing tests as code, k6 is usually preferred.

---

## 21. Code Quality (SonarQube, Coverage Gates, Mutation Testing)

**What it is:** Automated measurement of maintainability, reliability and test effectiveness.
- **SonarQube / SonarCloud (SonarQube Cloud):** Static analysis for bugs, code smells, vulnerabilities, security hotspots, duplication, complexity, and coverage; **Quality Gates** (pass/fail conditions — often on *new code*: coverage ≥ 80%, no new critical issues) that block merges.
- **Coverage:** Line/branch coverage via JaCoCo, Istanbul/nyc/c8, coverage.py; tools like Codecov to report on PRs.
- **Coverage gates:** Minimum thresholds or "no decrease" rules on PRs.
- **Mutation testing:** Tools (PIT for Java, Stryker for JS/.NET, mutmut for Python) introduce small changes (mutants) to code — if tests still pass, the mutant "survived" → tests are weak. Mutation score measures test effectiveness, not just execution.
- Other: complexity metrics, duplication detection, technical debt tracking.

### Interview Questions

**Q1. Is 100% code coverage a good goal?**
No. Coverage shows which code was executed, not whether behavior was verified — tests without meaningful assertions still produce coverage. Chasing 100% leads to low-value tests on trivial code. Better: a reasonable threshold on **new/changed code** (e.g. 80%), focusing on critical business logic, plus mutation testing on core modules to verify test quality.

**Q2. What is mutation testing and when is it worth it?**
It modifies code (e.g. `>` to `>=`, removing a call, changing a constant) and reruns tests; surviving mutants reveal untested behavior. It's computationally expensive, so use it on critical modules (payments, pricing, auth) or incrementally on changed code in CI rather than the whole codebase every time.

**Q3. How do you introduce quality gates on a legacy codebase without blocking all work?**
Apply gates to **new code only** ("clean as you code") — new changes must meet standards while legacy debt is tracked and reduced opportunistically (boy-scout rule). Set a baseline, prevent regressions (coverage must not decrease), and prioritize refactoring in hot spots (frequently changed + complex files).

**Q4. What should a SonarQube quality gate include?**
On new code: no new bugs/vulnerabilities above a severity, security hotspots reviewed, coverage ≥ threshold, duplication ≤ threshold, maintainability rating A. Keep it focused — too many conditions cause gate fatigue and bypassing.

**Q5. How do you measure and manage technical debt?**
Static analysis metrics (complexity, duplication, code smells), hotspot analysis combining churn and complexity (e.g. CodeScene), dependency age, incident/defect data per component, and developer surveys. Make debt visible in the backlog with business impact, allocate capacity (e.g. 15–20% per sprint), and tie debt reduction to outcomes (faster delivery, fewer incidents).

---

## 22. Test Strategy in CI (Parallelization, Flaky Tests, Test Impact Analysis, Preview Environments)

**What it is:** Making automated testing fast and reliable inside pipelines.
- **Parallel & sharded runs:** Split test suites across multiple runners (by timing data), run independent jobs concurrently (matrix builds).
- **Flaky tests:** Tests that pass and fail without code changes (timing, shared state, order dependence, network). Detect (rerun analysis), **quarantine**, fix, track flakiness rate.
- **Test impact analysis (TIA):** Run only tests affected by changed code (dependency graph, coverage mapping, Bazel/Nx affected, Launchable/predictive test selection).
- **Ephemeral / preview environments:** Spin up a full temporary environment per PR (namespace or vCluster on Kubernetes, Vercel/Netlify previews for frontends), run integration/E2E tests, share a URL for review, destroy on merge.
- **Fail fast ordering:** Lint → unit → build → integration → E2E.

### Interview Questions

**Q1. Your CI takes 45 minutes. How do you bring it under 10?**
Measure where time goes per stage. Then: cache dependencies and Docker layers, parallelize and shard tests by historical timing, run only affected tests/projects (TIA, monorepo tools), split slow E2E suites to run post-merge or nightly with only smoke E2E on PRs, use faster/larger runners, avoid redundant steps (build once and reuse artifacts across jobs), fail fast with quick checks first, and remove obsolete tests. Track CI duration as a metric.

**Q2. How do you deal with flaky tests?**
Detect automatically (same commit, different results; rerun-on-failure statistics), quarantine flaky tests into a non-blocking suite with an owner and deadline, fix root causes (shared state, time dependencies, async waits, random ports, test order), and track flakiness rate. Blind automatic retries hide real bugs and erode trust; developers start ignoring red builds.

**Q3. What are preview environments and what are their challenges?**
Per-PR ephemeral deployments of the changed service (and dependencies) so reviewers and E2E tests run against a realistic stack. Challenges: cost (auto-destroy, TTLs, scale to zero), startup time, data seeding, dependencies on shared services (use shared stable dependencies or virtualization), secrets management, and DNS/TLS per environment. Tools: Argo CD ApplicationSets PR generator, vCluster, Okteto, Vercel previews.

**Q4. How does test impact analysis work and what are the risks?**
It maps tests to the code they exercise (via the build dependency graph or coverage data) and runs only tests related to changed files. Risks: missing indirect dependencies (config files, reflection, shared data), so run the full suite periodically (nightly/on main) as a safety net.

**Q5. How do you order tests in a pipeline?**
Cheapest and most likely to fail first: formatting/lint/static checks (seconds) → unit tests → build/package → security scans (in parallel) → integration/contract tests → deploy to ephemeral/staging → E2E smoke → performance checks. Run independent stages in parallel and stop early on failure to save resources and give quick feedback.

---

# STAGE 5 — SECURITY (DevSecOps, "Shift Left")

---

## 23. SAST — Static Application Security Testing (Semgrep, CodeQL, SonarQube, Checkmarx)

**What it is:** Analyzing **your own source code** (without running it) for security vulnerabilities — SQL injection, XSS, command injection, hard-coded credentials, insecure crypto, path traversal, unsafe deserialization.

**Tools**
- **Semgrep:** Fast, pattern-based rules that look like code; easy custom rules; open-source engine + commercial platform (cross-file analysis, supply chain, secrets).
- **CodeQL (GitHub Advanced Security):** Treats code as a database queried with QL — deep data-flow/taint analysis; free for public repos.
- **SonarQube:** Quality + security rules, taint analysis in commercial editions.
- **Checkmarx, Veracode, Fortify, Snyk Code:** Enterprise SAST platforms.

**Key concepts:** Source → sink taint tracking, false positives vs false negatives, incremental scans on PRs, baseline/suppression management, SARIF output format to unify results in GitHub/GitLab security dashboards.

### Interview Questions

**Q1. What is SAST and what are its limitations?**
SAST scans source code for vulnerable patterns and tainted data flows from untrusted inputs (sources) to dangerous functions (sinks) early in development — cheap to fix. Limitations: false positives (noise causes alert fatigue), misses runtime/configuration issues and business-logic flaws, language/framework coverage gaps, and can't see how code is deployed. Combine with SCA, DAST, code review and threat modeling.

**Q2. How do you roll out SAST without overwhelming developers?**
Start in audit mode to baseline existing findings, then block PRs only on **new high-severity, high-confidence** findings; tune rules to the stack and suppress verified false positives with justification; show findings inline in PRs (SARIF annotations) with fix guidance; triage legacy findings into a prioritized backlog; track time-to-remediate. Use AI autofix suggestions (e.g. Copilot Autofix) to reduce effort.

**Q3. Semgrep vs CodeQL?**
Semgrep: fast, easy to write custom rules (pattern syntax resembles the target language), good for enforcing org-specific secure coding patterns; deeper inter-file analysis in the commercial tier. CodeQL: powerful semantic data-flow analysis via a query language, excellent for finding complex vulnerability classes, slower (needs a build/database extraction for compiled languages), tightly integrated with GitHub. Many teams use both.

**Q4. What is SARIF and why does it matter?**
Static Analysis Results Interchange Format — a standard JSON format for scanner results. Tools that output SARIF can be uploaded to GitHub code scanning or other platforms, giving a unified view, de-duplication, PR annotations and alert lifecycle regardless of which scanner produced them.

**Q5. How do you handle a SAST finding a developer believes is a false positive?**
Review the data flow together (is input actually user-controlled? is it sanitized?), if confirmed false positive, suppress it with an inline annotation and justification (auditable), and improve the rule/config (mark the sanitizer function) so it doesn't recur. If uncertain, treat it as real — fixing is often cheaper than debating.

---

## 24. SCA — Software Composition Analysis (Snyk, Dependabot, Renovate, OWASP Dependency-Check, Grype)

**What it is:** Identifying **third-party/open-source dependencies** (direct and transitive) and their known vulnerabilities (CVEs) and license risks. Since 70–90% of application code is often open source, this is one of the highest-value scans.

**Tools**
- **Snyk Open Source:** Vulnerability DB, fix PRs, reachability analysis, license compliance.
- **Dependabot (GitHub):** Security alerts and automated update PRs (version updates and security updates), grouped updates.
- **Renovate (Mend):** Highly configurable automated dependency updates across many ecosystems and platforms (also Docker images, Helm charts, Terraform providers, GitHub Actions); grouping, schedules, automerge rules.
- **OWASP Dependency-Check:** Open-source CVE scanner using the NVD.
- **Grype (Anchore) / Trivy / OSV-Scanner (Google):** Scan lockfiles, SBOMs, images.
- Data sources: NVD, GitHub Advisory Database, OSV.dev; prioritization with **CVSS**, **EPSS** (exploit probability), **CISA KEV** (known exploited).

### Interview Questions

**Q1. How do you prioritize hundreds of CVEs reported by SCA tools?**
Don't treat all CVEs equally. Prioritize by: known exploitation (CISA KEV), exploit probability (EPSS), severity (CVSS), **reachability** (is the vulnerable function actually called?), exposure (internet-facing vs internal), fix availability, and data sensitivity. Use VEX statements to document "not affected". Set SLAs by risk tier (e.g. critical exploitable internet-facing: 48 hours).

**Q2. Dependabot vs Renovate?**
Dependabot: built into GitHub, zero setup, security updates tied to GitHub advisories, simpler configuration. Renovate: far more configurable (grouping, scheduling, automerge for minor/patch with passing tests, monorepo support, regex managers for arbitrary files, supports GitLab/Bitbucket/Azure DevOps), and covers more ecosystems (Dockerfiles, Helm, Terraform, GitHub Actions digests). Large orgs often prefer Renovate.

**Q3. How do you keep dependencies up to date without drowning in PRs?**
Group related updates (e.g. all non-major updates weekly), automerge patch/minor updates when CI passes for well-tested repos, schedule updates (off-hours), separate security updates (immediate) from version updates, pin GitHub Actions and Docker images by digest with automated digest updates, and keep a good test suite so updates are low risk. Small frequent updates are safer than rare big-bang upgrades.

**Q4. What is transitive dependency risk and how do you fix vulnerable transitive dependencies?**
Your app depends on A, which depends on vulnerable B — you didn't choose B directly. Fix by upgrading A to a version using a patched B; if unavailable, use overrides/resolutions (`npm overrides`, `pnpm.overrides`, Maven `dependencyManagement`, Gradle constraints) to force a patched B version, then test. Track with SBOMs.

**Q5. How do you handle open-source license compliance?**
SCA tools detect licenses of all dependencies; define a policy (allowed: MIT, Apache-2.0, BSD; review: LGPL/MPL; deny: AGPL/GPL for distributed proprietary software, depending on legal advice), enforce in CI, generate SBOMs with license info, and handle exceptions through legal review.

---

## 25. Secret Scanning (Gitleaks, TruffleHog, GitHub Secret Scanning & Push Protection)

**What it is:** Detecting credentials (API keys, tokens, passwords, private keys) accidentally committed to source code, history, CI logs, images or chat.

**Tools**
- **Gitleaks:** Fast regex/entropy-based scanner for repos and history; pre-commit and CI.
- **TruffleHog:** Scans Git history, S3, Docker images, Slack, etc.; **verifies** secrets by testing them against provider APIs (reduces false positives).
- **GitHub secret scanning + push protection:** Partner patterns detected on push; **push protection blocks** pushes containing secrets; alerts can be sent to providers for auto-revocation. GitLab has equivalent secret detection and push protection.
- Others: detect-secrets (Yelp), ggshield (GitGuardian).

### Interview Questions

**Q1. A developer pushed an AWS access key to a public GitHub repo. What do you do?**
Treat it as compromised immediately (bots scrape public repos within minutes): **revoke/rotate the key first** (deactivate in IAM), then check CloudTrail for usage by that key and investigate any suspicious activity (new resources, IAM changes, crypto miners), remove from code and rewrite history if needed (BFG/`git filter-repo`) — but rewriting history doesn't make it safe; rotation does. Then do a blameless postmortem: add push protection, pre-commit scanning, and move to short-lived credentials (OIDC, IAM roles) so long-lived keys don't exist.

**Q2. Why isn't deleting the commit enough?**
The secret persists in Git history, forks, clones, CI caches, PR diffs and possibly in scrapers' databases. Once exposed, assume compromise — rotation is mandatory.

**Q3. How do you prevent secrets from entering repositories?**
Pre-commit hooks (gitleaks), server-side **push protection**, CI scanning as a backstop, `.gitignore` for env files, secret managers (Vault, AWS Secrets Manager) referenced at runtime instead of config files, OIDC/workload identity to eliminate static credentials, developer education, and scanning of CI logs/artifacts (masking).

**Q4. What makes TruffleHog's verification useful?**
Regex/entropy scanners produce many false positives (random-looking strings). TruffleHog calls the provider's API to check if a found credential is live/valid, letting teams prioritize verified, active secrets immediately while handling unverified matches later.

**Q5. How do you scan for secrets beyond source code?**
Container images (secrets baked into layers — Trivy, TruffleHog), CI/CD logs and artifacts, infrastructure state files (Terraform state contains secrets), wikis/Slack/Jira (TruffleHog, GitGuardian), S3 buckets, and Kubernetes manifests/Helm values. Secrets sprawl far beyond Git.

---

## 26. Container Image Scanning & IaC Scanning (Trivy, Grype, Docker Scout, Inspector, Checkov, KICS, Terrascan)

**What it is:**
- **Container image scanning:** Detect vulnerable OS packages and language libraries inside images, misconfigurations (running as root), embedded secrets.
  - **Trivy (Aqua)** ★ — all-in-one: images, filesystems, repos, Kubernetes clusters, IaC, SBOM, secrets, licenses; also the **Trivy Operator** for continuous in-cluster scanning. (tfsec's functionality has been merged into Trivy.)
  - **Grype** (Anchore, pairs with Syft SBOMs), **Docker Scout**, **Amazon Inspector** (ECR enhanced scanning), Snyk Container, Prisma Cloud, Wiz.
- **IaC scanning:** Detect insecure infrastructure definitions before deployment — public S3 buckets, open security groups (0.0.0.0/0 on SSH), unencrypted databases, privileged pods, missing resource limits.
  - **Checkov** (Prisma/Palo Alto — Terraform, CloudFormation, Kubernetes, Helm, Dockerfile, ARM/Bicep; graph-based policies), **KICS** (Checkmarx), **Terrascan** (Tenable — now archived), **tfsec** → Trivy, kube-score/kube-linter/Polaris for Kubernetes manifests.

### Interview Questions

**Q1. Where in the pipeline should image scanning happen?**
Multiple points: in CI after build (fail on fixable critical/high CVEs before pushing), in the registry continuously (new CVEs are disclosed after build — e.g. ECR enhanced scanning re-scans), at admission (only allow scanned/signed images), and at runtime (Trivy Operator/Inspector inventory of running images). Continuous scanning matters because an image clean today may be vulnerable tomorrow.

**Q2. The scanner reports 300 CVEs in your image. How do you reduce them?**
Most come from the base OS: switch to a minimal base (distroless/Chainguard/alpine/slim) — often eliminates the majority; rebuild regularly to pull patched packages; remove unnecessary packages/build tools via multi-stage builds; update language dependencies; filter by fixable and severity; use VEX/reachability to deprioritize unexploitable ones. Track CVE counts per image as a metric.

**Q3. What kinds of issues does IaC scanning catch, and why scan IaC rather than live resources?**
Misconfigurations like public storage, unencrypted volumes/databases, overly permissive IAM (`*:*`), security groups open to the internet, disabled logging, Kubernetes pods running privileged or without limits. Scanning IaC in PRs prevents issues **before** they're deployed (cheapest fix, developer sees it in context), whereas live scanning (CSPM) catches drift and click-ops changes — you need both.

**Q4. How do you handle accepted risks/exceptions in Checkov or Trivy?**
Inline skip annotations with a justification (`#checkov:skip=CKV_AWS_20:Public website bucket by design`) or central exception files with owner and expiry date, reviewed by security; avoid blanket disabling of rules. Report exceptions so they're auditable.

**Q5. Trivy vs Grype vs commercial tools?**
Trivy: broad all-in-one open-source scanner (vulns, IaC, secrets, licenses, Kubernetes), easy CI integration. Grype: focused vulnerability scanner with strong SBOM workflows (with Syft). Commercial (Snyk, Wiz, Prisma Cloud, Aqua): better prioritization (reachability, runtime context, attack paths), dashboards, policy management, support. Open-source is often enough for CI gating; commercial adds context at scale.

---

## 27. DAST — Dynamic Application Security Testing (OWASP ZAP, Burp Suite)

**What it is:** Testing a **running application** from the outside (black-box), like an attacker — sending malicious inputs to find vulnerabilities such as XSS, SQL injection, insecure headers, auth/session flaws, misconfigurations, exposed endpoints.

**Tools:** **OWASP ZAP** (now "ZAP by Checkmarx" — free, baseline/full/API scans, automation framework, Docker images for CI), **Burp Suite** (PortSwigger — industry standard for manual pentesting; Enterprise/DAST edition for automation), Nuclei (template-based scanning), StackHawk, Invicti. Related: **IAST** (instrumented app reports vulnerabilities during tests), **API security testing** (OpenAPI-driven fuzzing), penetration testing, bug bounties.

### Interview Questions

**Q1. SAST vs DAST — what's the difference?**
SAST analyzes source code without running it (white-box) — early, pinpoints code lines, but false positives and no runtime context. DAST attacks the running app (black-box) — finds runtime and configuration issues (headers, TLS, auth, server misconfigurations), fewer false positives, language-agnostic, but runs later, can't pinpoint code location and has limited coverage of deep paths. They're complementary.

**Q2. How do you integrate DAST into CI/CD?**
Run a **ZAP baseline scan** (passive, fast) against preview/staging environments on every deploy; run full active scans nightly or before releases (they're slower and can modify data, so never against production without care); use OpenAPI specs to drive API scans; authenticate the scanner (scripts/tokens) to reach protected pages; fail on high-risk findings; report results as SARIF/issues.

**Q3. What are the risks of running DAST against production?**
Active scans submit malicious payloads that can create junk data, trigger emails/payments, lock accounts, cause load spikes or outages, and trip WAF/IDS alerts. Run active scans against staging with production-like config; if production testing is required, use passive scans, coordinate with ops/security, allowlist scanner IPs and limit scope/rate.

**Q4. What is IAST and how does it differ from DAST?**
Interactive AST instruments the application runtime (agent) and observes code execution while tests (automated or DAST traffic) run — combining DAST's runtime accuracy with SAST's code-level visibility (pinpointing vulnerable lines) with low false positives. Downside: language-specific agents and performance overhead.

**Q5. How does DAST fit with penetration testing and bug bounties?**
DAST provides automated, continuous, broad coverage of common vulnerability classes. Penetration testers (manual, often with Burp) find business-logic flaws, chained exploits and authorization issues automation misses — typically annually or before major launches/compliance audits. Bug bounties provide continuous crowd-sourced testing for mature programs. Use all three in layers.

---

## 28. Software Supply Chain Security (SBOM, SLSA, Sigstore/cosign, in-toto, Provenance)

**What it is:** Protecting the integrity of everything that goes into and produces your software — dependencies, build systems, CI pipelines, artifacts — against attacks like SolarWinds (compromised build), Codecov (compromised CI script), xz-utils (malicious maintainer backdoor), and malicious packages/tj-actions-style compromised GitHub Actions.

**Key concepts**
- **SBOM (Software Bill of Materials):** Machine-readable inventory of all components and versions in an artifact. Formats: **SPDX** (Linux Foundation, ISO standard) and **CycloneDX** (OWASP). Generated with **Syft**, Trivy, cdxgen, Docker BuildKit (`--sbom`). Enables instant answers to "are we affected by log4shell?". **VEX** documents state exploitability.
- **SLSA (Supply-chain Levels for Software Artifacts, "salsa"):** Framework of increasing build-integrity levels (Build L1: provenance exists; L2: hosted build platform, signed provenance; L3: hardened, isolated builds with non-forgeable provenance).
- **Provenance:** Signed metadata describing **how, where and from what** an artifact was built (source repo/commit, builder, build parameters). GitHub artifact attestations, SLSA GitHub generator.
- **in-toto:** Framework for attestations about each supply-chain step (who did what), verifiable end-to-end; attestation format used by SLSA.
- **Sigstore:** Open-source signing infrastructure — **cosign** (sign/verify container images and artifacts), **Fulcio** (short-lived certificates from OIDC identity — **keyless signing**), **Rekor** (transparency log). Also gitsign for commits.
- **Verification at deploy:** Kubernetes admission (Kyverno `verifyImages`, Sigstore policy-controller, Ratify, Connaisseur) only admits images signed by your CI identity with valid attestations.
- **Other controls:** Pin dependencies and GitHub Actions by commit SHA, OpenSSF **Scorecard** for evaluating open-source project health, private mirrors, least-privilege CI tokens, two-person review, reproducible builds.

### Interview Questions

**Q1. What is an SBOM and why is it required more often now?**
A complete list of components (packages, versions, licenses, hashes) inside a piece of software. It enables rapid vulnerability impact analysis (search all SBOMs for a vulnerable library), license compliance, and transparency for customers. Regulations and procurement now demand it (US Executive Order 14028 for federal software suppliers, EU Cyber Resilience Act). Generate it at build time, attach it to the image as an attestation, and store it centrally (e.g. Dependency-Track) for querying.

**Q2. Explain keyless signing with Sigstore/cosign.**
Instead of managing long-lived private keys, the CI job authenticates with its OIDC identity (e.g. GitHub Actions workflow identity), **Fulcio** issues a short-lived signing certificate bound to that identity, cosign signs the image digest, and the signature + certificate are recorded in the **Rekor** transparency log. Verifiers check that the signature was made by the expected identity (`--certificate-identity` = your repo's workflow, `--certificate-oidc-issuer` = GitHub). No keys to leak or rotate.

**Q3. What are SLSA levels and how would you reach SLSA Build Level 3?**
L1: build process generates provenance. L2: builds run on a hosted build service that generates **signed** provenance. L3: the build platform is hardened — builds are isolated from each other, signing material is inaccessible to user-defined build steps, so provenance can't be forged. Achieve via GitHub Actions with the SLSA generator/reusable workflows or GitHub artifact attestations on isolated runners, ephemeral build environments, and verifying provenance at deploy time.

**Q4. How do you protect against compromised GitHub Actions or CI dependencies?**
Pin third-party actions to full **commit SHAs** (tags can be moved — as in the tj-actions/changed-files incident), use Renovate/Dependabot to update pinned SHAs, restrict allowed actions at org level, minimize `GITHUB_TOKEN` permissions (`permissions: read-all` default), avoid `pull_request_target` with untrusted code checkout, use OIDC instead of stored secrets, isolate sensitive jobs in protected environments, and audit workflow changes via CODEOWNERS.

**Q5. How do you enforce that only trusted images run in Kubernetes?**
Sign images in CI (cosign keyless), attach SBOM and provenance attestations, then use an admission controller (Kyverno `verifyImages`, Sigstore policy-controller) requiring a valid signature from your CI identity, optionally provenance from your trusted builder and a vulnerability scan attestation; restrict registries to your private registry; mutate tags to digests. Unsigned or tampered images are rejected.

**Q6. What lessons did the xz-utils backdoor teach about supply chain security?**
A trusted maintainer identity can be socially engineered over years, and malicious code can hide in release tarballs/build scripts rather than visible source. Mitigations: build from source repos rather than release tarballs, reproducible builds, monitoring of critical dependencies' health (Scorecard, maintainer changes), minimizing dependencies, defense in depth at runtime (least privilege, network policies), and SBOMs to locate affected systems quickly.

---

## 29. Runtime Security (Falco, Tetragon, GuardDuty, Microsoft Defender)

**What it is:** Detecting and responding to malicious or anomalous behavior in **running** workloads — the last line of defense when prevention fails.

**Tools**
- **Falco (CNCF graduated):** Runtime threat detection using kernel events (eBPF) with rules — e.g. "shell spawned in a container", "sensitive file read (`/etc/shadow`)", "unexpected outbound connection", "package manager run in container". Alerts via Falcosidekick to Slack/SIEM.
- **Tetragon (Cilium, eBPF):** Security observability **and enforcement** — can kill processes or block syscalls in-kernel based on policies (TracingPolicy).
- **Cloud services:** **AWS GuardDuty** (EKS/ECS/EC2 runtime monitoring, threat intel), **Microsoft Defender for Cloud/Containers**, GCP Security Command Center; commercial: Sysdig Secure, Aqua, Wiz Defend, CrowdStrike.
- Related hardening: seccomp profiles, AppArmor/SELinux, read-only root filesystem, dropping capabilities, network policies.

### Interview Questions

**Q1. Why do you need runtime security if you already scan images and code?**
Scanning finds *known* vulnerabilities before deploy; it can't detect zero-days, compromised credentials, misconfiguration exploitation, insider threats, or malicious behavior of trusted components. Runtime detection observes actual behavior (unexpected shells, privilege escalation, crypto mining, data exfiltration) and enables response. Defense in depth: prevent, detect, respond.

**Q2. How does eBPF enable runtime security?**
eBPF lets sandboxed programs run in the Linux kernel attached to hooks (syscalls, kprobes, tracepoints, LSM hooks, network events) safely and efficiently, without kernel modules. Security tools use it to observe process execution, file access and network connections with full container/Kubernetes context and low overhead; Tetragon can also enforce (block/kill) at the kernel level before the action completes.

**Q3. Give examples of Falco rules you'd enable.**
Terminal shell in container; write below `/etc` or binary directories; read of sensitive files; outbound connection to unusual ports/IPs (crypto mining pools); `kubectl exec` into production pods; container running as privileged; mounting host paths; launching package managers at runtime; modification of Kubernetes service account tokens. Tune for noise per environment.

**Q4. A runtime alert says a shell was spawned in a production pod. What's your response?**
Triage quickly: was it a legitimate `kubectl exec` (check audit logs for user identity and change ticket)? If unexpected: isolate the pod (NetworkPolicy deny-all, label to remove from Service), preserve evidence (don't delete immediately — capture logs, process tree, filesystem snapshot), revoke credentials the pod had access to (service account tokens, cloud IAM role sessions), investigate the entry vector (vulnerable app, exposed dashboard), then redeploy from a clean image and patch. Automate containment with Falco Talon or similar.

**Q5. Detection vs enforcement at runtime — trade-offs?**
Detection (alerting) is low-risk — no impact on workloads but relies on humans/automation to respond, attackers have time. Enforcement (blocking syscalls, killing processes) stops attacks immediately but false positives can break legitimate workloads and cause outages. Approach: start in detection mode, tune policies with real traffic, then enforce narrowly scoped, high-confidence rules on critical workloads.

---

## 30. Identity in Pipelines (OIDC Federation, Least-Privilege Deploy Roles)

**What it is:** How CI/CD systems authenticate to cloud providers and other systems — ideally with **short-lived, identity-based credentials** instead of long-lived secrets.

**Key concepts**
- **OIDC federation:** The CI platform (GitHub Actions, GitLab CI, CircleCI, Buildkite, Azure DevOps) issues a signed **OIDC token (JWT)** per job containing claims (repo, branch, environment, workflow). The cloud provider is configured to trust that issuer and exchanges the token for temporary credentials (AWS `AssumeRoleWithWebIdentity`, GCP Workload Identity Federation, Azure federated credentials).
- **Claim-based trust:** Restrict which repo/branch/environment can assume which role — e.g. `sub = repo:org/app:environment:production`.
- **Least-privilege deploy roles:** Separate roles per environment and per pipeline purpose (plan vs apply), scoped permissions, permission boundaries, no admin.
- **Protected environments:** Required reviewers, branch restrictions and wait timers before jobs can access production credentials.
- **Secrets in CI:** If static secrets are unavoidable — store in the CI secret store or fetch from Vault via OIDC, mask in logs, rotate, scope to environments.

### Interview Questions

**Q1. Why use OIDC instead of storing cloud access keys in CI secrets?**
Static keys are long-lived, can leak (logs, forks, compromised CI vendor or third-party action), are hard to rotate and are shared across jobs. With OIDC, each job gets a short-lived token (minutes to an hour) issued only to that specific workflow run; the cloud trust policy restricts which repo/branch/environment can assume which role. Nothing long-lived exists to steal, and every assumption is auditable in cloud logs.

**Q2. How do you configure GitHub Actions OIDC to AWS securely?**
Create the IAM OIDC identity provider for `token.actions.githubusercontent.com` with audience `sts.amazonaws.com`. Create a role with a trust policy conditioned on `token.actions.githubusercontent.com:aud = sts.amazonaws.com` and `sub` matching exactly the repo and environment/branch (`repo:org/app:environment:prod` — avoid wildcards like `repo:org/*`). In the workflow set `permissions: id-token: write, contents: read` and use `aws-actions/configure-aws-credentials` with `role-to-assume`. Give the role least privilege for its task.

**Q3. What's a common misconfiguration with OIDC trust policies?**
Overly broad `sub` conditions (e.g. `repo:org/*:*` or missing the `sub` condition entirely), which lets **any** repository — or any branch/PR including forks in some setups — assume a production role. Always pin to specific repos and protected branches/environments, and use separate roles for PR-triggered (read-only/plan) vs main/production (apply) workflows.

**Q4. How do you protect production deployments in a CI system?**
Production credentials bound to a protected environment requiring approvals, only from protected branches; OIDC role trusts only that environment; separate deploy role from build role; branch protection and CODEOWNERS on workflow files (so no one modifies the pipeline to exfiltrate creds); audit logs; restrict self-hosted runners for prod to trusted workflows; and break-glass procedures documented.

**Q5. How do pipelines access secrets like database passwords or API keys at deploy time?**
Best: the workload fetches secrets at runtime from a secret manager using its own workload identity — the pipeline never sees them. If the pipeline needs secrets (e.g. third-party API for a deploy step), fetch them dynamically from Vault/Secrets Manager using OIDC authentication, scoped to the job, masked in logs. Avoid storing them in repo variables shared broadly.

---

# STAGE 6 — CI/CD PIPELINES

---

## 31. GitHub Actions

**What it is:** GitHub's built-in CI/CD and automation platform; workflows defined in YAML in `.github/workflows/`, triggered by GitHub events.

**Key concepts**
- **Workflow → jobs → steps.** Jobs run in parallel by default on **runners** (GitHub-hosted Linux/Windows/macOS/ARM/GPU, larger runners, or **self-hosted**); `needs:` for dependencies.
- **Triggers (`on:`):** `push`, `pull_request`, `pull_request_target` (dangerous with untrusted code), `workflow_dispatch` (manual with inputs), `schedule` (cron), `release`, `workflow_call` (reusable), `repository_dispatch`, `merge_group`.
- **Actions:** Reusable steps from the Marketplace (`uses: actions/checkout@<sha>`) — JavaScript, Docker or composite actions.
- **Reusable workflows** (`workflow_call`) and **composite actions** for DRY pipelines across repos; organization **workflow templates** and required workflows via rulesets.
- **Matrix builds:** `strategy.matrix` to test multiple versions/OSes; `fail-fast`, `max-parallel`.
- **Caching:** `actions/cache`, built-in cache in setup actions (`setup-node` with `cache: pnpm`); artifacts (`upload-artifact`/`download-artifact`) to pass outputs between jobs.
- **Contexts & expressions:** `${{ github.sha }}`, `${{ secrets.X }}`, `${{ vars.X }}`, `if:` conditions, outputs between steps/jobs.
- **Security:** `permissions:` for `GITHUB_TOKEN` (least privilege), environments with protection rules and environment secrets, **OIDC** (`id-token: write`), pin actions to SHAs, artifact attestations.
- **Concurrency groups:** `concurrency: { group: deploy-prod, cancel-in-progress: false }` to prevent overlapping deploys.
- **Actions Runner Controller (ARC):** Kubernetes operator that autoscales ephemeral self-hosted runners.

```yaml
name: ci
on: { pull_request: {}, push: { branches: [main] } }
permissions: { contents: read }
jobs:
  test:
    runs-on: ubuntu-latest
    strategy: { matrix: { node: [20, 22] } }
    steps:
      - uses: actions/checkout@<pinned-sha>
      - uses: actions/setup-node@<pinned-sha>
        with: { node-version: "${{ matrix.node }}", cache: npm }
      - run: npm ci && npm test
  deploy:
    if: github.ref == 'refs/heads/main'
    needs: test
    runs-on: ubuntu-latest
    environment: production
    permissions: { id-token: write, contents: read }
    concurrency: { group: prod-deploy }
    steps:
      - uses: aws-actions/configure-aws-credentials@<pinned-sha>
        with: { role-to-assume: "arn:aws:iam::123456789012:role/deploy", aws-region: us-east-1 }
      - run: ./deploy.sh
```

### Interview Questions

**Q1. How do you share CI logic across 100 repositories in GitHub Actions?**
**Reusable workflows** (`on: workflow_call`) in a central repo for full pipelines (build-test-scan-deploy), **composite actions** for shared step sequences, versioned via tags/SHAs; organization **required workflows/rulesets** to enforce security scans; starter workflow templates for new repos. Keep inputs minimal and well documented, and roll out changes with versioning so a central change doesn't break every repo at once.

**Q2. What is the risk of `pull_request_target` and how do you use it safely?**
`pull_request_target` runs in the context of the **base** repository with access to secrets and a write-capable token, even for PRs from forks. If the workflow checks out and runs the PR's code (e.g. `npm install` executes scripts), an attacker can steal secrets ("pwn request"). Use `pull_request` for untrusted code (no secrets), and if `pull_request_target` is needed (e.g. labeling), never execute PR code, or split into a privileged workflow triggered by `workflow_run` that handles only safe artifacts.

**Q3. GitHub-hosted vs self-hosted runners?**
Hosted: zero maintenance, clean ephemeral VM per job, secure isolation, pay per minute (larger/GPU/ARM runners available). Self-hosted: access to private networks, custom hardware, cheaper at scale, caches persist — but you manage scaling, patching and security. Self-hosted runners on **public repos** are dangerous (forks can run code on your infra). Best practice: **ephemeral** self-hosted runners (one job per runner) autoscaled with **ARC** on Kubernetes or cloud-native runner solutions, segmented by trust level.

**Q4. How do you prevent two deployments to production running at the same time?**
`concurrency` groups at workflow or job level (`group: production`, `cancel-in-progress: false` to queue rather than cancel), plus environment protection rules. For GitOps, the deploy is just a Git commit — Argo CD serializes syncs.

**Q5. How do you speed up GitHub Actions workflows?**
Cache dependencies (keyed on lockfile hash) and Docker layers (`cache-from: type=gha`), run jobs in parallel, use path filters to skip unaffected workflows, matrix sharding of tests, larger runners for heavy jobs, avoid redundant checkouts/builds by passing artifacts, cancel superseded runs on PRs (`cancel-in-progress: true`), and use shallow clones.

**Q6. How do you secure GitHub Actions?**
Default `permissions: contents: read` and grant per job; pin third-party actions to commit SHAs and restrict allowed actions; OIDC instead of static cloud secrets; environment secrets with required reviewers for prod; avoid script injection by not interpolating untrusted input (`${{ github.event.issue.title }}`) directly into `run:` — pass via env vars; CODEOWNERS on `.github/workflows`; ephemeral runners; audit logs; artifact attestations for provenance.

---

## 32. GitLab CI/CD

**What it is:** GitLab's integrated CI/CD defined in `.gitlab-ci.yml`, part of GitLab's single DevSecOps platform (SCM, CI, registry, security scanning, environments, issue tracking).

**Key concepts**
- **Stages** (sequential) and **jobs** (parallel within a stage); **`needs:`** creates a DAG to run jobs as soon as dependencies finish.
- **Runners:** Shared/group/project runners with executors (Docker, Kubernetes, shell, docker-autoscaler); tags to select runners.
- **`rules:`** (replacing `only/except`) for conditional jobs; `workflow:rules` for pipeline creation.
- **Includes & templates:** `include:` local/remote/project/template files; **CI/CD Components** and the **CI/CD Catalog** for reusable, versioned pipeline building blocks; `extends:` and YAML anchors.
- **Parent-child pipelines** and **multi-project pipelines** (`trigger:`), dynamic child pipelines generated at runtime (great for monorepos).
- **Artifacts, caches, environments** (with deployment tracking, review apps, protected environments, manual `when: manual` jobs and approvals).
- **Merge trains** (merge queue), merge request pipelines, **built-in security scanners** (SAST, DAST, dependency, container, secret detection templates), **OIDC ID tokens** (`id_tokens:`) for cloud auth.

```yaml
stages: [test, build, deploy]
include:
  - template: Jobs/SAST.gitlab-ci.yml
test:
  stage: test
  image: node:22
  cache: { key: { files: [package-lock.json] }, paths: [.npm/] }
  script: [npm ci --cache .npm, npm test]
build:
  stage: build
  needs: [test]
  script: [docker build -t $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA ., docker push $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA]
deploy_prod:
  stage: deploy
  environment: production
  rules: [{ if: '$CI_COMMIT_BRANCH == "main"', when: manual }]
  script: [./deploy.sh]
```

### Interview Questions

**Q1. Stages vs `needs` (DAG) in GitLab CI?**
By default, all jobs in a stage must finish before the next stage starts — simple but slow when unrelated jobs wait. `needs:` declares explicit dependencies so a job starts as soon as its needed jobs complete (directed acyclic graph), cutting pipeline time significantly in monorepos and multi-component builds.

**Q2. How do you structure CI for a monorepo in GitLab?**
Use `rules: changes:` to run jobs only when relevant paths change, **parent-child pipelines** with a child pipeline per component (triggered conditionally), or **dynamic child pipelines** where a job generates YAML for only affected projects; share logic via CI/CD components; use `needs` for DAG execution.

**Q3. Cache vs artifacts in GitLab CI?**
**Cache** speeds up jobs by reusing dependencies between pipelines/jobs (best-effort, may be missing, keyed on lockfiles). **Artifacts** pass build outputs between stages/jobs in the same pipeline reliably (and can be downloaded/kept with expiry) — e.g. compiled binaries, test reports, coverage. Don't use cache for things you must have; don't use artifacts for large dependency directories.

**Q4. How do you implement reusable pipelines across many GitLab projects?**
**CI/CD Components** published to the **CI/CD Catalog** with versioned inputs (`include: component: gitlab.com/org/components/build@1.2.0`), or `include: project:` templates with `extends:`; compliance pipelines/frameworks to enforce security jobs; centrally managed runners.

**Q5. What are review apps in GitLab?**
Dynamic environments created per merge request (`environment: name: review/$CI_COMMIT_REF_SLUG`, `on_stop:` job to destroy), giving reviewers a live URL for the change. Implemented typically with Kubernetes namespaces or Helm releases per branch, auto-stopped after merge or after a timeout.

---

## 33. Jenkins

**What it is:** The long-standing open-source automation server — highly extensible (1,800+ plugins), self-hosted, still widespread in enterprises.

**Key concepts**
- **Pipeline as code:** `Jenkinsfile` — **Declarative** (structured `pipeline { agent; stages { stage { steps } } post {} }`) or **Scripted** (Groovy, flexible).
- **Controller and agents:** Controller schedules and hosts UI/config; agents (static, cloud, Kubernetes pods via the Kubernetes plugin) execute builds.
- **Shared libraries:** Reusable Groovy code across pipelines (`@Library('company-lib')`).
- **Multibranch pipelines & organization folders:** Auto-discover branches/PRs.
- **Credentials store**, plugins (Git, Docker, Kubernetes, Blue Ocean (legacy), Configuration as Code — **JCasC**).
- **Challenges:** Plugin dependency hell and security vulnerabilities, controller as single point of failure, Groovy sandbox issues, scaling and maintenance burden.

```groovy
pipeline {
  agent { kubernetes { yamlFile 'ci-pod.yaml' } }
  options { timeout(time: 30, unit: 'MINUTES') }
  stages {
    stage('Test')  { steps { sh 'npm ci && npm test' } }
    stage('Build') { steps { sh 'docker build -t app:$GIT_COMMIT .' } }
    stage('Deploy') {
      when { branch 'main' }
      input { message 'Deploy to production?' }
      steps { sh './deploy.sh' }
    }
  }
  post { always { junit 'reports/*.xml' } }
}
```

### Interview Questions

**Q1. Declarative vs Scripted pipelines?**
Declarative: structured, opinionated syntax with validation, `post` conditions, `when`, `options`, easier to read and maintain — recommended default. Scripted: full Groovy flexibility (loops, complex logic) but harder to maintain and test. Use declarative with `script {}` blocks or shared libraries for occasional complex logic.

**Q2. How do you scale Jenkins?**
Never run builds on the controller (set executors to 0); use ephemeral agents — Kubernetes plugin launching a pod per build with containers for each tool, or cloud agents (EC2 Fleet); multiple controllers per team/domain to reduce blast radius; configuration as code (JCasC) and pipelines in Git so controllers are reproducible; regular plugin updates; backups of `JENKINS_HOME`; monitoring of queue length.

**Q3. How do you manage Jenkins configuration reproducibly?**
**Jenkins Configuration as Code (JCasC)** YAML for system config, plugins pinned in `plugins.txt` and baked into a custom controller image, jobs defined via Job DSL or multibranch/organization folders discovering Jenkinsfiles, all stored in Git; controller deployed via Helm on Kubernetes. Treat the controller as cattle, not a pet.

**Q4. What are Jenkins shared libraries and best practices?**
Versioned Groovy libraries (in their own repo) providing reusable steps (`buildDockerImage()`, `deployToK8s()`) and even full pipeline templates (`standardPipeline { ... }`). Best practices: version and tag releases, keep logic testable (JenkinsPipelineUnit), keep heavy logic in scripts/tools instead of Groovy, and document parameters.

**Q5. Would you migrate from Jenkins to GitHub Actions/GitLab CI? How?**
Consider when maintenance burden, plugin security issues and scaling costs are high, and code is already on GitHub/GitLab. Migrate incrementally: inventory pipelines, standardize patterns into reusable workflows, use migration tooling (GitHub Actions Importer), migrate low-risk repos first, run both in parallel for validation, handle secrets via OIDC, and decommission Jenkins jobs progressively. Keep Jenkins where it provides unique value (complex on-prem integrations) if needed.

---

## 34. Other CI/CD Platforms (CircleCI, Azure DevOps, Buildkite, Tekton, AWS CodePipeline)

**What they are:**
| Tool | Key traits |
|---|---|
| **CircleCI** | Cloud CI with fast startup, orbs (reusable config packages), test splitting by timing, Docker layer caching, resource classes |
| **Azure DevOps Pipelines** | Microsoft's CI/CD (YAML multi-stage pipelines, templates, environments with approvals/checks, service connections incl. workload identity federation); part of Azure DevOps suite (Boards, Repos, Artifacts, Test Plans) |
| **Buildkite** | Hybrid model — SaaS control plane, **agents run on your infrastructure**; dynamic pipelines generated at runtime; scales to huge monorepos (used by Shopify, Uber, Canva) |
| **Tekton** | Kubernetes-native CI/CD framework (CNCF/CD Foundation) — Tasks, Pipelines, PipelineRuns as CRDs; building block for platforms (OpenShift Pipelines); Tekton Chains for supply-chain signing/provenance |
| **AWS CodePipeline/CodeBuild** | AWS-native orchestration and builds, IAM integrated (see AWS guide) |
| Others | Argo Workflows (Kubernetes workflow engine, also used for CI), Drone/Woodpecker, TeamCity, Harness CI, Google Cloud Build, Dagger (pipelines as code in real languages, portable across CI) |

### Interview Questions

**Q1. How do you choose a CI/CD platform?**
Consider where code lives (GitHub/GitLab integration), security model (OIDC, isolation, self-hosted options for private networks), scale and performance (concurrency, caching, monorepo support), cost model, ecosystem/reusability, compliance needs (on-prem/air-gapped), team skills and maintenance burden. Prefer the platform integrated with your SCM unless specific needs (huge monorepo scale → Buildkite; Kubernetes-native platform building → Tekton; Microsoft ecosystem → Azure DevOps).

**Q2. What is Buildkite's hybrid model and why do large companies use it?**
Buildkite hosts the orchestration UI/API, while build agents run on your own infrastructure (any cloud, any hardware, autoscaled). Code and secrets never leave your network, you control machine types and caching, and it scales to thousands of parallel agents; dynamic pipelines let a script generate steps based on changed code — ideal for big monorepos.

**Q3. What is Tekton and when would you use it?**
A Kubernetes-native framework where CI/CD primitives are CRDs: a **Task** is a series of steps (containers) in a pod, a **Pipeline** chains tasks, **PipelineRuns/TaskRuns** execute them; Triggers handle events; Chains signs results and generates SLSA provenance. Use it to build an internal CI/CD platform on Kubernetes or with OpenShift; it's lower-level than GitHub Actions (more assembly required).

**Q4. What is Dagger and why is "pipelines as code" in real languages appealing?**
Dagger lets you write pipeline logic in Go/Python/TypeScript that runs in containers via BuildKit — identical locally and in any CI, with automatic caching. It addresses YAML sprawl and "push and pray" debugging: you can run and debug the pipeline locally, and CI configuration becomes a thin wrapper calling Dagger functions.

**Q5. How do Azure DevOps environments and checks work?**
Deployment jobs target environments (e.g. `production`) that can have approvals, business hours, required template checks, branch control, and gates (query Azure Monitor, REST APIs) evaluated before the deployment proceeds; service connections (ideally with workload identity federation) provide scoped credentials to Azure.

---

## 35. Pipeline Design & Concepts

**What it is:** Designing end-to-end delivery pipelines that are fast, safe, secure and maintainable.

**Reference pipeline**
```
trigger (PR / merge to main)
  → lint & static checks → unit tests
  → build artifact/image (once) → SAST/SCA/secret/image scans (parallel)
  → SBOM + sign + push to registry
  → deploy to dev/ephemeral → integration & contract tests
  → deploy to staging → E2E smoke, performance checks
  → approval (manual or automated policy)
  → progressive deploy to production (canary) → automated verification (SLO metrics)
  → promote or roll back → notify, record deployment (DORA metrics)
```

**Concepts**
- **Continuous Integration vs Continuous Delivery vs Continuous Deployment:** CI = merge frequently with automated build/tests. Continuous Delivery = every change is releasable; prod deploy may need a manual approval. Continuous Deployment = every passing change deploys to prod automatically.
- **Runners/agents:** Hosted vs self-hosted, ephemeral vs persistent, autoscaling with **ARC (Actions Runner Controller)** on Kubernetes, isolation per trust level.
- **Caching:** Dependency caches, Docker layer/registry caches, build-tool remote caches.
- **Matrix builds:** Test across versions/OS/architectures.
- **Reusable workflows/templates:** Central, versioned pipeline building blocks ("golden pipelines").
- **Environments & approvals:** Protected environments, required reviewers, deployment windows.
- **Pipeline secrets:** CI secret stores, OIDC, masking, scoping per environment.
- **Pipeline observability:** Duration, failure rate, queue time, flaky tests; DORA metrics from deploy events.

### Interview Questions

**Q1. Continuous Delivery vs Continuous Deployment?**
Both require every change to pass an automated pipeline and be deployable. In Continuous Delivery, the production release is a business decision triggered manually (one click). In Continuous Deployment, every change that passes all stages goes to production automatically with no human gate — requiring high test confidence, progressive delivery, automated rollback and feature flags.

**Q2. Design a CI/CD pipeline for 50 microservices.**
Standardize with **reusable "golden" pipeline templates** (build → test → scan → sign → push) owned by the platform team and versioned; each service repo has a thin config calling the template. Artifacts promoted by digest; deployment via GitOps (each service's manifests in a config repo, Argo CD ApplicationSets per environment); progressive delivery with Argo Rollouts and automated analysis; OIDC-based credentials; environment promotions via PRs to the config repo; centralized dashboards for DORA metrics and pipeline health; contract tests to keep services independently deployable.

**Q3. How do you implement rollback in your pipeline?**
Keep previous artifacts immutable and deployable; for GitOps, revert the commit in the config repo (Argo CD syncs the previous version) or use Argo Rollouts abort; for Kubernetes deployments `kubectl rollout undo` as an emergency; automated rollback triggered by failed analysis (error rate/latency SLO breach) during canary; database migrations must be backward compatible (expand/contract) or rollback of code breaks. Practice rollbacks regularly.

**Q4. How do you make pipelines secure?**
Least-privilege tokens, OIDC to cloud, secrets scoped to protected environments, pinned dependencies/actions, ephemeral isolated runners, no untrusted code with secrets, signed artifacts and provenance, scans as gates, review required for pipeline file changes (CODEOWNERS), audit logs, separation between build and deploy credentials.

**Q5. How do you measure pipeline health?**
Lead time from commit to production, pipeline duration (p50/p90) per stage, queue/wait time for runners, success/failure rate (and failure reasons: tests, infra, flaky), flaky test rate, deployment frequency, change failure rate, and mean time to restore. Treat the pipeline as a product with SLOs.

**Q6. Ephemeral vs persistent runners — why prefer ephemeral?**
Ephemeral runners execute one job and are destroyed: no leftover state, credentials or malicious persistence between jobs (critical security property), consistent clean environments, easy autoscaling. Persistent runners keep warm caches (faster) but risk cross-job contamination and drift. Use ephemeral runners with remote caches to get both safety and speed.

---

## 36. Deployment Strategies

**What they are:** Ways to roll out a new version to production while managing risk.
| Strategy | How it works | Pros | Cons |
|---|---|---|---|
| **Recreate** | Stop old, start new | Simple | Downtime |
| **Rolling** | Replace instances gradually (`maxSurge`, `maxUnavailable`) | No extra full environment; Kubernetes default | Mixed versions during rollout; slow rollback |
| **Blue/Green** | Deploy full new environment (green) alongside old (blue); switch traffic at once | Instant rollback; test green before switch | Double resources during deploy; DB compatibility needed |
| **Canary** | Send small % of traffic to new version, increase gradually while monitoring | Limits blast radius; real-traffic validation | Needs traffic shaping and good metrics; slower |
| **A/B testing** | Route specific user segments to variants to measure business metrics | Product experimentation | Not primarily a safety mechanism |
| **Shadow (mirroring/dark launch)** | Copy production traffic to new version; discard responses | Test with real traffic, zero user impact | Side effects must be suppressed; cost |
| **Feature-flag rollout** | Deploy dark, enable features gradually per user/segment | Decouples deploy from release | Flag management overhead |

**Rollback vs roll-forward:** Rollback reverts to the previous version (fast, safe if compatible); roll-forward deploys a fix (needed when rollback is impossible, e.g. irreversible data migration). Mature teams prefer fast rollback for mitigation, then fix forward.

### Interview Questions

**Q1. Blue/green vs canary — when do you use which?**
Blue/green: when you want a fully tested environment and instant all-or-nothing switch and rollback (e.g. major version changes, monoliths, legacy apps where partial traffic splitting is hard) — costs double capacity briefly. Canary: when you want to limit blast radius and validate with real traffic progressively (e.g. 1% → 10% → 50% → 100%) with automated metric analysis — best for high-traffic microservices with good observability. They can be combined (blue/green infrastructure with canary traffic shifting).

**Q2. How does a rolling update work in Kubernetes and how do you make it zero-downtime?**
The Deployment controller creates a new ReplicaSet and scales it up while scaling the old one down, respecting `maxSurge` (extra pods allowed) and `maxUnavailable`. Zero downtime requires: readiness probes (only ready pods receive traffic), graceful shutdown (handle SIGTERM, `preStop` sleep so endpoints update before the process stops), `maxUnavailable: 0` for full capacity, PodDisruptionBudgets, and backward-compatible API/DB changes because old and new versions run simultaneously.

**Q3. What metrics would you use to automatically promote or abort a canary?**
Compare canary vs baseline/stable: HTTP error rate (5xx), latency p95/p99, saturation (CPU/memory), business KPIs (checkout success rate), and logs/exception counts. Use statistical analysis over a sufficient sample (Argo Rollouts AnalysisTemplates with Prometheus queries, Flagger, Kayenta). Abort and roll back automatically when thresholds are breached.

**Q4. What are the challenges of blue/green with databases?**
Both blue and green typically share one database, so schema changes must be compatible with both versions (expand/contract migrations). If green writes data in a new format, rolling back to blue may break. Separate databases per color require data synchronization — complex. Also long-lived connections/sessions and caches need handling during the switch.

**Q5. What is a shadow deployment and what must you be careful about?**
Production requests are duplicated to the new version asynchronously; responses are compared/discarded — validating performance and correctness under real load without affecting users. Careful: side effects (writes, emails, payments, third-party calls) must be disabled or sandboxed, and doubled load on shared dependencies. Service meshes (Istio mirroring) support it.

**Q6. When would you choose to roll forward instead of rolling back?**
When rollback isn't safe: irreversible data migrations already applied, external contract changes already consumed by clients, or the previous version has a critical security vulnerability. Also for tiny, obvious fixes when the pipeline is fast. Default for incidents: mitigate fast (rollback or disable flag), then fix forward calmly.

---

## 37. Feature Flags (LaunchDarkly, Unleash, Flagsmith, OpenFeature, AWS AppConfig)

**What it is:** Runtime switches that turn functionality on/off or vary behavior **without deploying** — decoupling **deploy** (code in production) from **release** (users see the feature).

**Flag types:** Release flags (short-lived, gradual rollout), experiment flags (A/B tests), ops flags/kill switches (disable expensive features under load), permission/entitlement flags (long-lived, plan-based).

**Tools**
- **LaunchDarkly** (market leader SaaS — targeting rules, percentage rollouts, experimentation, guarded releases with automatic rollback on metric regressions).
- **Unleash** (open-source, self-hostable), **Flagsmith** (open-source + SaaS), **Flipt**, GrowthBook (experimentation), Split (now part of Harness), ConfigCat.
- **OpenFeature (CNCF):** Vendor-neutral **standard API/SDKs** for feature flagging — swap providers without changing app code; **flagd** reference implementation.
- **AWS AppConfig** (feature flags + config with validators and gradual deployment with rollback on alarms), Azure App Configuration, Firebase Remote Config.

### Interview Questions

**Q1. How do feature flags enable trunk-based development and safer releases?**
Unfinished features can be merged to main behind a flag that's off in production, so branches stay short-lived. Releases become gradual: enable for internal users → 1% → 10% → 100%, monitor metrics, and disable instantly (kill switch) if issues arise — no redeploy or rollback needed. It turns releases into a business decision and reduces deployment risk.

**Q2. What is flag debt and how do you manage it?**
Stale flags left in code after full rollout add branching complexity, dead code paths, testing combinations and risk (an old flag accidentally toggled). Manage it: every release flag has an owner and expiry date, track flag age and usage in the flag platform, create cleanup tickets automatically when a flag reaches 100%, lint for stale flags, and limit the number of active flags per service.

**Q3. Why does OpenFeature matter?**
Without a standard, application code is coupled to a vendor's SDK; switching vendors requires code changes everywhere. OpenFeature provides a common evaluation API and provider model (plus hooks for telemetry), so you can change backends (LaunchDarkly, Flagsmith, flagd, in-house) by swapping the provider, and standardize flag telemetry across languages.

**Q4. How do you test code with feature flags?**
Test both states of each flag in unit/integration tests (parameterize tests over flag values), use test providers/in-memory flag configurations, test critical combinations rather than all permutations, ensure default/fallback values are safe if the flag service is unavailable, and include flag state in logs/traces for debugging.

**Q5. What happens if the feature flag service goes down?**
Well-designed SDKs evaluate flags locally from a cached ruleset (streaming updates), so outages only stop updates. Code must define **safe defaults** for every flag (usually "off" for new features), bootstrap from cached/last-known configuration at startup, and not block requests on flag evaluation. For self-hosted systems, run them highly available or use relay proxies.

---

## 38. Database Changes in CI/CD (Flyway, Liquibase, Expand/Contract)

**What it is:** Versioning and automating database schema changes alongside application code, without downtime.

**Tools**
- **Flyway (Redgate):** Versioned SQL migrations (`V1__create_users.sql`), schema history table, simple, repeatable migrations.
- **Liquibase:** Changesets in SQL/XML/YAML/JSON, rollback definitions, preconditions, contexts/labels, diff/changelog generation.
- Framework-native: Rails/Django/Alembic/Prisma/TypeORM/Knex migrations; **Atlas** (declarative schema-as-code with diffing and linting); gh-ost/pt-online-schema-change (online MySQL migrations); pgroll (zero-downtime Postgres migrations).

**Expand/Contract (parallel change) pattern**
1. **Expand:** Add new structures in a backward-compatible way (new column/table, nullable; dual-write from app).
2. **Migrate:** Backfill data in batches; app reads from new structure (behind a flag).
3. **Contract:** After all app versions use the new structure, remove old columns/code in a later release.

### Interview Questions

**Q1. How do you rename a column in a production database with zero downtime?**
Never rename in place (old app versions break mid-deploy). Expand: add the new column; deploy app version that writes to both columns and reads old. Backfill new column in batches. Deploy version that reads from the new column (still dual-writing). Verify. Contract: stop writing the old column, then drop it in a later release. Each step is independently deployable and rollback-safe.

**Q2. Should migrations run as part of application startup or as a separate pipeline step?**
Separate step is generally safer: a dedicated job (Kubernetes Job, Helm pre-upgrade hook, Argo CD PreSync hook, or pipeline stage) runs migrations once before rolling out new pods, with clear logs and failure handling. Running at app startup with many replicas causes concurrent migration races (mitigated by locks, as Flyway/Liquibase use), slow startups and failed rollouts if a migration fails. Either way, migrations must be backward compatible with the currently running version.

**Q3. Flyway vs Liquibase?**
Flyway: SQL-first, simple versioned files, minimal concepts — great for teams comfortable with SQL. Liquibase: database-agnostic changesets (YAML/XML/SQL), explicit rollback blocks, preconditions, contexts per environment, diff tooling — more powerful for multi-database products. Both track applied migrations in a history table and integrate into CI/CD.

**Q4. How do you handle long-running migrations on big tables?**
Avoid locking operations: add columns as nullable without defaults (or use DB features that make defaults metadata-only), create indexes concurrently (`CREATE INDEX CONCURRENTLY` in Postgres), backfill in small batches with throttling (background job), use online schema change tools (gh-ost, pt-osc, pgroll), run during low traffic, and monitor replication lag and locks. Test on production-sized data first.

**Q5. How do you test database migrations in CI?**
Spin up the target DB version with Testcontainers, apply all migrations from scratch and from the last production version (upgrade path), run integration tests, lint migrations for dangerous operations (Atlas lint, squawk for Postgres — e.g. flags locking `ALTER TABLE`), verify rollback scripts if used, and test against an anonymized production-like dataset for performance.

---

# STAGE 7 — INFRASTRUCTURE AS CODE (IaC) & CONFIGURATION

---

## 39. Terraform / OpenTofu

**What it is:** **Terraform** (HashiCorp, now IBM) is the most widely used declarative IaC tool: you describe desired infrastructure in **HCL**, and Terraform computes and applies the changes across 3,000+ **providers** (AWS, Azure, GCP, Kubernetes, GitHub, Datadog, Cloudflare...). In 2023 HashiCorp changed the license from MPL to the **Business Source License (BSL)**; the community forked the last open version as **OpenTofu** (Linux Foundation/CNCF), which remains open source and largely compatible (and adds features like state encryption).

**Core workflow:** `init` (download providers/modules, configure backend) → `plan` (diff desired vs actual state) → `apply` → `destroy`. Also `fmt`, `validate`, `import`, `state` subcommands, `output`, `console`.

**Key concepts**
- **State:** `terraform.tfstate` maps config resources to real resource IDs and stores attributes. **Remote backends** (S3 — with native S3 lockfile locking in newer versions, previously DynamoDB lock table; GCS; azurerm; HCP Terraform) for team use with **state locking** to prevent concurrent applies. State contains **secrets in plaintext** → encrypt, restrict access.
- **Providers** (version pinned with `required_providers` + `.terraform.lock.hcl`), **resources**, **data sources** (read existing infra), **variables**, **outputs**, **locals**.
- **Modules:** Reusable units (root module + child modules), versioned via registries/Git tags; composition over deep nesting.
- **Meta-arguments:** `count`, `for_each`, `depends_on`, `lifecycle` (`prevent_destroy`, `create_before_destroy`, `ignore_changes`), `provider`.
- **`count` vs `for_each`:** `count` indexes by number (removing a middle item shifts indexes → destroys/recreates); `for_each` keys by map/set (stable addresses).
- **Refactoring:** `moved` blocks (rename/move resources without destroy), `import` blocks (config-driven import), `removed` blocks, `terraform state mv/rm`.
- **Environments:** Separate state per environment — directory per env or **workspaces** (same config, different state; limited isolation) or Terragrunt.
- **Drift:** Real infra changed outside Terraform — detected by `plan` (`-refresh-only`), scheduled drift detection.
- **Testing:** `terraform test` (native `.tftest.hcl`), Terratest; checks/preconditions/postconditions; `validation` blocks on variables.
- **Secrets:** Avoid hard-coding; use secret managers, `sensitive = true` (hides from output, not from state), ephemeral resources/values (newer feature to avoid persisting secrets in state).

### Interview Questions

**Q1. What is Terraform state, why is it needed and how do you manage it in a team?**
State maps resources in configuration to real-world objects (IDs), caches attributes and dependency info, and enables Terraform to compute diffs and perform updates/deletes. In a team: store it in a **remote backend** (S3/GCS/Azure Blob/HCP Terraform) with **locking**, encryption at rest, versioning (to recover from corruption), and tight access control because it contains secrets. Split state into smaller units (per component/environment) to reduce blast radius and plan time. Never edit state manually except via `terraform state` commands.

**Q2. `count` vs `for_each`?**
`count` creates N instances addressed by index (`aws_instance.web[0]`). Removing an element from the middle of a list shifts indexes, causing Terraform to destroy and recreate resources unexpectedly. `for_each` iterates over a map or set of strings and addresses instances by key (`aws_instance.web["api"]`), so adding/removing one item affects only that item. Use `count` mainly for conditional creation (`count = var.enabled ? 1 : 0`) and `for_each` for collections.

**Q3. Someone changed a resource manually in the console. How do you handle drift?**
`terraform plan` (or `plan -refresh-only`) shows the drift. Decide: if the manual change was wrong, `apply` to revert to code; if it was a needed hotfix, update the code to match and apply (no-op). Prevent: restrict console write access in prod (read-only roles, SCPs), all changes via PR pipelines, scheduled drift detection (HCP Terraform health assessments, Spacelift/env0, CI cron plan) alerting on differences, and `ignore_changes` only for attributes legitimately managed elsewhere (e.g. autoscaling desired count).

**Q4. How do you structure Terraform for multiple environments and teams?**
Reusable, versioned **modules** (networking, EKS cluster, service) in a module registry; separate **root modules/stacks** per environment and component (`envs/prod/network`, `envs/prod/eks`) each with its own state; environment differences via variables (`.tfvars`), not code duplication; Terragrunt or a TACOS platform to keep backend/provider config DRY and orchestrate dependencies; CODEOWNERS per directory; pipeline running plan on PR and apply on merge.

**Q5. How do you rename or move a resource without destroying it?**
Add a **`moved` block** (`moved { from = aws_s3_bucket.old to = aws_s3_bucket.new }`) — Terraform updates the state address during plan/apply without touching infrastructure; this is reviewable in code and works across modules. Older approach: `terraform state mv`. Similarly use `import` blocks to adopt existing resources and `removed` blocks to stop managing a resource without destroying it.

**Q6. Workspaces vs separate directories for environments?**
Workspaces share the same configuration and backend with separate state files — convenient for identical ephemeral environments but risky for prod/dev separation (easy to apply to the wrong workspace, same credentials/backend, hard to have env-specific differences). Separate directories/stacks per environment with separate backends and credentials give stronger isolation and clearer reviews — preferred for production.

**Q7. Terraform vs OpenTofu — what changed and how do you decide?**
Terraform moved to BSL in 2023 (restricts offering competing commercial products; normal internal use remains allowed); OpenTofu forked from Terraform 1.5.x under MPL in the Linux Foundation, is mostly drop-in compatible, and has added features such as client-side state encryption and early variable evaluation, while newer Terraform features may diverge. Decide on licensing policy, vendor support (HCP Terraform vs open ecosystem), needed features, and ecosystem compatibility — migration between them is usually straightforward for common configs.

**Q8. How do you handle secrets in Terraform?**
Don't put secrets in `.tf` or `.tfvars` files; read them at apply time from Vault/AWS Secrets Manager via data sources or have resources generate them (e.g. RDS managed master password) — but remember values read or generated may still be stored in state, so encrypt and restrict state access. Mark variables/outputs `sensitive`, use newer ephemeral values/write-only arguments where supported to avoid persisting secrets, and prefer letting the platform manage secrets (managed passwords, IAM auth).

---

## 40. Pulumi, CloudFormation/CDK & Azure Bicep (Other Declarative IaC)

**What they are:**
- **Pulumi:** IaC using general-purpose languages (TypeScript, Python, Go, C#, Java, YAML). Declarative desired-state engine under the hood (it diffs and applies like Terraform), with state in Pulumi Cloud or self-managed backends (S3, etc.). Supports most Terraform providers via bridging, policy as code (CrossGuard), ESC (secrets/config), Automation API (embed IaC in programs), and AI features (Pulumi Copilot/Neo).
- **AWS CloudFormation / CDK:** AWS-native IaC — managed state, automatic rollback; CDK generates CloudFormation from code (detailed in the AWS guide).
- **Azure Bicep:** Microsoft's domain-specific language compiling to ARM templates — concise, type-safe, day-0 Azure support, no state file (Azure Resource Manager is the source of truth), `what-if` previews, deployment stacks.
- **Google Cloud:** Infrastructure Manager (managed Terraform), Config Connector (Kubernetes CRDs for GCP resources).

### Interview Questions

**Q1. Pulumi vs Terraform?**
Pulumi: real programming languages (loops, functions, classes, packages, IDE support, unit tests with familiar frameworks), good for teams of software engineers and for building higher-level abstractions/platforms (Automation API). Terraform: HCL is simpler and more constrained (easier for ops teams to read and review), bigger community, modules and talent pool, many TACOS tools. Both are declarative with state. Risk with Pulumi: over-engineered code that's hard to review; with Terraform: verbose duplication and limited logic.

**Q2. When would you choose cloud-native IaC (CloudFormation/Bicep) over Terraform?**
Single-cloud organizations wanting no state files to manage (the cloud service holds state), first-party support and day-0 features, native rollback (CloudFormation), integration with cloud governance (StackSets, deployment stacks), and no extra licensing. Choose Terraform/OpenTofu/Pulumi for multi-cloud, SaaS providers (Datadog, GitHub, Cloudflare) and a uniform workflow.

**Q3. How does Bicep differ from ARM templates?**
Bicep is a cleaner DSL (much less verbose than JSON ARM templates), with type safety, modules, loops and conditions, transpiled to ARM JSON for deployment. No state file — Azure Resource Manager compares desired vs actual. `what-if` previews changes. ARM templates are effectively the "assembly language" beneath it.

**Q4. What is the Pulumi Automation API used for?**
Running Pulumi programmatically inside another application — e.g. a self-service platform that provisions environments on demand from an API call, an internal developer portal backend, or custom CLIs — without shelling out to the CLI. Useful for platform engineering.

**Q5. How do you test infrastructure code written in Pulumi or CDK?**
Unit tests with mocks (assert properties: buckets are encrypted, no public SGs) using the language's test framework, snapshot tests of synthesized output (CDK templates), policy-as-code checks (Pulumi CrossGuard, cdk-nag, Checkov), and integration tests deploying to ephemeral stacks and verifying behavior, then destroying them.

---

## 41. Crossplane (Kubernetes-Native IaC)

**What it is:** A CNCF project that extends Kubernetes to provision and manage **cloud infrastructure as Kubernetes resources** (CRDs). The Kubernetes control plane continuously reconciles external resources (RDS databases, S3 buckets, VPCs) just like it reconciles pods.

**Key concepts**
- **Providers:** Install CRDs and controllers for a cloud (provider-aws, provider-gcp, provider-azure; Upbound "official" providers, many generated from Terraform providers).
- **Managed Resources (MRs):** 1:1 representation of an external resource (`RDSInstance`, `Bucket`).
- **Composite Resources (XRs) & Compositions:** Platform teams define higher-level abstractions — e.g. `XPostgresDatabase` composed of RDS instance + subnet group + security group + secret — via **CompositeResourceDefinitions (XRDs)**; **Composition functions** (pipelines written in Go/Python/KCL/templating) generate resources. Crossplane v2 simplifies by making composite resources namespaced and allowing composition of any Kubernetes resources.
- **Claims** (v1) / namespaced XRs (v2): Developers request infra in their namespace (`kind: PostgresDatabase`), getting connection secrets.
- **Continuous reconciliation:** Drift is automatically corrected.
- Works naturally with **GitOps** (Argo CD syncs Crossplane claims).

### Interview Questions

**Q1. Crossplane vs Terraform?**
Terraform runs on demand (plan/apply in a pipeline), stores state in a file, and only corrects drift when someone runs it. Crossplane runs as controllers in Kubernetes, **continuously reconciling** desired state (self-healing drift), uses the Kubernetes API (RBAC, namespaces, admission policies, GitOps tools) as the interface, and lets platform teams publish **custom APIs** for developers. Trade-offs: Crossplane requires operating a Kubernetes control plane, debugging is via Kubernetes events/conditions, and provider maturity varies; Terraform has a larger ecosystem and simpler mental model for one-off infra.

**Q2. How would a platform team use Crossplane to offer self-service databases?**
Define an XRD for `PostgresDatabase` with a simple schema (size: small/medium/large, version), write a Composition mapping it to an RDS instance with company defaults (encryption, backups, private subnets, parameter group, monitoring) and writing connection details to a Kubernetes secret. Developers commit a short YAML claim to their app repo; Argo CD applies it; Crossplane provisions it; policies (Kyverno) restrict allowed sizes per namespace. Developers never touch raw cloud APIs.

**Q3. What are the risks of running infrastructure controllers in Kubernetes?**
The management cluster becomes critical infrastructure (if it's down, no changes or reconciliation — existing resources keep running), cloud credentials live in the cluster (use workload identity, tight RBAC), accidental deletion of a claim can delete production databases (use `deletionPolicy: Orphan` for critical resources and admission policies), and controller bugs can mass-modify resources. Run a dedicated, well-secured management cluster.

**Q4. What are Composition Functions?**
A pipeline model for compositions where each step is a function (container) that receives the desired composite resource and outputs managed resources — allowing logic in real languages (Go, Python), templating (go-templating, KCL), or patch-and-transform. They replaced the limited native patch-and-transform syntax for complex logic.

**Q5. How does Crossplane fit with GitOps?**
All infrastructure claims/XRs are YAML in Git; Argo CD or Flux syncs them to the management cluster; Crossplane reconciles to the cloud. Git holds the desired state for both apps and infrastructure in one workflow, with PR reviews, audit and rollback by revert.

---

## 42. Ansible (and Chef/Puppet)

**What it is:** **Ansible** (Red Hat) is an agentless **configuration management** and automation tool: connects over SSH (or WinRM) and executes tasks described in YAML **playbooks**, using idempotent **modules**.

**Key concepts**
- **Inventory:** Hosts and groups (static INI/YAML or **dynamic inventory** from cloud APIs).
- **Playbooks → plays → tasks** using modules (`apt`, `copy`, `template`, `service`, `user`), **handlers** (run on change, e.g. restart nginx), variables and facts, Jinja2 templates.
- **Roles** (reusable structure) and **Collections** (distribution format), Ansible Galaxy.
- **Idempotency:** Modules check current state and only change what's needed; `--check` (dry run) and `--diff`.
- **Ansible Vault:** Encrypt secrets in files.
- **Ansible Automation Platform / AWX:** UI, RBAC, scheduling, credentials, Event-Driven Ansible (rulebooks reacting to events).
- **Chef** (Ruby DSL, pull-based agents, cookbooks) and **Puppet** (declarative DSL, agent pull model, catalog compilation) — older, agent-based configuration management, still found in legacy enterprises (Chef is now owned by Progress; Puppet by Perforce).

### Interview Questions

**Q1. Terraform vs Ansible — what's the difference?**
Terraform is **declarative provisioning** of infrastructure (create VPCs, VMs, databases) with state and dependency graph — ideal for cloud resources lifecycle. Ansible is primarily **procedural configuration management and orchestration** of existing systems (install packages, configure files, run deployments, patching) — agentless and stateless. Common pattern: Terraform provisions, Ansible configures (or better, bake images with Packer and avoid post-boot configuration entirely). Ansible can provision cloud resources too, but without state-based drift handling.

**Q2. What does idempotency mean in Ansible and how do you ensure it?**
Running a playbook multiple times yields the same end state without unintended changes — the second run should report `changed=0`. Use state-based modules (`apt: state=present`, `template`, `lineinfile`) instead of `shell`/`command`; when shell is unavoidable, use `creates:`/`removes:` or `changed_when:` conditions; test with Molecule and run twice in CI to verify no changes on the second run.

**Q3. Agentless (Ansible) vs agent-based (Chef/Puppet)?**
Agentless: no software to install/maintain on targets, uses SSH, push model, simple to start; scaling to thousands of hosts requires tuning (forks, pipelining, mitogen) and connectivity from the control node. Agent-based: agents pull configuration periodically, enforce continuous compliance (drift correction), scale well, but require agent lifecycle management and a central server. With immutable infrastructure and containers, configuration management needs have shrunk overall.

**Q4. How do you manage secrets in Ansible?**
Ansible Vault to encrypt variable files (with vault password from a secure source in CI), or better, lookup plugins fetching secrets at runtime from HashiCorp Vault/AWS Secrets Manager (`community.hashi_vault`), `no_log: true` on tasks handling secrets to keep them out of logs.

**Q5. How do you test Ansible roles?**
**Molecule**: spins up test instances (Docker/Podman/VMs), applies the role, runs an idempotency check (second converge must not change anything), verifies with tests (Testinfra/Ansible verify playbooks), and destroys. Plus `ansible-lint`, syntax checks, and `--check --diff` against staging.

---

## 43. Image Building with Packer & EC2 Image Builder (Immutable Infrastructure)

**What it is:**
- **Packer (HashiCorp):** Builds identical machine images (AWS AMIs, Azure images, GCP images, VMware, Docker) from a single HCL template — launching a temporary instance, running provisioners (shell, Ansible), and snapshotting the result.
- **EC2 Image Builder:** AWS-managed pipeline to build, test, patch and distribute AMIs and container images on a schedule, with components and recipes.
- **Immutable infrastructure / golden images:** Servers are never modified in place; to change anything, build a new image and replace instances. Pairs with ASG instance refresh or blue/green.

### Interview Questions

**Q1. What is immutable infrastructure and why is it preferred?**
Instead of patching and configuring running servers (mutable — leads to configuration drift and "snowflake" servers), you build a new versioned image with changes baked in and replace instances. Benefits: consistency between environments, predictable deployments, easy rollback (redeploy previous image), faster scaling (no boot-time configuration), better security (no SSH changes, smaller attack surface). Containers are the extreme form of this.

**Q2. Describe a golden AMI pipeline.**
Packer template (or Image Builder recipe) starting from a hardened base → install agents (monitoring, security), apply CIS hardening, OS patches → run tests (InSpec/Goss, vulnerability scan) → produce a versioned AMI, tag with metadata → share/copy to accounts and regions → publish ID to SSM Parameter Store → ASGs/launch templates consume the latest approved AMI via instance refresh. Scheduled rebuilds (e.g. weekly) to pick up patches; old AMIs deprecated.

**Q3. Packer vs configuration at boot (user data/cloud-init)?**
Baking with Packer moves slow, failure-prone installation to build time → fast, reliable boots and scaling, tested images. Boot-time configuration is flexible and simple for small tweaks (environment-specific values) but slows startup and can fail during scaling events (e.g. package repository down). Common: bake everything static, inject only environment config at boot.

**Q4. How do you keep golden images patched without breaking workloads?**
Automated scheduled rebuilds, automated tests on the image, staged rollout (dev → staging → prod) via launch template versions/instance refresh with health checks and auto rollback, image lifecycle policies (deprecate after N days), and dashboards showing which instances run outdated images.

**Q5. Do you still need Packer if everything runs in containers?**
Less so for apps — container images are your artifacts. But nodes still need images: Kubernetes worker AMIs (often use vendor-provided like Bottlerocket/EKS-optimized, or custom built with Packer for agents/hardening), bastion/VPN/legacy VMs, and on-prem/VMware images.

---

## 44. IaC Workflow Automation (HCP Terraform, Atlantis, Spacelift, env0, Terragrunt)

**What it is:** Tooling to run IaC safely as a team — PR-based plan/apply, locking, policy, approvals, drift detection, and DRY configuration. Often called **TACOS** (Terraform Automation and Collaboration Software).

| Tool | Notes |
|---|---|
| **HCP Terraform** (formerly Terraform Cloud) / Terraform Enterprise | HashiCorp SaaS/self-hosted — remote runs, state, private registry, Sentinel/OPA policies, run tasks, drift detection, no-code modules |
| **Atlantis** | Open-source, self-hosted; comments `atlantis plan` / `atlantis apply` on PRs, locks directories per PR, simple GitOps workflow for Terraform/OpenTofu |
| **Spacelift** | SaaS — stacks, dependencies between stacks, OPA policies, drift detection, supports Terraform/OpenTofu/Pulumi/CloudFormation/Ansible/Kubernetes |
| **env0, Scalr, Harness IaCM** | Similar platforms with cost estimation, self-service environments, governance |
| **Terragrunt** | Thin wrapper keeping Terraform/OpenTofu DRY: shared backend/provider config, inputs, dependencies between modules, `run-all`/stacks across many modules |

### Interview Questions

**Q1. Describe a safe team workflow for Terraform changes.**
Developer opens PR → automation runs `fmt`, `validate`, lint (tflint), security scan (Checkov/Trivy), cost estimate (Infracost) and `plan`, posting the plan as a PR comment → reviewers (CODEOWNERS) review code and plan → merge (or `apply` comment in Atlantis) triggers apply with the **same plan** → state locked during runs → notifications and drift detection afterward. Credentials via OIDC; prod applies require approval.

**Q2. How does Atlantis work and what are its security considerations?**
Atlantis is a server receiving VCS webhooks; on PR it runs `plan` in affected directories and comments results; `atlantis apply` applies and it locks those directories so other PRs can't plan/apply conflicting changes until merged. Security: it holds powerful cloud credentials (use OIDC/IRSA, separate per environment), restrict who can run apply (`apply_requirements: [approved, mergeable]`), protect against malicious PRs running arbitrary code during plan (external data sources/providers can execute code) — use repo allowlists and server-side config.

**Q3. Why use Terragrunt?**
In large estates with many root modules (per account/region/component), Terraform repeats backend and provider config everywhere. Terragrunt generates them from a single definition, passes inputs, declares dependencies between modules (outputs of VPC feed EKS), and runs commands across many modules in dependency order. Downside: extra tool and abstraction to learn; OpenTofu/Terraform have added some features (e.g. stacks in HCP Terraform, early evaluation in OpenTofu) narrowing the gap.

**Q4. How do you implement drift detection at scale?**
Scheduled plan runs (HCP Terraform health assessments, Spacelift/env0 drift detection, or a CI cron running `plan -detailed-exitcode` per stack — exit code 2 means changes) that alert to Slack/tickets with the diff; optionally auto-remediate for low-risk stacks; plus preventive controls (no console write access in prod).

**Q5. HCP Terraform vs self-hosted Atlantis?**
HCP Terraform: managed state, runs, private module registry, policies, SSO, drift detection, no servers to run — but cost per resource (RUM pricing) and BSL Terraform only. Atlantis: free, simple, self-hosted, works with OpenTofu — but you operate it, store state yourself, and build extras (policy, drift, cost) via other tools. Choose by scale, budget, compliance and features needed.

---

## 45. IaC Testing, Policy & Cost (Terratest, `terraform test`, Checkov, Conftest/OPA, Sentinel, Infracost)

**What it is:** Verifying infrastructure code before and after deployment, enforcing organizational policies, and showing cost impact in PRs.
- **Static checks:** `terraform fmt -check`, `validate`, **tflint** (provider-specific linting), **Checkov/Trivy/KICS** (security misconfigurations).
- **`terraform test`:** Native testing framework (`.tftest.hcl`) — `run` blocks with `plan` or `apply` commands and `assert` conditions; mock providers for unit-style tests.
- **Terratest (Gruntwork):** Go library to deploy real infrastructure in a test account, validate it (HTTP checks, AWS API calls), then destroy — integration/E2E tests for modules.
- **Policy as code:** **OPA/Conftest** (Rego policies against plan JSON — `terraform show -json plan.out | conftest test -`), **Sentinel** (HashiCorp's policy language in HCP Terraform/Enterprise), Checkov custom policies, HCP Terraform OPA support.
- **Infracost:** Estimates monthly cost of Terraform changes and posts a cost diff comment on PRs; policies on cost thresholds (e.g. require approval if +$1,000/month).

### Interview Questions

**Q1. How do you test Terraform modules?**
Layers: static (fmt, validate, tflint, Checkov) on every PR; **unit-style tests** with `terraform test` using `plan` and mocked providers to assert computed values and validation rules; **integration tests** with `terraform test` (apply mode) or Terratest deploying to a sandbox account and asserting behavior (e.g. bucket is private, endpoint returns 200), then destroying; examples directory used as test fixtures; run integration tests on module release rather than every PR due to cost and time.

**Q2. How do you enforce policies like "all S3 buckets must be encrypted" or "no instance larger than xlarge in dev"?**
Policy as code evaluated against the **plan JSON** in the pipeline: Conftest/OPA (Rego), Sentinel in HCP Terraform, or Checkov custom policies. Policies are versioned and tested, with enforcement levels (advisory, soft-mandatory with override, hard-mandatory). Complement with cloud-side guardrails (SCPs, Azure Policy) because IaC policies don't cover click-ops.

**Q3. Why evaluate policies against the plan rather than the code?**
The plan contains the **resolved** values after variables, modules, functions and data sources are evaluated — what will actually be created. Code-level static scanning can miss values computed at runtime or coming from modules; plan-based policies see the final attributes and the type of change (create/update/delete), enabling rules like "no deletes of databases".

**Q4. What does Infracost add to the workflow?**
Cost visibility at review time: a PR comment showing the monthly cost difference per resource (e.g. "+$2,340/month: m6i.4xlarge × 3"), enabling engineers to catch expensive mistakes before merge and making cost an engineering concern (shift-left FinOps). It can enforce cost policies and track estimates vs actuals.

**Q5. What's the testing pyramid for infrastructure code?**
Many fast static checks (format, lint, security, policy) → unit-ish tests on plan with mocks → fewer integration tests that deploy real resources in sandbox accounts → a few end-to-end tests of full environments (e.g. nightly ephemeral environment build and smoke test) → production monitoring and drift detection. Real cloud tests are slow and costly, so reserve them for modules and critical paths.

---

# STAGE 8 — RUNTIME PLATFORM: KUBERNETES & ECOSYSTEM

---

## 46. Kubernetes Architecture

**What it is:** Kubernetes (K8s) is an open-source container orchestration platform that automates deployment, scaling, self-healing and networking of containerized workloads using a **declarative desired-state** model and **control loops** (reconciliation).

**Control plane components**
- **kube-apiserver:** Front door — all reads/writes go through its REST API; authentication, authorization (RBAC), admission control, validation; the only component talking to etcd.
- **etcd:** Distributed, strongly consistent key-value store (Raft consensus) holding all cluster state. Needs an odd number of members (3/5) for quorum; back it up.
- **kube-scheduler:** Assigns unscheduled pods to nodes — **filtering** (resources, taints, affinity, volume topology) then **scoring** (spread, resource balance).
- **kube-controller-manager:** Runs controllers (Deployment, ReplicaSet, Node, Job, EndpointSlice, ServiceAccount...) that watch state and act to reach desired state.
- **cloud-controller-manager:** Cloud-specific logic (load balancers, node lifecycle, routes).

**Node components**
- **kubelet:** Agent on each node — ensures containers described in PodSpecs are running and healthy (via CRI runtime), reports status, runs probes.
- **kube-proxy:** Implements Service virtual IPs using iptables/IPVS/nftables rules (can be replaced by eBPF — Cilium).
- **Container runtime:** containerd/CRI-O.
- Add-ons: CoreDNS, CNI plugin, metrics-server, ingress/gateway controllers.

### Interview Questions

**Q1. What happens when you run `kubectl apply -f deployment.yaml`?**
kubectl sends the object to the **API server** → authentication → authorization (RBAC) → **mutating admission** (defaults, sidecar injection) → schema validation → **validating admission** (policies) → persisted in **etcd**. The **Deployment controller** sees the new Deployment and creates a ReplicaSet; the **ReplicaSet controller** creates Pod objects; the **scheduler** sees unscheduled pods and binds each to a node; the **kubelet** on that node sees the pod assigned, asks the container runtime to pull images and start containers, sets up networking via CNI and volumes via CSI, runs probes, and reports status back. EndpointSlices are updated when pods become ready so Services route traffic to them.

**Q2. What is the reconciliation loop / controller pattern?**
Controllers continuously **watch** resources through the API server, compare **desired state** (spec) with **actual state** (status/real world), and take actions to converge (create/delete pods, call cloud APIs), then repeat. It's level-triggered and self-healing: if a pod dies or someone deletes it, the controller recreates it. This pattern is also how Operators, Argo CD and Crossplane work.

**Q3. What happens if etcd is lost or the control plane goes down?**
If the control plane is unavailable, running pods keep running (kubelet keeps containers alive) and existing Services keep routing, but nothing can change: no scheduling, scaling, self-healing of failed nodes, or deployments. If etcd data is lost without backup, cluster state is gone — you must rebuild from manifests (GitOps makes this feasible). Hence: HA control plane (3+ etcd members across AZs), regular etcd snapshots, and managed control planes (EKS/GKE/AKS) handle this for you.

**Q4. How does the scheduler decide where to put a pod?**
**Filtering** removes infeasible nodes: insufficient CPU/memory **requests**, node selectors/affinity mismatch, untolerated taints, port conflicts, volume zone constraints, topology spread rules. **Scoring** ranks remaining nodes (least/most allocated resources, image locality, inter-pod affinity, spread). Highest score wins; pod is bound. If none fit, the pod stays Pending (and cluster autoscalers may add nodes). Scheduling uses requests, not actual usage.

**Q5. Why is etcd sensitive to performance, and what are best practices?**
etcd uses Raft consensus and fsyncs every write to disk; slow disks or network latency cause leader elections, timeouts and an unstable cluster. Best practices: fast SSDs, low-latency network between members, odd number of members (3 or 5), regular defragmentation and compaction, snapshot backups, monitoring of `etcd_disk_wal_fsync_duration` and leader changes, and avoiding huge objects or excessive watch/list load (e.g. too many CRDs/events).

**Q6. What does kube-proxy do and what replaces it?**
kube-proxy watches Services and EndpointSlices and programs node-level rules (iptables, IPVS, or nftables) to DNAT traffic from a Service's ClusterIP to backend pod IPs. At large scale iptables rules become slow to update and evaluate; **eBPF-based** dataplanes (Cilium's kube-proxy replacement) implement Services more efficiently with better observability.

---

## 47. Kubernetes Workloads (Pod, Deployment, ReplicaSet, StatefulSet, DaemonSet, Job, CronJob)

**What they are:**
| Object | Purpose |
|---|---|
| **Pod** | Smallest deployable unit — one or more containers sharing network namespace (same IP, localhost) and volumes; ephemeral. Patterns: sidecar (now native **sidecar containers** via init containers with `restartPolicy: Always`), init containers, ambassador, adapter |
| **ReplicaSet** | Ensures N identical pod replicas run; rarely used directly |
| **Deployment** | Manages ReplicaSets for **stateless** apps — rolling updates, rollbacks (`kubectl rollout undo`), revision history |
| **StatefulSet** | For **stateful** apps — stable pod names (`db-0`, `db-1`), stable network identity via headless Service, per-pod PersistentVolumeClaims (`volumeClaimTemplates`), ordered deployment/scaling/termination |
| **DaemonSet** | One pod per (matching) node — log shippers, monitoring agents, CNI, CSI node plugins |
| **Job** | Run-to-completion tasks with retries (`backoffLimit`), parallelism, completions, indexed jobs |
| **CronJob** | Scheduled Jobs (cron syntax), `concurrencyPolicy`, history limits, timezone support |

### Interview Questions

**Q1. Deployment vs StatefulSet?**
Deployments treat pods as interchangeable: random names, any pod can be replaced, shared or no persistent storage — for stateless web/API services. StatefulSets give each pod a **stable identity** (ordinal name and DNS via a headless Service), **its own persistent volume** that follows it across rescheduling, and **ordered** startup/scale/termination — needed for clustered databases, Kafka, ZooKeeper, Elasticsearch where members have identities and data. For production databases, managed services or operators (which use StatefulSets) are often better than hand-rolled StatefulSets.

**Q2. What are init containers and sidecar containers?**
**Init containers** run to completion sequentially before app containers start — for setup tasks (wait for a dependency, run migrations, fetch config, set permissions). **Sidecars** run alongside the main container for the pod's lifetime — log shippers, proxies (Envoy in service meshes), secret agents. Kubernetes 1.29+ supports **native sidecars** (init containers with `restartPolicy: Always`) that start before and stop after the main container, fixing old issues like Jobs never completing because the sidecar keeps running.

**Q3. When would you use a DaemonSet?**
When a pod must run on every node (or a subset): node-level log collectors (Fluent Bit), metrics agents (node-exporter, Datadog agent), CNI and CSI node plugins, security agents (Falco), and local caches/proxies. DaemonSet pods are scheduled automatically when nodes join. Use tolerations to run on tainted nodes (e.g. control plane, GPU nodes).

**Q4. How do you ensure a CronJob doesn't run twice concurrently?**
Set `concurrencyPolicy: Forbid` (skip new run if previous still running) or `Replace`; set `startingDeadlineSeconds` to bound late starts; make the job **idempotent** because Kubernetes CronJobs can occasionally create zero or two jobs in rare cases; set `activeDeadlineSeconds` and `backoffLimit`; keep `successfulJobsHistoryLimit`/`failedJobsHistoryLimit` small.

**Q5. How do you run a batch processing job across 1,000 items in parallel?**
A Job with `parallelism` (concurrent pods) and `completions` (total), using **Indexed Jobs** (`completionMode: Indexed`) so each pod gets `JOB_COMPLETION_INDEX` to pick its shard; or a work-queue pattern where pods pull items from a queue (SQS/Redis) until empty; set `backoffLimitPerIndex` for per-item retries; for large/complex batch use Kueue, Argo Workflows or Volcano.

**Q6. Why shouldn't you create bare Pods in production?**
A bare pod isn't recreated if its node fails or it's evicted — no controller manages it. Use Deployments/StatefulSets/Jobs which provide self-healing, scaling and rollout management.

---

## 48. Kubernetes Networking (Services, Ingress, Gateway API, CoreDNS, CNI)

**Networking model:** Every pod gets its own IP; all pods can communicate with all other pods without NAT (flat network) — implemented by the **CNI** plugin; NetworkPolicies restrict it.

**Services** (stable virtual IP + DNS name for a set of pods selected by labels):
- **ClusterIP** (default) — internal only.
- **NodePort** — exposes on a port (30000–32767) on every node.
- **LoadBalancer** — provisions a cloud/external load balancer.
- **Headless** (`clusterIP: None`) — DNS returns pod IPs directly (StatefulSets, client-side load balancing).
- **ExternalName** — DNS CNAME to an external name.
- **EndpointSlices** track backend pod IPs; `externalTrafficPolicy: Local` preserves client IP; topology-aware routing / `trafficDistribution` to keep traffic in-zone.

**Ingress:** L7 HTTP routing rules (host/path → Service) implemented by an **Ingress controller** (NGINX, Traefik, HAProxy, AWS LB Controller). Limited, annotation-heavy, frozen API.

**Gateway API** (successor to Ingress, GA since 2023): Role-oriented resources — **GatewayClass** (infra provider), **Gateway** (listeners, owned by platform team), **HTTPRoute/GRPCRoute/TLSRoute/TCPRoute** (owned by app teams), with traffic splitting, header matching, cross-namespace delegation via ReferenceGrant — portable across implementations (Envoy Gateway, Istio, Cilium, NGINX Gateway Fabric, Kong, Traefik, cloud gateways).

**CoreDNS:** Cluster DNS — `service.namespace.svc.cluster.local`; `ndots:5` default causes extra lookups for external names; NodeLocal DNSCache improves performance.

**CNI plugins:** Calico, **Cilium** (eBPF), Flannel, cloud CNIs (AWS VPC CNI, Azure CNI, GKE Dataplane V2 = Cilium); responsible for pod IPs, routing, and NetworkPolicy enforcement.

### Interview Questions

**Q1. Explain the Service types and when to use each.**
ClusterIP for internal service-to-service communication (most services). NodePort mainly as a building block or for simple/bare-metal setups. LoadBalancer to expose a service externally via a cloud L4 load balancer (one LB per service — costly at scale). Headless for stateful sets or clients that need individual pod IPs. ExternalName to alias external services. For HTTP, expose many services through a single Ingress/Gateway instead of many LoadBalancers.

**Q2. How does a request reach a pod from outside the cluster?**
Client → DNS → cloud load balancer (created by a LoadBalancer Service or Gateway/Ingress controller) → either node ports (then kube-proxy/eBPF forwards to a pod, possibly on another node) or directly to pod IPs (IP target mode with VPC CNI) → Ingress/Gateway proxy pods (L7 routing, TLS termination) → Service → selected backend pod via EndpointSlices. Readiness probes determine which pods are endpoints.

**Q3. Ingress vs Gateway API — why is Gateway API the future?**
Ingress only standardizes basic host/path routing; everything else (timeouts, canary weights, header routing, TLS options) relies on controller-specific annotations — non-portable and error-prone, and a single resource mixes platform and app concerns. Gateway API is expressive (weighted traffic splitting, header/query matching, gRPC/TCP/TLS routes, filters), extensible, role-oriented (platform owns Gateways, app teams own Routes, with cross-namespace controls), and portable. Ingress is frozen; new features land in Gateway API. Also, the community ingress-nginx project announced retirement (end of maintenance in early 2026), pushing migrations to Gateway API implementations.

**Q4. A pod can't resolve or reach another service. How do you debug?**
Check DNS: `kubectl exec -it pod -- nslookup svc.namespace` (CoreDNS pods healthy? correct namespace?). Check Service: selector matches pod labels, ports/targetPort correct, `kubectl get endpointslices -l kubernetes.io/service-name=svc` shows ready endpoints (empty → readiness probe failing or label mismatch). Test pod-to-pod IP connectivity, then NetworkPolicies (default deny?), then application listening on the right interface (`0.0.0.0` vs `127.0.0.1`) and port. Use `kubectl debug` with a netshoot image for tools.

**Q5. What is the `ndots` issue in Kubernetes DNS?**
Pods' `/etc/resolv.conf` has `ndots:5` and several search domains. A name with fewer than 5 dots (e.g. `api.example.com`) is first tried with each search suffix (`api.example.com.default.svc.cluster.local`, etc.) before the absolute name — multiplying DNS queries and latency, overloading CoreDNS. Fixes: use fully-qualified names with trailing dot (`api.example.com.`), lower `ndots` via `dnsConfig`, NodeLocal DNSCache, scale CoreDNS.

**Q6. What does a CNI plugin do and how do you choose one?**
CNI plugins allocate pod IPs, configure interfaces/routes, and (for many) enforce NetworkPolicies. Choices: cloud-native CNI (AWS VPC CNI — pods get VPC IPs, native routing; watch IP exhaustion), overlay CNIs (Flannel/Calico VXLAN — simple, IP-independent), **Cilium** (eBPF — high performance, kube-proxy replacement, L7 policies, Hubble observability, encryption), Calico (mature policy engine, BGP). Choose based on scale, policy needs, observability, cloud integration and team expertise.

---

## 49. Kubernetes Configuration & Storage (ConfigMaps, Secrets, PV/PVC, StorageClass, CSI)

**What they are:**
- **ConfigMap:** Non-sensitive configuration (key-values or files) injected as env vars or mounted volumes.
- **Secret:** Sensitive data (base64-encoded — **not encrypted by default**); types: Opaque, TLS, dockerconfigjson, service-account tokens. Enable **encryption at rest** in etcd (KMS provider), restrict RBAC, prefer external secret managers.
- **Volumes:** emptyDir (scratch, pod lifetime), configMap/secret, projected, hostPath (avoid), persistent volumes.
- **PersistentVolume (PV):** A piece of storage in the cluster (provisioned statically or dynamically).
- **PersistentVolumeClaim (PVC):** A user's request for storage (size, access mode); binds to a PV.
- **StorageClass:** Defines a storage "tier" and provisioner (e.g. `gp3` EBS via EBS CSI driver), parameters, `reclaimPolicy` (Delete/Retain), `volumeBindingMode: WaitForFirstConsumer` (create volume in the zone where the pod is scheduled), `allowVolumeExpansion`.
- **Access modes:** ReadWriteOnce (one node), ReadOnlyMany, ReadWriteMany (shared — EFS/NFS/CephFS), ReadWriteOncePod.
- **CSI (Container Storage Interface):** Standard plugin interface for storage vendors (EBS, EFS, Azure Disk, GCE PD, Ceph, Portworx) — supports snapshots (`VolumeSnapshot`), cloning, resizing.

### Interview Questions

**Q1. Are Kubernetes Secrets secure?**
Not by default: they're only base64-encoded and stored in etcd (possibly unencrypted), readable by anyone with `get secrets` RBAC in the namespace or who can create pods mounting them. Harden: enable **etcd encryption at rest with a KMS provider**, strict least-privilege RBAC (avoid broad `list/watch secrets`), use external secret managers (Vault, AWS Secrets Manager) via External Secrets Operator or Secrets Store CSI, avoid env vars for highly sensitive secrets (can leak via logs/dumps — prefer files), audit access, and never commit Secret manifests unencrypted (use SOPS/Sealed Secrets).

**Q2. How does dynamic volume provisioning work?**
A PVC references a StorageClass; the CSI provisioner (external-provisioner sidecar) sees the unbound PVC and calls the storage backend (e.g. EBS CreateVolume) to create a volume, creates a matching PV, and binds it. With `WaitForFirstConsumer`, provisioning waits until the pod is scheduled so the volume is created in the same AZ as the node. The CSI node plugin then attaches and mounts the volume on the node.

**Q3. A pod with a PVC is stuck Pending after its node died. Why?**
For zonal block storage (EBS, Azure Disk), the volume exists in one AZ; if no schedulable node exists in that AZ (or the volume is still attached to the dead node), the pod can't start. Kubernetes waits for the volume to detach (may take minutes, force-detach after timeout). Mitigations: node groups in every AZ with capacity, use replication at the application level (databases with replicas in different AZs), or use regional/multi-AZ storage (EFS, regional persistent disks) where appropriate.

**Q4. How do you update a ConfigMap and have pods pick up changes?**
Environment variables from ConfigMaps are read only at container start — need a rollout restart. Mounted ConfigMap files are updated eventually (kubelet sync, ~1 minute; not for `subPath` mounts), but the app must re-read them. Common practice: include a hash of the config in the pod template annotation (Helm `checksum/config`, Kustomize configMapGenerator name suffix) so config changes trigger a rolling update — making config changes versioned and roll-back-able.

**Q5. What is the reclaim policy and why does it matter?**
It determines what happens to the underlying storage when a PVC is deleted: `Delete` (default for dynamic provisioning — volume destroyed) or `Retain` (PV kept for manual recovery). For production databases, use `Retain` storage classes or snapshots/backups (VolumeSnapshots, Velero) to prevent data loss from accidental deletes (e.g. deleting a namespace or Helm release).

---

## 50. Kubernetes Health & Resources (Probes, Requests/Limits, QoS, PDBs)

**Probes**
- **Liveness:** Is the container alive? Failure → kubelet **restarts** the container. For deadlocks/hung processes.
- **Readiness:** Can it serve traffic? Failure → pod **removed from Service endpoints** (not restarted). For startup warm-up, temporary overload, dependency loss.
- **Startup:** For slow-starting apps — disables liveness/readiness until it succeeds, preventing premature restarts.
- Types: HTTP GET, TCP socket, exec command, gRPC.

**Resources**
- **Requests:** Guaranteed amount used by the **scheduler** for placement.
- **Limits:** Maximum allowed. **CPU** over limit → **throttled** (CFS quota); **memory** over limit → **OOMKilled**.
- **QoS classes:** **Guaranteed** (requests = limits for all containers), **Burstable** (requests < limits or some set), **BestEffort** (none set). Under node memory pressure, eviction order: BestEffort → Burstable (exceeding requests) → Guaranteed.
- **LimitRange** (defaults/min/max per namespace) and **ResourceQuota** (total per namespace).
- **PodDisruptionBudget (PDB):** Minimum available (or max unavailable) pods during **voluntary disruptions** (node drains, upgrades, Karpenter consolidation).
- **Graceful shutdown:** `terminationGracePeriodSeconds`, `preStop` hooks.

### Interview Questions

**Q1. Liveness vs readiness vs startup probes — and common mistakes?**
Liveness restarts hung containers; readiness gates traffic; startup protects slow starters. Mistakes: liveness checks that call downstream dependencies (DB outage → all pods restart in a loop → cascading failure); identical liveness and readiness endpoints; too-aggressive timeouts causing restarts under load (making overload worse); no startup probe for slow JVM apps (killed before ready); readiness never failing during shutdown. Liveness should be cheap and local; readiness can reflect ability to serve.

**Q2. Should you set CPU limits?**
Debated. CPU limits cause CFS **throttling** even when the node has idle CPU, increasing latency (especially for multi-threaded runtimes). Many teams set CPU **requests** accurately (for scheduling and fair share under contention) and omit CPU limits for latency-sensitive services, while always setting **memory limits** (memory isn't compressible — avoid node OOM). Use limits where you need strict isolation (multi-tenant, batch) or Guaranteed QoS. Monitor throttling (`container_cpu_cfs_throttled_periods_total`).

**Q3. What are QoS classes and how do they affect eviction?**
Determined by requests/limits: Guaranteed (requests = limits for CPU and memory in every container), Burstable (at least one request/limit set but not Guaranteed), BestEffort (none). When a node is under memory pressure, the kubelet evicts BestEffort first, then Burstable pods using more than their requests, and Guaranteed last. Critical workloads should be Guaranteed or at least have memory requests close to usage, plus PriorityClasses.

**Q4. What is a PodDisruptionBudget and what happens without one?**
A PDB limits how many pods of an app can be voluntarily evicted at once (e.g. `minAvailable: 2` or `maxUnavailable: 1`). Node drains (upgrades, autoscaler consolidation) respect it via the eviction API. Without PDBs, draining nodes can evict all replicas simultaneously → outage. Caveat: PDBs don't protect against involuntary disruptions (node crash), and overly strict PDBs (`maxUnavailable: 0`) block node upgrades indefinitely.

**Q5. How do you right-size requests and limits?**
Observe actual usage (p95/p99 CPU and peak memory over representative periods) from Prometheus/metrics-server, use VPA in recommendation mode or tools like Goldilocks/Kubecost/StormForge, set requests near typical usage with headroom and memory limits above peak, revisit after releases, and enforce sane defaults with LimitRanges and policies. Over-requesting wastes money (bin-packing); under-requesting causes noisy neighbors and OOMs.

**Q6. How do you achieve graceful shutdown of a pod?**
On termination: pod is marked Terminating and removed from endpoints (asynchronously), `preStop` hook runs, then SIGTERM is sent; after `terminationGracePeriodSeconds` SIGKILL. Because endpoint removal propagates asynchronously, add a short `preStop` sleep (5–10 s) so load balancers stop sending new requests, handle SIGTERM in the app to finish in-flight requests and close connections, and set the grace period long enough for that.

---

## 51. Kubernetes Scheduling (Selectors, Affinity, Taints/Tolerations, Topology Spread, Priority)

**What they are:**
- **nodeSelector:** Simple label match (`disktype: ssd`).
- **Node affinity:** Expressive rules — `requiredDuringSchedulingIgnoredDuringExecution` (hard) or `preferred...` (soft).
- **Pod affinity / anti-affinity:** Place pods near (same node/zone) or away from other pods (e.g. spread replicas across nodes).
- **Taints & tolerations:** Taints on nodes **repel** pods unless they tolerate them (`NoSchedule`, `PreferNoSchedule`, `NoExecute` — evicts running pods). Used for dedicated nodes (GPU, system) and node problems (`not-ready`).
- **Topology spread constraints:** Evenly distribute pods across zones/nodes (`maxSkew`, `topologyKey`, `whenUnsatisfiable`).
- **PriorityClasses & preemption:** Higher-priority pods can evict lower-priority ones when resources are scarce (`system-cluster-critical`).
- Others: Descheduler (rebalances), resource bin-packing, scheduling gates, Dynamic Resource Allocation for devices.

### Interview Questions

**Q1. How do you ensure replicas of a service are spread across availability zones?**
Use `topologySpreadConstraints` with `topologyKey: topology.kubernetes.io/zone`, `maxSkew: 1`, and `whenUnsatisfiable: DoNotSchedule` (hard) or `ScheduleAnyway` (soft), plus a second constraint on `kubernetes.io/hostname` for node spread. Ensure node groups exist in each zone with capacity. Pod anti-affinity also works but is less flexible and more expensive to compute at scale.

**Q2. How do you dedicate GPU nodes only to GPU workloads?**
Taint GPU nodes (`nvidia.com/gpu=true:NoSchedule`) so regular pods aren't scheduled there; GPU pods add the matching toleration **and** a node selector/affinity (toleration allows but doesn't attract) and request `nvidia.com/gpu` resources. Autoscalers (Karpenter NodePools) create tainted GPU nodes on demand.

**Q3. Taints/tolerations vs node affinity?**
Taints are node-centric "keep out unless tolerated" — protect special nodes from general workloads. Node affinity is pod-centric "I want to go to these nodes". To dedicate nodes to specific pods, you typically need both: taint to keep others out, affinity to make those pods go there.

**Q4. What is preemption and how can it cause problems?**
When a high-priority pod can't be scheduled, the scheduler evicts lower-priority pods to make room. Useful to guarantee critical workloads, but misconfigured PriorityClasses (e.g. everything high priority, or batch jobs higher than services) can cause cascading evictions. Reserve high priority for truly critical system/business pods, and combine with PDBs and capacity planning.

**Q5. A pod is Pending with "0/10 nodes are available: 3 Insufficient cpu, 7 node(s) had untolerated taint". What do you do?**
The event explains the filter results: 3 nodes lack requested CPU (requests too high or cluster full — check requests vs allocatable, scale nodes or right-size), and 7 nodes have taints the pod doesn't tolerate (they're dedicated nodes — add tolerations if the pod belongs there, or add capacity to the general pool). Check autoscaler logs to see why it didn't scale up (max size, instance types, constraints).

---

## 52. Kubernetes Security (RBAC, ServiceAccounts, NetworkPolicies, Pod Security Standards, Admission Controllers)

**Key concepts**
- **Authentication:** Users via OIDC/cloud IAM (no built-in user objects), certificates; workloads via **ServiceAccounts** (projected, short-lived, audience-bound tokens).
- **RBAC:** **Role** (namespaced permissions) / **ClusterRole** (cluster-wide or reusable) bound via **RoleBinding** / **ClusterRoleBinding** to users, groups or ServiceAccounts. Verbs: get, list, watch, create, update, patch, delete, escalate, bind, impersonate.
- **NetworkPolicies:** Firewall rules for pod traffic (ingress/egress) using pod/namespace selectors and CIDRs; **default allow** until a policy selects a pod; enforced by the CNI. Best practice: default-deny per namespace, then allow explicitly.
- **Pod Security Standards (PSS):** Profiles **privileged**, **baseline**, **restricted**, enforced by the built-in **Pod Security Admission** via namespace labels (`pod-security.kubernetes.io/enforce: restricted`). (PodSecurityPolicy was removed in 1.25.)
- **securityContext:** `runAsNonRoot`, `readOnlyRootFilesystem`, `allowPrivilegeEscalation: false`, drop capabilities, `seccompProfile: RuntimeDefault`.
- **Admission controllers:** Mutating and validating webhooks (OPA Gatekeeper, Kyverno), built-in ValidatingAdmissionPolicy (CEL).
- **Other:** Secrets encryption, audit logs, API server private endpoint, image signature verification, runtime security, node hardening, CIS benchmarks (kube-bench).

### Interview Questions

**Q1. How do you implement least privilege with RBAC?**
Grant namespaced Roles rather than ClusterRoles where possible; bind to groups from the IdP (not individual users); avoid wildcards (`*` verbs/resources); be careful with dangerous permissions — `create pods` (can mount any secret/service account in the namespace), `get/list secrets`, `escalate`, `bind`, `impersonate`, `nodes/proxy`, and `create` on `pods/exec`; separate read-only and admin roles; give each workload its own ServiceAccount with only needed permissions (most need none — set `automountServiceAccountToken: false`); review with `kubectl auth can-i --list` and tools like rbac-lookup/KubiScan.

**Q2. How do NetworkPolicies work and what's a good baseline?**
Once any NetworkPolicy selects a pod for a direction (ingress/egress), only explicitly allowed traffic is permitted for that direction. Baseline: default-deny ingress and egress in each namespace, allow DNS egress to kube-dns, allow ingress from the ingress controller/gateway namespace to frontends, allow specific service-to-service paths by labels, allow egress to required external CIDRs. Requires a CNI that enforces policies (Calico, Cilium, VPC CNI with policy enabled). Cilium adds L7 and DNS-aware (FQDN) policies.

**Q3. What are Pod Security Standards and how do you roll them out?**
Three cumulative profiles: privileged (no restrictions), baseline (prevents known privilege escalations — no privileged, hostNetwork, hostPath), restricted (hardened — non-root, drop all capabilities, seccomp, no privilege escalation). Enforced by Pod Security Admission via namespace labels with modes `enforce`, `audit`, `warn`. Roll out by labeling namespaces with `warn`/`audit` restricted first, fixing workloads, then `enforce`; keep system namespaces at baseline/privileged as needed.

**Q4. How would an attacker escalate privileges in a Kubernetes cluster, and how do you prevent it?**
Common paths: compromised pod with a powerful ServiceAccount token (list secrets, create pods), privileged pods/hostPath mounts → node root → kubelet credentials, accessing cloud metadata service to steal node IAM credentials, exposed dashboards or kubelet APIs, overly permissive RBAC (`create pods` → mount privileged). Prevent with least-privilege RBAC, disable token automount, PSS restricted, block metadata access (IMDSv2 hop limit 1 / NetworkPolicy), workload identity per pod, admission policies, audit logs and runtime detection.

**Q5. What are admission controllers and give examples of policies you'd enforce?**
Plugins that intercept API requests after authn/authz, before persistence — **mutating** (modify objects: inject sidecars, add default labels/resources) and **validating** (accept/reject). Example policies: images only from approved registries and signed; no `:latest` tags; required resource requests/limits; required labels (team, cost-center); disallow privileged containers and hostPath; require probes; restrict Service type LoadBalancer; enforce Ingress hostnames per namespace.

**Q6. How do you secure ServiceAccount tokens?**
Modern clusters use projected, time-bound, audience-bound tokens (rotated automatically) instead of legacy long-lived secret tokens; disable automount where not needed; one ServiceAccount per workload; minimal RBAC; for cloud access use workload identity (IRSA/Pod Identity/GKE Workload Identity) rather than cloud keys in Secrets.

---

## 53. Kubernetes Extensibility (CRDs, Operators, Controllers & the Reconcile Loop)

**What it is:**
- **CustomResourceDefinition (CRD):** Extends the Kubernetes API with new resource types (`kind: PostgresCluster`) with OpenAPI schema validation, versions, status subresource.
- **Controller:** A loop watching resources and reconciling actual state toward desired state.
- **Operator:** CRD + controller encoding **domain-specific operational knowledge** (how to deploy, scale, back up, upgrade, fail over a database) — "software SRE". Examples: CloudNativePG, Strimzi (Kafka), Prometheus Operator, cert-manager, Argo CD, Crossplane, Elastic ECK.
- **Frameworks:** **Kubebuilder** / controller-runtime (Go), **Operator SDK** (Go/Ansible/Helm), kopf (Python), Metacontroller.
- **Key ideas:** Level-triggered reconciliation, idempotent reconcile functions, status conditions, finalizers (cleanup before deletion), owner references (garbage collection), leader election, requeue with backoff, informers/caching.
- **Aggregated API servers** — alternative extension for special cases.

### Interview Questions

**Q1. What is an Operator and when should you write one?**
An Operator automates the full lifecycle of a complex application using custom resources and a controller — e.g. `kind: KafkaCluster` and the controller handles provisioning, configuration, rolling upgrades, scaling, rebalancing and recovery. Write one when an application needs ongoing, stateful operational logic beyond what Deployments/Helm provide, and the logic is reused across many instances. Don't write one for simple stateless apps (Helm/Kustomize suffice) or if a mature community operator exists.

**Q2. Explain the reconcile loop and why it must be idempotent.**
The controller receives events (resource created/changed, or periodic resync), reads the desired spec and current world state, computes differences and performs actions, updates status, and requeues if needed. Because reconciliation can run many times for the same state (retries, resyncs, multiple events coalesced), it must be **idempotent** — running it again with no changes produces no side effects. It should not rely on event types (level-triggered, not edge-triggered).

**Q3. What are finalizers and owner references?**
**Finalizers** are keys on an object that block deletion until a controller performs cleanup (e.g. delete an external cloud resource) and removes the finalizer — avoids orphaned external resources. Stuck finalizers (controller gone) leave objects in Terminating forever. **Owner references** link dependent objects to their owner (ReplicaSet → Pods), enabling garbage collection when the owner is deleted.

**Q4. How do you version and evolve a CRD API?**
Use API versions (`v1alpha1` → `v1beta1` → `v1`), mark one storage version, provide **conversion webhooks** between versions when schemas differ, use structural schemas with validation (CEL rules), make additive, backward-compatible changes, deprecate fields gracefully, and migrate stored objects before removing old versions.

**Q5. What are risks of installing many third-party operators?**
Each operator runs with powerful cluster permissions (potential privilege escalation and supply-chain risk), adds CRDs and API server load (watches, many objects), can conflict with others, needs upgrades and compatibility testing on Kubernetes upgrades, and bugs can affect many resources. Vet operators (maturity, maintainers, RBAC scope), pin versions, and monitor them.

---

## 54. Kubernetes Troubleshooting

**What it is:** Systematic debugging of failing workloads using `kubectl describe`, `logs`, `exec`, `debug`, and **events**.

**Common states and causes**
| Symptom | Likely causes | Where to look |
|---|---|---|
| **Pending** | Insufficient resources, untolerated taints, affinity unsatisfiable, PVC unbound/zone mismatch, quota exceeded | `kubectl describe pod` events, autoscaler logs |
| **ImagePullBackOff / ErrImagePull** | Wrong image name/tag, private registry without imagePullSecret/IAM, rate limits, architecture mismatch | Pod events |
| **CrashLoopBackOff** | App crashes on start: bad config/env, missing secret, dependency unreachable, failing liveness probe, wrong command | `kubectl logs --previous`, describe (exit code) |
| **OOMKilled (exit 137)** | Memory limit too low, memory leak, JVM heap > container limit | describe `Last State`, metrics |
| **CreateContainerConfigError** | Missing ConfigMap/Secret key referenced | Events |
| **Running but not Ready** | Readiness probe failing | describe, probe endpoint |
| **Terminating forever** | Finalizers, node unreachable, volume detach | `kubectl get -o yaml` finalizers |
| **Evicted** | Node pressure (disk, memory) | Node conditions, events |
| **Service not reachable** | Selector/label mismatch, wrong targetPort, no ready endpoints, NetworkPolicy | EndpointSlices, `kubectl get svc -o wide` |

**Toolkit:** `kubectl get events --sort-by=.lastTimestamp`, `kubectl describe`, `kubectl logs -f [-c container] [--previous]`, `kubectl exec -it`, **`kubectl debug`** (ephemeral debug containers into distroless pods, or `kubectl debug node/x`), `kubectl top pods/nodes`, `kubectl get pod -o yaml`, `kubectl auth can-i`, `kubectl rollout status/history/undo`, k9s, stern (multi-pod logs), netshoot image.

### Interview Questions

**Q1. A pod is in CrashLoopBackOff. Walk through your debugging.**
`kubectl describe pod` → check events and **Last State**: exit code (1 = app error, 137 = OOMKilled/SIGKILL, 143 = SIGTERM, 127 = command not found), reason, restart count. `kubectl logs pod --previous` to see the crash output of the last run. Check config: env vars, mounted ConfigMaps/Secrets, command/args, image version change. Check liveness probe (too aggressive?). If logs are empty, override the command to `sleep` (or use `kubectl debug --copy-to` with a shell) to inspect the environment interactively. Check dependencies (DB reachable, DNS). Compare with the last working revision (`kubectl rollout history`).

**Q2. A pod is OOMKilled repeatedly. What do you do?**
Confirm via `Last State: OOMKilled` and exit 137. Check memory usage trends (Prometheus `container_memory_working_set_bytes`): steady growth = leak → profile heap; spike at startup/load = limit too low → raise limit/request. For JVMs ensure container-aware heap settings (`-XX:MaxRAMPercentage=75`) so heap + metaspace + native memory fit under the limit; for Node.js set `--max-old-space-size`. Also check node-level OOMs (pods without limits on overcommitted nodes).

**Q3. ImagePullBackOff — what are the causes?**
Typo in image name/tag, tag doesn't exist (not pushed yet / deleted by retention), private registry authentication missing (imagePullSecrets or node IAM permissions for ECR), Docker Hub rate limiting, network/egress blocked to registry (private cluster without NAT/VPC endpoint), or image architecture not matching node (arm64 vs amd64 → `exec format error` actually shows as crash). `kubectl describe pod` events show the exact error.

**Q4. How do you debug a distroless container with no shell?**
`kubectl debug -it pod/mypod --image=busybox --target=app` attaches an **ephemeral container** sharing the target's process namespace — you can inspect processes, `/proc/1/root` filesystem and network. Or `kubectl debug pod/mypod --copy-to=debug --image=...` to clone with a debug image, or use the image's `:debug` variant. For node issues, `kubectl debug node/name -it --image=ubuntu`.

**Q5. A deployment rollout is stuck. How do you investigate?**
`kubectl rollout status deploy/x` and `kubectl describe deploy/x` (conditions: `ProgressDeadlineExceeded`), check new ReplicaSet pods — are they Pending (capacity), CrashLooping (bad release), not Ready (readiness failing)? Check PDBs or quotas blocking, image pull errors. Roll back quickly with `kubectl rollout undo` (or GitOps revert) to restore service, then debug the new version.

**Q6. Pods are being evicted. Why?**
Node pressure: disk (ephemeral storage filled by logs, emptyDir, images), memory (usage over requests on overcommitted node), PID pressure. Check node conditions (`kubectl describe node`) and kubelet eviction thresholds. Fix by setting ephemeral-storage requests/limits, log rotation, image garbage collection, correct memory requests, and right-sized nodes. Also evictions occur from preemption by higher-priority pods or API-initiated drains.

---

## 55. Helm & Kustomize (Packaging)

**What they are:**
- **Helm:** The Kubernetes **package manager**. A **chart** contains templated manifests (Go templates), `values.yaml` defaults, `Chart.yaml` metadata and dependencies. `helm install/upgrade/rollback` manage **releases** with revision history stored in cluster secrets. Hooks (pre-install, pre-upgrade, post-upgrade, test), library charts, chart repositories and **OCI registries** (`helm push oci://...`), `helm template` to render, `helm diff` plugin. Helm 4 (2025) brought improvements such as server-side apply support and a modernized plugin system.
- **Kustomize:** Template-free customization built into kubectl (`kubectl apply -k`). A **base** of plain YAML plus **overlays** per environment that apply patches (strategic merge, JSON 6902), `namePrefix`, labels, `images` overrides, `configMapGenerator`/`secretGenerator` (hash suffix triggers rollouts), components.

### Interview Questions

**Q1. Helm vs Kustomize — when do you use each?**
Helm: distributing reusable, parameterized applications (third-party software like ingress controllers, Prometheus), packaging with versioned charts, dependencies and lifecycle hooks — good for "one chart, many consumers". Kustomize: managing your own manifests across environments with minimal differences, readable plain YAML, no templating language. Common pattern: use Helm charts for third-party apps and Kustomize (or Helm with environment values files) for internal apps; Argo CD supports both and even Kustomize-with-Helm.

**Q2. What are the drawbacks of Helm templating?**
Templates mix logic with YAML text, causing whitespace/indentation bugs, hard-to-read charts with many conditionals, values files that expose every field (leaky abstraction), weak validation (mitigate with `values.schema.json`), and drift between rendered output and expectations — always review rendered output (`helm template`, `helm diff`). Helm hooks also behave differently under GitOps tools.

**Q3. How does `helm rollback` work and what are its limits?**
Helm stores each release revision (rendered manifests and values) in a Secret; `helm rollback app 3` re-applies revision 3's manifests as a new revision. Limits: it doesn't roll back data or database migrations, CRD changes are not managed by upgrades/rollbacks the same way, and external side effects of hooks remain. In GitOps workflows, rollback by reverting Git instead.

**Q4. How do Kustomize overlays work?**
A `base/` directory holds common manifests; `overlays/prod/kustomization.yaml` references the base and applies patches (replica counts, resources, env vars), sets image tags (`images:` field), adds labels/annotations, and generates ConfigMaps. `kustomize build overlays/prod` outputs the final YAML. Changes are explicit patches, so diffs between environments are easy to review.

**Q5. How do you manage CRDs with Helm?**
Helm installs CRDs from the `crds/` directory only on first install and never upgrades or deletes them (to avoid data loss). Options: manage CRDs in a separate chart or templates (with care), install them via GitOps separately, or use the operator's recommended upgrade process. Always upgrade CRDs before controllers that need new fields.

**Q6. How do you store and distribute Helm charts?**
Push charts to **OCI registries** (ECR, GHCR, Artifactory, Harbor) — same auth and replication as images — version charts with SemVer, sign them (cosign) and reference by version/digest in GitOps configs. Chart museums/classic `index.yaml` repos are older alternatives.

---

## 56. Kubernetes Autoscaling (HPA, VPA, Cluster Autoscaler, Karpenter, KEDA)

**What they are:**
| Autoscaler | Scales | Based on |
|---|---|---|
| **HPA (Horizontal Pod Autoscaler)** | Number of pod replicas | CPU/memory utilization vs requests, custom metrics (RPS), external metrics; `behavior` controls scale up/down rates and stabilization windows |
| **VPA (Vertical Pod Autoscaler)** | Pod requests/limits | Historical usage; modes Off (recommend), Initial, Recreate/Auto; in-place pod resize is becoming available in newer Kubernetes versions |
| **Cluster Autoscaler** | Nodes (via node groups/ASGs) | Pending pods that can't schedule; removes underutilized nodes |
| **Karpenter** | Nodes (directly, just-in-time) | Pending pods' requirements → picks optimal instance types; consolidation; Spot handling; NodePools & NodeClasses (AWS, Azure; CNCF-sponsored project) |
| **KEDA (Kubernetes Event-Driven Autoscaling)** | Pods (via HPA), including **scale to zero** | Event sources: queue length (SQS, Kafka lag, RabbitMQ), Prometheus queries, cron, HTTP (add-on) — 60+ scalers |

### Interview Questions

**Q1. How does HPA work and what are its limitations?**
Every ~15 s the HPA controller fetches metrics (metrics-server for CPU/memory; custom/external metrics adapters like Prometheus Adapter or KEDA) and computes `desiredReplicas = ceil(currentReplicas × currentMetric / targetMetric)`, bounded by min/max and `behavior` policies. Limitations: depends on correct resource **requests** (utilization is relative to requests), reacts after load arrives (lag + pod startup time), CPU isn't always the right signal (I/O-bound or queue workers), can't scale to zero (without KEDA), and conflicts with VPA on the same metric.

**Q2. HPA vs VPA — can you use both?**
HPA scales out (more pods) — for stateless services with variable load. VPA right-sizes requests (bigger/smaller pods) — for workloads that can't scale horizontally or to discover correct sizing. Don't use both on the **same CPU/memory metric** (they fight). Common: VPA in recommendation mode to size requests + HPA on CPU or custom metrics; or HPA on custom metrics with VPA managing memory.

**Q3. Karpenter vs Cluster Autoscaler?**
Cluster Autoscaler adjusts the size of predefined node groups (each with fixed instance types), is slower and less cost-optimal. Karpenter provisions nodes directly from cloud APIs choosing among many instance types/sizes/architectures/purchase options to fit pending pods within seconds, consolidates by replacing/removing underutilized nodes, handles Spot interruptions and node expiry/drift for upgrades. Trade-offs: consolidation churn needs PDBs and disruption budgets; mainly AWS (Azure supported via AKS node autoprovisioning).

**Q4. What is KEDA and when would you use it?**
KEDA scales workloads based on **event sources** (e.g. SQS queue depth, Kafka consumer lag, Redis list length, Prometheus metrics, cron schedules) by feeding external metrics to HPA, and adds **scale to zero/from zero**. Ideal for queue consumers and event-driven processors where CPU isn't a good signal and idle cost matters. ScaledJobs can spawn Jobs per batch of events.

**Q5. Scaling is too slow during traffic spikes. How do you improve it?**
Pod level: scale on leading indicators (RPS/queue depth rather than CPU), lower HPA targets for headroom, tune `behavior` scale-up policies (allow 100% increase quickly), reduce pod startup time (smaller images, faster readiness, pre-pulled images). Node level: Karpenter (faster than CA), overprovisioning with low-priority placeholder pods that get preempted (instant capacity), warm pools, larger nodes. Plus scheduled scaling for predictable peaks and load shedding for protection.

**Q6. How do HPA scale-down settings prevent flapping?**
`behavior.scaleDown.stabilizationWindowSeconds` (default 300 s) uses the highest recommendation in the window, avoiding rapid down-up oscillations; policies limit how many pods/percent can be removed per period. Scale up is typically fast (0 s window) and scale down conservative.

---

## 57. Cilium & eBPF Networking

**What it is:** **eBPF** lets safe, sandboxed programs run inside the Linux kernel at hook points (network, syscalls, tracing) without kernel modules. **Cilium** (CNCF graduated, created by Isovalent — now part of Cisco) uses eBPF to provide Kubernetes networking, security and observability.

**Cilium features**
- **CNI** with high-performance eBPF dataplane (bypassing iptables), native routing or overlay.
- **kube-proxy replacement** — Service load balancing in eBPF (scales to many services, faster updates), Maglev consistent hashing, DSR.
- **Network policies:** Kubernetes NetworkPolicy plus CiliumNetworkPolicy with **L7 (HTTP/gRPC/Kafka) and DNS/FQDN-aware rules**, identity-based (labels → security identities instead of IPs).
- **Hubble:** Network observability — flow logs, service dependency maps, DNS/HTTP visibility, metrics.
- **Encryption:** Transparent WireGuard/IPsec node-to-node encryption.
- **Cilium Service Mesh:** Sidecar-less mesh features (L7 via per-node Envoy), Gateway API implementation, mTLS (mutual authentication).
- **Cluster Mesh:** Multi-cluster connectivity and global services.
- **Tetragon** (sibling project): runtime security.
- Used by default in GKE Dataplane V2, and available as an option on EKS/AKS (Azure CNI powered by Cilium).

### Interview Questions

**Q1. Why is eBPF considered a big shift for Kubernetes networking and observability?**
It moves packet processing and instrumentation into the kernel with programmable, safe hooks — no need for long iptables chains (which scale poorly and are slow to update with thousands of services), no kernel modules, and no sidecars for many features. It provides high-performance load balancing, identity-aware policy, and deep visibility (every flow, DNS query, HTTP request) with low overhead, plus security enforcement at the syscall level.

**Q2. How does Cilium's identity-based security differ from IP-based policies?**
Pod IPs are ephemeral; IP-based rules are brittle. Cilium assigns each set of pods with the same labels a numeric **security identity**, propagated with traffic (or resolved via maps), and policies are evaluated on identities — they keep working as pods churn and scale, across nodes and clusters.

**Q3. What is Hubble and how would you use it during an incident?**
Hubble exposes Cilium's flow data: you can see which pods talk to which, dropped packets with the policy verdict and reason, DNS queries and failures, HTTP status codes and latency between services, via CLI/UI/Prometheus metrics. In an incident ("service A can't reach B"), `hubble observe --from-pod A --to-pod B --verdict DROPPED` immediately shows if a NetworkPolicy blocks it.

**Q4. What are FQDN-based egress policies and why are they useful?**
Policies allowing egress to DNS names (`toFQDNs: matchName: api.stripe.com`) instead of IP CIDRs; Cilium intercepts DNS responses to learn the IPs dynamically. Useful for restricting pods to specific external SaaS APIs whose IPs change — a key data-exfiltration control.

**Q5. Cilium as kube-proxy replacement — benefits and risks?**
Benefits: O(1) eBPF map lookups vs iptables chains, faster Service updates at scale, better source IP preservation, DSR, lower latency and CPU. Risks: requires recent kernels, operational expertise in a newer stack, debugging differs from iptables, and migration must be planned (in-place migration of a running cluster's dataplane is delicate — often done with new node pools/clusters).

---

## 58. Service Mesh (Istio, Linkerd, Envoy) & Ingress Controllers

**What it is:** A **service mesh** is an infrastructure layer handling service-to-service communication: **mTLS** encryption and identity, traffic management (retries, timeouts, circuit breaking, traffic splitting, mirroring), and observability (golden metrics, traces) — without changing application code.

**Implementations**
- **Istio:** Most feature-rich. Control plane `istiod`; data plane historically **Envoy sidecars** per pod; **Ambient mode** (GA 2024) removes sidecars: a per-node **ztunnel** handles L4 (mTLS, identity) and optional per-namespace **waypoint proxies** (Envoy) handle L7 — lower resource usage and easier upgrades. CRDs: VirtualService, DestinationRule, Gateway (and Gateway API support), PeerAuthentication, AuthorizationPolicy.
- **Linkerd:** Simpler, lightweight Rust micro-proxy sidecars, strong mTLS defaults, easy to operate (stable releases now distributed by Buoyant).
- **Cilium Service Mesh:** eBPF + per-node Envoy, sidecar-less.
- **Envoy proxy:** High-performance L7 proxy (CNCF) — the data plane of Istio, many API gateways (Envoy Gateway, Gloo, Contour, Emissary) and cloud load balancers; dynamic configuration via xDS APIs.
- Others: Consul service mesh, Kuma, AWS App Mesh (being retired — AWS recommends VPC Lattice/ECS Service Connect).

**Ingress controllers / gateways:** **ingress-nginx** (community project — announced retirement with maintenance ending around March 2026; migrate to Gateway API implementations), **NGINX Inc. controller/NGINX Gateway Fabric**, **Traefik** (dynamic config, Let's Encrypt integration, also implements Gateway API), HAProxy, Kong, **Envoy Gateway**, **AWS Load Balancer Controller** (ALB/NLB per Ingress/Service), GKE/Azure application gateways.

### Interview Questions

**Q1. What problems does a service mesh solve, and when is it not worth it?**
It provides uniform mTLS and service identity (zero-trust), fine-grained traffic control (canary splits, retries, timeouts, circuit breaking, fault injection), and consistent telemetry across polyglot services — without library changes. Not worth it for small systems with few services, teams without capacity to operate it (it's complex: upgrades, debugging, resource overhead), or when simpler alternatives suffice (Cilium for mTLS/policies, Gateway API for edge routing, libraries for retries).

**Q2. Sidecar vs ambient (sidecar-less) mesh?**
Sidecars: a proxy per pod — strong isolation and full L7 features per workload, but CPU/memory overhead per pod, increased latency hops, operational pain (injecting, restarting pods to upgrade proxies, Job/sidecar lifecycle issues). Ambient: shared per-node L4 proxy (ztunnel) for mTLS and identity, plus optional L7 waypoint proxies only where needed — much lower overhead and upgrades without restarting apps; trade-off is a newer model and shared node component.

**Q3. How does mTLS work in Istio?**
istiod acts as a CA issuing short-lived X.509 certificates to each workload's proxy with a **SPIFFE identity** (`spiffe://cluster.local/ns/prod/sa/payments`) based on its ServiceAccount, rotating them automatically. Proxies establish mutual TLS between workloads transparently. `PeerAuthentication` sets STRICT mode (reject plaintext), and `AuthorizationPolicy` allows/denies based on source identity, namespace, paths, methods — enabling zero-trust service-to-service authorization.

**Q4. How would you implement a canary release with Istio?**
Define subsets in a DestinationRule by version label (`v1`, `v2`), and a VirtualService (or HTTPRoute) routing 95% to v1 and 5% to v2 (or based on headers for internal testers); progressively shift weights while monitoring mesh metrics (error rate, latency per version); automate with Argo Rollouts or Flagger which modify the routing and analyze Prometheus metrics, rolling back on failures.

**Q5. What is Envoy and why is it everywhere?**
Envoy is a high-performance, extensible L4/L7 proxy designed for cloud-native architectures: dynamic configuration via xDS APIs (no restarts), advanced load balancing, retries, circuit breaking, rate limiting, rich observability (stats, tracing), HTTP/2/gRPC/HTTP3 support, and filter extensibility (Lua, WASM). Its API-driven design made it the standard data plane for meshes and gateways.

**Q6. The community ingress-nginx is being retired. What's your migration plan?**
Inventory all Ingress resources and especially controller-specific annotations (rewrites, auth, rate limits, snippets); choose a replacement — ideally a **Gateway API** implementation (Envoy Gateway, Istio, Cilium, NGINX Gateway Fabric, Traefik, cloud gateway controllers); translate rules (tools like `ingress2gateway`); run the new controller in parallel with a separate load balancer, shift DNS/traffic gradually per host, validate metrics, then decommission. Also an opportunity to remove risky features like configuration snippets.

---

## 59. Managed Kubernetes & Alternatives (EKS, GKE, AKS, k3s/kind/minikube, ECS, Cloud Run)

**What they are:**
- **Managed Kubernetes:** **EKS** (AWS — control plane managed, Auto Mode for nodes, deep IAM/VPC integration), **GKE** (Google — most mature, **Autopilot** mode with per-pod billing and fully managed nodes, release channels), **AKS** (Azure — free control plane tier, Entra ID integration, node autoprovisioning). Provider manages control plane HA, upgrades, etcd; you manage workloads and (depending on mode) nodes.
- **Local/lightweight distributions:** **kind** (Kubernetes in Docker — great for CI testing), **minikube** (local single-node cluster with addons), **k3s** (lightweight certified Kubernetes by Rancher/SUSE — edge, IoT, small clusters), k3d (k3s in Docker), MicroK8s, Talos Linux (immutable Kubernetes OS), OpenShift (Red Hat enterprise platform), Rancher (multi-cluster management).
- **Simpler alternatives:** **AWS ECS/Fargate**, **Google Cloud Run** (serverless containers, scale to zero, Knative-based API), Azure Container Apps (KEDA + Dapr based), AWS App Runner, Fly.io/Render, Nomad (HashiCorp scheduler).

### Interview Questions

**Q1. Managed vs self-managed Kubernetes?**
Managed: provider runs the control plane (HA, etcd backups, patching, upgrades available on demand), integrates IAM/networking/load balancers — you focus on workloads. Self-managed (kubeadm, Rancher, OpenShift on-prem): needed for on-prem/edge/air-gapped environments, special compliance, or full control — but requires deep expertise and on-call for the control plane. For most cloud workloads, managed is the default.

**Q2. When would you choose Cloud Run/ECS instead of Kubernetes?**
When the team needs to run stateless containers without operating a platform: no cluster upgrades, node management or a huge ecosystem to learn; scale-to-zero (Cloud Run); per-request billing; small number of services; limited platform team capacity. Choose Kubernetes when you need its ecosystem (operators, GitOps at scale, service mesh, custom scheduling, GPUs at scale, portability across clouds/on-prem) and have the team to run it.

**Q3. kind vs minikube vs k3s?**
kind runs Kubernetes nodes as Docker containers — fast to create/destroy, multi-node, ideal for CI integration tests and testing controllers. minikube targets local developer use with addons (dashboard, ingress) and multiple drivers (VMs, Docker). k3s is a production-grade lightweight distribution (single binary, low memory) for edge, IoT, homelabs and small clusters; k3d runs it in Docker for local use.

**Q4. What is GKE Autopilot / EKS Auto Mode and their trade-offs?**
Fully managed node layers: the provider provisions, scales, patches and secures nodes; you pay per pod resources (Autopilot) or with a management fee (EKS Auto Mode). Pros: less ops, built-in best practices and security. Cons: restrictions (no privileged pods/host access in Autopilot, limited node customization, some DaemonSets/agents unsupported), possibly higher cost for well-optimized clusters, less control for special hardware.

**Q5. How do you plan Kubernetes upgrades on managed services?**
Track version support windows (each minor version ~14 months standard support; extended support costs extra), read deprecation/removal notes, scan manifests and Helm charts for removed APIs (pluto, kubent), upgrade non-prod first, upgrade control plane one minor version at a time, then add-ons (CNI, CoreDNS, CSI, ingress), then node pools (surge upgrades or Karpenter drift) protected by PDBs; or blue/green clusters for risky jumps. Automate and do it regularly — falling behind makes upgrades painful.

---

## 60. Multi-Cluster Management & Backup (Cluster API, Karmada, Velero)

**What it is:**
- **Why multiple clusters:** Isolation (blast radius, tenants, environments), regions/latency, compliance, scale limits, separate upgrade cadence.
- **Cluster API (CAPI):** Kubernetes-style declarative APIs to create, scale, upgrade and delete **clusters themselves** (`Cluster`, `MachineDeployment` CRDs) across providers (AWS, Azure, vSphere, bare metal) from a management cluster.
- **Karmada (CNCF):** Multi-cluster orchestration — propagate workloads to member clusters with placement and override policies, failover across clusters. (KubeFed is deprecated.)
- **Fleet management:** Argo CD ApplicationSets (cluster generator), Rancher Fleet, Open Cluster Management, GKE Fleet, Azure Kubernetes Fleet Manager, EKS dashboard.
- **Multi-cluster networking:** Cilium Cluster Mesh, Istio multi-cluster, Submariner, multi-cluster Services API, global load balancing (DNS/GSLB).
- **Velero:** Backup and restore of Kubernetes resources (API objects) and persistent volumes (snapshots or file-level via Kopia), scheduled backups, migration between clusters, disaster recovery.

### Interview Questions

**Q1. One large cluster vs many smaller clusters?**
One large cluster: better utilization and bin-packing, fewer control planes to manage, simpler service discovery; but larger blast radius (bad upgrade, CRD/controller bug, control-plane overload affects everyone), noisy neighbors, complex multi-tenancy (RBAC, quotas, policies), scale limits. Many clusters: strong isolation, independent upgrades, regional/compliance separation, but need fleet tooling (GitOps, policy, observability) and multi-cluster networking. Common: clusters per environment and region, sometimes per business unit or for sensitive workloads.

**Q2. How do you manage configuration consistently across 50 clusters?**
GitOps with Argo CD/Flux: a fleet repo defining cluster add-ons (CNI, ingress/gateway, monitoring agents, policies) applied to all clusters via **ApplicationSets with cluster generators** using labels (env, region) for targeting and overlays for differences; cluster lifecycle via Cluster API/Terraform; centralized policy (Kyverno) and observability; progressive rollout of platform changes across cluster waves (dev → canary prod cluster → all prod).

**Q3. What does Velero back up and what doesn't it cover?**
Velero backs up Kubernetes API objects (as JSON to object storage) and persistent volume data (via cloud snapshots or file-system backup with Kopia), with schedules, retention, namespace/label selection and restore hooks. It doesn't guarantee application-consistent database backups (use pre-backup hooks to freeze/flush or database-native backups), doesn't back up external cloud resources (RDS, S3) and doesn't replace GitOps (manifests should already be in Git). Test restores regularly.

**Q4. How do you design disaster recovery for Kubernetes workloads?**
Infrastructure as code for clusters (recreate quickly), GitOps for workloads (re-sync everything to a new cluster), data layer DR (managed databases with cross-region replicas, Velero/volume snapshots copied cross-region for in-cluster state), secrets in external managers replicated across regions, DNS/global load balancing for failover, and regular DR drills measuring RTO/RPO. Prefer stateless clusters and keep state in managed services.

**Q5. What is Cluster API and why use it?**
It applies Kubernetes' declarative model to cluster lifecycle: define a cluster as YAML (control plane, machine deployments, infrastructure provider), and controllers in a management cluster create and reconcile it — scaling, upgrading (rolling machine replacement) and repairing (MachineHealthChecks) clusters consistently across clouds and on-prem. Useful for platform teams managing many clusters, especially on-prem/vSphere/bare metal where no managed service exists.

---

# STAGE 9 — GITOPS & PROGRESSIVE DELIVERY

---

## 61. GitOps Principles (and Push vs Pull Deployments)

**What it is:** An operating model where **Git is the single source of truth** for the desired state of infrastructure and applications, and automated agents continuously **reconcile** the live system to match Git.

**OpenGitOps principles (CNCF)**
1. **Declarative:** Desired state expressed declaratively (Kubernetes YAML, Helm, Kustomize, Crossplane).
2. **Versioned and immutable:** Stored in Git with full history.
3. **Pulled automatically:** Agents pull desired state from the source.
4. **Continuously reconciled:** Agents observe actual state and correct **drift**.

**Push vs pull deployments**
| | Push (CI-driven) | Pull (GitOps) |
|---|---|---|
| How | CI pipeline runs `kubectl apply`/`helm upgrade` against the cluster | Agent inside the cluster (Argo CD/Flux) pulls from Git and applies |
| Credentials | CI needs cluster-admin-ish credentials (exposed outside cluster) | No inbound cluster credentials in CI; agent uses in-cluster permissions |
| Drift | Not detected until next deploy | Continuously detected and corrected |
| Audit & rollback | Pipeline logs | Git history; rollback = `git revert` |
| Multi-cluster | CI must reach every cluster | Each cluster pulls its own config (works behind firewalls) |

**Typical repo structure:** App source repo (code + Dockerfile) separate from a **config/environment repo** (manifests per environment); CI builds image and updates the image tag in the config repo via PR or bot commit (or Argo CD Image Updater / Flux image automation).

### Interview Questions

**Q1. What is GitOps and what are its benefits?**
Git holds the declarative desired state; agents continuously reconcile clusters to it. Benefits: every change is a reviewed, auditable commit; easy rollback via revert; drift detection and self-healing (manual `kubectl edit` gets reverted); no CI credentials to production clusters (better security); consistent multi-cluster management; disaster recovery by pointing a new cluster at the repo; developer-friendly workflow using PRs.

**Q2. Push-based vs pull-based deployment — which is more secure?**
Pull-based: the cluster agent pulls from Git/registry, so CI never holds cluster credentials and the cluster API need not be exposed to CI — reducing the attack surface (a compromised CI can't directly deploy to prod, only propose Git changes subject to review/branch protection). Push-based requires storing powerful kubeconfigs/tokens in CI. Push can still be secured (OIDC to cluster, scoped RBAC), and some teams use hybrid approaches.

**Q3. How do you promote a release from staging to production in GitOps?**
Environment directories/overlays (or branches — generally less recommended) in the config repo. CI updates the staging image tag; after verification, promotion is a PR copying the same tag/digest to the prod overlay (automated by tools like Kargo, Argo CD Image Updater, Flux image automation, or a pipeline step), reviewed and merged; Argo CD syncs prod. Approvals = PR reviews; audit = Git history.

**Q4. How do you handle secrets in a GitOps repo?**
Never commit plaintext secrets. Options: **External Secrets Operator** (commit an ExternalSecret reference; the operator fetches from Vault/AWS Secrets Manager) — most common; **Sealed Secrets** (encrypt with a cluster public key, only the controller can decrypt); **SOPS** (encrypt values with KMS/age keys, decrypted by Flux/Argo CD plugins); Secrets Store CSI driver. Prefer references to external managers so rotation doesn't require Git commits.

**Q5. What are challenges with GitOps?**
Secrets management, many repos/folders to manage at scale, ordering dependencies (CRDs before CRs, DB migrations before app — sync waves/hooks), imperative operations (one-off jobs, emergency changes), promotion workflows between environments, drift from autoscalers (ignore `replicas` managed by HPA), and Git becoming a bottleneck if CI commits too frequently. Tools (ApplicationSets, Kargo, sync waves) address many of these.

**Q6. Should you use branches or directories for environments?**
Directories (or Kustomize overlays/Helm values per environment) on a single main branch are generally recommended: diffs between environments are visible, no complex merges between long-lived branches, and promotions are explicit file changes. Branch-per-environment leads to merge conflicts, cherry-picking and drift between branches.

---

## 62. Argo CD

**What it is:** A declarative GitOps continuous delivery tool for Kubernetes (CNCF graduated). It watches Git repos (and Helm/OCI registries) and syncs manifests to clusters, with a powerful UI showing resource trees, health and diffs.

**Key concepts**
- **Application:** Maps a source (repo, path, revision; Helm/Kustomize/plain YAML/plugins) to a destination (cluster, namespace); multiple sources supported.
- **Sync status** (Synced/OutOfSync) and **health status** (Healthy/Progressing/Degraded — with custom health checks in Lua).
- **Sync policies:** Manual or **automated**, `prune` (delete resources removed from Git), `selfHeal` (revert manual drift), sync options (server-side apply, `CreateNamespace`), retry.
- **Sync waves & hooks:** `argocd.argoproj.io/sync-wave` annotations order resources (CRDs → operators → apps); hooks PreSync (migrations), Sync, PostSync (smoke tests), SyncFail.
- **App of Apps:** One root Application that creates other Applications — bootstrap clusters declaratively.
- **ApplicationSets:** Generate many Applications from templates using **generators** — list, cluster (all registered clusters with labels), Git directory/file, matrix/merge, pull request (preview environments), SCM provider.
- **Projects (AppProjects):** Multi-tenancy — restrict source repos, destination clusters/namespaces, allowed resource kinds, RBAC roles per team.
- **SSO** (OIDC/Dex), RBAC, notifications, **Image Updater**, multi-cluster management from a central instance (hub-spoke) or instance per cluster; **diffing customization** (`ignoreDifferences` for HPA-managed replicas, mutating webhooks).
- Related: **Kargo** (promotion orchestration across stages from the Argo creators — Akuity).

### Interview Questions

**Q1. Explain the App of Apps pattern vs ApplicationSets.**
App of Apps: a parent Application points to a directory of child Application manifests — good for bootstrapping a cluster's set of add-ons and apps declaratively. ApplicationSets: a controller generates Applications from a template plus generators — e.g. deploy the monitoring stack to **every cluster** labeled `env=prod`, one Application per directory in a monorepo, or a preview environment per open PR. ApplicationSets scale better for fleets and dynamic sets; both can be combined.

**Q2. How do you order deployments with dependencies in Argo CD?**
**Sync waves** (lower numbers sync first; Argo CD waits for health before the next wave): e.g. wave -1 namespaces and CRDs, wave 0 operators/configs, wave 1 database migration Job (or PreSync hook), wave 2 application. **Hooks** for PreSync tasks like migrations and PostSync smoke tests. Custom health checks ensure CRD-based resources report readiness correctly.

**Q3. What do `prune` and `selfHeal` do, and what are the risks?**
`prune: true` deletes cluster resources that were removed from Git — keeps cluster clean, but a mistaken deletion in Git (or wrong path) can delete production resources; protect critical resources with `Prune=false` annotations and review. `selfHeal: true` reverts manual changes made directly in the cluster — enforces Git as source of truth, but during incidents you must make emergency changes through Git (or temporarily disable auto-sync), so ensure fast PR paths for hotfixes.

**Q4. How do you implement multi-tenancy in Argo CD?**
**AppProjects** per team restricting allowed source repos, destination clusters/namespaces, cluster-scoped resource kinds (deny ClusterRoles), and sync windows; RBAC policies mapping SSO groups to project roles (team A can sync its apps only); apps-in-any-namespace feature for teams managing their own Application objects; separate Argo CD instances for strong isolation where needed.

**Q5. How do you prevent Argo CD from showing OutOfSync for fields changed by controllers (e.g. HPA replicas)?**
Remove `replicas` from the Deployment manifest when HPA manages it, or configure `ignoreDifferences` for `/spec/replicas` (and for fields mutated by webhooks — injected sidecars, CA bundles); use server-side diff/apply so managed fields are handled properly; `RespectIgnoreDifferences=true` sync option.

**Q6. Argo CD hub-and-spoke vs instance per cluster?**
Central (hub) instance managing many clusters: single pane of glass, simpler management, but a single point of failure, needs network access and credentials to all clusters, and scaling the controller (sharding) at large fleets. Instance per cluster: isolation and works behind firewalls, but more instances to operate and no unified view (mitigated by tools like Akuity platform/agent-based models). Choose by fleet size, network topology and security boundaries.

---

## 63. Flux

**What it is:** A CNCF graduated GitOps toolkit for Kubernetes — a set of composable controllers (the **GitOps Toolkit**), CLI-first with no built-in UI (UIs available via third parties like Weave GitOps OSS, Headlamp plugin, or Capacitor). After Weaveworks shut down in early 2024, Flux continued as a community-maintained CNCF project with support from vendors (e.g. ControlPlane).

**Controllers**
- **source-controller:** Fetches artifacts from GitRepository, OCIRepository, HelmRepository, Bucket sources.
- **kustomize-controller:** Applies `Kustomization` resources (path in a source) with pruning, health checks, dependencies (`dependsOn`), SOPS decryption, post-build variable substitution.
- **helm-controller:** Manages `HelmRelease` resources declaratively (install, upgrade, test, rollback, drift detection).
- **notification-controller:** Alerts to Slack/Teams/etc. and receives webhooks.
- **image-reflector & image-automation controllers:** Scan registries for new tags and commit updated tags back to Git.
- `flux bootstrap` installs Flux and stores its own config in Git (self-managing).

### Interview Questions

**Q1. Argo CD vs Flux?**
Argo CD: rich UI with resource trees and diffs, Application/ApplicationSet abstractions, strong multi-tenancy via AppProjects and SSO RBAC, hub-and-spoke multi-cluster, larger adoption. Flux: lightweight, modular controllers, Kubernetes-native CRDs (everything is a CR — no separate API server/UI), strong Helm controller, native SOPS decryption, OCI artifact sources, image automation built in, natural per-cluster pull model. Choose Argo CD for UI-driven visibility and centralized management; Flux for a composable, CLI/CRD-centric approach and edge/many-cluster pull setups.

**Q2. How does Flux handle dependencies between components?**
`Kustomization` and `HelmRelease` objects support `dependsOn`, so infrastructure (CRDs, cert-manager, ingress) is reconciled and healthy before apps that depend on it; health checks (`wait: true`, `healthChecks`) gate readiness; intervals and retries control reconciliation frequency.

**Q3. How does Flux image automation work?**
`ImageRepository` scans a registry for tags, `ImagePolicy` selects the latest tag matching a policy (SemVer range, regex, alphabetical/numerical order), and `ImageUpdateAutomation` commits the new tag to Git at marker comments in manifests (`# {"$imagepolicy": "flux-system:app"}`). Git remains the source of truth while automating updates — usually for dev/staging; prod via reviewed PRs.

**Q4. How does Flux decrypt SOPS-encrypted secrets?**
Secrets are encrypted in Git with SOPS (values only, keys visible) using KMS (AWS/GCP/Azure), age or PGP keys. The kustomize-controller is configured with a `decryption` provider and access to the key (e.g. via workload identity for KMS or an age private key Secret) and decrypts at apply time — plaintext never stored in Git.

**Q5. What does it mean that Flux supports OCI artifacts as sources?**
Instead of pulling manifests from Git, Flux can pull versioned bundles of manifests pushed to an OCI registry (`flux push artifact`), which can be signed with cosign and verified by Flux. Benefits: immutable versioned deployment artifacts, use of existing registry infrastructure/replication, and decoupling clusters from Git availability and credentials.

---

## 64. Progressive Delivery (Argo Rollouts, Flagger)

**What it is:** Gradually exposing a new version to users while **automatically analyzing metrics** and promoting or rolling back — combining deployment strategies with automated verification.

**Argo Rollouts**
- A controller and `Rollout` CRD (drop-in replacement for Deployment) supporting **canary** (steps: setWeight, pause, analysis) and **blue/green** (active and preview Services, promotion).
- **Traffic management** integrations: Istio, Linkerd (SMI), NGINX, ALB, Gateway API plugin, Traefik, Ambassador, Apisix — precise traffic weights; without a mesh it approximates weights by replica ratio.
- **AnalysisTemplates/AnalysisRuns** query Prometheus, Datadog, New Relic, CloudWatch, Kayenta, web hooks or Jobs; failed analysis → automatic abort and rollback.
- Experiments, header-based routing, anti-affinity, kubectl plugin & dashboard.

**Flagger** (Flux project): Works with existing Deployments, creates canary/primary copies, integrates with service meshes and ingress controllers, automated canary analysis with Prometheus metrics, A/B testing (header/cookie routing), blue/green, webhooks for load testing/acceptance tests.

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Rollout
spec:
  strategy:
    canary:
      steps:
        - setWeight: 10
        - pause: { duration: 5m }
        - analysis: { templates: [ { templateName: success-rate } ] }
        - setWeight: 50
        - pause: { duration: 10m }
```

### Interview Questions

**Q1. What is progressive delivery and how is it different from continuous delivery?**
Continuous delivery ensures every change is deployable and deployed via automated pipelines. Progressive delivery adds **controlled exposure**: release to a small percentage or segment first, measure real-world health (errors, latency, business metrics), and automatically expand or roll back — reducing blast radius. It combines canaries/blue-green, feature flags and automated analysis.

**Q2. How does Argo Rollouts perform an automated canary?**
On a new image, the Rollout controller creates a canary ReplicaSet and executes steps: shifts e.g. 10% of traffic via the configured traffic router (mesh/ingress/ALB), pauses, runs an AnalysisRun querying metrics (e.g. success rate ≥ 99%, p99 latency < threshold) at intervals; if all measurements pass, it proceeds to the next weight, finally promoting the canary to stable; if analysis fails, it aborts — shifting traffic back to stable and scaling down the canary.

**Q3. Argo Rollouts vs Flagger?**
Argo Rollouts replaces Deployments with a `Rollout` resource, offers fine-grained step definitions, rich UI/kubectl plugin, and integrates naturally with Argo CD. Flagger keeps your existing Deployment and manages generated primary/canary deployments, configured via a `Canary` resource; integrates with Flux and many meshes/ingresses. Both are capable — choose based on GitOps tool and preference for resource model.

**Q4. What makes a good canary analysis metric?**
Metrics that reflect user impact and differentiate canary vs stable: request success rate (non-5xx), latency percentiles, saturation, plus business KPIs where possible (orders completed); compared against the stable baseline over enough traffic to be statistically meaningful; with failure limits and consecutive-failure thresholds to avoid noise. Low-traffic services need longer analysis windows or synthetic traffic.

**Q5. How do you handle database changes with canary releases?**
Both versions run simultaneously and share the database, so schema changes must be backward- and forward-compatible (expand/contract): apply expand migrations before the canary, ensure the new version tolerates old data and the old version tolerates new data, and contract only after full rollout. Otherwise rollback breaks.

---

# STAGE 10 — CONFIGURATION & SECRETS MANAGEMENT

---

## 65. HashiCorp Vault & OpenBao

**What it is:** **Vault** is a secrets management and data protection system: centralized storage of secrets, **dynamic secrets**, encryption as a service, PKI, identity-based access with fine-grained policies and audit logging. After HashiCorp's 2023 license change to BSL, the Linux Foundation forked it as **OpenBao** (open source, API-compatible).

**Key concepts**
- **Secrets engines:** KV (versioned static secrets), **database** (generates short-lived DB credentials on demand), AWS/GCP/Azure (dynamic cloud credentials), **PKI** (issue X.509 certificates), **Transit** (encrypt/decrypt/sign data without exposing keys — encryption as a service), SSH, TOTP.
- **Auth methods:** Kubernetes (ServiceAccount tokens), AWS IAM, OIDC/JWT (incl. CI OIDC tokens), AppRole, LDAP, certificates.
- **Policies:** HCL path-based ACLs (`path "secret/data/app/*" { capabilities = ["read"] }`).
- **Leases, TTLs, renewal and revocation:** Every dynamic secret has a lease; revoke individually or by prefix (incident response).
- **Seal/unseal:** Vault starts sealed; the master key is split (Shamir) or **auto-unsealed** via cloud KMS/HSM.
- **HA & storage:** Integrated storage (Raft), performance/DR replication (Enterprise), namespaces (Enterprise).
- **Kubernetes integration:** Vault Agent Injector (sidecar writes secrets to files), Vault Secrets Operator, CSI provider, External Secrets Operator.
- **Audit devices:** Log every request.

### Interview Questions

**Q1. What are dynamic secrets and why are they better than static secrets?**
Dynamic secrets are generated on demand per client with a short TTL — e.g. Vault creates a unique PostgreSQL user for each app instance valid for 1 hour and revokes it automatically on expiry. Benefits: no long-lived shared passwords, a leaked credential expires quickly, every credential is attributable to a specific client (audit), and revocation is targeted (revoke one lease or all leases for a compromised service). Static secrets require manual rotation and are often shared widely.

**Q2. How does an application on Kubernetes authenticate to Vault?**
Using the **Kubernetes auth method**: the pod presents its ServiceAccount JWT; Vault validates it with the Kubernetes TokenReview API (or JWT validation with the cluster's OIDC issuer), maps the ServiceAccount/namespace to a Vault role and policies, and returns a Vault token with a TTL. The app (or Vault Agent/Secrets Operator/ESO) uses the token to read secrets. No secret zero needs to be distributed manually.

**Q3. What is the Transit secrets engine used for?**
Encryption as a service: applications send plaintext to Vault and receive ciphertext (and vice versa) without ever handling the encryption keys; keys can be rotated (`vault:v2:` prefixed ciphertexts, rewrap without exposing plaintext), support for signing, HMAC and convergent encryption. Useful for encrypting PII fields in databases with centralized key management and audit.

**Q4. What is the "secret zero" problem and how is it solved?**
To fetch secrets, an application needs an initial credential to authenticate to the secret manager — where does that come from securely? Solved by **platform identity**: Kubernetes ServiceAccount tokens, cloud IAM identities (AWS IAM auth, instance identity documents), CI OIDC tokens, SPIFFE identities — credentials issued by the platform to the workload automatically, not distributed by humans.

**Q5. How do you run Vault in production reliably?**
HA cluster (3–5 nodes with integrated Raft storage across AZs), auto-unseal with cloud KMS/HSM, TLS everywhere, audit devices enabled (to two destinations — Vault blocks requests if no audit device can log), regular snapshots, DR replication (Enterprise) or tested restore procedures, least-privilege policies, root token revoked after setup, monitoring (seal status, leader, request latency, lease counts). Or use a managed offering (HCP Vault) or cloud-native secret managers if Vault's features aren't needed.

**Q6. Vault vs cloud secret managers (AWS Secrets Manager, Azure Key Vault, GCP Secret Manager)?**
Cloud managers: fully managed, IAM integrated, simple, cheap at small scale, rotation for native databases — ideal for single-cloud. Vault/OpenBao: multi-cloud and on-prem, dynamic secrets for many systems, PKI, transit encryption, rich policy/auth methods — but requires operating a critical system (or paying for HCP). Many orgs use cloud managers for cloud-native apps and Vault where advanced capabilities or hybrid environments demand it.

---

## 66. Kubernetes Secrets Integration & Cloud Secret Stores (External Secrets Operator, Secrets Store CSI, Sealed Secrets, SOPS)

**What they are:**
| Tool | How it works | Best for |
|---|---|---|
| **External Secrets Operator (ESO)** ★ | `ExternalSecret` CR references a key in an external store (AWS Secrets Manager, SSM, Vault, Azure Key Vault, GCP Secret Manager, 1Password...) via a `SecretStore`/`ClusterSecretStore`; the operator syncs it into a native Kubernetes Secret and refreshes periodically | GitOps-friendly, apps consume normal Secrets |
| **Secrets Store CSI Driver** | Mounts secrets from external stores as files via a CSI volume at pod start (optionally syncs to K8s Secrets); providers for AWS, Azure, GCP, Vault | Avoiding secrets in etcd; file-based consumption |
| **Sealed Secrets** (Bitnami) | Encrypt a Secret with the controller's public key (`kubeseal`) → commit `SealedSecret` to Git; only the in-cluster controller can decrypt | Simple GitOps without external store |
| **SOPS** (CNCF) + age/KMS | Encrypts values in YAML/JSON files with KMS/age/PGP keys; decrypted by Flux, Argo CD plugins, Helm secrets, or CI | Encrypted config files in Git |
| **Cloud secret stores** | AWS Secrets Manager/SSM Parameter Store, Azure Key Vault, GCP Secret Manager | Source of truth with rotation and IAM |

### Interview Questions

**Q1. External Secrets Operator vs Secrets Store CSI Driver?**
ESO syncs external secrets into native Kubernetes Secrets — apps use standard env vars/volumes, works for any resource needing a Secret (Ingress TLS, image pull secrets), with periodic refresh; but secrets live in etcd (encrypt etcd). CSI driver mounts secrets directly as files into pods at start without necessarily creating Kubernetes Secrets (reduced exposure), rotation via remount/poll; but requires pods to be running to fetch, and env var consumption needs sync anyway. ESO is more popular for GitOps; CSI for minimizing secret copies.

**Q2. Sealed Secrets vs SOPS vs ESO?**
Sealed Secrets: encrypted secrets live in Git, tied to a cluster's key pair (key backup and per-cluster re-sealing needed), no external dependency. SOPS: encrypted files in Git with KMS/age keys — works beyond Kubernetes (Terraform, Helm values), flexible key management. ESO: Git holds only references; the secret lives in a managed store with rotation, auditing and access control — scales best for large organizations. Prefer ESO with a managed store for production; SOPS/Sealed Secrets for simpler setups.

**Q3. How do you rotate a database password used by pods without downtime?**
Rotate in the secret manager (e.g. Secrets Manager rotation with the alternating-users strategy so old credentials stay valid during transition), ESO refreshes the Kubernetes Secret on its refresh interval, and pods either reload the mounted file dynamically (app watches the file) or are restarted via a rolling update triggered by a hash annotation (Reloader controller). Better still: dynamic short-lived credentials (Vault database engine) or IAM database authentication.

**Q4. How do you prevent one team from reading another team's secrets via ESO?**
Use namespaced `SecretStore`s with credentials/roles scoped to that team's secret paths (e.g. IAM role allowed only `arn:...:secret:team-a/*`), restrict `ClusterSecretStore` usage with conditions on namespaces, RBAC on ExternalSecret creation, and policies (Kyverno) preventing references to other teams' paths.

**Q5. Why is storing secrets in environment variables considered risky?**
Env vars can leak through process listings, crash dumps, debug endpoints, logs (apps printing their environment), child processes inheriting them, and `kubectl describe`/inspect outputs. Mounted files with restricted permissions or fetching from a secret manager at runtime reduce exposure. Env vars are still common for convenience — at least avoid logging them and limit access.

---

## 67. Certificates (cert-manager, Let's Encrypt, ACM)

**What it is:** Automated issuance and renewal of TLS certificates — expired certificates are a classic cause of outages.
- **cert-manager (CNCF graduated):** Kubernetes controller issuing certificates from **Issuers/ClusterIssuers** (ACME/Let's Encrypt, Vault, private CAs, AWS Private CA, Venafi, self-signed) via `Certificate` CRDs, storing them in Secrets and renewing automatically before expiry; annotations on Ingress/Gateway create certificates automatically. Also trust-manager (distribute CA bundles) and csi-driver-spiffe.
- **Let's Encrypt:** Free, automated public CA using the **ACME** protocol with challenges — **HTTP-01** (serve token at `/.well-known/acme-challenge/`), **DNS-01** (TXT record — required for wildcards), TLS-ALPN-01. Certificates are short-lived (90 days, with shorter lifetimes being introduced).
- **AWS Certificate Manager (ACM):** Free public certificates for AWS-integrated services (ALB, CloudFront, API Gateway) with automatic renewal; private CA for internal PKI; exportable public certificates are now available for a fee.
- **Industry trend:** CA/Browser Forum decided to reduce maximum public TLS certificate validity progressively to 47 days by 2029 — automation is mandatory.

### Interview Questions

**Q1. How does cert-manager obtain a Let's Encrypt certificate?**
A `Certificate` resource (or Ingress annotation) references a ClusterIssuer configured for ACME. cert-manager creates an Order and Challenge: for HTTP-01 it creates a temporary solver pod/Ingress route serving the token; for DNS-01 it creates a TXT record via the DNS provider API (Route 53, Cloudflare). Let's Encrypt validates domain control, issues the certificate; cert-manager stores it in a Secret and renews ~30 days before expiry.

**Q2. HTTP-01 vs DNS-01 challenges?**
HTTP-01: simple, requires the domain to be publicly reachable on port 80 to the cluster — doesn't support wildcards. DNS-01: proves control via DNS TXT records — works for wildcard certificates and internal services not reachable from the internet, but requires DNS API credentials (scope them tightly).

**Q3. How do you prevent outages from expired certificates?**
Automate issuance and renewal (cert-manager, ACM, ACME clients), monitor expiry (cert-manager metrics `certmanager_certificate_expiration_timestamp_seconds`, blackbox exporter TLS probes, synthetic monitoring) with alerts weeks in advance, inventory all certificates (including on appliances, third-party services, internal CAs, client certs), and avoid manually installed certificates. Shorter certificate lifetimes make automation non-negotiable.

**Q4. How do you manage internal (private) certificates for service-to-service TLS?**
Use a private CA (Vault PKI, AWS Private CA, cert-manager CA issuer, or service mesh CA like istiod) with short-lived certificates and automatic rotation, distribute CA trust bundles (trust-manager), and use workload identities (SPIFFE) for mTLS. Avoid long-lived self-signed certs scattered across services.

**Q5. Why does ACM require certificates for CloudFront in us-east-1?**
CloudFront is a global service whose configuration is managed from us-east-1, so ACM certificates attached to distributions must be requested/imported in that region; regional services (ALB) need certificates in their own region.

---

## 68. Workload Identity (SPIFFE/SPIRE, IRSA/Pod Identity, GCP & Azure Workload Identity)

**What it is:** Giving **workloads (not humans)** cryptographically verifiable identities so they can authenticate to other services and clouds without static secrets.
- **SPIFFE** (Secure Production Identity Framework for Everyone, CNCF graduated): Standard for workload identity — **SPIFFE IDs** (`spiffe://trust-domain/ns/prod/sa/payments`) and **SVIDs** (X.509 certificates or JWTs) issued via the Workload API.
- **SPIRE:** Reference implementation — server and node agents perform **attestation** (node attestation via cloud instance identity, workload attestation via Kubernetes pod info/Unix process) before issuing short-lived SVIDs; federation across trust domains. Service meshes use SPIFFE-compatible identities.
- **Cloud workload identity for Kubernetes:** **AWS IRSA** (OIDC-based) and **EKS Pod Identity**; **GKE Workload Identity Federation** (KSA ↔ Google service account/principal); **Azure Workload Identity** (federated credentials with Entra ID).
- **Workload identity federation** generally: exchange an external OIDC token (CI, other cloud, SPIFFE JWT) for cloud credentials.

### Interview Questions

**Q1. Why is workload identity better than API keys or static credentials?**
Identities are issued automatically to verified workloads (attestation), credentials are short-lived and auto-rotated, nothing to store in config or code, permissions bound to the identity (least privilege per service), and access is auditable per workload. Leaked short-lived tokens expire quickly. It removes the "secret zero" problem.

**Q2. How does GKE/Azure workload identity work conceptually?**
The cluster acts as an OIDC issuer for ServiceAccount tokens. The cloud IAM is configured to trust that issuer and map a specific Kubernetes ServiceAccount (namespace/name) to a cloud identity (Google service account or principal; Azure managed identity/app registration with federated credential). Pods using that ServiceAccount get a projected token that the cloud SDK exchanges for cloud access tokens — same pattern as AWS IRSA.

**Q3. What is attestation in SPIRE?**
Proving a workload is what it claims before issuing an identity. **Node attestation** verifies the node (e.g. AWS instance identity document, Kubernetes PSAT tokens); **workload attestation** on the node agent inspects the calling process (Kubernetes namespace, ServiceAccount, pod labels, container image, Unix UID) and matches registration entries to decide which SPIFFE ID to issue. Identity is based on verifiable properties, not secrets.

**Q4. How would you enable secure communication between services running in different clouds?**
Use a common identity framework — SPIFFE/SPIRE with federation between trust domains, or a service mesh with multi-cluster trust — for mTLS between services; and workload identity federation to let a workload in cloud A obtain credentials in cloud B by exchanging its OIDC token (no static keys). Combine with network connectivity (VPN/interconnect) and authorization policies.

**Q5. IRSA vs EKS Pod Identity — quick comparison?**
Both give pods IAM roles. IRSA uses each cluster's OIDC provider in IAM trust policies (one trust entry per cluster — cumbersome across many clusters, IAM OIDC provider limits). Pod Identity uses an EKS agent and association API; roles trust the EKS service principal so the same role works across clusters without trust policy edits, and it supports session tags. Prefer Pod Identity for new EKS setups.

---

# STAGE 11 — POLICY AS CODE & GOVERNANCE

---

## 69. Policy as Code (OPA/Gatekeeper, Kyverno, Conftest, ValidatingAdmissionPolicy)

**What it is:** Expressing organizational rules (security, compliance, best practices) as **versioned, testable code** enforced automatically — in CI (shift-left) and at runtime (admission control).

**Tools**
- **OPA (Open Policy Agent, CNCF graduated):** General-purpose policy engine using the **Rego** language; evaluates JSON input → decisions. Used for Kubernetes, Terraform plans, API authorization, CI.
- **Gatekeeper:** OPA-based Kubernetes admission controller with `ConstraintTemplates` (Rego) and `Constraints` (parameters), audit of existing resources, mutation.
- **Kyverno (CNCF):** Kubernetes-native policy engine with policies written in **YAML** (plus CEL) — validate, **mutate**, **generate** (e.g. create default NetworkPolicy per namespace), **verifyImages** (cosign signatures/attestations), cleanup policies; Kyverno CLI for CI; policy reports.
- **Conftest:** Runs OPA/Rego policies against structured config files (Kubernetes YAML, Terraform plan JSON, Dockerfiles) in CI.
- **ValidatingAdmissionPolicy (VAP, GA in Kubernetes 1.30):** Built-in, in-process validation using **CEL** expressions — no webhook to run; MutatingAdmissionPolicy is following.
- **Typical policies:** No `:latest` tags, required labels (owner, cost-center), required resource requests/limits, only signed images from approved registries, no privileged pods/hostPath, required probes, restrict LoadBalancer services, enforce Ingress hostnames.

### Interview Questions

**Q1. OPA Gatekeeper vs Kyverno?**
Gatekeeper: uses Rego (powerful, general-purpose, reusable across non-Kubernetes domains like Terraform and APIs) but has a learning curve; validation-centric with mutation support. Kyverno: Kubernetes-native YAML policies (easier for platform teams), strong mutation and generation, built-in image verification, policy exceptions and reports. Choose Kyverno for Kubernetes-focused teams wanting simplicity; OPA/Gatekeeper when Rego is used org-wide. VAP/CEL reduces the need for webhooks for simple validations.

**Q2. How do you roll out a new admission policy without breaking deployments?**
Start in **audit/warn mode** (Kyverno `validationFailureAction: Audit`, Gatekeeper `enforcementAction: dryrun/warn`), review policy reports to find violations, notify owners and fix workloads, provide documented exceptions (Kyverno PolicyExceptions) with expiry, run the same policies in CI (Kyverno CLI/Conftest) so developers see violations early, then switch to enforce per namespace/environment progressively. Exclude system namespaces carefully.

**Q3. Why run policies in CI as well as at admission?**
CI gives developers fast feedback in the PR (shift-left) before anything reaches a cluster; admission is the last line of defense that catches everything (including manual applies and other pipelines). Using the same policy source in both avoids drift between "what CI checks" and "what the cluster enforces".

**Q4. What is ValidatingAdmissionPolicy and why does it matter?**
A native Kubernetes API for validation rules written in CEL, evaluated inside the API server — no external webhook service to run, scale or keep highly available (webhook outages can block deployments cluster-wide). Good for simple, common rules; complex policies (image verification, generation, external data) still need Kyverno/Gatekeeper.

**Q5. What are the risks of admission webhooks and how do you mitigate them?**
If a webhook is down or slow and has `failurePolicy: Fail`, all matching API requests fail — potentially blocking deployments, scaling or even system components. Mitigate: run webhooks highly available (multiple replicas, PDBs, priority), exclude critical system namespaces, set short timeouts, monitor webhook latency, consider `Ignore` failure policy for non-security policies, and use in-process VAP where possible.

**Q6. Give an example Kyverno policy.**
Require resource limits:
```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata: { name: require-limits }
spec:
  validationFailureAction: Enforce
  rules:
    - name: check-limits
      match: { any: [ { resources: { kinds: [Pod] } } ] }
      validate:
        message: "CPU and memory requests and memory limits are required."
        pattern:
          spec:
            containers:
              - resources:
                  requests: { memory: "?*", cpu: "?*" }
                  limits: { memory: "?*" }
```

---

## 70. Compliance (CIS Benchmarks, kube-bench, Audit Logging, SOC 2/ISO Evidence Automation)

**What it is:** Demonstrating that systems meet security standards and regulatory/contractual requirements — continuously, with automated evidence.
- **CIS Benchmarks:** Consensus security configuration guides for OSes, Docker, Kubernetes, cloud providers.
- **kube-bench (Aqua):** Checks Kubernetes nodes/control plane against the CIS Kubernetes Benchmark (managed services have their own variants — e.g. CIS EKS benchmark). Related: kube-hunter (deprecated), Trivy Kubernetes compliance scans (NSA/CISA, CIS), Kubescape (frameworks like NSA, MITRE).
- **Audit logging:** Kubernetes API audit logs (who did what — policy levels Metadata/Request/RequestResponse), cloud audit logs (CloudTrail, GCP Cloud Audit Logs, Azure Activity Log), Git and CI audit logs, centralized in a SIEM with retention.
- **Compliance frameworks:** **SOC 2** (Trust Services Criteria — security, availability, confidentiality...), **ISO 27001** (ISMS), PCI DSS, HIPAA, GDPR, FedRAMP.
- **Evidence automation:** Platforms like **Vanta, Drata, Secureframe** continuously collect evidence (MFA enabled, encryption, access reviews, background checks, change management via PR history) via integrations; cloud-native: AWS Audit Manager, Security Hub standards, Azure Policy regulatory compliance; **compliance as code** (InSpec, OPA policies) and immutable logs.

### Interview Questions

**Q1. How do DevOps practices help with compliance (e.g. SOC 2)?**
Automation produces evidence by default: every change is a reviewed PR (change management), CI enforces tests and security scans, IaC shows exact configuration, deployments are logged with who/what/when, access is via SSO with MFA and least privilege, audit logs are centralized and immutable, policies are enforced as code. Compliance becomes continuous rather than an annual scramble, and tools like Vanta/Drata collect evidence automatically.

**Q2. What should Kubernetes audit logging capture and how do you manage the volume?**
Capture authentication identities, verbs and resources — especially write operations, secrets access (metadata only, never request bodies for secrets), exec/attach/port-forward, RBAC changes and admission decisions. Use an audit policy with levels: `Metadata` for secrets/configmaps, `RequestResponse` for critical resources like RBAC, `None` for noisy read-only system traffic (health checks, kube-proxy watches). Ship to a SIEM with retention and alerts on suspicious actions.

**Q3. What is kube-bench and how do you use it?**
It runs CIS Kubernetes Benchmark checks on nodes (and control plane for self-managed clusters) — file permissions, kubelet settings (anonymous auth disabled, authorization mode), API server flags, etcd settings — and reports pass/fail/warn with remediation. Run as a Job on each node type periodically or in CI for node images; for managed clusters use the provider-specific benchmark profile since the control plane is out of scope.

**Q4. How do you implement separation of duties in a CI/CD pipeline for compliance?**
Require PR approvals from someone other than the author (branch protection), protect production environments with approvals from authorized groups, prevent developers from deploying directly to production outside the pipeline (no direct prod access, break-glass with logging), keep pipeline definitions protected by CODEOWNERS, and log all approvals and deployments as evidence.

**Q5. What is "compliance as code"?**
Expressing compliance controls as automated, version-controlled checks — Chef InSpec profiles, OPA/Kyverno policies, Config rules, Security Hub standards, Checkov policies mapped to frameworks — run continuously in CI and against live environments, generating machine-readable results that map to control requirements. Gives continuous assurance and reproducible audit evidence.

---

# STAGE 12 — OBSERVABILITY

---

## 71. Observability Concepts (Golden Signals, RED/USE, Cardinality, Sampling, Correlation, Exemplars, Synthetics vs RUM)

**What it is:** **Monitoring** tells you *whether* something is wrong (known failure modes, dashboards, alerts). **Observability** is the ability to understand *why* — asking new questions about system behavior from its outputs, without shipping new code.

**Key concepts**
- **Telemetry signals:** **Metrics** (numeric aggregates over time — cheap, alerting), **logs** (discrete events — detail), **traces** (request path across services — latency attribution), plus **profiles** (code-level resource usage) and events.
- **Four golden signals (Google SRE):** **Latency**, **Traffic**, **Errors**, **Saturation**.
- **RED method (request-driven services):** **R**ate, **E**rrors, **D**uration.
- **USE method (resources):** **U**tilization, **S**aturation, **E**rrors — for CPU, memory, disks, network, pools.
- **Cardinality:** Number of unique label combinations per metric (e.g. `user_id` as a label creates millions of series) → memory/cost explosion in Prometheus/Datadog. Keep high-cardinality data in logs/traces, not metric labels.
- **Sampling:** **Head sampling** (decide at trace start — cheap, may miss rare errors) vs **tail sampling** (decide after trace completes — keep all errors/slow traces; needs buffering in collectors).
- **Correlation:** Shared trace IDs in logs, consistent resource attributes (`service.name`, `deployment.environment`, `k8s.pod.name`) to jump between signals.
- **Exemplars:** Trace IDs attached to metric data points (e.g. a slow request in a latency histogram bucket) → click from a metric spike to an example trace.
- **Synthetic monitoring vs RUM:** **Synthetics** run scripted checks (uptime, API flows, browser journeys) from probes on a schedule — proactive, consistent, catches outages with no traffic; **RUM (Real User Monitoring)** measures actual users' experience in browsers/mobile (Core Web Vitals, errors, geography) — real but needs traffic.
- **Histograms vs averages**, percentiles, alerting on symptoms.

### Interview Questions

**Q1. Monitoring vs observability?**
Monitoring is collecting predefined metrics and alerting on known conditions ("CPU > 90%", "error rate > 1%") — it answers known questions. Observability is a property of the system: rich, correlated telemetry (high-cardinality events, traces, logs, metrics) that lets engineers investigate unknown-unknowns ("why are only Android users in Brazil seeing slow checkouts since 14:05?") without deploying new instrumentation. You need both: monitoring to detect, observability to diagnose.

**Q2. Explain RED vs USE and when to use each.**
RED applies to services handling requests: monitor request rate, error rate, and duration distribution per endpoint — directly reflects user experience. USE applies to resources: utilization (% busy), saturation (queued work — run queue, connection pool waits), errors (hardware/driver errors) — for finding bottlenecks. Use RED for dashboards/alerts on every service and USE for infrastructure and capacity troubleshooting.

**Q3. What is high cardinality and why is it a problem for metrics?**
Every unique combination of metric name and label values is a separate time series. Labels like user ID, request ID, full URL path with IDs, or pod name on high-churn workloads multiply series into millions, exhausting Prometheus memory, slowing queries, and in SaaS tools causing large bills. Fix: avoid unbounded labels, normalize paths (`/users/{id}`), drop/relabel unneeded labels, aggregate with recording rules, and put high-cardinality detail in traces/logs (which are designed for it).

**Q4. Head vs tail sampling for traces?**
Head sampling decides at the root (e.g. keep 5%) and propagates the decision — simple and cheap, but rare errors and slow requests are mostly dropped. Tail sampling buffers complete traces in a collector tier and decides afterward — keep 100% of errors, slow traces and specific tenants, sample the rest — better signal at the cost of collector memory and a load-balanced collector architecture (all spans of a trace must reach the same collector instance — OTel loadbalancing exporter).

**Q5. What are exemplars and why are they useful?**
Exemplars attach a sample trace ID (and labels) to specific metric observations, such as a latency histogram bucket. In Grafana you see dots on a latency graph and click one to open the exact trace that was slow — bridging aggregate metrics and request-level detail during incident investigation.

**Q6. Synthetic monitoring vs RUM — do you need both?**
Yes, they complement each other. Synthetics detect outages and regressions proactively 24/7, even without traffic (e.g. at night or for new regions), and give consistent baselines for SLAs. RUM shows actual user experience across devices, networks and geographies — catching problems synthetics don't simulate (slow 3G, specific browsers, third-party script issues). Synthetics for availability alerting, RUM for experience optimization.

---

## 72. OpenTelemetry (OTel)

**What it is:** The CNCF standard (second most active CNCF project after Kubernetes) for **vendor-neutral instrumentation** — APIs, SDKs, auto-instrumentation agents, the **Collector**, and the **OTLP** protocol for traces, metrics, logs (and profiles emerging).

**Components**
- **API & SDKs** per language (Java, Python, Go, JS, .NET...), **auto-instrumentation** (Java agent, Python/Node instrumentation, Kubernetes OTel Operator injection, eBPF-based zero-code instrumentation).
- **Semantic conventions:** Standard attribute names (`http.request.method`, `db.system`, `service.name`, `k8s.pod.name`, and **GenAI conventions** for LLM calls).
- **Context propagation:** W3C Trace Context (`traceparent` header) and Baggage across services and messaging.
- **OTel Collector:** Pipeline of **receivers** (OTLP, Prometheus, Jaeger, filelog, k8s events) → **processors** (batch, memory_limiter, attributes/redaction, k8sattributes, tail_sampling, filter) → **exporters** (OTLP to any backend, Prometheus remote write, Loki, Datadog, X-Ray...). Deployed as **agent** (DaemonSet/sidecar) and/or **gateway** (central deployment). Distributions: contrib, AWS ADOT, Grafana Alloy, vendor distros.
- **OTLP:** gRPC/HTTP protocol for telemetry.

### Interview Questions

**Q1. Why has OpenTelemetry become the standard?**
It decouples instrumentation from backends: instrument once with OTel and send to any vendor or open-source backend by changing Collector config — no lock-in. It unifies traces, metrics and logs with shared context and semantic conventions, has broad auto-instrumentation, and is supported by every major vendor and cloud. Before OTel, switching APM vendors meant re-instrumenting everything.

**Q2. Explain the OTel Collector architecture and deployment patterns.**
Receivers ingest data, processors transform it (batching, memory limits, adding Kubernetes metadata, redacting PII, sampling), exporters send it to backends; pipelines are defined per signal. Patterns: **agent** per node (DaemonSet) collecting local telemetry and host metrics with low latency, forwarding to a **gateway** tier (scalable Deployment) that does heavy processing (tail sampling, routing, export to multiple backends, credential handling). This centralizes config and protects backends.

**Q3. How does context propagation work across services and queues?**
The instrumentation injects trace context (trace ID, span ID, flags) into outgoing requests via the W3C `traceparent` header (and `tracestate`/baggage); the receiving service extracts it and creates child spans. For async messaging, context is injected into message attributes/headers (Kafka headers, SQS message attributes) and extracted by consumers, often using span links for batch processing. Missing propagation (e.g. a proxy stripping headers, custom thread pools) breaks traces.

**Q4. Auto-instrumentation vs manual instrumentation?**
Auto-instrumentation (agents, OTel Operator injection, eBPF) gives immediate coverage of common libraries — HTTP servers/clients, DB drivers, messaging — with no code changes. Manual instrumentation adds business context: custom spans around important operations, attributes like `order.id`/`tenant`, custom metrics (orders processed) and events. Best: start with auto, then add targeted manual instrumentation for business-critical paths.

**Q5. How do you control telemetry costs with OpenTelemetry?**
In the Collector: tail sampling (keep errors/slow, sample normal), filter noisy spans (health checks), drop unused attributes and high-cardinality labels, aggregate metrics (delta to cumulative, recording), redact large payloads, route data by value (critical services to premium backend, debug data to cheaper storage), and set SDK sampling ratios. Measure ingest volumes per service.

**Q6. How do you protect sensitive data in telemetry?**
Avoid capturing request bodies/PII in instrumentation by default; use Collector processors (`attributes`, `redaction`, `transform`) to hash/remove sensitive attributes (emails, tokens, card numbers) centrally; scrub logs; restrict access to telemetry backends; and define data retention policies.

---

## 73. Prometheus (& Thanos/Mimir/VictoriaMetrics, Alertmanager)

**What it is:** The CNCF graduated open-source **metrics** monitoring system and time-series database — the de facto standard for Kubernetes metrics.

**Key concepts**
- **Pull model:** Prometheus **scrapes** HTTP `/metrics` endpoints at intervals; **service discovery** (Kubernetes, EC2, Consul); Pushgateway only for short-lived batch jobs.
- **Data model:** Metric name + labels → time series. **Metric types:** Counter (monotonic — use `rate()`), Gauge (up/down), Histogram (buckets — compute quantiles server-side with `histogram_quantile`, aggregatable; native histograms newer), Summary (client-side quantiles — not aggregatable).
- **PromQL:** `rate(http_requests_total{status=~"5.."}[5m])`, `sum by (service)`, `histogram_quantile(0.99, sum by (le) (rate(http_request_duration_seconds_bucket[5m])))`.
- **Recording rules** (precompute expensive queries) and **alerting rules** (`expr`, `for:` duration, labels, annotations).
- **Alertmanager:** Receives alerts → **grouping**, **deduplication**, **inhibition** (suppress dependent alerts), **silences**, routing tree to receivers (PagerDuty, Slack, email, webhooks).
- **Exporters:** node-exporter (hosts), kube-state-metrics (Kubernetes object state), cAdvisor (containers via kubelet), blackbox-exporter (probes), database exporters.
- **kube-prometheus-stack / Prometheus Operator:** `ServiceMonitor`/`PodMonitor`/`PrometheusRule` CRDs.
- **Limitations & scaling:** Single-node local TSDB with limited retention; scale out and long-term storage with **Thanos** (sidecar + object storage, global query, downsampling), **Grafana Mimir** (horizontally scalable, multi-tenant, remote write), **Cortex**, **VictoriaMetrics** (efficient storage, PromQL-compatible MetricsQL); managed: Amazon Managed Prometheus, Grafana Cloud, Google Managed Prometheus. **Remote write**, agent mode, federation.

### Interview Questions

**Q1. Why does Prometheus use a pull model and what are its trade-offs?**
Pull: Prometheus controls scrape rate (no overload from clients), knows when targets are down (`up == 0`), easy to debug (curl `/metrics`), and service discovery drives what to monitor. Trade-offs: targets must be reachable from Prometheus (hard across NAT/firewalls), short-lived jobs may finish before scraping (use Pushgateway or push via OTLP/remote write), and scaling needs sharding. Push-based remote write/OTLP is used for edge or serverless cases.

**Q2. Counter vs gauge vs histogram vs summary?**
Counter only increases (resets on restart) — requests, errors, bytes; query with `rate()`/`increase()`. Gauge goes up and down — memory, queue length, temperature. Histogram counts observations into configurable buckets — latency/size distributions; quantiles computed at query time and **aggregatable across instances**. Summary computes quantiles in the client — accurate per instance but **can't be aggregated** across instances, so histograms are generally preferred.

**Q3. Write a PromQL query for p99 latency per service and error ratio.**
p99: `histogram_quantile(0.99, sum by (le, service) (rate(http_request_duration_seconds_bucket[5m])))`. Error ratio: `sum by (service) (rate(http_requests_total{status=~"5.."}[5m])) / sum by (service) (rate(http_requests_total[5m]))`. Use `rate` over counters, aggregate `by (le)` before `histogram_quantile`.

**Q4. How does Alertmanager reduce alert noise?**
**Grouping** bundles related alerts (e.g. all pods of a service failing) into one notification; **deduplication** across HA Prometheus replicas; **inhibition** suppresses lower-level alerts when a higher-level alert fires (node down inhibits pod alerts on it); **silences** mute during maintenance; routing sends critical alerts to paging and warnings to Slack; `group_wait`, `group_interval`, `repeat_interval` control timing.

**Q5. How do you scale Prometheus for many clusters and long retention?**
Run Prometheus (or agent mode) per cluster, shard scraping if needed, and **remote write** to a horizontally scalable backend (Mimir, Thanos Receive, VictoriaMetrics, or managed Prometheus) with object storage for long-term retention, global querying, deduplication and multi-tenancy. Alternatively Thanos sidecars upload blocks to object storage and Thanos Query federates. Control cardinality and use recording rules for heavy dashboards.

**Q6. What is the `for` clause in an alerting rule?**
The condition must be continuously true for that duration before the alert fires (pending → firing), filtering out brief spikes and reducing flapping. Too long delays detection; too short creates noise. SLO burn-rate alerts use multiple windows instead.

---

## 74. Logging (Loki, ELK/EFK, Fluentd/Fluent Bit, Vector, Splunk, Datadog Logs)

**What it is:** Collecting, shipping, storing and querying logs from applications and infrastructure.

**Pipeline:** Application writes **structured JSON logs to stdout** → node agent collects container log files → parse/enrich (Kubernetes metadata) → route/filter → backend storage and indexing → query/visualize/alert.

**Tools**
- **Collectors/shippers:** **Fluent Bit** (lightweight C agent, CNCF — most common DaemonSet), **Fluentd** (Ruby, plugin-rich aggregator), **Vector** (Rust, high-performance, transforms with VRL, by Datadog), OTel Collector filelog receiver, Grafana Alloy, Logstash (heavy, ELK).
- **Backends:**
  - **ELK/EFK:** **Elasticsearch/OpenSearch** (full-text indexing, powerful search, expensive at scale) + Logstash/Fluentd + **Kibana/OpenSearch Dashboards**.
  - **Grafana Loki:** Indexes only **labels** (not full text), stores compressed chunks in object storage — much cheaper; queried with **LogQL**; pairs with Grafana.
  - **Commercial:** **Splunk** (powerful SPL search, enterprise/security, costly), **Datadog Logs** (indexing vs archiving, log pipelines, Flex Logs), Elastic Cloud, New Relic, Sumo Logic, CloudWatch Logs.
- **Practices:** Structured logging, log levels, correlation/trace IDs, sampling debug logs, PII redaction, retention tiers (hot/warm/cold/archive), log-based metrics.

### Interview Questions

**Q1. Loki vs Elasticsearch for logs?**
Elasticsearch indexes full log content — fast arbitrary full-text search and analytics, but high storage and compute costs and operational complexity at scale (shards, heap, cluster management). Loki indexes only a small set of labels (namespace, app, pod) and stores compressed log chunks in object storage — dramatically cheaper and simpler, query by labels then grep-like filtering (LogQL); slower for broad full-text searches across huge time ranges. Loki suits Kubernetes ops logs with Grafana; Elasticsearch/OpenSearch suits search-heavy and security analytics use cases.

**Q2. Fluent Bit vs Fluentd vs Vector?**
Fluent Bit: very lightweight, fast, low memory — ideal as a node-level DaemonSet collector. Fluentd: heavier, huge plugin ecosystem — used as an aggregator for complex routing/transformation. Vector: high performance, strong transformation language (VRL), unified for logs and metrics, good as agent or aggregator. Common architecture: Fluent Bit/Vector agents → optional aggregator (Fluentd/Vector/OTel) → backends.

**Q3. What are logging best practices for microservices?**
Structured JSON logs with consistent fields (timestamp, level, service, environment, trace_id/span_id, request_id, user/tenant where allowed), log to stdout (let the platform collect), meaningful messages with context, appropriate levels (don't log everything at INFO), never log secrets/PII (or redact), sample high-volume debug logs, include error stack traces, and correlate with traces. Use log-based metrics sparingly — emit metrics directly when possible.

**Q4. Your logging bill doubled. How do you reduce it?**
Identify top sources by volume (per service/namespace), lower verbosity (debug logs in production, noisy health-check logs, chatty libraries), drop or sample at the collector, convert repetitive logs into metrics, apply tiered retention (short hot retention, archive to object storage for compliance), use cheaper backends/tiers (Loki, Datadog Flex/archives, CloudWatch Infrequent Access), and set per-team quotas/showback.

**Q5. How are container logs collected in Kubernetes?**
Containers write to stdout/stderr; the container runtime writes them to files on the node (`/var/log/pods/...`, symlinked in `/var/log/containers`); kubelet handles rotation; a DaemonSet log agent (Fluent Bit) tails those files, parses (CRI format, JSON), enriches with Kubernetes metadata from the API (pod labels, namespace), and ships to the backend. Logs written to files inside containers are lost unless collected via sidecars or volumes — prefer stdout.

---

## 75. Distributed Tracing Backends (Jaeger, Grafana Tempo, Zipkin, Datadog APM, X-Ray, Honeycomb)

**What it is:** Storing and visualizing **traces** — the path of a request across services as a tree of **spans** (operation name, start/end time, attributes, status, events, links, parent).

**Tools**
- **Jaeger (CNCF graduated):** Open-source tracing backend (originally Uber) with UI, storage on Elasticsearch/OpenSearch/Cassandra/Badger; Jaeger v2 is built on the OTel Collector framework and natively ingests OTLP.
- **Grafana Tempo:** High-scale, cost-efficient tracing storing traces in object storage with minimal indexing (lookup by trace ID, TraceQL search), integrated with Loki/Mimir/Grafana (exemplars, logs-to-traces).
- **Zipkin:** Early open-source tracer (Twitter), simple, B3 propagation format.
- **Commercial/cloud:** **Datadog APM**, **AWS X-Ray** (OTLP-compatible), **Honeycomb** (high-cardinality event analysis, BubbleUp), Lightstep (ServiceNow Cloud Observability), New Relic, Dynatrace, Elastic APM.

### Interview Questions

**Q1. How do you use a trace to debug a slow request?**
Open the trace (from an exemplar, a log's trace_id or search by attributes), look at the waterfall: identify the critical path and the longest spans, check if latency is in a downstream call (DB query span with statement, external API), in queueing/gaps between spans (thread pool saturation, GC), or in many sequential calls (N+1 pattern — parallelize/batch). Compare with a normal trace of the same endpoint and check span attributes (payload size, tenant, region).

**Q2. Jaeger vs Tempo?**
Jaeger: mature UI, indexes span attributes in a search backend (Elasticsearch/Cassandra) — flexible searches but storage cost/ops; v2 aligns with OTel. Tempo: stores traces cheaply in object storage, minimal indexing, TraceQL for searching, tight integration with Grafana's metrics and logs for correlation — designed for 100% trace retention at low cost. Choose by scale/cost and ecosystem.

**Q3. What information should spans contain?**
Standard OTel semantic attributes (HTTP method/route/status, DB system and sanitized statement, messaging destination, RPC service/method), resource attributes (service.name, version, environment, k8s metadata, cloud region), error status and exception events, and business context (tenant, order type) — avoiding PII and unbounded payloads.

**Q4. Why might traces be broken (missing spans or disconnected)?**
Context not propagated — a service not instrumented, a proxy/load balancer stripping headers, mismatched propagation formats (B3 vs W3C), async boundaries (queues, thread pools, callbacks) not carrying context, sampling decisions inconsistent across services, clock skew showing odd timing, or spans dropped by collectors due to limits.

**Q5. What does Honeycomb's approach ("observability 2.0") emphasize?**
Treating telemetry as wide, high-cardinality structured events (spans with many attributes) stored in a column store, enabling arbitrary slicing and dicing ("which attribute values are over-represented among slow requests?" — BubbleUp) instead of predefined metrics and dashboards. It encourages a single source of truth (events/traces) from which metrics can be derived.

---

## 76. Grafana & Observability Platforms (Datadog, New Relic, Dynatrace)

**What it is:**
- **Grafana:** Open-source visualization and observability platform — dashboards querying many data sources (Prometheus/Mimir, Loki, Tempo, Elasticsearch, CloudWatch, SQL, etc.), alerting (Grafana Alerting), correlations between signals, dashboards as code (JSON, Grafonnet, Terraform provider, Grafana Operator). The **LGTM stack**: **L**oki (logs), **G**rafana, **T**empo (traces), **M**imir (metrics), plus Pyroscope (profiles), Alloy (collector), Beyla (eBPF auto-instrumentation), k6, OnCall/IRM; Grafana Cloud as SaaS.
- **Commercial all-in-one platforms:** **Datadog** (infrastructure, APM, logs, RUM, synthetics, security, LLM observability; huge integrations; usage-based pricing), **New Relic** (full-stack, consumption pricing by data + users), **Dynatrace** (OneAgent auto-discovery, Smartscape topology, Davis AI causal analysis), **Elastic Observability**, Splunk Observability, Honeycomb, Chronosphere (cost control).

### Interview Questions

**Q1. Open-source stack (Prometheus/Grafana/Loki/Tempo) vs Datadog — how do you decide?**
Open source: no license costs and no vendor lock-in, full control, OTel-native — but requires engineering effort to operate at scale (storage, upgrades, HA, multi-tenancy), building integrations and on-call for the observability stack itself; managed versions (Grafana Cloud, AMP/AMG) reduce ops. Datadog: fastest time to value, unified UX across signals, 800+ integrations, advanced features (APM, RUM, security, AI) — but costs grow quickly with hosts, custom metrics, log volume and cardinality, and you're locked into its agents/pricing. Decide by team capacity, scale, budget predictability, and required features; OTel instrumentation keeps the option to switch.

**Q2. What makes a good dashboard?**
Purpose-driven (one audience and question per dashboard — service overview, on-call triage, capacity, business KPIs), start with SLOs and golden signals at the top, then drill-down panels, consistent units and time ranges, annotations for deploys/incidents, templated variables (cluster/service/env), links to runbooks/traces/logs, and no vanity panels. Manage dashboards as code and review them like code.

**Q3. How do you manage Grafana dashboards and alerts as code?**
Store dashboard JSON (or generate it with Grafonnet/Jsonnet, the Grafana Foundation SDK) in Git; provision via Grafana provisioning files, the Grafana Operator CRDs, or the Terraform Grafana provider; alert rules as code too; CI validates and deploys; prevent click-ops drift (or export changes back to Git).

**Q4. How do you control costs on a commercial observability platform?**
Govern custom metric cardinality (tags), use metrics-without-limits/tag configuration features, sample traces (ingestion controls), index only valuable logs and archive the rest, set retention appropriately, limit APM/host coverage to what matters (e.g. no APM on dev), use committed-use contracts, monitor usage dashboards per team with showback, and keep OTel pipelines that can route/drop data before it reaches the vendor.

**Q5. What is Dynatrace's Davis AI / causal AI approach?**
Dynatrace builds a real-time dependency topology (Smartscape) from its OneAgent instrumentation and uses deterministic causal analysis on that graph plus anomaly detection to identify the root-cause component of a problem and group related alerts into one "problem" — reducing alert storms. It's an example of AIOps built on topology, alongside newer generative AI assistants.

---

## 77. Profiling & eBPF-Based Observability (Pyroscope, Parca, Datadog Profiler, Pixie, Hubble, Beyla)

**What it is:**
- **Continuous profiling:** Always-on, low-overhead sampling of where code spends CPU, memory allocations, locks and I/O in production — visualized as **flame graphs**; the "fourth pillar" of observability.
  - **Grafana Pyroscope** (open source, merged with Phlare), **Parca** (Polar Signals, eBPF-based, open source), **Datadog Continuous Profiler**, Elastic Universal Profiling (contributed to OTel as the eBPF profiling agent), Google Cloud Profiler, AWS CodeGuru Profiler. **OpenTelemetry profiles** signal is emerging.
- **eBPF-based observability:** Instrumentation from the kernel — no code changes or sidecars.
  - **Pixie** (CNCF, New Relic-originated): Auto-captures HTTP/gRPC/DB requests, CPU flame graphs in Kubernetes via eBPF, scriptable with PxL.
  - **Cilium Hubble:** Network flow observability.
  - **Grafana Beyla** (donated to OTel as OBI — OpenTelemetry eBPF Instrumentation): Zero-code RED metrics and traces for HTTP/gRPC services via eBPF.
  - Others: Coroot, Odigos, Groundcover, Inspektor Gadget, bpftrace/BCC tools.

### Interview Questions

**Q1. What is continuous profiling and why add it to metrics, logs and traces?**
Metrics tell you CPU is high, traces tell you which service/span is slow — profiles tell you **which function/line** consumes CPU or memory. Continuous profiling samples stacks all the time in production with ~1–2% overhead, so you can compare before/after deploys, find hot paths, memory leaks and lock contention, and cut compute costs. It's especially valuable for performance regressions that don't show up in tests.

**Q2. How do you read a flame graph?**
Each box is a function in a stack; the x-axis width represents the proportion of samples (time/resources) — **not** time order; the y-axis is stack depth (callers below, callees above, or inverted in icicle graphs). Wide boxes at the top (leaf functions) are where CPU is actually spent; wide plateaus indicate hot code paths. Compare differential flame graphs between versions to find regressions.

**Q3. What are the advantages and limitations of eBPF-based auto-instrumentation?**
Advantages: zero code changes, language-agnostic, captures everything on the node (including legacy or third-party services), low overhead, fast rollout across a fleet. Limitations: less application context (no business attributes or custom spans), trace context propagation is harder (some tools inject headers, others can't), encrypted traffic needs uprobes on TLS libraries, kernel version requirements, and privileged agents on nodes (security review). Use it for baseline coverage, plus SDK instrumentation for depth.

**Q4. How would you investigate rising memory usage in a production service?**
Check metrics for growth pattern (steady leak vs load-correlated), compare across versions via deploy annotations, use continuous memory/allocation profiles (Pyroscope/Datadog) to see which code paths allocate and retain memory, take heap dumps if needed (JVM heap dump, Go pprof heap, Node heap snapshot) in a safe manner, and correlate with traffic patterns (caches without bounds, connection leaks, large payloads).

**Q5. What is Pixie and when would you use it?**
An in-cluster, eBPF-based observability tool for Kubernetes that automatically captures service maps, request-level data (HTTP, gRPC, MySQL, Postgres, Redis, Kafka, DNS), and CPU profiles without instrumentation, stored in-cluster for a short window and queried with PxL scripts. Useful for rapid debugging in clusters lacking instrumentation, or as a complement to OTel.

---

# STAGE 13 — RELIABILITY (SRE) & INCIDENT MANAGEMENT

---

## 78. SLI, SLO, SLA, Error Budgets & Burn-Rate Alerting

**What they are:**
- **SLI (Service Level Indicator):** A quantitative measure of service level from the user's perspective — e.g. proportion of successful requests, proportion of requests served under 300 ms, freshness of data, durability.
- **SLO (Service Level Objective):** Target for an SLI over a window — e.g. "99.9% of requests succeed over 28 days".
- **SLA (Service Level Agreement):** A contractual commitment with consequences (credits) — looser than internal SLOs.
- **Error budget:** `1 − SLO` — the allowed unreliability (99.9% over 30 days ≈ 43 minutes of full downtime or 0.1% of requests). It balances velocity and reliability: budget left → ship features faster; budget exhausted → focus on reliability (feature freeze per policy).
- **Burn rate:** How fast the error budget is being consumed relative to the SLO window (burn rate 1 = exactly exhaust at window end; 14.4 over 1 hour = consumes 2% of a 30-day budget per hour).
- **Multi-window, multi-burn-rate alerts** (Google SRE workbook): e.g. page if burn rate > 14.4 over 1 h **and** over 5 min; page if > 6 over 6 h and 30 min; ticket if > 1 over 3 days. Fast detection of severe issues, low noise.
- **Tools:** Sloth, Pyrra, OpenSLO spec, Google Cloud SLO monitoring, Datadog/Dynatrace/Grafana SLOs, Nobl9, CloudWatch Application Signals.

### Interview Questions

**Q1. SLI vs SLO vs SLA?**
SLI is the measurement (e.g. % of requests returning non-5xx within 500 ms). SLO is the internal target for that measurement (99.9% over 30 days) that guides engineering decisions. SLA is the external contract with customers and penalties, typically less strict than the SLO so you have a buffer. You can have SLOs without SLAs, but SLAs should be backed by tighter SLOs.

**Q2. How do you choose good SLIs and SLO targets?**
Start from critical user journeys (login, search, checkout) and what users care about (availability, latency, correctness, freshness). Measure as close to the user as possible (load balancer/edge or RUM rather than CPU). Set targets based on historical performance, user expectations and business needs — not 100% (impossible and infinitely expensive) and not higher than dependencies allow. Iterate: tighten or loosen based on whether SLO breaches correlate with user pain.

**Q3. What is an error budget policy?**
An agreement between product and engineering on what happens when the budget is consumed — e.g. if budget is exhausted, freeze non-critical feature releases, prioritize reliability work, require postmortems for incidents consuming >X% budget; if budget remains healthy, teams can take more risk (faster releases, experiments). It turns reliability debates into data-driven decisions.

**Q4. Why use burn-rate alerts instead of threshold alerts on error rate?**
Simple thresholds ("error rate > 1% for 5 min") either page for brief, harmless blips or miss slow, sustained degradation that silently consumes the budget. Burn-rate alerts relate errors to the SLO: high burn over short windows pages quickly for severe incidents; low burn over long windows creates tickets for slow leaks. Multi-window conditions reduce false positives and reset quickly after recovery.

**Q5. Calculate the error budget for a 99.95% SLO over 30 days.**
Budget = 0.05% of the window: 30 days × 24 × 60 = 43,200 minutes × 0.0005 = **21.6 minutes** of total unavailability (or 0.05% of requests failing). If a single incident caused 10 minutes of full outage, ~46% of the monthly budget is consumed.

**Q6. How do you set SLOs for a service that depends on other services?**
Your achievable SLO is bounded by critical dependencies' reliability (serial dependencies multiply). Options: negotiate SLOs with dependency owners, add resilience (caching, retries with budgets, graceful degradation, async processing, fallbacks) to decouple from dependency failures, and measure your SLI at the user boundary so the SLO reflects real experience regardless of internal causes.

---

## 79. DORA Metrics

**What it is:** Metrics from the **DevOps Research and Assessment (DORA)** program (Accelerate book, annual State of DevOps reports) that predict software delivery and organizational performance.

**The metrics**
| Metric | Measures | Elite (approx.) |
|---|---|---|
| **Deployment frequency** | How often you deploy to production | On demand (multiple per day) |
| **Lead time for changes** | Commit → running in production | Less than a day |
| **Change failure rate** | % of deployments causing a failure needing remediation | Low (~0–15%) |
| **Time to restore service / Failed deployment recovery time** | How quickly you recover from a failed deployment/incident | Less than an hour |
| **Rework rate** (added in newer reports) | Share of deployments that are unplanned fixes for user-facing issues | Low |

Throughput (frequency, lead time) and stability (failure rate, recovery time) **go together** in high performers — speed and stability are not trade-offs. DORA's research also covers capabilities (trunk-based development, CI, loosely coupled architecture, observability, culture) and, in recent reports, the impact of **AI adoption** on delivery (AI increases individual productivity but can hurt delivery stability if batch sizes grow — the "AI amplifies existing practices" finding).

**Tools:** Four Keys (Google), Sleuth, LinearB, Swarmia, Jellyfish, DX, GitLab Value Stream Analytics, Backstage plugins, custom pipelines from deploy and incident events.

### Interview Questions

**Q1. What are the DORA metrics and why are they useful?**
Deployment frequency, lead time for changes, change failure rate and time to restore (plus rework rate). They balance speed and stability, are outcome-oriented (not activity counts like lines of code), and are backed by multi-year research correlating them with organizational performance. They help teams identify bottlenecks (e.g. long lead time due to slow reviews or manual approvals) and track improvement.

**Q2. How do you measure DORA metrics in practice?**
Collect events: commits/PR merges (Git), deployments (CI/CD or GitOps tool with commit SHAs), incidents and their resolution (PagerDuty/incident tool, linked to deployments), and rollbacks/hotfixes. Lead time = deploy time − commit time per change; change failure rate = deployments linked to incidents/rollbacks ÷ total deployments; time to restore from incident timestamps. Automate via tools (Four Keys, Sleuth, GitLab, DX) and present trends per team, not leaderboards.

**Q3. How can DORA metrics be misused?**
Gaming (splitting deploys to boost frequency, not recording incidents), comparing teams with different contexts, using them for individual performance evaluation, optimizing one metric at the expense of others, and focusing on metrics instead of underlying capabilities. Use them for team-level learning and improvement conversations.

**Q4. Your lead time for changes is 2 weeks. How do you improve it?**
Map the value stream to see where time goes: waiting for code review (set review SLAs, smaller PRs), long-lived branches (trunk-based development), slow or flaky CI (speed up, parallelize), manual QA and change approval boards (automate testing, use peer review and automated checks as approval), batched releases/release trains (deploy independently and continuously), environment bottlenecks (ephemeral environments). Fix the biggest wait first.

**Q5. What did recent DORA research say about AI-assisted development?**
AI adoption is widespread and increases perceived individual productivity, code output and documentation quality, but early reports found delivery throughput and stability could decline when AI leads to larger batches or less review rigor. The takeaway: AI amplifies the strengths and weaknesses of existing practices — teams with small batches, strong testing, platform engineering and clear AI policies benefit most.

---

## 80. On-Call & Incident Management (PagerDuty, incident.io, Rootly, FireHydrant, Grafana OnCall, Postmortems, Runbooks)

**What it is:** The people, process and tools for responding to production problems.

**Tools**
- **Alerting/on-call:** **PagerDuty** (market leader — schedules, escalation policies, event orchestration, AIOps), **incident.io** (Slack-native incident response plus on-call), **Rootly**, **FireHydrant** (incident workflows, status pages, retrospectives), **Grafana OnCall/IRM**, Splunk On-Call (VictorOps). Atlassian **Opsgenie** is being phased out (end of sale 2025, migration to Jira Service Management/Compass).
- **Status pages:** Atlassian Statuspage, incident.io status pages, Instatus.

**Incident process**
- **Severity levels** (SEV1 critical customer-facing outage → SEV4 minor) with defined response expectations.
- **Roles:** **Incident Commander** (coordinates, decides — doesn't debug), **Communications lead** (stakeholders/status page), **Operations/subject-matter experts** (investigate and mitigate), scribe.
- **Lifecycle:** Detect → triage/declare → mobilize → **mitigate first** (rollback, failover, feature flag off) → resolve → communicate → **postmortem** → follow-up actions.
- **Blameless postmortems:** Timeline, impact, root cause(s) and contributing factors (5 whys / causal analysis), what went well, what went poorly, action items with owners and due dates; focus on systems, not individuals.
- **Runbooks/playbooks:** Step-by-step guides linked from alerts — increasingly automated (runbook automation, SSM documents, Rundeck).
- **Healthy on-call:** Reasonable rotation size (≥6–8 people), follow-the-sun, compensation, alert hygiene (every page actionable), handoffs, on-call reviews.

### Interview Questions

**Q1. Walk me through how you handle a major production incident.**
Acknowledge the page, assess impact quickly (SLO dashboards, error rates, scope), declare an incident with a severity and open a dedicated channel/bridge, assign an incident commander and comms lead. Focus on **mitigation first**: check recent changes (deploys, config, feature flags, infra) and roll back or disable; fail over or scale if capacity-related. Communicate regularly (internal updates every 15–30 min, status page for customers). Once mitigated, verify recovery with metrics, then resolve. Hold a blameless postmortem within days, track action items to completion.

**Q2. What makes a postmortem blameless and why does it matter?**
It assumes people acted reasonably given the information and tools they had, and asks what in the system (process, tooling, design, alerts, documentation) allowed the failure — instead of who made a mistake. Blame leads people to hide information, preventing learning and repeat prevention. Blameless doesn't mean no accountability: owners commit to concrete systemic improvements.

**Q3. What should be in a good postmortem document?**
Summary, impact (users affected, duration, SLO/error budget consumed, revenue), detection (how and how fast), timeline with timestamps, root cause and contributing factors, what went well/poorly (including response process), lessons learned, and prioritized action items (prevent, detect faster, mitigate faster) with owners and deadlines; shared widely and reviewed in a postmortem meeting.

**Q4. How do you reduce on-call burden and alert fatigue?**
Alert only on actionable, user-impacting symptoms (SLO burn rate) — not on causes like CPU; route non-urgent issues to tickets; review every page weekly (was it actionable? needed a human?), delete or tune noisy alerts, automate common remediations, add runbooks, fix recurring root causes with dedicated time, measure pages per shift and off-hours pages, and ensure enough people in the rotation.

**Q5. What is the role of an Incident Commander?**
Coordinates the response rather than debugging: establishes roles, keeps a clear picture of status and hypotheses, prioritizes mitigation, makes decisions (e.g. rollback now), avoids too many cooks, ensures regular communication, manages handoffs for long incidents, and declares resolution. Separating coordination from investigation speeds up resolution.

**Q6. What is the difference between MTTD, MTTA, MTTR?**
MTTD — mean time to detect (from failure start to detection/alert); MTTA — mean time to acknowledge (alert to human response); MTTR — mean time to restore/recover (from start to service restoration). Improving detection (better SLO alerts, synthetics), response (on-call process) and mitigation (fast rollbacks, feature flags, runbooks) each reduce customer impact. Averages can mislead — look at distributions per severity.

---

## 81. Chaos Engineering (Chaos Mesh, LitmusChaos, Gremlin, AWS FIS, Game Days)

**What it is:** The discipline of **experimenting on a system to build confidence** in its ability to withstand turbulent conditions — deliberately injecting failures (instance/pod kills, network latency/loss, CPU/memory stress, AZ outages, dependency failures) in a controlled way to find weaknesses before they cause incidents. Popularized by Netflix (Chaos Monkey, Simian Army).

**Principles:** Define **steady state** (measurable normal behavior, e.g. SLIs) → hypothesize it continues during the fault → inject realistic faults → observe → minimize **blast radius** (start small, abort conditions) → automate and run continuously.

**Tools**
- **Chaos Mesh (CNCF):** Kubernetes-native chaos via CRDs — PodChaos, NetworkChaos, StressChaos, IOChaos, TimeChaos, DNSChaos, workflows, dashboard.
- **LitmusChaos (CNCF):** Kubernetes chaos with experiment hub (ChaosHub), workflows, probes for steady-state validation (Harness offers a commercial version).
- **Gremlin:** Commercial SaaS chaos platform — safe, broad fault library, reliability scoring.
- **AWS Fault Injection Service (FIS):** Managed experiments on EC2, ECS, EKS, RDS, network (AZ power interruption scenario), with stop conditions tied to CloudWatch alarms. Azure Chaos Studio as equivalent.
- **Game days:** Planned exercises where teams simulate incidents (failover drills, region evacuation, dependency outage) to test systems **and** people/processes/runbooks.

### Interview Questions

**Q1. What is chaos engineering and how do you start safely?**
Controlled experiments that inject faults to verify resilience hypotheses. Start safely: ensure good observability and defined steady-state metrics first, begin in staging with small faults (kill one pod), define abort/stop conditions (SLO burn alarm), limit blast radius (one instance, small % traffic), communicate with stakeholders, run during business hours with the team present, document findings and fixes, then gradually move to production experiments and automation.

**Q2. Give examples of chaos experiments for a Kubernetes microservice.**
Kill random pods (verify replicas, PDBs, readiness and retries handle it); drain a node or terminate it (rescheduling time, stateful workload behavior); inject network latency/packet loss between service and database (timeouts, circuit breakers, user impact); make a dependency return errors (fallbacks, graceful degradation); CPU/memory stress (autoscaling, limits, OOM behavior); DNS failures; AZ outage simulation (topology spread, capacity headroom); clock skew.

**Q3. What's the difference between chaos engineering and testing?**
Testing verifies known expected behavior with deterministic assertions (unit/integration tests). Chaos engineering is experimentation to discover unknown weaknesses in complex system behavior under real conditions — emergent properties like retry storms, cascading failures, misconfigured timeouts, missing alerts — and validates operational readiness (alerts fired? runbooks work? on-call knows what to do?).

**Q4. What is a game day and how do you run one?**
A scheduled exercise simulating a realistic failure scenario (e.g. primary database failover, region evacuation, loss of a critical third party). Plan scenario, success criteria and safety measures; inform participants (or not, for realism of detection); execute; observe detection, communication, decision-making and technical recovery; measure RTO/RPO; debrief blamelessly with action items. Valuable for testing DR plans and training new on-call engineers.

**Q5. How do AWS FIS stop conditions work and why are they important?**
An FIS experiment template references CloudWatch alarms as stop conditions; if any alarm enters ALARM state (e.g. SLO burn or error rate too high), FIS stops the experiment and rolls back actions where possible. They act as guardrails that limit customer impact automatically — essential for production chaos experiments.

---

## 82. Resilience, Capacity Planning & Disaster Recovery

**What it is:** Designing and operating systems that degrade gracefully, scale with demand, and recover from failures.

**Key concepts**
- **Capacity planning:** Forecast demand (organic growth, launches, seasonality), measure capacity per instance via load tests, maintain **headroom** (e.g. N+1 or N+2, AZ-loss headroom), track utilization and saturation, plan lead times (quota increases, reserved capacity, GPUs).
- **Load shedding:** Reject excess requests early (HTTP 429/503) to protect core functionality and keep goodput high under overload; prioritize critical traffic.
- **Graceful degradation:** Disable non-critical features (recommendations, personalization) or serve cached/stale data when dependencies fail.
- **Timeouts, retries with backoff + jitter, retry budgets, circuit breakers, bulkheads, rate limiting, backpressure.**
- **Redundancy:** Multi-AZ by default, multi-region for critical systems; avoid single points of failure (including people and third parties).
- **Backup & DR:** RTO/RPO targets, backups (and tested restores!), replication, DR strategies (backup/restore, pilot light, warm standby, active-active), immutable/offsite backups against ransomware.
- **Static stability:** Keep working during failures without needing control-plane actions.

### Interview Questions

**Q1. How do you do capacity planning for a service?**
Establish the capacity model: load test to find throughput per instance at the SLO latency (e.g. 500 RPS/pod at p99 < 200 ms); gather demand forecasts (historical growth trends, seasonality, marketing launches); compute required capacity at peak plus headroom for failures (e.g. survive one AZ loss → size so N−1 AZs handle peak) and autoscaling lag; verify dependencies (DB connections, third-party rate limits, cloud quotas); revisit regularly and after architecture changes; monitor saturation signals as early warnings.

**Q2. What is load shedding and how do you implement it?**
When demand exceeds capacity, accepting all requests causes queues to grow, latency to explode and throughput of successful requests (goodput) to collapse — everyone gets errors. Load shedding rejects some requests quickly to keep serving the rest within SLO. Implement with concurrency limits/adaptive concurrency (Envoy adaptive concurrency, Netflix concurrency-limits), queue length or latency-based admission control, priority classes (shed background/batch first, protect checkout), rate limits at the edge, and fast 429/503 responses with Retry-After.

**Q3. How do retries cause outages and how do you prevent it?**
Retries multiply load on an already struggling service (retry storms) — with retries at multiple layers (client, gateway, service, SDK), one failure can produce exponential amplification. Prevent: retry only idempotent/safe operations, exponential backoff with jitter, retry at a single layer, cap attempts, **retry budgets** (retries ≤ 10% of requests), circuit breakers to stop calling failing dependencies, and deadline propagation (don't retry if the caller's deadline has passed).

**Q4. Explain graceful degradation with an example.**
An e-commerce product page depends on catalog, pricing, inventory, reviews and recommendations. If recommendations or reviews services fail, the page still renders without those sections (timeouts + fallbacks) instead of failing entirely; if pricing is slow, serve a recently cached price with a notice; the critical path (view product, add to cart, checkout) is protected with stricter resources and priority. Design which features are critical vs optional upfront.

**Q5. How do you ensure backups are actually usable?**
Automate regular **restore tests** (restore into an isolated environment and run validation queries/integrity checks), monitor backup jobs and age of the latest successful backup, keep backups in a separate account/region with immutability (object lock/vault lock), document and rehearse the restore procedure, measure restore time against RTO. "A backup you haven't restored is a hope, not a backup."

---

## 83. Toil Reduction & Automation

**What it is:** **Toil** (Google SRE definition) is operational work that is **manual, repetitive, automatable, tactical (interrupt-driven), without enduring value, and scales linearly with service growth** — e.g. manually restarting services, handling routine tickets, rotating certificates by hand, provisioning accounts manually. Google SRE aims to keep toil below **50%** of an SRE's time, investing the rest in engineering that reduces future toil.

**Approaches:** Measure toil (track time spent per task type), prioritize by frequency × time × risk, automate (scripts → self-service tools → fully autonomous systems/operators), eliminate the need entirely (fix root causes, redesign), self-service portals, runbook automation, auto-remediation with guardrails, and now AI agents for triage.

### Interview Questions

**Q1. What is toil and how is it different from overhead?**
Toil is manual, repetitive, automatable work tied to running a service that grows with scale and produces no lasting improvement (e.g. manually resizing disks each week). Overhead is administrative work not directly tied to running the service (meetings, planning, HR tasks). Not all operational work is toil — incident response requiring judgment or one-time architecture work isn't toil.

**Q2. How do you decide what to automate first?**
Quantify toil: frequency × time per occurrence × number of people, plus error risk and interrupt cost. Prioritize high-frequency, error-prone, well-understood tasks (with stable procedures) whose automation pays back quickly; consider eliminating the cause instead of automating the symptom (e.g. fix the memory leak instead of automating restarts). Use the "automation ladder": document → script → self-service tool → autonomous system.

**Q3. What are the risks of automation and how do you mitigate them?**
Automation can amplify mistakes at scale (a script that deletes the wrong resources everywhere), hide system understanding (people forget how to do it manually), and fail silently. Mitigate: dry-run modes, rate limits and blast-radius limits (act on one shard first), idempotency, guardrails/approvals for destructive actions, logging and alerting on automation outcomes, tests for automation code, and manual fallback runbooks.

**Q4. Give an example of toil you've eliminated.**
Example: engineers manually created database users and credentials for each new service (tickets taking days). Replaced with self-service: a Terraform module + Backstage template that provisions the database and a Vault dynamic-secrets role, and the service gets credentials through workload identity. Result: provisioning went from days to minutes, no shared passwords, zero tickets.

**Q5. How do you measure success of toil reduction?**
Hours of toil per week per team (tracked via ticket categories or time logs), ticket volume for routine requests, pages per shift, lead time for requests (e.g. environment provisioning), percentage of time spent on engineering projects vs operations, and error rates of manual operations — tracked over time.

---

# STAGE 14 — PLATFORM ENGINEERING & DEVELOPER EXPERIENCE

---

## 84. Internal Developer Platforms (IDP), Golden Paths & Platform-as-a-Product

**What it is:** **Platform engineering** builds and operates an **Internal Developer Platform (IDP)** — a curated, self-service layer of tools, workflows and infrastructure that lets application teams build, deploy and run software with low cognitive load, without filing tickets.

**Key concepts**
- **Self-service:** Developers create services, environments, databases, pipelines via portal/CLI/API/GitOps, within guardrails.
- **Golden paths / paved roads:** Opinionated, well-supported default ways to do common tasks (create a new Go service with CI, observability, security scans, and deployment pre-wired). Optional but so convenient that most teams choose them.
- **Platform as a product:** Treat developers as customers — user research, roadmaps, documentation, onboarding, SLOs for the platform, measuring adoption and satisfaction; not a mandated ticket queue.
- **Thinnest viable platform:** Start small (maybe a wiki + templates), grow based on real needs.
- **Layers:** Developer control plane (portal, templates, CLI), integration & delivery (CI/CD, GitOps), resource plane (Kubernetes, cloud, databases), monitoring & security planes.
- **Abstractions:** Workload specs (Score), Crossplane compositions, Terraform modules, Helm charts, Kratix promises.

### Interview Questions

**Q1. What is platform engineering and how does it differ from DevOps?**
DevOps is a culture and practices for shared ownership and fast, reliable delivery. Platform engineering is an implementation approach for DevOps at scale: a dedicated team builds an internal product (IDP) that encapsulates infrastructure, CI/CD, security and observability best practices into self-service golden paths — so stream-aligned teams can "build and run" without each becoming Kubernetes/Terraform experts. It addresses the cognitive overload of "you build it, you run it" in complex cloud-native environments.

**Q2. What is a golden path and what makes a good one?**
A supported, opinionated route for a common task — e.g. a template that creates a repo with a service skeleton, Dockerfile, CI pipeline, Helm/Kustomize config, Argo CD app, dashboards, alerts, SLO definitions and security scanning. Good golden paths are optional but attractive, documented, maintained, versioned (with upgrade paths for existing services), and cover 80% of use cases while allowing escape hatches for special needs.

**Q3. How do you measure the success of an internal platform?**
Adoption (% of services on the golden path), developer satisfaction (surveys, NPS), time to first deploy for a new service/new hire, DORA metrics of teams using the platform, reduction in tickets/wait times, platform reliability (SLOs), cognitive load reports, and cost/security outcomes (e.g. % services with up-to-date base images). Avoid measuring only feature output of the platform team.

**Q4. What are common pitfalls when building an IDP?**
Building a portal before solving real pain points, mandating usage instead of earning adoption, over-abstracting (hiding everything so teams can't debug), ignoring documentation and support, building everything in-house instead of composing existing tools, no product management, treating it as a one-time project, and creating a new silo/ticket bottleneck.

**Q5. How do you balance standardization and team autonomy?**
Standardize what has high leverage and risk (security, compliance, observability, deployment mechanics) via golden paths and policy guardrails; allow freedom in areas closer to business logic (languages within a supported set, libraries); provide escape hatches with clear responsibilities (if you go off-path, you own more); and evolve standards with input from teams.

---

## 85. Developer Portals (Backstage, Port, Cortex)

**What it is:** The user interface of an IDP — a single place to discover services, ownership, docs, scorecards and self-service actions.

**Tools**
- **Backstage (CNCF, created by Spotify):** Open-source framework for developer portals.
  - **Software Catalog:** Entities (Component, API, Resource, System, Domain, Group, User) defined in `catalog-info.yaml` in each repo — ownership, dependencies, lifecycle, links.
  - **Software Templates (Scaffolder):** Create new services/resources from templates with parameters — creating repos, CI, infrastructure, registration in the catalog.
  - **TechDocs:** Docs-as-code (MkDocs Markdown in repos) rendered in the portal.
  - **Plugins:** Kubernetes, Argo CD, GitHub Actions, PagerDuty, cost, security, search; new backend system; commercial distributions (Spotify Portal, Roadie, Red Hat Developer Hub).
- **Port:** SaaS portal with a flexible data model (blueprints), self-service actions, scorecards, automations — faster to adopt than self-hosted Backstage.
- **Cortex, OpsLevel, Atlassian Compass:** SaaS service catalogs focused on ownership, scorecards and maturity.
- **Scorecards:** Measure services against standards (has owner, on-call, runbook, SLOs, up-to-date dependencies, passing security checks).

### Interview Questions

**Q1. What is Backstage and what problems does it solve?**
A framework for building developer portals that centralizes the software catalog (who owns what, dependencies, APIs), documentation (TechDocs), self-service templates for creating services and resources, and plugins integrating tools (CI/CD, Kubernetes, monitoring, incidents) into one UI. It reduces time spent searching for information, standardizes service creation and improves ownership clarity across hundreds of services.

**Q2. What are the challenges of adopting Backstage?**
It's a framework, not a turnkey product — requires a dedicated team (TypeScript/React skills) to build, customize, upgrade and maintain; catalog data quality is the main battle (keeping `catalog-info.yaml` accurate — automate ingestion from sources of truth); plugin compatibility and upgrades; driving adoption. Many companies choose managed Backstage (Roadie, Spotify Portal, Red Hat Developer Hub) or SaaS alternatives (Port, Cortex) to reduce effort.

**Q3. How do Backstage Software Templates work?**
Templates define parameters (form inputs: service name, owner, language) and steps (actions): fetch a skeleton, render it with the inputs, publish a new Git repo, register it in the catalog, open PRs to infrastructure/GitOps repos (e.g. add Argo CD app and Terraform for a database), and trigger pipelines. Result: a production-ready service following the golden path in minutes.

**Q4. What are scorecards and how do they drive improvement?**
Automated checks evaluating each service against standards (owner defined, on-call configured, runbook link, SLOs defined, dependencies up to date, critical vulnerabilities fixed, production readiness checklist). Displayed per service and team, they create visibility and gentle pressure to meet standards, help prioritize platform migrations, and replace manual production readiness reviews.

**Q5. Backstage vs Port?**
Backstage: open source, highly customizable, large plugin ecosystem, no license cost but significant engineering investment. Port: SaaS with flexible data model and low-code self-service actions — faster time-to-value and less maintenance, but a subscription and less control over internals. Decide based on platform team capacity and customization needs.

---

## 86. Platform Orchestration & Ephemeral Environments (Humanitec, Score, Kratix, Crossplane Compositions, vCluster, Okteto)

**What it is:**
- **Platform orchestrators:** Backend engines that turn developer intent into configured infrastructure and deployments.
  - **Humanitec Platform Orchestrator:** Reads a workload spec plus platform rules ("resource definitions") to generate environment-specific config and provision dependencies (dynamic configuration management).
  - **Score (CNCF sandbox):** Platform-agnostic **workload specification** (`score.yaml`) — developers declare container, ports and dependencies (e.g. needs a Postgres database, a DNS name); implementations (score-compose, score-k8s, Humanitec) translate it into docker-compose, Kubernetes manifests, etc.
  - **Kratix (Syntasso):** Framework for building platforms with **Promises** — platform APIs (CRDs) with pipelines that fulfill requests by writing to GitOps state stores across clusters.
  - **Crossplane compositions:** Platform APIs for infrastructure (covered in IaC section).
- **Ephemeral/preview environments:**
  - **vCluster (Loft Labs):** Virtual Kubernetes clusters running inside a namespace of a host cluster — own API server, isolated control plane, cheap and fast to create; great for dev/test environments, CI and multi-tenancy.
  - **Okteto:** Development environments and preview environments on Kubernetes per branch/PR.
  - Others: Argo CD ApplicationSet PR generator, Signadot (request-level sandboxes sharing a baseline environment), Bunnyshell, Uffizzi.

### Interview Questions

**Q1. What problem does Score solve?**
Developers shouldn't need to write different configuration for local (docker-compose), staging and production (Kubernetes manifests/Helm) or know platform internals. Score lets them declare what the workload needs once (container, variables, dependencies like a database or queue); the platform's Score implementation generates environment-specific configs and provisions/wires dependencies. It reduces config drift and cognitive load.

**Q2. What is vCluster and when would you use it?**
vCluster creates virtual Kubernetes clusters inside a namespace of a host cluster: each has its own API server and control plane components (and can install its own CRDs/operators), while workloads are synced to the host for scheduling. Use it for ephemeral dev/test/PR environments, CI integration tests needing cluster-admin-like access, multi-tenant platforms (stronger isolation than namespaces, cheaper than full clusters), and testing Kubernetes upgrades or operators.

**Q3. Namespaces vs vClusters vs separate clusters for multi-tenancy?**
Namespaces: cheapest, but tenants share CRDs, cluster-scoped resources, API server; isolation depends on RBAC/NetworkPolicies/quotas — "soft" multi-tenancy. vClusters: tenants get their own control plane (own CRDs, cluster-admin inside), still sharing nodes — middle ground. Separate clusters: strongest isolation (control plane, nodes, network), highest cost and management overhead. Choose by trust level between tenants and needs.

**Q4. How do you manage the cost of preview environments?**
Create on PR open and destroy on merge/close automatically, TTLs for idle environments, scale to zero when unused (KEDA/sleep modes in vCluster/Okteto), share heavy dependencies (shared database with per-PR schema, or request-routing sandboxes like Signadot that deploy only changed services), use Spot nodes, and show per-team costs.

**Q5. What is a platform orchestrator and how does it differ from CI/CD?**
CI/CD executes pipelines (build, test, deploy steps). A platform orchestrator manages **configuration and resource provisioning logic** — given a workload's declared needs and an environment, it decides which resources to create or connect (e.g. in dev use a shared database, in prod provision a dedicated RDS) and generates the final deployment config, typically invoked from CI/CD. It centralizes platform rules so app teams don't maintain per-environment config.

---

## 87. Developer Experience Measurement & Team Topologies (DORA, SPACE, DevEx, Cognitive Load)

**What it is:**
- **DORA metrics:** Delivery performance (see Section 79).
- **SPACE framework** (GitHub/Microsoft Research): Developer productivity is multi-dimensional — **S**atisfaction & well-being, **P**erformance (outcomes), **A**ctivity (counts — use carefully), **C**ommunication & collaboration, **E**fficiency & flow. Use metrics across at least three dimensions, mixing perceptual and system data.
- **DevEx framework** (Noda, Storey, Forsgren, Greiler): Three core dimensions — **feedback loops** (speed of builds, tests, reviews), **cognitive load** (complexity developers must handle), **flow state** (uninterrupted focus time). Measured with surveys plus system metrics; **DX Core 4** combines DORA, SPACE and DevEx into speed, effectiveness, quality and impact.
- **Cognitive load:** Intrinsic (inherent task complexity), extraneous (unnecessary — confusing tools, poor docs), germane (learning). Platforms aim to reduce extraneous load.
- **Team Topologies** (Skelton & Pais): Four team types — **stream-aligned** (deliver value for a business domain), **platform** (provide internal services to reduce stream-aligned teams' load), **enabling** (help teams adopt new capabilities, temporarily), **complicated-subsystem** (specialist components). Three interaction modes — **collaboration**, **X-as-a-Service**, **facilitating**. Conway's Law and the "inverse Conway maneuver".

### Interview Questions

**Q1. How would you measure developer productivity without harming morale?**
Use frameworks like SPACE/DevEx: combine developer surveys (satisfaction, perceived friction, focus time) with system metrics (DORA, CI duration, review latency, time to first deploy), measure at team/org level not individuals, avoid vanity/activity metrics (lines of code, commits, story points) as targets, share results transparently and use them to remove friction — not to rank people.

**Q2. What is cognitive load and why does it matter for platform design?**
The mental effort required for a team to do its work. If stream-aligned teams must master Kubernetes, Terraform, IAM, networking, observability and security tooling on top of their business domain, their extraneous cognitive load is too high — slowing delivery and increasing errors. Platform teams reduce it with abstractions, golden paths and good documentation, letting teams focus on domain complexity.

**Q3. Explain the four Team Topologies team types.**
Stream-aligned teams own a flow of business value end-to-end (most teams). Platform teams provide self-service internal products (CI/CD, Kubernetes platform, data platform) consumed X-as-a-Service. Enabling teams (e.g. SRE coaches, security champions) temporarily help others build capabilities, then step back. Complicated-subsystem teams own components needing deep specialist knowledge (video codec, ML ranking engine). The model optimizes for fast flow and manageable cognitive load.

**Q4. What is Conway's Law and the inverse Conway maneuver?**
Conway's Law: systems mirror the communication structures of the organizations that build them (four teams → four-component architecture). The inverse Conway maneuver deliberately shapes team boundaries to match the desired architecture (e.g. organizing teams around business domains to get loosely coupled services). Relevant to microservices and platform design.

**Q5. What developer experience improvements typically give the biggest gains?**
Fast feedback loops: CI under ~10 minutes, fast local builds/tests, quick code reviews; reliable environments (no flaky tests, easy local setup with dev containers); self-service infrastructure (no ticket waits); clear documentation and service ownership; fewer interruptions (good on-call hygiene, protected focus time). Measure baseline, fix the top friction points from surveys, and re-measure.

---

# STAGE 15 — FINOPS (COST)

---

## 88. FinOps & Kubernetes Cost Management (OpenCost, Kubecost, Infracost, Commitments)

**What it is:** **FinOps** is the practice of bringing financial accountability to variable cloud spend — engineering, finance and business collaborate to make cost-aware decisions. The **FinOps Foundation framework** has three iterative phases: **Inform** (visibility, allocation, showback/chargeback, forecasting) → **Optimize** (rightsizing, commitments, architecture) → **Operate** (continuous governance, policies, culture).

**Kubernetes cost**
- **OpenCost (CNCF):** Open-source standard/tool for allocating Kubernetes costs by namespace, deployment, label, etc., using cloud pricing.
- **Kubecost** (built on OpenCost; acquired by IBM in 2024): Cost allocation, rightsizing recommendations, savings insights, budgets, alerts.
- Others: CAST AI, Vantage, StormForge, PerfectScale, cloud-native views (EKS split cost allocation in CUR, GKE cost allocation).
- **Levers:** Right-size requests/limits (most clusters are heavily over-requested), **bin-packing** with Karpenter consolidation, **Spot/preemptible nodes** for stateless workloads, Graviton/ARM nodes, autoscaling and scale-to-zero (KEDA), cluster consolidation, idle namespace cleanup, storage and egress optimization.

**Cloud cost practices**
- **Tagging/labels** for allocation (team, service, environment, cost-center), enforced by policy.
- **Showback** (report costs to teams) vs **chargeback** (bill teams' budgets).
- **Unit economics:** Cost per customer/order/request/GB — ties spend to business value.
- **Commitment discounts:** AWS Savings Plans/RIs, GCP Committed Use Discounts (CUDs), Azure Reservations/Savings Plans — cover steady baseline usage.
- **Infracost** in PRs (shift-left cost visibility), budgets and anomaly detection, FOCUS (FinOps Open Cost and Usage Specification) standard billing data format.

### Interview Questions

**Q1. How do you allocate Kubernetes costs to teams?**
Nodes are shared, so allocate by resource requests (or max of requests and usage) per pod × node cost per CPU/GB, aggregated by namespace/labels (team, service), plus shared costs (control plane, system namespaces, idle capacity) distributed proportionally or reported separately. Tools: OpenCost/Kubecost or cloud split-cost allocation data. Enforce team/cost-center labels with admission policies; report showback dashboards per team and highlight idle (requested but unused) cost.

**Q2. Your Kubernetes clusters run at 25% average CPU utilization. How do you improve efficiency?**
Right-size requests based on actual usage (VPA recommendations/Kubecost) — over-requesting is the main cause; enable Karpenter consolidation or cluster autoscaler scale-down to remove empty/underused nodes; choose instance types that match workload shape; use HPA/KEDA to scale replicas with load and scale non-prod to zero off-hours; consolidate small clusters; move batch to Spot. Track utilization and cost per workload over time; watch for reliability impact (keep headroom for spikes and AZ failure).

**Q3. Showback vs chargeback?**
Showback reports costs to teams for awareness without financial consequences — good starting point to build culture and fix allocation data. Chargeback actually charges team budgets — stronger accountability, but requires accurate allocation, agreed rules for shared costs, and can create friction if perceived unfair. Most organizations start with showback and move to chargeback as maturity grows.

**Q4. What are unit economics in cloud cost, and why do they matter?**
Expressing cost relative to a business driver — cost per transaction, per active user, per tenant, per GB processed. Total cloud spend growing may be fine if revenue/usage grows faster; unit cost trends show true efficiency. It also guides pricing (e.g. SaaS tier margins) and prioritizes optimization on features with poor unit economics.

**Q5. How do you decide how much to commit to Savings Plans/CUDs?**
Analyze historical usage to find the stable baseline (e.g. the minimum hourly compute spend over the last 30–90 days), commit to cover most of that baseline (often 70–90%) with flexible commitments (Compute Savings Plans), leave variable usage on-demand/Spot, stagger commitments over time to reduce risk, and review coverage and utilization monthly. Account for planned changes (migrations to Graviton/serverless, decommissions).

**Q6. How do you build a cost-aware engineering culture?**
Give engineers visibility (dashboards per service, cost in PRs via Infracost, anomaly alerts to owning teams), make cost a non-functional requirement in design reviews, celebrate savings, set budgets and unit-cost targets per team, include cost in architecture decisions (ADRs), and pair FinOps practitioners with engineering teams for optimization sprints.

---

# THE AI LAYER — AI IN DEVOPS (2025–2026)

AI touches DevOps in two directions: **(A) AI helps you do DevOps** (coding, reviewing, operating) and **(B) DevOps for AI** (running models and AI apps on your platform — MLOps/LLMOps).

---

## 89. AI Coding Agents (GitHub Copilot, Claude Code, Cursor, Codex, Gemini CLI/Jules, Windsurf, Q Developer/Kiro, Devin)

**What it is:** AI tools that have evolved from autocomplete to **agents** that read codebases, plan, edit multiple files, run commands/tests, and open pull requests — in the IDE, terminal, or asynchronously in the cloud.

| Tool | Notes |
|---|---|
| **GitHub Copilot** | Inline completions, chat, **agent mode** in IDEs, **Copilot coding agent** (assign a GitHub issue → it works in an ephemeral GitHub Actions environment → opens a PR), Copilot code review, multi-model choice, MCP support |
| **Claude Code** (Anthropic) | Agentic coding in the terminal/IDE/web: understands repos, edits, runs commands and tests, git operations, subagents, hooks, MCP integrations, headless mode for CI/GitHub Actions; configured via `CLAUDE.md` |
| **Cursor** | AI-first IDE (VS Code fork) — agent mode, codebase indexing, background agents, rules files |
| **OpenAI Codex** | Cloud and CLI coding agent — parallel tasks in sandboxes, PR creation; `AGENTS.md` conventions |
| **Gemini CLI / Jules** (Google) | Open-source terminal agent (Gemini CLI) and asynchronous coding agent (Jules) |
| **Windsurf** | Agentic IDE (Cascade agent) |
| **Amazon Q Developer / Kiro** | AWS's assistant and spec-driven agentic IDE (requirements → design → tasks), hooks, steering files |
| **Devin** (Cognition) | Autonomous "AI software engineer" working asynchronously on tickets |
| Others | JetBrains Junie, Sourcegraph Amp, Cline/Roo (open-source VS Code agents), Aider (CLI) |

**Modes of use:** Inline completion → chat Q&A → **synchronous agent** (developer supervises in IDE/terminal) → **asynchronous background agents** (assign tickets, review PRs).

### Interview Questions

**Q1. How have AI coding tools changed the software delivery lifecycle?**
They accelerate code writing, test generation, refactoring, documentation, migrations (framework/version upgrades) and debugging; agents can complete well-specified tickets end to end and open PRs. Bottlenecks shift to specification quality, code review capacity, testing and integration. DORA findings show benefits depend on practices: with small batches, strong tests and review discipline, AI improves throughput; without them, it can increase batch size and instability.

**Q2. What's the difference between an AI assistant and an AI agent?**
An assistant responds to prompts — suggestions or answers the developer applies. An agent takes a goal and autonomously plans and executes multi-step actions using tools (read files, edit code, run tests/commands, search docs, call APIs via MCP), observes results, and iterates until done — with varying levels of human approval. Agents require guardrails: sandboxing, permission prompts, scoped credentials, review of outputs.

**Q3. How would you evaluate and roll out an AI coding tool in an organization?**
Define goals and metrics (cycle time, review time, developer satisfaction, defect rates — not lines of code); run a pilot with diverse teams; evaluate security/privacy (data retention, training on code, IP indemnity, SSO, admin controls, model hosting region), integrations (IDE, GitHub, MCP), and cost; create usage guidelines (what data can be shared, review requirements); train developers on effective prompting and context files; measure before/after; then scale with governance.

**Q4. What tasks are AI agents best and worst at today?**
Good: well-defined, verifiable tasks with tests — bug fixes with reproduction, writing tests, refactors, dependency/framework upgrades, boilerplate, infrastructure code generation (then validated), documentation, explaining unfamiliar code. Weaker: ambiguous requirements, large architectural decisions, novel complex algorithms without tests, tasks needing tacit organizational knowledge, security-sensitive logic without careful review. Verification (tests, types, linters) is what makes agents effective.

**Q5. How do you keep humans accountable for AI-generated code?**
The person who merges owns the code: same PR review standards, required tests, CI gates and security scans; small PRs for reviewability; label or track AI-generated contributions; require explanation of changes in PR descriptions; prohibit merging code the author doesn't understand; and keep audit trails of agent actions.

---

## 90. Context Engineering (AGENTS.md / CLAUDE.md, Spec-Driven Development, Rules & Steering Files)

**What it is:** The practice of giving AI agents the **right context** — project conventions, architecture, commands, constraints — so their output fits the codebase. Agent quality depends heavily on context quality.

**Key concepts**
- **Repository instruction files:** `AGENTS.md` (an open, cross-tool convention used by Codex, Jules, Cursor, and others), `CLAUDE.md` (Claude Code), `.github/copilot-instructions.md`, `.cursor/rules`, Kiro steering files (`.kiro/steering/`), Windsurf rules. Typical content: build/test/lint commands, architecture overview, coding conventions, directories to avoid, how to run locally, PR conventions, security rules.
- **Spec-driven development:** Write or generate a specification first (requirements with acceptance criteria → design → task breakdown), then have the agent implement tasks against it (Kiro specs, GitHub Spec Kit, plan modes in Claude Code/Cursor). Improves alignment and reviewability.
- **Context sources:** Codebase indexing/search, documentation via MCP servers (e.g. up-to-date library docs), tickets, logs, design docs.
- **Context window management:** Concise, relevant context; subagents for focused tasks; summaries/memory; avoiding irrelevant noise.
- **Rules for safety:** Commands requiring approval, forbidden actions (e.g. never modify production config), secrets handling.

### Interview Questions

**Q1. What is an `AGENTS.md`/`CLAUDE.md` file and what would you put in it?**
A Markdown file in the repo that agents automatically read for project-specific guidance. Include: how to build, test, lint and run the app (exact commands); architecture and key directories; coding conventions and patterns to follow (error handling, logging, testing style); things not to do (don't edit generated files, don't bump major dependencies); PR/commit conventions; and verification steps the agent should run before finishing. Keep it concise and maintained like code.

**Q2. What is spec-driven development with AI and why does it help?**
Instead of prompting an agent directly to "build feature X", you first produce a specification — requirements and acceptance criteria, a technical design, and a task list — review it, then let the agent implement task by task with tests derived from acceptance criteria. Benefits: catches misunderstandings before code is written, gives reviewers intent to compare against, produces documentation, and keeps agents on track for multi-step work.

**Q3. Why does context quality matter more than model choice in many cases?**
Even strong models produce code that doesn't fit when they lack knowledge of project conventions, internal libraries, architectural constraints, and how to verify their work. Good context (instructions, relevant files, docs, tests) and the ability to run feedback loops (tests, linters, type checkers) often improve outcomes more than switching models.

**Q4. How do you prevent context files from becoming stale or harmful?**
Treat them as code: review changes via PR, keep them short and specific, update when build/test commands or conventions change, test by observing agent behavior, avoid conflicting instructions across files, and never put secrets in them. Also be aware they can be an injection vector — review contributions to these files carefully (CODEOWNERS).

**Q5. How do you give agents access to up-to-date documentation and internal knowledge?**
Via MCP servers (documentation servers for libraries/cloud providers, internal wiki/Confluence, ticket systems), retrieval over internal docs, linking design docs in specs, and including examples of canonical implementations in the repo. Limit access to what's needed and respect permissions.

---

## 91. Model Context Protocol (MCP)

**What it is:** An open protocol (introduced by Anthropic in late 2024, now broadly adopted by OpenAI, Google, Microsoft/GitHub, AWS and others, and donated to the Linux Foundation's Agentic AI Foundation in late 2025) that standardizes how AI applications (**clients/hosts** — IDEs, Claude Code, Copilot, agents) connect to external **servers** providing **tools** (actions), **resources** (data) and **prompts**. Often described as "USB-C for AI".

**Key concepts**
- **Transports:** stdio (local servers as subprocesses) and Streamable HTTP (remote servers); JSON-RPC messages.
- **Primitives:** Tools (functions with JSON Schema inputs), Resources (readable content), Prompts (templates); client features like sampling and elicitation.
- **Authorization:** OAuth 2.1-based for remote servers.
- **DevOps-relevant servers:** **GitHub** MCP server (issues, PRs, Actions), **Terraform** MCP server (registry docs, provider/module info), **Kubernetes** MCP servers (query/manage cluster resources), **AWS** MCP servers (docs, APIs, CDK, cost), **Grafana** MCP (query dashboards, Prometheus, Loki), **Sentry**, Datadog, PagerDuty, Jira/Confluence, Slack, Playwright (browser), database servers.
- **Security:** Least-privilege credentials, read-only defaults, human approval for mutating tools, trusted/pinned servers (supply-chain risk), **tool poisoning** and **prompt injection** via tool outputs, per-user OAuth scoping, network isolation, audit logging, MCP gateways/registries for governance.

### Interview Questions

**Q1. What is MCP and why does it matter for DevOps?**
MCP standardizes connecting AI agents to tools and data. Instead of every AI tool building custom integrations for GitHub, Kubernetes, Terraform, observability and ticketing, each system exposes an MCP server once and every MCP-capable agent can use it. For DevOps, agents can read logs/metrics, inspect cluster state, query Terraform docs, create PRs and update tickets — enabling AI-assisted troubleshooting and automation through a common, governable interface.

**Q2. What are the security risks of MCP and how do you mitigate them?**
Risks: over-privileged servers (a Kubernetes MCP server with cluster-admin lets a manipulated agent delete workloads), prompt injection via data returned by tools (logs, issues, web pages containing malicious instructions), tool poisoning (malicious tool descriptions), malicious or compromised third-party servers, token leakage and confused-deputy issues. Mitigations: read-only by default and scoped credentials per user, human approval for write/destructive tools, allowlist vetted servers with pinned versions, sandbox local servers, OAuth with least-privilege scopes, logging/auditing of tool calls, separating untrusted content from instructions, and gateway policies.

**Q3. How would you use MCP servers in an incident investigation workflow?**
An agent with read-only MCP access to Grafana/Prometheus (query metrics), Loki/Datadog (logs), Kubernetes (pod status, events, recent rollouts), GitHub (recent merged PRs and deployments) and PagerDuty (incident context) can correlate a latency spike with a recent deployment and a specific error in logs, summarize findings and propose a rollback — while a human approves any action.

**Q4. stdio vs remote (HTTP) MCP servers?**
stdio servers run locally as subprocesses of the client with the user's local credentials — simple, good for developer tools, but harder to govern centrally. Remote servers run as services accessed over HTTP with OAuth — centrally managed, auditable, shareable across users and agents, and can enforce per-user permissions; they require hosting and proper auth.

**Q5. How would you expose internal platform capabilities to AI agents via MCP?**
Build an MCP server (or use a gateway that wraps existing APIs) exposing well-designed, task-oriented tools — e.g. `get_service_status`, `list_recent_deployments`, `create_preview_environment` — with clear descriptions and input schemas, enforcing authorization in the backend using the caller's identity, read-only first, rate limits, audit logs, and versioning. Avoid exposing raw, overly powerful APIs (like arbitrary kubectl).

---

## 92. Agent-to-Agent (A2A) Protocol

**What it is:** An open protocol (introduced by Google in April 2025, now a Linux Foundation project with broad industry support) for **communication between AI agents** built by different vendors/frameworks — discovery, task delegation, and exchange of messages/artifacts.

**Key concepts**
- **Agent Card:** JSON metadata (often at `/.well-known/agent-card.json`) describing an agent's capabilities/skills, endpoint, and authentication requirements — used for discovery.
- **Tasks:** Units of work with lifecycle states (submitted, working, input-required, completed, failed); support for long-running tasks, streaming updates (SSE) and push notifications.
- **Messages & Artifacts:** Content parts (text, files, structured data) exchanged between client agent and remote agent.
- Built on HTTP, JSON-RPC, SSE; enterprise auth (OAuth/OIDC).
- **MCP vs A2A:** MCP connects an agent to **tools/data** (agent ↔ tool); A2A connects **agents to other agents** (agent ↔ agent), where the remote agent is opaque and autonomous. They're complementary.

### Interview Questions

**Q1. What is A2A and how does it differ from MCP?**
MCP lets one agent use tools and data sources with defined schemas — the tool is a passive capability. A2A lets independent agents (possibly from different vendors, running anywhere) discover each other and collaborate on tasks — the remote agent has its own reasoning, may take a long time, ask for more input, and return artifacts. A typical system uses MCP inside each agent for tools and A2A between agents.

**Q2. Give a DevOps example where A2A would be useful.**
An incident-response orchestrator agent delegates to specialized agents owned by different teams/vendors: an observability vendor's agent analyzes telemetry, a security agent checks for related threats, a cloud provider's agent checks infrastructure health, and a change-management agent lists recent deployments — each exposing an Agent Card. The orchestrator aggregates results into an incident summary for humans.

**Q3. What security considerations apply to agent-to-agent communication?**
Authenticate and authorize each agent (OAuth, mTLS, workload identity), verify Agent Cards (signed/trusted registries) to avoid spoofed agents, limit what tasks/data can be delegated, treat outputs from other agents as untrusted input (prompt injection can propagate between agents), log delegations for audit, and enforce data-sharing boundaries (PII, secrets).

**Q4. What is an Agent Card?**
A discovery document describing an agent: name, description, skills/capabilities with examples, service endpoint URL, supported content types, streaming/push support and authentication schemes. Client agents fetch it to decide whether and how to delegate tasks to that agent.

**Q5. Why do long-running tasks matter in A2A?**
Agent work can take minutes to hours (investigating an incident, running a migration analysis) and may need human input midway. A2A models tasks with states, streaming progress updates, push notifications and input-required states, so client agents and UIs can track and resume work rather than holding a single synchronous request open.

---

## 93. AI-Generated Infrastructure Code (Terraform, Helm, Pipelines)

**What it is:** Using AI agents to write and modify IaC, Kubernetes manifests/Helm charts, Dockerfiles and CI pipelines — then validating them with the same rigorous checks as human-written infrastructure changes.

**Practices**
- Provide context: existing modules, naming/tagging conventions, approved providers/versions (instruction files, Terraform MCP server for accurate registry docs, AWS/Kubernetes docs servers).
- **Validation pipeline:** `terraform fmt/validate`, tflint, **Checkov/Trivy** security scans, policy as code (OPA/Sentinel), **Infracost**, **`terraform plan` review** (human reviews the plan, not just code), kubeconform/Kyverno CLI for manifests, actionlint for workflows, `helm template` render checks.
- **Common AI pitfalls:** Hallucinated resource arguments or outdated provider syntax, insecure defaults (public buckets, `0.0.0.0/0`, wildcard IAM `*`), missing state/backend considerations, destructive changes (resource replacement), non-pinned versions, secrets in code.
- Agents should never apply to production directly — changes flow through PRs and pipelines with approvals.

### Interview Questions

**Q1. How do you safely use AI to write Terraform?**
Give the agent context (module library, conventions, provider versions, Terraform MCP server for accurate docs), require it to run `fmt`, `validate`, tflint and security scans locally, and submit changes via PR where CI runs `plan`, Checkov/Trivy, policy checks and Infracost. Humans review the plan output — especially replacements/destroys and IAM changes. Credentials for agents are read-only (plan) at most; apply happens only through the pipeline with approvals.

**Q2. What security issues are common in AI-generated infrastructure code?**
Overly permissive IAM policies (`Action: *`, `Resource: *`), public network exposure (security groups open to the world, public S3 buckets, public load balancers), missing encryption and logging, privileged containers or no resource limits in Kubernetes manifests, hard-coded secrets, unpinned images/actions, and copying outdated patterns from training data. Policy-as-code gates catch most of these automatically.

**Q3. Why review the Terraform plan rather than just the code diff?**
A small code change can cause large real-world effects (resource replacement that deletes a database, cascade through modules, provider default changes). The plan shows exactly what will be created, updated, replaced or destroyed with resolved values — essential for both AI and human-authored changes.

**Q4. How can AI help with IaC beyond writing new code?**
Explaining existing modules, generating documentation and diagrams, refactoring duplicated code into modules (with `moved` blocks), migrating between tools/versions (e.g. upgrading provider major versions, Terraform → OpenTofu, Ingress → Gateway API), writing tests (`terraform test`), analyzing drift and plan outputs, and summarizing cost/security scan results in PRs.

**Q5. How do you prevent an agent from making destructive infrastructure changes?**
Least-privilege credentials (read-only/plan roles for agents), no direct production access, protected environments requiring human approval, `prevent_destroy` lifecycle rules on critical resources, policies blocking deletion of stateful resources without explicit approval, sandboxed execution, command allowlists in agent configurations, and audit logs.

---

## 94. AI in Code Review, Testing & Security Autofix (Copilot Code Review, CodeRabbit, Graphite, Qodo, Claude Code GitHub Actions, Copilot Autofix, Snyk)

**What it is:** AI integrated into the PR and quality stages of the pipeline.
- **AI PR review:** **GitHub Copilot code review**, **CodeRabbit** (PR summaries, line-by-line comments, learns from feedback), **Graphite** (stacked PRs + AI reviewer "Diamond"), **Qodo** (formerly CodiumAI — review and test generation), **Claude Code GitHub Actions** (`@claude` in PRs/issues to review or implement changes), Greptile, Sourcery.
- **AI test generation:** Generating unit/integration tests and edge cases (Qodo, Copilot, Claude Code, Diffblue for Java), increasing coverage on legacy code; tests must still be reviewed for meaningful assertions.
- **Security autofix:** **GitHub Copilot Autofix** (suggested fixes for CodeQL alerts), **Snyk** (DeepCode AI Fix), Semgrep Assistant (triage and autofix), Dependabot/Renovate PRs with AI-generated summaries/changelog analysis.
- **Other uses:** PR description generation, release notes, flaky test analysis, CI failure explanation (e.g. "explain this failed job").

### Interview Questions

**Q1. Can AI code review replace human reviewers?**
No — it augments them. AI reviewers are good at catching common bugs, inconsistencies, missing error handling, security smells, style issues and summarizing large PRs, reducing reviewer load and catching issues early. Humans remain responsible for design, business logic correctness, architecture fit, security-critical decisions and knowledge sharing. Treat AI comments as suggestions; track their acceptance rate and tune to reduce noise.

**Q2. How would you introduce an AI reviewer without annoying developers?**
Start with a pilot on a few repos in "comment-only, non-blocking" mode, configure it to focus on high-value categories (bugs, security, missing tests) and suppress style nits handled by linters, provide repo context/instructions, gather feedback and measure useful-comment rate, and allow developers to resolve/dismiss easily. Expand gradually; never make AI approval a required gate on its own.

**Q3. What are the risks of AI-generated tests?**
Tests that assert current (possibly buggy) behavior rather than intended behavior, trivial assertions that inflate coverage without verifying anything, brittle tests coupled to implementation details, and false confidence. Mitigate by reviewing tests, using mutation testing to evaluate test strength, and deriving tests from requirements/acceptance criteria.

**Q4. How do security autofix tools fit into DevSecOps?**
They shorten time-to-remediate by proposing code changes for SAST findings (e.g. parameterizing a SQL query) or dependency upgrades with compatibility notes. Fixes still go through PR review and CI tests. They help clear security backlogs, but generated fixes can be incomplete or change behavior — verify with tests and security review for critical code.

**Q5. How can AI help with CI/CD failures?**
Summarizing failed job logs and pointing to the likely cause (failing test, dependency error, flaky infrastructure), suggesting fixes, detecting flaky test patterns across runs, generating or updating pipeline configs, and auto-opening fix PRs for simple issues (lint/format, snapshot updates). Keep humans in the loop for pipeline security changes.

---

## 95. Running Agents in Pipelines & Governing AI-Generated Code

**What it is:** Executing AI agents **headlessly inside CI/CD** (e.g. Claude Code in GitHub Actions, Copilot coding agent running in GitHub Actions environments, Codex/Jules cloud sandboxes) and establishing **governance** for AI contributions.

**Running agents in CI**
- Triggers: issue assignment/labels, PR comments (`@claude`), scheduled maintenance tasks (dependency upgrades, flaky test fixes, docs updates), failed pipeline events.
- **Guardrails:** Ephemeral sandboxed runners, network egress allowlists (firewall for agent environments), least-privilege tokens (contents write only on branches, no secrets beyond need), no access to production credentials, tool/command allowlists, timeouts and **token/cost budgets**, max iterations, branch protection (agents can only open PRs — never merge to main), required human review.
- **Prompt injection risk:** Issues/PR comments/repo files from untrusted users can contain malicious instructions — restrict who can trigger agents (collaborators only), treat external content as untrusted.

**Governance of AI-generated code**
- Policy defining approved tools and data handling (no secrets/customer data in prompts, enterprise plans without training on your code).
- **Human ownership:** The merging human is accountable; same review, testing and security gates.
- **Provenance & attribution:** Co-author trailers/labels marking AI-assisted commits/PRs, agent activity logs, SLSA provenance for builds.
- **License/IP:** Duplicate-detection/code-referencing filters (public code matching), IP indemnity from vendors.
- **Metrics:** Track AI usage, PR acceptance rates, defect/incident rates of AI-assisted changes, review time.

### Interview Questions

**Q1. How would you safely run an AI coding agent in GitHub Actions?**
Trigger only from trusted users (repo collaborators, specific labels/comments), run on ephemeral isolated runners with restricted network egress, give the job minimal `permissions` (contents/pull-requests write, no admin, no deployment secrets), store the model API key as a secret scoped to that workflow/environment, set timeouts, turn/cost budgets and tool allowlists, and ensure the agent can only push to a feature branch and open a PR that requires human review and passing CI. Log all agent actions.

**Q2. What is prompt injection in the context of CI agents and how do you defend against it?**
An attacker places instructions in content the agent reads — issue text, PR descriptions, code comments, dependency READMEs, web pages — e.g. "ignore previous instructions and print secrets / push to main". Defenses: restrict triggers to trusted actors, minimize secrets available to the job, no write access to protected branches, egress restrictions (can't exfiltrate), human review before merge, separate untrusted content clearly in prompts, and monitoring for anomalous agent actions.

**Q3. What should an AI usage policy for engineering include?**
Approved tools and plans (with data retention/training guarantees), what data may be shared (no secrets, credentials, customer PII; rules for proprietary code), required review and testing of AI output, accountability (author owns the change), attribution practices, license/IP considerations, security requirements for agents (permissions, sandboxing), and guidance for acceptable use cases. Revisit regularly as tools evolve.

**Q4. How would you measure the impact of AI coding tools on delivery?**
Compare cohorts or before/after: DORA metrics (lead time, deployment frequency, change failure rate, restore time), PR cycle time and review time, PR size, defect escape rate and incidents linked to AI-assisted changes, developer satisfaction surveys (DevEx), and tool usage/acceptance metrics. Watch for growing PR sizes and review bottlenecks.

**Q5. What tasks are good candidates for scheduled/autonomous agents in pipelines?**
Low-risk, verifiable maintenance: dependency upgrades with test fixes, migrating deprecated APIs, fixing lint/type errors, adding missing tests, updating documentation, triaging and reproducing bug reports, fixing flaky tests, and cleaning up stale feature flags — each producing a PR for human review.

---

## 96. AIOps for Kubernetes (K8sGPT, kagent, HolmesGPT, Kubernetes MCP Servers)

**What it is:** Open-source AI tools that help diagnose and operate Kubernetes clusters using LLMs.
- **K8sGPT (CNCF sandbox):** Scans clusters with built-in analyzers (pods, services, PVCs, ingress, nodes, events...), finds problems, and uses an LLM (OpenAI, Bedrock, Azure, local models via Ollama/LocalAI) to explain issues and suggest fixes in plain language; anonymization option; operator mode for continuous analysis; integrations (Trivy, Prometheus).
- **kagent (CNCF sandbox, from Solo.io):** Kubernetes-native framework for building and running AI agents in-cluster, defined as CRDs (agents, tools, model configs), with built-in tools for Kubernetes, Prometheus, Istio, Argo, Helm and MCP support.
- **HolmesGPT (CNCF sandbox, from Robusta):** AI troubleshooting agent that investigates alerts by pulling data from Kubernetes, Prometheus, logs (Loki/Elasticsearch/Datadog), runbooks and other toolsets, producing root-cause analyses; integrates with Alertmanager/PagerDuty/Slack.
- **Kubernetes MCP servers:** Expose cluster operations (get pods/logs/events, describe, apply) to any MCP-capable agent (Claude Code, Copilot, Cursor) — configure read-only access for safety.

### Interview Questions

**Q1. What does K8sGPT do and how is it different from kubectl?**
kubectl shows raw state; K8sGPT runs analyzers that detect common issues (CrashLoopBackOff, failing probes, unbound PVCs, misconfigured Services with no endpoints, ingress pointing to missing services) and uses an LLM to explain the probable cause and remediation in natural language — speeding up triage, especially for less experienced engineers. It can run as a CLI or an operator producing results as CRDs/metrics.

**Q2. What are the risks of letting an AI agent operate a Kubernetes cluster?**
Wrong diagnoses applied confidently, destructive actions (deleting resources, scaling to zero), privilege escalation if the agent's ServiceAccount is powerful, leakage of sensitive data (secrets, logs with PII) to external LLM providers, and prompt injection through cluster data (e.g. annotations or log lines containing instructions). Mitigate with read-only RBAC by default, explicit approval for any change, data anonymization or self-hosted models, audit logging and restricted scope (namespaces).

**Q3. How would you integrate an AI troubleshooting agent into on-call?**
When Alertmanager/PagerDuty fires, trigger HolmesGPT (or a similar agent) to investigate automatically using read-only toolsets (Kubernetes state, recent deployments, metrics, logs, runbooks), and post its findings and suggested next steps into the incident Slack channel/ticket. On-call engineers validate and act. Track accuracy over time and feed postmortem learnings into runbooks the agent can use.

**Q4. How do you keep cluster data private when using LLM-based tools?**
Use anonymization/masking features (K8sGPT `--anonymize`), self-hosted or VPC-private model endpoints (Bedrock/Azure OpenAI in your account, local models via Ollama/vLLM), restrict which data sources the agent can read (no Secrets), redact logs, and ensure provider contracts prohibit training on your data.

**Q5. kagent vs using a general coding agent with a Kubernetes MCP server?**
kagent runs agents inside the cluster as Kubernetes resources — declarative, shareable across the team, integrated with cluster RBAC and tooling, suitable for persistent operational agents (e.g. an SRE assistant in Slack). A coding agent with an MCP server is ad hoc and runs on a developer's machine with their credentials — great for individual debugging. Both should follow least-privilege principles.

---

## 97. AI in Observability, Incident Platforms & Cloud Provider Agents (Datadog Bits AI, Dynatrace Davis, New Relic AI, Grafana Assistant, Elastic, Splunk, PagerDuty, incident.io, Rootly, AWS DevOps Agent, Azure SRE Agent, Gemini Cloud Assist)

**What it is:** Vendors embedding AI assistants and **AI SRE agents** into observability, incident and cloud platforms.

| Category | Examples & capabilities |
|---|---|
| **Observability vendors** | **Datadog Bits AI** (assistant + Bits AI SRE agent that investigates alerts autonomously, Bits AI dev agent proposing code fixes), **Dynatrace Davis AI** (causal AI root cause + generative CoPilot, agentic workflows), **New Relic AI** (natural language to NRQL, alert insights), **Grafana Assistant** (natural language queries, dashboard creation, investigations in Grafana Cloud), **Elastic AI Assistant** (observability and security), **Splunk AI Assistant** (SPL generation, investigations) |
| **Incident platforms** | **PagerDuty** AI agents (SRE agent, scribe for meeting/incident summaries, insights), **incident.io** AI (investigations, summaries, postmortem drafts), **Rootly AI**, FireHydrant AI — incident summaries, timeline generation, similar-incident lookup, postmortem drafting |
| **Cloud provider agents** | **AWS DevOps Agent** (frontier agent investigating incidents across telemetry, code and deployments) and CloudWatch investigations, **Azure SRE Agent** (monitors and diagnoses Azure resources, suggests/performs mitigations with approval), **Gemini Cloud Assist** (GCP investigations and design help) |

### Interview Questions

**Q1. What can AI SRE agents realistically do today?**
They can quickly gather context across tools (metrics, logs, traces, deploy history, config changes, similar past incidents), form and test hypotheses, summarize likely root causes with evidence, suggest mitigations and runbook steps, draft status updates and postmortems, and reduce the time spent on initial triage. They are less reliable for novel failure modes, complex multi-system interactions, and decisions requiring business judgment — humans should validate conclusions and approve actions.

**Q2. How would you evaluate an AI SRE agent before trusting it?**
Replay historical incidents (with known root causes) and measure accuracy, time to correct hypothesis and quality of evidence; run it in shadow mode on live incidents (suggestions only) and compare to human findings; measure false confident conclusions; check data handling and permissions; collect on-call feedback; then gradually allow low-risk automated actions with approval.

**Q3. How does AI help with postmortems and incident communication?**
Generating timelines from Slack/incident channels, alerts and deploy events; summarizing impact and actions; drafting status page updates and stakeholder summaries; drafting postmortem documents for humans to refine; identifying similar past incidents and recurring contributing factors across postmortems to prioritize systemic fixes.

**Q4. What prerequisites make AI-assisted operations effective?**
Good, correlated telemetry (OpenTelemetry with consistent service names and trace IDs), deployment and change events recorded with metadata, up-to-date service catalog/ownership and dependencies, runbooks in accessible formats, well-defined SLOs and alerts, and secure, scoped API access for the agent. AI amplifies the quality of your operational data — garbage in, garbage out.

**Q5. What are the risks of autonomous remediation by AI agents?**
Incorrect actions that worsen incidents (restarting the wrong service, rolling back unrelated changes), cascading automation loops, actions taken without understanding business context (e.g. scaling during a fraud spike), and security exposure if agent credentials are broad. Use tiered autonomy: suggest → execute pre-approved low-risk runbooks → broader actions only with human approval, with blast-radius limits and audit trails.

---

## 98. AIOps Patterns (Anomaly Detection, Alert Correlation & Noise Reduction, Automated RCA, Auto-Remediation)

**What it is:** Applying ML/AI techniques to operations data.
- **Anomaly detection:** Learning baselines with seasonality (daily/weekly) and flagging deviations — e.g. CloudWatch anomaly detection, Datadog Watchdog, Prometheus with forecasting, Dynatrace adaptive baselines. Useful where static thresholds fail.
- **Alert correlation & noise reduction:** Grouping related alerts into a single incident by time, topology/dependencies and similarity; deduplication; suppressing symptom alerts when a root cause alert fires (PagerDuty Event Intelligence, BigPanda, Moogsoft).
- **Automated root cause analysis (RCA):** Using topology graphs, change events and causal inference or LLM-based investigation to identify likely root causes.
- **Auto-remediation:** Triggering runbooks automatically (restart, scale, rollback, clear disk, failover) with guardrails — via event-driven automation (EventBridge/SSM Automation, Ansible Event-Driven, Rundeck/PagerDuty Process Automation, StackStorm, Kubernetes operators).
- **Predictive operations:** Forecasting capacity exhaustion (disk full in 4 hours), predictive autoscaling, failure prediction.
- **LLM-based operations:** Natural language querying of telemetry, summarization, agentic investigation (previous sections).

### Interview Questions

**Q1. What is AIOps and what problems does it address?**
AIOps applies machine learning and AI to IT operations data (metrics, logs, events, traces, changes, tickets) to detect anomalies, reduce alert noise by correlating related events, accelerate root cause analysis, predict issues and automate remediation. It addresses the scale and complexity of modern distributed systems where humans can't manually watch thousands of signals and alert storms overwhelm on-call teams.

**Q2. Anomaly detection vs static thresholds — when to use each?**
Static thresholds are simple, predictable and best for hard limits and SLO-based alerts (error budget burn, disk > 90%). Anomaly detection suits metrics with seasonality or trends where "normal" varies (traffic by hour, queue depth patterns), catching unusual drops (e.g. traffic suddenly zero) that thresholds miss. Anomaly alerts can be noisy (false positives around holidays/launches), so use them for investigation signals or tickets more than paging.

**Q3. How does alert correlation reduce noise?**
When a database fails, dozens of services alert simultaneously. Correlation groups these by time window, shared dependencies (service topology) and textual similarity into one incident pointing to the likely root cause, deduplicates repeated alerts, and suppresses downstream symptoms (inhibition). On-call receives one actionable page instead of an alert storm.

**Q4. Design an auto-remediation system with guardrails.**
Detection (alert/SLO burn/Config rule) → event bus → decision logic checks preconditions (is this a known pattern? is remediation already running? is the blast radius acceptable?) → executes a versioned, tested runbook (SSM Automation/Rundeck/Kubernetes operator) with rate limiting (max N actions per hour, one shard at a time) → verifies outcome (metrics recovered?) → escalates to humans if not, or if the action fails → full audit logging and notifications. Start with safe, reversible actions and require approval for risky ones.

**Q5. What data foundations are needed for AIOps to work?**
Consistent, high-quality telemetry with standard naming/labels, a service topology/dependency map (service catalog, tracing), change/deployment events, historical incident data with labeled root causes, CMDB/ownership information, and integration of tools via APIs. Without these, AIOps produces noise.

---

## 99. GPUs on Kubernetes (NVIDIA GPU Operator, DRA, MIG/Time-Slicing, Kueue, Volcano)

**What it is:** Running AI/ML training and inference on GPU (and other accelerator) nodes in Kubernetes efficiently.

**Key concepts**
- **NVIDIA GPU Operator:** Automates GPU node software — drivers, container toolkit, **device plugin** (advertises `nvidia.com/gpu` resources), DCGM exporter (GPU metrics), GPU Feature Discovery (node labels), MIG manager.
- **Device plugins** expose GPUs as extended resources requested in pod specs (`resources.limits: nvidia.com/gpu: 1`) — whole-GPU granularity by default.
- **GPU sharing:**
  - **MIG (Multi-Instance GPU):** Hardware partitioning of A100/H100-class GPUs into isolated instances (own memory, compute) — strong isolation.
  - **Time-slicing:** Oversubscribe a GPU by time-sharing among pods — no memory isolation.
  - **MPS (Multi-Process Service):** Concurrent kernels sharing a GPU with some partitioning.
- **Dynamic Resource Allocation (DRA):** Kubernetes API (beta 1.32, GA in 1.34) for flexible device requests via `ResourceClaims` and `DeviceClasses` — request GPUs by attributes (model, memory), share devices between containers, and support complex topologies — replacing limits of the simple device plugin model.
- **Batch scheduling:** **Kueue** (Kubernetes SIG project — job queueing, quotas and fair sharing across teams, all-or-nothing admission, works with Jobs, JobSet, Kubeflow, Ray), **Volcano** (CNCF batch scheduler — gang scheduling, queue management for AI/HPC), JobSet, LeaderWorkerSet (multi-node inference).
- **Topology-aware scheduling** (NVLink, network topology), GPU node autoscaling (Karpenter, cluster autoscaler), Spot GPUs for fault-tolerant jobs, GPU observability (DCGM: utilization, memory, SM occupancy).

### Interview Questions

**Q1. How do pods get access to GPUs in Kubernetes?**
The GPU node runs NVIDIA drivers and container toolkit; the device plugin advertises `nvidia.com/gpu` capacity to the kubelet; pods request GPUs in resource limits; the scheduler places them on nodes with free GPUs; the device plugin allocates specific GPU devices and the container runtime mounts the device files and libraries into the container. The GPU Operator automates installing and managing all these components. DRA provides a richer, attribute-based alternative.

**Q2. GPUs are expensive and underutilized. How do you improve utilization?**
Measure actual GPU utilization and memory with DCGM metrics; share GPUs for small workloads (MIG for isolation, time-slicing/MPS for dev and light inference); right-size models and batch sizes; use efficient inference servers (vLLM continuous batching); consolidate workloads with Kueue quotas and fair sharing across teams; autoscale GPU nodes down when idle (Karpenter), use Spot GPUs for interruptible training with checkpointing; schedule batch training during off-peak hours.

**Q3. What is gang scheduling and why does distributed training need it?**
Distributed training jobs need all N workers (pods) running simultaneously; if only some get scheduled, they hold GPUs idle while waiting — wasting resources and possibly deadlocking with other partial jobs. Gang (all-or-nothing) scheduling admits/schedules the job only when resources for all pods are available — provided by Kueue admission, Volcano, or the scheduler plugins (coscheduling).

**Q4. MIG vs time-slicing?**
MIG partitions supported GPUs at the hardware level into isolated instances with dedicated memory and compute — predictable performance and fault isolation, but fixed partition sizes and only on specific GPU models. Time-slicing lets multiple pods share a GPU by alternating time — works on any GPU and is flexible, but no memory isolation (one pod can OOM others) and unpredictable latency. Use MIG for production multi-tenant inference; time-slicing for dev/test and light workloads.

**Q5. What is Kueue and why use it for ML platforms?**
Kueue is a Kubernetes-native job queueing system: teams submit jobs to LocalQueues mapped to ClusterQueues with resource quotas (GPU flavors, Spot/On-Demand), borrowing between cohorts and fair sharing, priority and preemption, and all-or-nothing admission. It decides **when** jobs start (admission), leaving pod placement to the scheduler — enabling multi-team GPU sharing with policies instead of ad-hoc contention.

---

## 100. Model Serving (vLLM, KServe, llm-d, NVIDIA Triton/TensorRT-LLM/Dynamo, Ray Serve, SGLang, TGI, Ollama)

**What it is:** Deploying ML/LLM models as scalable, efficient inference services.

| Tool | Notes |
|---|---|
| **vLLM** ★ | High-throughput LLM inference engine — **PagedAttention** (efficient KV-cache memory management), **continuous batching**, prefix caching, speculative decoding, quantization, tensor/pipeline parallelism, OpenAI-compatible API; broad hardware support (NVIDIA, AMD, TPU, Trainium/Inferentia via plugins) |
| **KServe** (CNCF) | Kubernetes standard for model serving — `InferenceService` CRD, autoscaling (incl. scale-to-zero with Knative), canary rollouts, multiple runtimes (vLLM, Triton, TorchServe, sklearn/XGBoost), model storage integration; LLM-specific features |
| **llm-d** | Kubernetes-native distributed LLM inference stack (Red Hat, Google, IBM, NVIDIA and others) built on vLLM — **disaggregated prefill/decode**, KV-cache-aware routing via the Gateway API Inference Extension, scaling for large models |
| **NVIDIA Triton / TensorRT-LLM / Dynamo** | Triton Inference Server for multi-framework models; TensorRT-LLM optimized kernels for NVIDIA GPUs; **Dynamo** distributed inference framework (disaggregated serving, smart routing) |
| **Ray Serve** | Scalable serving on Ray — model composition, Python-native, multi-model pipelines; KubeRay operator |
| **SGLang** | Fast serving framework with RadixAttention prefix caching and structured generation |
| **TGI (Text Generation Inference)** | Hugging Face's LLM server (now in maintenance mode — HF recommends vLLM/SGLang) |
| **Ollama** | Easy local model running (llama.cpp-based) for developers and edge — not for high-scale production |
| Gateway API Inference Extension | Kubernetes Gateway API extension for model-aware routing (InferencePool, InferenceObjective) |

**Key LLM serving metrics:** **TTFT** (time to first token), **TPOT/ITL** (time per output token / inter-token latency), throughput (tokens/sec), requests/sec, GPU utilization, KV-cache utilization, queue depth, cost per million tokens.

### Interview Questions

**Q1. Why is vLLM so widely used for LLM serving?**
LLM inference is memory-bound by the KV cache. vLLM's **PagedAttention** manages KV cache in non-contiguous pages (like virtual memory), eliminating fragmentation and allowing many more concurrent sequences per GPU; **continuous batching** adds and removes requests from the running batch at each token step rather than waiting for a whole batch to finish — together delivering much higher throughput and lower cost. It also offers prefix caching, quantization, parallelism and an OpenAI-compatible API, with broad model and hardware support.

**Q2. How would you deploy and autoscale an open-weight LLM on Kubernetes?**
GPU node pool (Karpenter/GPU Operator), model weights cached on fast storage (pre-pulled to node local disks or using a model cache/OCI artifacts to reduce cold start), vLLM via KServe `InferenceService` or a Deployment/LeaderWorkerSet for multi-GPU/multi-node, Gateway API Inference Extension or an AI gateway for routing, autoscaling on **queue depth / in-flight requests / KV-cache utilization** (KEDA or KServe autoscaler) rather than CPU, readiness gated on model loaded, and monitoring of TTFT, tokens/sec and GPU metrics. Consider quantization to fit smaller GPUs and scale-to-zero only where cold-start latency is acceptable.

**Q3. What is disaggregated prefill/decode serving?**
LLM inference has two phases: **prefill** (process the whole prompt — compute-bound) and **decode** (generate tokens one by one — memory-bandwidth bound). Running both on the same GPUs causes interference (long prefills stall decoding, hurting latency). Disaggregation runs them on separate pools optimized for each phase and transfers the KV cache between them — improving throughput and latency SLOs at scale (llm-d, NVIDIA Dynamo, vLLM support).

**Q4. Which metrics matter for LLM serving SLOs?**
Time to first token (responsiveness, especially for streaming chat), time per output token / inter-token latency (streaming smoothness), end-to-end latency, throughput (tokens/sec per GPU), request success rate, queue time, KV-cache usage and preemptions, GPU utilization, and cost per million tokens. Define SLOs per use case (interactive chat vs batch summarization).

**Q5. KServe vs Ray Serve vs plain Deployment?**
Plain Deployment with vLLM: simplest, full control, but you build autoscaling, rollout and routing yourself. KServe: Kubernetes-native standard abstraction (InferenceService), autoscaling incl. scale-to-zero, canaries, multiple runtimes, model storage — good for platform teams standardizing serving. Ray Serve: Python-native, great for complex pipelines composing multiple models and business logic, runs on Ray clusters (KubeRay). Choose based on team skills and complexity of inference pipelines.

**Q6. How do you reduce cold-start time for large models?**
Model weights are tens to hundreds of GB: cache weights on node-local NVMe or a shared high-throughput file system, use streaming model loaders, pre-pull container images and model artifacts onto warm nodes, keep minimum replicas warm for latency-sensitive services, use smaller/quantized models, and use snapshotting features where available.

---

## 101. ML Pipelines & Experiment Tracking (Kubeflow, MLflow, Airflow, Argo Workflows, Weights & Biases)

**What it is:** Tools for orchestrating ML workflows (data prep → training → evaluation → registration → deployment) and tracking experiments and models — the **MLOps** lifecycle.

| Tool | Purpose |
|---|---|
| **Kubeflow** (CNCF) | ML platform on Kubernetes — **Kubeflow Pipelines** (containerized ML DAGs, Python SDK), Notebooks, Katib (hyperparameter tuning), Training Operator (distributed PyTorch/TF jobs), KServe, Model Registry |
| **MLflow** ★ | Open-source (Databricks) — **experiment tracking** (params, metrics, artifacts), **Model Registry** (versions, stages/aliases), model packaging, evaluation, and GenAI features (tracing, prompt registry, LLM evaluation) |
| **Apache Airflow** | General workflow orchestrator (Python DAGs) widely used for data/ML pipelines; managed versions (MWAA, Cloud Composer, Astronomer) |
| **Argo Workflows** (CNCF) | Kubernetes-native workflow engine (container steps as DAG/steps) — used for ML pipelines, CI, data processing; underlying engine of Kubeflow Pipelines |
| **Weights & Biases** | SaaS experiment tracking, visualization, sweeps, artifacts, model registry, and Weave for LLM app evaluation/tracing |
| Others | Prefect, Dagster, Metaflow, ZenML, Vertex AI Pipelines, SageMaker Pipelines, DVC (data versioning), Feast (feature store) |

**MLOps concepts:** Reproducibility (versioned data, code, config, environment), experiment tracking, model registry and promotion, CI/CD/CT (continuous training), model monitoring (drift), feature stores, lineage.

### Interview Questions

**Q1. What is MLOps and how does it differ from DevOps?**
MLOps applies DevOps principles to machine learning, but ML systems depend on **data and models** as well as code: you must version datasets and models, track experiments, validate data quality, evaluate model performance (not just pass/fail tests), handle **continuous training** when data changes, and monitor for data/concept drift in production. Artifacts are larger, results are probabilistic, and failures can be silent (model accuracy degrading without errors).

**Q2. Why is experiment tracking important and what do you track?**
ML development involves many experiments; without tracking, results aren't reproducible or comparable. Track code version (commit), data version/hash, hyperparameters, environment (library versions), metrics (training/validation curves), artifacts (models, plots), and hardware/time. MLflow or W&B record these, enabling comparison and promotion of the best model to the registry with lineage.

**Q3. How do you design a CI/CD pipeline for ML models?**
Code CI (lint, unit tests for data processing and feature code), data validation (schema, distribution checks — Great Expectations), training pipeline (Kubeflow/Airflow/SageMaker) producing a candidate model logged to MLflow, automated evaluation against a test set and the current production model (quality, fairness, latency), registration with approval, deployment via canary/shadow (KServe), monitoring for drift and performance, and triggers for retraining. Everything versioned and reproducible.

**Q4. Airflow vs Kubeflow Pipelines vs Argo Workflows?**
Airflow: general-purpose Python DAG scheduler with huge integration ecosystem — great for data engineering and scheduled batch ML pipelines; not Kubernetes-native by design (though can run on it). Kubeflow Pipelines: ML-specific on Kubernetes with artifact/metadata tracking and ML components. Argo Workflows: Kubernetes-native generic container workflow engine — high scalability and flexibility, used under the hood by Kubeflow and for CI/data jobs. Choose by team skills, infrastructure and ML-specific needs.

**Q5. What is a model registry and how is it used in deployment?**
A central store of model versions with metadata (metrics, lineage, training data version, owner) and lifecycle states or aliases (staging, production/"champion"). CI/CD deploys the model version referenced by the production alias; promotion is a controlled, audited action (with evaluation gates and approvals); rollback means pointing the alias to a previous version.

---

## 102. AI Gateways (LiteLLM, Envoy AI Gateway, Kong AI Gateway, Portkey)

**What it is:** A proxy layer between applications and LLM providers/self-hosted models that centralizes **routing, reliability, security, governance and cost control** for AI traffic.

**Capabilities:** Unified API (often OpenAI-compatible) across providers (OpenAI, Anthropic, Bedrock, Azure OpenAI, Vertex, self-hosted vLLM), **model routing** (by cost/latency/task), **fallbacks and retries** across providers/regions, load balancing, **rate limiting and token quotas** per team/user/key, **token usage and cost tracking**, caching (exact and semantic), API key management (virtual keys — apps never see provider keys), guardrails (PII redaction, prompt-injection filters), logging/observability (OTel), and policy enforcement (allowed models).

| Tool | Notes |
|---|---|
| **LiteLLM** | Open-source Python proxy/SDK supporting 100+ providers, virtual keys, budgets, spend tracking, fallbacks — very popular |
| **Envoy AI Gateway** | Open-source, built on Envoy Gateway — token-based rate limiting, provider routing, upstream auth; Kubernetes-native |
| **Kong AI Gateway** | AI plugins for Kong — multi-LLM routing, semantic caching, prompt guards, PII sanitization |
| **Portkey** | AI gateway with observability, guardrails, prompt management (open-source gateway + SaaS) |
| Others | Cloudflare AI Gateway, Azure API Management AI gateway policies, AWS (Bedrock + API Gateway patterns), Apigee, Gloo AI Gateway, OpenRouter (hosted routing) |

### Interview Questions

**Q1. Why would an organization put an AI gateway in front of LLM providers?**
To centralize control as many teams adopt LLMs: one integration point and OpenAI-compatible API (easy provider switching), provider keys stored centrally with per-team virtual keys, budgets/quotas and cost attribution per team, rate limiting to stay within provider limits, automatic fallbacks for reliability, caching to reduce cost/latency, consistent logging/observability and guardrails (PII masking, allowed-model policies) for security and compliance.

**Q2. How do you implement rate limiting for LLM traffic differently from normal APIs?**
LLM cost and provider limits are in **tokens**, not just requests — a single request can use 100 or 100,000 tokens. AI gateways enforce token-based limits (tokens per minute per key/team), estimate input tokens before the call, account for output tokens after, combine with request-rate limits and concurrency limits, and apply budgets ($ per month) per team.

**Q3. What is semantic caching and when is it risky?**
Caching responses keyed by embedding similarity of prompts, so semantically equivalent questions reuse a cached answer — reducing cost and latency for repetitive queries (FAQs). Risky when answers depend on user-specific context, permissions, or freshness (could leak another user's data or return stale info); use exact-match caching or scope caches per user/tenant, set TTLs, and avoid caching for personalized or sensitive prompts.

**Q4. How do you design LLM fallbacks across providers?**
Define a primary model and ordered fallbacks (another region of the same provider, an equivalent model from another provider, a self-hosted model) triggered on errors, rate limits (429) or timeouts; ensure prompts/outputs are compatible (tool calling formats, context window sizes), validate output quality of fallbacks with evals, use circuit breakers to avoid hammering a failing provider, and track which model served each request for observability.

**Q5. How do you attribute and control LLM costs across teams?**
Issue per-team/app virtual keys through the gateway, tag requests with metadata (team, feature, environment), record token usage and computed cost per request (by model pricing), dashboards and alerts per team, hard/soft budgets, model allowlists per environment (no premium models in dev without approval), and caching/routing to cheaper models where quality allows.

---

## 103. LLM Observability (Langfuse, LangSmith, Arize Phoenix, Helicone, OpenTelemetry GenAI Semantic Conventions)

**What it is:** Monitoring and debugging LLM applications and agents — capturing prompts, completions, tool calls, retrieval steps, token usage, latency, cost and quality scores — beyond traditional APM.

**What to capture:** Full traces of chains/agent runs (each LLM call, retrieval, tool invocation as spans), model and parameters, prompt templates and versions, input/output (with PII controls), token counts and cost, latency (TTFT), errors, user feedback, evaluation scores (faithfulness, relevance, toxicity), session/user IDs.

| Tool | Notes |
|---|---|
| **Langfuse** | Open-source (self-hostable) LLM engineering platform — tracing, prompt management, evaluations, datasets, cost tracking; OTel-compatible |
| **LangSmith** | LangChain's platform — tracing/debugging, evaluation datasets, prompt hub, monitoring; framework-agnostic SDKs |
| **Arize Phoenix** | Open-source tracing and evaluation built on OpenTelemetry/OpenInference; Arize AX for enterprise |
| **Helicone** | Open-source proxy-based LLM observability (log via gateway), cost tracking, caching |
| **OpenTelemetry GenAI semantic conventions** | Standard attributes for LLM spans (`gen_ai.operation.name`, `gen_ai.request.model`, `gen_ai.usage.input_tokens`/`output_tokens`, agent and tool spans) so any OTel backend can understand LLM telemetry; instrumentation libraries like OpenLLMetry (Traceloop) and OpenInference |
| Others | Datadog LLM Observability, W&B Weave, MLflow Tracing, Braintrust, Galileo, Opik (Comet), cloud-native (Bedrock invocation logging, AgentCore Observability) |

### Interview Questions

**Q1. Why is traditional APM not enough for LLM applications?**
APM shows latency and errors, but LLM failures are often **semantic**: a 200 OK response that's hallucinated, off-topic, unsafe, or uses the wrong tool. You need visibility into prompts, retrieved context, model outputs, tool calls and reasoning steps, token costs, and quality evaluations to debug and improve behavior — plus versioning of prompts and models to correlate changes with quality shifts.

**Q2. What should be in a trace of an AI agent run?**
A root span for the user request with session/user metadata; child spans for each LLM call (model, parameters, prompt template version, input/output tokens, latency, cost, finish reason), retrieval steps (query, documents retrieved with scores), tool calls (name, arguments, results, errors), guardrail checks, and the final answer; plus attached evaluation scores and user feedback. This lets you see why an agent took a wrong path.

**Q3. Why do OpenTelemetry GenAI semantic conventions matter?**
They standardize how LLM and agent operations are represented in telemetry (model names, token usage, operation types, tool calls), so instrumentation from different frameworks and providers produces consistent data that any OTel-compatible backend can analyze — avoiding lock-in to a single LLM observability vendor and allowing LLM traces to be correlated with the rest of your distributed traces.

**Q4. How do you handle privacy when logging prompts and completions?**
Prompts may contain PII, secrets or confidential documents: apply redaction/masking (at the SDK, gateway or collector), configurable capture levels (metadata only in production, full content in sampled/consented cases), encryption and strict access controls on observability storage, retention limits, self-hosted options (Langfuse/Phoenix) for sensitive data, and compliance with data residency.

**Q5. What production metrics would you monitor for an LLM application?**
Latency (TTFT, end-to-end), error/timeout rates per provider/model, token usage and cost per request/user/feature, cache hit rate, fallback rate, guardrail trigger rates (PII, prompt injection blocks), tool-call failure rates for agents, user feedback (thumbs up/down), online evaluation scores (sampled LLM-as-judge for faithfulness/relevance), and drift in input topics.

---

## 104. Evaluation in CI (promptfoo, DeepEval, Ragas, LLM-as-Judge Regression Suites)

**What it is:** Treating prompts, models, RAG pipelines and agents like code — with **automated evaluation suites** that run in CI and block regressions.

**Tools**
- **promptfoo:** Open-source CLI/library — YAML test cases with assertions (contains, regex, JSON schema, similarity, LLM-rubric), compare prompts/models side by side, **red teaming** (jailbreaks, prompt injection, PII leaks), CI integration.
- **DeepEval (Confident AI):** Pytest-style LLM testing framework with metrics (answer relevancy, faithfulness, hallucination, contextual precision/recall, G-Eval custom metrics, agent/tool-use metrics).
- **Ragas:** Metrics specifically for RAG — faithfulness, answer relevancy, context precision, context recall; test set generation.
- Others: OpenAI Evals, LangSmith/Langfuse/Braintrust evaluations, Inspect (UK AISI), Arize Phoenix evals, MLflow evaluate.

**Approach:** Golden datasets of representative inputs (including edge cases and adversarial prompts) with expected outputs or rubrics → deterministic checks (format, schema, contains/excludes) → model-graded checks (**LLM-as-judge** with clear rubrics, calibrated against human labels) → thresholds as CI gates → track scores over time; add production failures to datasets continuously.

### Interview Questions

**Q1. How do you test an LLM application when outputs are non-deterministic?**
Use assertions that tolerate variation: structural checks (valid JSON schema, required fields, length), property checks (contains key facts, excludes forbidden content), semantic similarity to reference answers, and LLM-as-judge rubrics (correctness, faithfulness to context, tone). Run multiple samples or temperature 0 where appropriate, use aggregate pass rates with thresholds rather than single exact matches, and track trends across versions.

**Q2. What is LLM-as-a-judge and what are its pitfalls?**
Using an LLM to grade outputs against a rubric or reference (e.g. "score 1–5 for faithfulness to the provided context"). It scales evaluation beyond human capacity. Pitfalls: judge bias (verbosity, position, self-preference), inconsistency, sensitivity to rubric wording, cost. Mitigate with precise rubrics and examples, pairwise comparisons with randomized order, a different/stronger judge model, calibration against human-labeled samples, and monitoring judge agreement.

**Q3. How would you evaluate a RAG system?**
Separate **retrieval** and **generation** quality: retrieval metrics (context recall — were the needed documents retrieved? context precision — how much retrieved content is relevant? hit rate/MRR on labeled queries), generation metrics (faithfulness/groundedness — claims supported by context, answer relevance, correctness vs reference, citation accuracy). Tools: Ragas, DeepEval, Bedrock/LangSmith evaluations. Diagnose whether failures come from retrieval or generation before tuning.

**Q4. How do you integrate LLM evals into CI/CD?**
On PRs that change prompts, models, retrieval config or agent logic: run a fast eval subset (smoke) with thresholds that fail the build on regressions; nightly run the full suite and red-team tests; store results and diffs vs baseline as PR comments/dashboards; version datasets; for releases, require eval sign-off; after deployment, monitor online metrics and sample production traffic for judge-based scoring. Control cost by caching responses and using smaller judge models for routine checks.

**Q5. What is LLM red teaming and how is it automated?**
Adversarial testing to find unsafe or insecure behavior — jailbreaks, prompt injection (direct and indirect via documents/tools), data leakage (system prompt, PII), harmful content, excessive agency (unauthorized tool use). Automated with tools like promptfoo red team, garak (NVIDIA), PyRIT (Microsoft), generating attack variants and checking responses against policies; results feed guardrail improvements and CI gates.

---

## 105. Agent Frameworks to Deploy & Operate (LangGraph, CrewAI, LlamaIndex, OpenAI Agents SDK, Strands, Google ADK)

**What it is:** Frameworks developers use to build AI agents — DevOps/platform engineers need to know how to **deploy, scale, secure and observe** them.

| Framework | Characteristics |
|---|---|
| **LangGraph** (LangChain) | Graph-based stateful agent orchestration — nodes/edges, cycles, checkpointing/persistence, human-in-the-loop interrupts, multi-agent patterns; LangGraph Platform for deployment |
| **CrewAI** | Role-based multi-agent "crews" (agents with roles, goals, tasks) and Flows for deterministic orchestration |
| **LlamaIndex** | Data framework for RAG (indexing, retrieval, query engines) plus agent workflows |
| **OpenAI Agents SDK** | Lightweight agents with tools, handoffs between agents, guardrails, tracing; provider-agnostic via adapters |
| **Strands Agents** (AWS, open source) | Model-driven agent loop, tools/MCP, multi-agent patterns, deployable to Lambda/Fargate/AgentCore |
| **Google ADK (Agent Development Kit)** | Open-source framework for multi-agent systems, integrates with Vertex AI Agent Engine, supports A2A |
| Others | Microsoft Agent Framework (successor to AutoGen and Semantic Kernel), PydanticAI, Mastra (TypeScript), Claude Agent SDK |

**Operational concerns:** Long-running and stateful execution (checkpointing, durable execution — e.g. Temporal, LangGraph persistence), session isolation, scaling (queue-based workers), timeouts and max iterations, cost controls, secrets and tool credentials (per-user OAuth), sandboxing code execution, observability (OTel GenAI traces), evaluation, versioning and rollout of prompts/agents, human-in-the-loop approvals. Managed runtimes: AWS Bedrock AgentCore, Vertex AI Agent Engine, Azure AI Foundry Agent Service, LangGraph Platform.

### Interview Questions

**Q1. What operational challenges do AI agents introduce compared with regular microservices?**
Non-deterministic behavior and variable execution length (seconds to hours), multi-step workflows needing state persistence and resumability, unpredictable cost (token usage loops), external tool calls with side effects requiring authorization and idempotency, security risks (prompt injection, excessive permissions, code execution), harder testing (evals rather than unit tests), and the need for specialized observability (traces of reasoning/tool calls). Plus human-in-the-loop flows that pause execution.

**Q2. How would you deploy a LangGraph (or similar) agent to production?**
Containerize the agent service; persist graph state/checkpoints in a durable store (Postgres/Redis) so runs can resume after failures or human approval; run long tasks asynchronously via a queue with worker autoscaling (KEDA) or use a managed runtime (LangGraph Platform, AgentCore); route model calls through an AI gateway (keys, budgets, fallbacks); expose tools via MCP with scoped credentials; isolate code execution in sandboxes; instrument with OTel GenAI conventions → Langfuse/LangSmith; set iteration/time/cost limits; ship with eval suites and canary releases.

**Q3. Deterministic workflows vs autonomous agents — how do you choose?**
If the steps are known (classify → retrieve → draft → validate), use a deterministic workflow/graph with LLM calls at specific nodes — more predictable, testable, cheaper. Use autonomous agents (LLM chooses tools and steps) when tasks are open-ended and paths vary widely. Many production systems are hybrids: constrained graphs with agentic sub-steps and explicit guardrails.

**Q4. Why is durable execution important for agents?**
Agent runs can be long and involve many external calls; process restarts, deployments or failures mid-run shouldn't lose progress or repeat side effects. Durable execution (checkpointing state after each step, workflow engines like Temporal/Step Functions, LangGraph persistence) allows resuming from the last completed step, waiting for human approval for hours/days, and exactly-once-like semantics with idempotent tool calls.

**Q5. How do you prevent runaway agent costs?**
Max iterations/tool calls per run, timeouts, token budgets per request/user/tenant enforced in code and at the AI gateway, model routing to cheaper models for simple steps, caching, loop detection (repeated identical tool calls), alerts on cost anomalies, and dashboards of cost per task — plus evaluation of whether agentic approaches are necessary at all.

---

## 106. Vector Databases (pgvector, Qdrant, Weaviate, Milvus, Pinecone, OpenSearch)

**What it is:** Databases optimized to store **embeddings** (high-dimensional vectors representing meaning of text/images) and perform fast **similarity search** (nearest neighbors) — the retrieval backbone of RAG, semantic search and recommendations.

**Key concepts**
- **Similarity metrics:** Cosine similarity, dot product, Euclidean (L2).
- **ANN (Approximate Nearest Neighbor) indexes:** **HNSW** (graph-based — fast, high recall, memory-heavy), **IVF** (inverted file clustering), **PQ/quantization** (compress vectors — less memory, some accuracy loss), DiskANN (disk-based).
- **Hybrid search:** Combine vector similarity with keyword/BM25 search and **metadata filtering** (tenant, permissions, date); reranking.
- **Operational aspects:** Index build time, recall vs latency tuning (ef/M parameters), memory sizing, sharding/replication, backups, re-embedding when models change, multi-tenancy, freshness of updates.

| Option | Notes |
|---|---|
| **pgvector** | PostgreSQL extension — vectors alongside relational data, HNSW/IVFFlat indexes, transactions; great default if you already run Postgres (Aurora/RDS/Cloud SQL support) |
| **Qdrant** | Open-source (Rust), strong filtering, quantization, cloud offering |
| **Weaviate** | Open-source, built-in vectorization modules, hybrid search, multi-tenancy |
| **Milvus** (LF AI) / Zilliz | Open-source, distributed, designed for billions of vectors, GPU indexing |
| **Pinecone** | Fully managed serverless vector DB |
| **OpenSearch / Elasticsearch** | k-NN vector search plus mature full-text search — strong hybrid search |
| Others | Redis/Valkey vector search, MongoDB Atlas Vector Search, Chroma (lightweight/dev), LanceDB, S3 Vectors, Vespa |

### Interview Questions

**Q1. Do you need a dedicated vector database?**
Not always. If you already run PostgreSQL and have up to millions of vectors with moderate QPS, **pgvector** keeps data, metadata filtering and transactions in one system with less operational overhead. Dedicated vector DBs make sense at large scale (hundreds of millions+ vectors), high QPS with strict latency, advanced features (quantization, distributed indexing, multi-tenancy at scale) or when you need hybrid search engines like OpenSearch. Decide by scale, latency, filtering needs, and team operations capacity.

**Q2. What is HNSW and its trade-offs?**
Hierarchical Navigable Small World: a multi-layer proximity graph where search starts at the sparse top layer and greedily navigates closer neighbors down to the dense bottom layer. It provides excellent recall/latency, supports incremental inserts, but uses significant memory (graph + vectors), index builds can be slow, and deletes/updates fragment the graph. Parameters (`M`, `ef_construction`, `ef_search`) trade accuracy for speed and memory.

**Q3. How do you enforce permissions in vector search for RAG?**
Store access control metadata (tenant ID, groups, document ACLs) with each vector and apply **filters** in the query based on the authenticated user's entitlements (server-side), or use separate collections/namespaces per tenant for stronger isolation. Ensure filtered search keeps good recall (some indexes degrade with highly selective filters — use databases with efficient filtered HNSW). Never rely on the LLM to hide unauthorized content.

**Q4. What happens when you change the embedding model?**
Embeddings from different models aren't comparable — you must **re-embed the entire corpus** and rebuild indexes. Plan for it: version embeddings and indexes, run re-embedding as a batch pipeline into a new index, evaluate retrieval quality on both, then switch traffic (blue/green indexes) and delete the old index. Budget cost and time for large corpora.

**Q5. How do you operate a vector database in production?**
Capacity planning (vector count × dimensions × bytes + index overhead, memory vs disk-based indexes, quantization), replication for HA and read scaling, sharding for large collections, backups/snapshots, monitoring (query latency percentiles, recall on test queries, index size, memory, ingestion lag), handling updates/deletes (compaction), multi-tenancy design, security (authn/authz, encryption, network isolation), and freshness pipelines from source data.

---

## 107. AI Security (OWASP Top 10 for LLMs, Prompt Injection, Guardrails, Model & Dataset Supply Chain)

**What it is:** Securing AI applications, agents and the AI supply chain.

**OWASP Top 10 for LLM Applications (2025 edition)**
1. **Prompt Injection** (direct and indirect via documents, web pages, tool outputs)
2. **Sensitive Information Disclosure**
3. **Supply Chain** (models, datasets, plugins, dependencies)
4. **Data and Model Poisoning**
5. **Improper Output Handling** (unvalidated output → XSS, SQL injection, code execution)
6. **Excessive Agency** (too many permissions/tools/autonomy)
7. **System Prompt Leakage**
8. **Vector and Embedding Weaknesses** (RAG access control, poisoning, inversion)
9. **Misinformation** (hallucinations, overreliance)
10. **Unbounded Consumption** (cost/DoS from unlimited usage)
(OWASP also publishes guidance for agentic AI threats.)

**Defenses & tools**
- **Guardrails:** **NVIDIA NeMo Guardrails** (programmable rails — topical, safety, jailbreak, fact-checking), **Llama Guard** (Meta safety classifier model) and Prompt Guard, cloud guardrails (Bedrock Guardrails, Azure AI Content Safety, Google Model Armor), Guardrails AI (output validation), Lakera, LLM Guard.
- **Architecture controls:** Least-privilege tools and credentials, human approval for sensitive actions, separating trusted instructions from untrusted data, output validation/encoding, sandboxed code execution, rate limits and budgets, permission-aware retrieval, logging/monitoring.
- **Model & dataset supply chain:** Download models only from trusted sources, prefer **safetensors** over pickle-based formats (pickle can execute arbitrary code), scan model files (ModelScan, Protect AI), verify hashes/signatures (**OpenSSF Model Signing** with Sigstore), track provenance and licenses, **AI-BOM/ML-BOM** (CycloneDX supports ML BOMs) listing models, datasets and dependencies, dataset integrity and poisoning checks.
- **Red teaming** (garak, PyRIT, promptfoo) and governance frameworks (NIST AI RMF, ISO/IEC 42001, EU AI Act).

### Interview Questions

**Q1. What is prompt injection and why is it hard to solve?**
An attacker supplies input that the model interprets as instructions, overriding intended behavior — directly ("ignore previous instructions...") or **indirectly** through content the system processes (a web page, email, document, issue comment, tool output). It's hard because LLMs process instructions and data in the same channel with no reliable separation, so filters can be bypassed by novel phrasing. Defense in depth: assume injection will sometimes succeed and limit impact — least-privilege tools, human approval for consequential actions, isolate untrusted content, output validation, monitoring, and guardrail classifiers.

**Q2. What is "excessive agency" and how do you mitigate it?**
Giving an LLM agent more functionality, permissions or autonomy than needed — e.g. an email-summarizing agent that can also send and delete emails, or a support bot with database write access. If manipulated (via injection) or mistaken, it causes damage. Mitigate: minimal tool sets, narrowly scoped functions (not generic "run SQL"), least-privilege credentials tied to the end user's permissions, human-in-the-loop for high-impact actions, rate limits, and audit logs.

**Q3. Why are pickle-based model files a supply chain risk?**
Python pickle deserialization can execute arbitrary code; loading a malicious `.pkl`/`.pt` (older PyTorch format) model from an untrusted source can compromise the host. Mitigate by using **safetensors** (pure tensor data format), scanning models (ModelScan, Hugging Face scanners), loading with safe options (`weights_only=True` in PyTorch), sourcing from trusted publishers, verifying signatures/hashes, and running model loading in sandboxed environments.

**Q4. How do you prevent sensitive data leakage in LLM applications?**
Don't put secrets in prompts or system prompts, redact PII before sending to models (gateway/guardrails), permission-aware retrieval (users can only retrieve documents they're allowed to see), output filters for sensitive patterns, contracts ensuring providers don't train on your data, data residency controls, logging with redaction, and access controls on logs/traces containing prompts.

**Q5. What is an AI-BOM and why is it needed?**
An AI Bill of Materials extends SBOMs to AI systems: models (name, version, source, hash, license), datasets used for training/fine-tuning (provenance, licenses), dependencies, and configurations. It supports vulnerability and license management, provenance verification, regulatory compliance (EU AI Act transparency), and incident response ("which applications use the compromised model?").

**Q6. How do guardrail tools like NeMo Guardrails or Llama Guard fit into an architecture?**
They sit before and after model calls: input rails classify/block jailbreaks, prompt injections, off-topic or unsafe requests and redact PII; output rails check for unsafe content, policy violations, hallucination against sources, or sensitive data leakage; dialog rails constrain conversation flows. They're one layer of defense — combined with least-privilege design, validation and monitoring — and must be evaluated for false positives/negatives and latency impact.

---

# CONNECTING THE DOTS

---

## 108. End-to-End Scenario: One Commit's Journey (and Senior System-Design Questions)

**The full flow**
1. **Plan:** Jira/Linear ticket (or an AI coding agent is assigned the issue).
2. **Code:** Developer or agent (guided by `AGENTS.md`/`CLAUDE.md`) writes code on a short-lived branch; pre-commit runs formatters, linters and **gitleaks**.
3. **PR:** GitHub PR with branch protection, CODEOWNERS, AI reviewer comments, human approval; merge queue keeps main green.
4. **CI (GitHub Actions, OIDC — no static keys):** unit tests → build → **Semgrep/CodeQL** (SAST) and **Snyk/Trivy** (SCA) → **Docker** multi-stage build on distroless → **Trivy** image scan → **Syft** SBOM → **cosign** keyless signing + provenance → push to **ECR/Artifactory** (immutable SHA tag).
5. **Infra change (if any):** **Terraform** plan in PR via **Atlantis**, **Checkov** and **OPA** policies, **Infracost** comment → apply after approval.
6. **Deploy:** CI updates the image tag in the GitOps config repo (Helm/Kustomize overlay) → **Argo CD** syncs to **Kubernetes**.
7. **Admission:** **Kyverno** verifies signature/provenance, required labels, limits, non-root; secrets from **Vault/Secrets Manager** via **External Secrets**; workload identity for cloud access.
8. **Release:** **Argo Rollouts** canary 10% → 50% → 100% with Prometheus analysis; **feature flag** (OpenFeature/LaunchDarkly) controls exposure; DB changes via expand/contract migrations (Flyway in a PreSync hook).
9. **Run:** **Karpenter/KEDA** scale capacity; **Cilium/Istio** provide mTLS and network policy; Gateway API routes traffic.
10. **Observe:** **OpenTelemetry** → Prometheus/Mimir, Loki, Tempo → **Grafana**; SLO burn-rate alerts via Alertmanager.
11. **Incident:** **PagerDuty** pages on-call; an **AI SRE agent** (HolmesGPT/Bits AI/AWS DevOps Agent) correlates the new deploy with the error spike and suggests rollback; human approves; Argo Rollouts abort/Git revert restores the stable version.
12. **Learn:** Blameless postmortem, action items (better tests/alerts), **DORA** metrics updated, **Backstage** scorecards and **OpenCost** dashboards reflect impact → back to Plan.

### Interview Questions

**Q1. Design a CI/CD and runtime platform for a company with 30 teams and 200 microservices on Kubernetes.**
Platform team owns an IDP: Backstage catalog and templates (golden path creates repo, pipeline, manifests, dashboards, SLOs); reusable GitHub Actions workflows (build/test/scan/sign/push with OIDC); GitOps with Argo CD ApplicationSets across dev/staging/prod clusters per region; Argo Rollouts for progressive delivery; Kyverno policies (signed images, limits, PSS restricted); External Secrets + cloud secret manager; Karpenter + KEDA autoscaling; Cilium networking/policies; OTel → Prometheus/Loki/Tempo/Grafana (or vendor); SLO-based alerting with PagerDuty; Terraform/Crossplane for cloud resources; FinOps via OpenCost with showback; DORA and DevEx measurement; AI agents/reviewers integrated with guardrails. Multi-cluster DR with GitOps re-sync and managed databases.

**Q2. Production latency increased 5× right after a deployment. Walk through your response.**
Confirm impact via SLO dashboards (latency p99, error rate, which endpoints/regions). Check deployment events correlated in Grafana annotations / Argo CD history. If the canary is still in progress, abort; if fully rolled out, roll back immediately (Git revert / rollout undo) or disable the feature flag — mitigate first. Verify recovery. Then investigate: traces of slow requests (new N+1 query? new dependency call?), profiles (CPU hot path), DB metrics (missing index from migration), resource limits (CPU throttling). Postmortem: why didn't canary analysis catch it (missing latency metric, low traffic)? Add tests/analysis checks.

**Q3. How would you secure the software supply chain end to end?**
Source: branch protection, signed commits, CODEOWNERS, secret scanning with push protection. Dependencies: lockfiles, Renovate updates, SCA with reachability, private proxies against dependency confusion, pinned actions by SHA. Build: ephemeral isolated runners, OIDC credentials, SLSA L3 provenance, SBOM generation, cosign keyless signing. Artifact: immutable tags/digests, registry scanning, promotion between registries. Deploy: admission verification of signatures/attestations, restricted registries, policy as code. Runtime: least privilege, NetworkPolicies, Falco/Tetragon detection, continuous vulnerability scanning. AI: vetted MCP servers/models, AI-BOM.

**Q4. Your team spends 40% of its time on operational toil and on-call is burning people out. What do you do?**
Measure: categorize pages and tickets, quantify time. Fix alerting: page only on SLO burn and actionable symptoms, route the rest to tickets, delete noisy alerts. Address top recurring causes with engineering work (reserve capacity — e.g. 30% of sprint), automate repetitive tasks (runbook automation, self-service tools), introduce error budget policy to balance features vs reliability, improve deployment safety (canaries, flags, faster rollback), add AI-assisted triage for faster investigation, ensure rotation size and compensation are fair. Track pages per shift and toil hours monthly.

**Q5. How would you introduce AI agents across the DevOps lifecycle responsibly?**
Start with high-value, low-risk uses: AI code review (non-blocking), test generation, CI failure explanation, incident summarization and read-only AI investigation. Establish governance: approved tools, data policy, human accountability, PR-only changes by agents, least-privilege credentials, sandboxed execution, MCP server allowlist, audit logs. Measure impact with DORA/DevEx metrics and quality indicators. Gradually expand to autonomous maintenance tasks (dependency upgrades) and pre-approved auto-remediation with guardrails as trust is established.

**Q6. Push vs pull deployment, mesh vs no mesh, monorepo vs polyrepo, Datadog vs open source — how do you make these big platform decisions?**
Start from requirements and constraints (scale, team skills, compliance, budget, existing investments), identify the few criteria that matter most, evaluate options with small proofs of concept and real workloads, consider reversibility and lock-in (open standards like OTel, OCI, Gateway API reduce lock-in), estimate total cost of ownership (licenses + people + operations), involve stakeholders, document the decision in an ADR with trade-offs, and define success metrics to revisit later.

---

## Category → Tool Map (Quick Reference)

| Category | Open-source / generic | AWS equivalent |
|---|---|---|
| Source control | GitHub, GitLab | (CodeCommit — closed to new customers) |
| CI | GitHub Actions, GitLab CI, Jenkins, Buildkite, Tekton | CodeBuild, CodePipeline |
| Artifact registry | Artifactory, Nexus, Harbor, GHCR | ECR, CodeArtifact |
| IaC | Terraform/OpenTofu, Pulumi, Crossplane, Ansible | CloudFormation, CDK |
| Orchestration | Kubernetes (+ Helm/Kustomize) | EKS, ECS |
| GitOps / CD | Argo CD, Flux, Argo Rollouts, Flagger | CodeDeploy |
| Secrets | Vault/OpenBao, External Secrets, SOPS | Secrets Manager, SSM Parameter Store |
| Policy | OPA/Gatekeeper, Kyverno, VAP | SCPs, Config rules |
| Observability | OTel, Prometheus, Loki, Tempo, Grafana | CloudWatch, X-Ray, AMP, AMG, ADOT |
| Security scanning | Trivy, Semgrep, CodeQL, Snyk, Checkov | Inspector, GuardDuty, Security Hub |
| Runtime security | Falco, Tetragon | GuardDuty Runtime Monitoring |
| Chaos | Chaos Mesh, LitmusChaos | Fault Injection Service |
| Developer portal | Backstage, Port | — |
| Cost | OpenCost, Kubecost, Infracost | Cost Explorer, CUR, Compute Optimizer |
| AI coding | Claude Code, Copilot, Cursor, Codex | Q Developer, Kiro |
| AI ops | K8sGPT, HolmesGPT, kagent, Datadog Bits AI | AWS DevOps Agent, CloudWatch investigations |
| Model serving | vLLM, KServe, llm-d, Triton | SageMaker AI, Bedrock |
| AI gateway / LLM observability | LiteLLM, Envoy AI Gateway, Langfuse | Bedrock + API Gateway, AgentCore Observability |

---

## Quick Revision Checklist

- [ ] Explain how containers work (namespaces, cgroups) and debug a slow Linux server
- [ ] Walk through "what happens when you type a URL" including DNS and TLS
- [ ] Write an optimized multi-stage Dockerfile and explain every line
- [ ] Explain the Kubernetes request path from `kubectl apply` to a running pod
- [ ] Debug CrashLoopBackOff, OOMKilled, Pending, ImagePullBackOff
- [ ] Explain probes, requests/limits, QoS, PDBs and graceful shutdown
- [ ] Compare HPA/VPA/Karpenter/KEDA and Ingress vs Gateway API
- [ ] Design a CI/CD pipeline with OIDC, scans, SBOM, signing and promotion
- [ ] Explain GitOps (Argo CD) and progressive delivery (Argo Rollouts)
- [ ] Explain Terraform state, modules, drift, `for_each` vs `count`, `moved` blocks
- [ ] Define SLI/SLO/SLA, error budgets, burn-rate alerts and DORA metrics
- [ ] Run an incident end to end and write a blameless postmortem
- [ ] Explain supply-chain security (SLSA, Sigstore, SBOM) and admission verification
- [ ] Explain OpenTelemetry architecture and cardinality/sampling trade-offs
- [ ] Discuss platform engineering, golden paths, Backstage and Team Topologies
- [ ] Discuss AI coding agents, MCP, A2A and governance of AI-generated code
- [ ] Explain LLM serving (vLLM), AI gateways, LLM observability, evals and OWASP LLM Top 10
