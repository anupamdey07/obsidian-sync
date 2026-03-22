# FABRICOS WHITEPAPER
## The "OpenClaw" style OS for Physical devices, Home Lab and ML

**Date:** March 22, 2026
**Author:** Onu 
**Version:** 3.0 (Idea → Architecture → Specs)
**Status:** Active Development (PoC Phase)

---

# PART I — THE IDEA

*Why FabricOS exists. The vision, the philosophy, and the experience it delivers.*

---

## 1. VISION & PHILOSOPHY

If Linux was the operating system of the Server/Desktop era, FabricOS is the "OpenClaw" of the Edge/Physical era.

FabricOS is a local-first, agentic OS designed for human-interface devices (Screen+GPU) and physical robotics. It treats intelligence as fluid and competence as deterministic, linked by a rigorous evolutionary filter. It is mesh-native, leveraging the Zenoh protocol for universal communication with zero cloud dependency and zero vendor lock-in.

FabricOS runs on an immutable, container-native host (Talos Linux) with Kubernetes orchestrating all application and AI workloads. The entire application layer loads as containers — no package manager at the OS layer. The OS boots directly into the container runtime. New workloads are scheduled as Pods, not installed on the host.

### Core Philosophy

* **Intelligence is Fluid:** The Left Brain can hallucinate, experiment, and generate new capabilities without constraint. Creativity is never gated at the source.
* **Competence is Deterministic:** The Right Brain and System 2 enforce strict rules. Nothing reaches the hardware without passing formal verification.
* **The Filter is Evolutionary:** The Biological CI/CD pipeline is the bridge between fluid creativity and deterministic action. Capabilities must survive the filter to be promoted.
* **Compute-First, Hardware-Second:** FabricOS decouples the "Brain" from the "Body." The same OS runs on a $10,000 workstation or a discarded Android phone.
* **Zero Cloud Dependency:** Zenoh replaces cloud brokers. Tailscale replaces public endpoints. The mesh is the network.

---

## 2. THE TRI-BRAIN GUARDIAN ARCHITECTURE

FabricOS moves beyond standard General Purpose OS (GPOS) vulnerabilities by implementing a three-tiered hierarchical architecture. Each brain has a distinct role, and System 2 sits physically between the other two — no action passes without its approval.

* **Left Brain (Creative/Fluid):** A remote or high-compute large VLA (Vision-Language-Action) model (e.g., Nemotron). Handles user intent, chat, and "revolutionary" skill generation. Features a semantic tool-calling interface capable of accessing external intelligence (e.g., a Perplexity Computer via plugin API) to digest new frontier knowledge. Runs as dedicated Kubernetes Pods (`model-runner-*`, `llm-router`).

* **System 2 (The Hardened Conscience):** A local SLM (Small Language Model, like Phi-4-mini or Gemma 3) running as a Kubernetes Deployment (`policy-engine`). Acts as the ultimate deterministic bouncer — physically isolating the Left Brain from the Right Brain, reasoning over every `/action` to validate safety and hardware constraints before execution. Backed by a formally verified seL4 Microkernel (Type-1 Hypervisor) where hardware-level isolation is required.

* **Right Brain (Deterministic Body):** An immutable Talos Linux environment serving as the Kubernetes host. Talos provides the immutable rootfs, real-time guarantees, and container runtime where stable Docker (heavy) and WASM (light) skills execute with surgical precision. All application behavior runs as Kubernetes Pods — the host itself never changes at runtime.

### Why Three Brains?

A two-brain system (creative + deterministic) creates a single point of failure: either the creative brain has unrestricted access to hardware (dangerous), or the deterministic brain must do everything (slow, inflexible). System 2 is the firewall — a lightweight, always-on guardian that enforces safety without constraining creativity.

---

## 3. THE TWO-FACE INTERFACE (OPENCLAW UX)

FabricOS presents a dual UI for operators — a fluid creative face and a deterministic terminal face, both served by `fabric-ui` Pods.

### 3.1. The Crystal Touchball / Fluid Face (Left Brain)

A fluid, chat-based VLA interface for creative flow and intent processing. The chat is the central "gravity source" around which all other capabilities orbit.

* **The "/" Command:** Universal context switch combining system search (Spotlight) with slash commands. `/camera` surfaces the camera node, `/code` opens the editor skin, `/research` spawns an agent task.
* **Floating Neural Nodes:** Recently used apps/tools appear as **proximity-weighted orbs** orbiting the central chat. Placement mimics spatial memory:
  * **High Frequency:** Most-used nodes orbit closest to the center.
  * **Low Frequency:** Rarely-used nodes drift to the periphery, fade to low opacity.
* **Context Capsules:** Each node is not just an app window but a "capsule" containing the tool's recent state and its historical connections to other nodes it interacted with.

### 3.2. The Deterministic Terminal (Right Brain)

A classic Bash/Linux-like interface for system auditing, augmented with Monaco-style plugins to visualize Zenoh/ROS2 tf-trees, edit code, and manage hardware deterministically.

### 3.3. Async (Optimistic) UI Pattern

To ensure the interface feels responsive even when System 2 is evaluating an action, the UI uses an optimistic pattern:

1. When a user submits a request, the UI **immediately** shows the action as pending with a glow/pulse animation.
2. System 2 evaluates in the background (Risk Tier + manifest check).
3. If **approved** → state resolves, animation confirms success.
4. If **rejected** → UI visually **snaps back** with a structured `reason_code` badge from the JSON verdict.

For the MVP, floating nodes are rendered as SVG/Canvas elements using CSS physics or libraries like Framer Motion to simulate gravitational pull and spring animations.

### 3.4. Latency Expectations

System 2 evaluation latency depends on hardware:
* **RPi / Edge Nodes:** 2–8 second evaluation delay (creates a "blocking" feel on the UI).
* **Jetson AGX Orin:** Under 0.3 seconds — spatial interface operates in near-real-time.
* **ASUS GX10 (Core Node):** Sub-100ms for Tier 0/1; Tier 2 Deep Eval depends on linter complexity.

### 3.5. Node Console (First Screen)

The fabric-agent node console is the initial entry point via Tailscale. It shows status cards and an **Open Fabric** button that redirects to the `fabric-ui` Service or Tailscale-exposed ingress. If Fabric UI is unhealthy, the node console stays on rescue mode with host-level recovery actions.

---

## 4. KEY DESIGN PRINCIPLES

1. **Intelligence is Fluid, Competence is Deterministic:** Never gate creativity at the source. Gate it at the boundary between thought and action.
2. **Three Brains, One Firewall:** System 2 physically isolates creative compute from deterministic execution.
3. **Host layer for reachability + recovery:** Talos extensions provide the safety net. Nothing else belongs at the host level.
4. **App layer for product behavior:** Kubernetes Services/Deployments for everything the user interacts with.
5. **Workload layer for models + compute:** Dedicated Pods so inference can restart, scale, or migrate without touching the app layer.
6. **Git as source of truth:** Configs, manifests, desired state — all in Git, all reconciled by Flux.
7. **Tailscale everywhere:** Private access without public exposure. No VPN, no port forwarding, no certs to manage.
8. **Container-native OS:** The OS boots into the runtime. The application IS the container layer. No package manager on the host. Ever.
9. **Separation of concerns:** Fabric control plane never touches LLM inference directly; LLM restarts never take down the UI.
10. **Structured evaluation:** System 2 communicates only via JSON verdicts, never free text — preventing context rot and conversational loops.
11. **Risk-tiered access:** Not all actions need deep evaluation. Fast-path for low-risk operations, deep eval only where safety demands it.

---

# PART II — THE ARCHITECTURE

*How the system fits together. Layers, components, data flow, and evolutionary pipelines.*

---

## 5. SYSTEM ARCHITECTURE

FabricOS separates concerns into three clean layers: immutable host for reachability and safety, mutable app for product behavior, and dedicated compute for models.

### Core Access Flow
```
Phone/Laptop (Tailscale)
    ↓
Talos Node (Tailscale extension + fabric-agent extension)
    ↓ (node console checks service health)
Kubernetes Services (fabric-ui, fabric-api, git-reconciler)
    ↓ (orchestration requests)
LLM Pods (llm-router → model-runners)
```

### 5.1. Talos Extension Layer (Host)

**Purpose:** Stable host entrypoint, recovery console, node health dashboard.

**Components:**
```
tailscale                    # Official extension: private network access
fabric-agent (custom ext)    # Node console: health/status/fallback UI
```

**fabric-agent responsibilities:**
- Node boot state, Talos health, disk/network status
- Tailscale connectivity status
- Fabric backend service health checks
- Actions: reconnect, reload config, request upgrade, "Open Fabric UI"
- Break-glass recovery API

Do NOT put heavy orchestration logic, a large web frontend, or model execution here. Those evolve too often and are better handled as Pods.

### 5.2. Kubernetes System Pods (Fabric OS Core)

**Purpose:** Main product platform — UI, API, orchestration, GitOps, System 2 enforcement.

| Component | Controller | Purpose |
|-----------|------------|---------|
| `fabric-ui` | Deployment | Two-sided left/right brain fluid UI |
| `fabric-api` | Deployment | Central orchestration API |
| `git-reconciler` | Deployment/CronJob | Fetches desired state from Git |
| `policy-engine` | Deployment | System 2 approval logic (SLM guardian) |
| `fabric-db` | StatefulSet | Persistent app state |

A Kubernetes Service gives each component a stable endpoint even as Pods restart or move. The node console targets Services (not individual Pod IPs) via stable DNS:
```
fabric-ui.fabric.svc.cluster.local:8080
fabric-api.fabric.svc.cluster.local:3000
git-reconciler.fabric.svc.cluster.local:9000
llm-router.ai.svc.cluster.local:8000
```

### 5.3. LLM Workload Pods (AI Layer)

**Purpose:** Model inference, routing, tool execution. Separated from the control plane so inference can move to another node, dedicate GPUs, or restart without taking down the main UI/API layer.

| Component | Controller | Purpose |
|-----------|------------|---------|
| `llm-router` | Deployment | Routes requests to model backends |
| `model-runner-*` | Deployment/StatefulSet | Chat/completion models |
| `embedding-service` | Deployment | Embedding models |
| `tool-worker-*` | Deployment/Job | Long-running tool tasks |

Use StatefulSets only when you need sticky identity or persistent storage (model caches, checkpoints). Stateless inference gateways use Deployments.

### 5.4. Access Path (Tailscale)

Tailscale is the private transport. Two layers:
- **Node-level**: reachability through the Talos fabric-agent extension
- **Cluster-level**: optional exposure of Services using the Tailscale Kubernetes operator

Two access paths:
- **Primary UX path**: phone → Tailscale → `fabric-ui` Service on Kubernetes
- **Fallback path**: phone → Tailscale → node console on Talos extension (if Fabric UI is down)

The phone does not need to run Kubernetes or be "part of the cluster." It only needs Tailscale and permission to reach the exposed node or service.

### 5.5. Interaction Model

1. User opens `fabric-node.tailnet.ts.net` over Tailscale
2. Node console checks `fabric-ui` and `fabric-api` Service health
3. If healthy → opens the full left/right brain UI served by `fabric-ui` Pods
4. The UI talks to `fabric-api`, which orchestrates workloads and calls `llm-router`
5. If unhealthy → node console stays on rescue screen and offers host-level actions

### 5.6. Tri-Brain to System Architecture Mapping

| Tri-Brain Layer | System Component | Runtime |
|-----------------|------------------|---------|
| Left Brain (Creative) | `model-runner-*`, `llm-router`, `tool-worker-*` | K8s Pods (AI layer) |
| System 2 (Guardian) | `policy-engine` + seL4 hypervisor | K8s Deployment / host firmware |
| Right Brain (Body) | `fabric-api`, `fabric-ui`, `git-reconciler`, `fabric-db` | K8s Pods (app layer) |
| Host (Immutable) | Talos Linux + Tailscale + fabric-agent | Talos extensions |

---

## 6. EDGE-DISTRIBUTED TOPOLOGY & UNIVERSAL COMPUTE

FabricOS decouples the "Brain" from the "Body," scaling from a $10,000 workstation to a discarded Android phone. It operates on a Compute-First, Hardware-Second philosophy:

* **Core Nodes (DGX/GB10):** High-end servers hosting the heavy Left Brain LLM for complex reasoning, RAG vector storage, and fine-tuning/NIM orchestration. Runs the full Kubernetes stack.
* **Edge Nodes (Jetson/Raspberry Pi/Phones):** Devices running the local System 2 Guardian and Right Brain execution via k3s. **Fail-Safe Protocol:** If the Left Brain remote connection drops, System 2 triggers a Fail-Safe Protocol, defaulting to local safe-stop or autonomous survival states.
* **Micro Nodes (ESP32S3):** Sensors and actuators functioning as the "Nervous System" via Zenoh-pico.
* **The Zenoh Bus:** Replaces ROS 2 (DDS) to prevent "discovery storms." Zenoh treats all nodes (over local Wi-Fi, Ethernet, or LoRa transport) as a single, unified address space.

---

## 7. COLD START: SELF-DISCOVERY PROTOCOL

Upon initialization or hardware change, FabricOS executes a dual-phase introspection:
1. **Internal Discovery:** The OS audits its own neural network (weights, context window, loaded skills, quantization state).
2. **External Discovery:** The Right Brain (Talos/Kubernetes) probes the hardware (Jetson Orin stats, GPIO status, mesh signal, node health) and generates a JSON manifest.
3. **Adaptation:** This manifest updates the Left Brain's System Instructions, allowing the LLM to adapt to physical constraints (e.g., routing a heavy task to a Docker container on the Jetson, but routing a lightweight task to a WASM module on an ESP32S3).

The flow:
```
Cold Start → Talos boots → fabric-agent probes hardware
    → generates manifest.json → fabric-api reads manifest
    → updates LLM system instructions → Left Brain adapts
```

---

## 8. KNOWLEDGE MATURATION & BIOLOGICAL CI/CD

The transition from creative thought to robotic action is an automated Continual Evolution pipeline. Every action the Left Brain proposes passes through a rigorously defined evaluation and promotion chain before reaching the hardware layer.

### 8.1. The Command Manifest & Risk Tiers

The heart of the evaluation system is a **pre-generated, signed registry** of every action the Left Brain is permitted to propose. System 2 (the policy-engine SLM) is trained on this manifest — a compiled library of command sheets, packages, and metadata.

Every request is categorized into a **Risk Tier** that determines the depth of the CI/CD check:

* **Risk Tier 0 (Auto-Approved):** Read-only queries, status checks, UI navigation. System 2 does not wake up; action executes immediately.
* **Risk Tier 1 (Fast-Path):** Simple match against the manifest for instant verification. Examples: launching known containers, querying Zenoh nodes.
* **Risk Tier 2 (Deep Eval):** Required for any action that mutates configurations, installs packages, or writes to the Right Brain filesystem. Triggers full Deterministic Linter pipeline.
* **Unknown Commands:** Hard block with no negotiation permitted. The Left Brain receives a rejection and must throw a `RejectedActionError` immediately.

### 8.2. The Three-Stage Maturation Pipeline

A new capability must pass through three distinct phases before it is permanently added to the system's repertoire:

* **Phase I: Left-Brain CI (Creative Sandbox):** Rapid, iterative testing where the LLM is allowed to hallucinate and prototype new Python/Docker drivers when new hardware connects. Runs as ephemeral tool-worker Pods. "Hallucinations" are permitted as part of the creative prototyping process.
* **Phase II: Right-Brain CI (Deterministic Linter):** The strict gatekeeper. To pass, experimental code must satisfy three requirements:
  1. **Formal Schema Check:** Matches strict Zenoh-RPC naming conventions.
  2. **Resource Sandboxing:** Simulates memory/CPU limits to prevent OOM crashes (Kubernetes resource limits).
  3. **Property-Based Testing:** Fuzz-tests code with edge-case physical inputs using frameworks like Hypothesis.
* **Phase III: Promotion:** Once the code survives the linter, it is signed by System 2 (policy-engine) and deployed as an immutable Kubernetes workload via GitOps, permanently upskilling the system.

### 8.3. Structured Evaluation: JSON Verdicts

To prevent "context rot" or endless rephrasing loops, System 2 communicates with the Left Brain exclusively through **structured JSON enums** rather than free-text prose. The verdict schema includes:

```json
{
  "decision": "approved | rejected | deferred",
  "risk_tier": 0,
  "reason_code": "MANIFEST_MATCH | RESOURCE_OVERFLOW | SCHEMA_VIOLATION",
  "retry_permitted": false,
  "message": "Human-readable summary for UI display"
}
```

If `retry_permitted` is `false`, the Left Brain must throw a `RejectedActionError` immediately rather than attempting to rephrase the failed intent. This deterministic communication protocol ensures System 2 never gets trapped in conversational loops.

### 8.4. GitOps Flow
```
Left Brain proposes skill → tool-worker tests it
    → policy-engine evaluates (risk tier + manifest check)
    → deterministic linter validates (schema + sandbox + fuzz)
    → policy-engine signs verdict
    → git-reconciler commits to Git
    → Flux deploys as new Pod/workload
    → skill is live and immutable
```

---

## 9. DATA MANAGEMENT & STATEFUL SERVICES

FabricOS treats all state as Kubernetes-native primitives. The immutable host provides zero persistent storage — all durable data lives in PVCs, StatefulSets, and object storage managed by the cluster layer.

### 9.1. Storage Architecture

Three storage tiers serve different needs:

* **Local-Path Provisioner:** Dynamic node-local PVs for homelab and single-node deployments. Watches unbound PVCs and creates volumes automatically under `/var/lib/local-path-provisioner/`. Ideal for app data, model caches, and DB state.
* **Longhorn (RWX):** k3s-native distributed storage providing ReadWriteMany volumes for multi-Pod collaboration. Enables shared workspaces where OpenClaw agents, Fabric API, and terminal Pods see the same files simultaneously.
* **MinIO (S3-Compatible):** Object storage for large files, fine-tuning datasets, model checkpoints, and backup. Acts as the "GDrive-like" layer accessible from any Pod via S3 API.

### 9.2. Stateful Components

| Component | Controller | Storage | Purpose |
|-----------|------------|---------|---------|
| `fabric-db` | StatefulSet | PVC (local-path) | Metadata, user data, sessions |
| `karakeep` | StatefulSet | PVC (local-path) | Bookmarks, search index |
| Model caches | PVC | local-path → fast SSD | LLM weights, embeddings |
| MinIO | StatefulSet | PVC/hostPath (50Gi+) | Object storage (datasets, backups) |
| OpenClaw workspace | RWX PVC | Longhorn | Shared agent workspace |

### 9.3. Data Flow
```
Git (desired state)
    ↓
fabric-api Pod → git-reconciler → PVC-backed DB
    ↓
LLM requests → llm-router → model-runner PVCs
    ↓
Large files → MinIO object storage
    ↓
Shared workspace → Longhorn RWX volume (multi-Pod access)
    ↓
Backups → Velero / rsync to external storage
```

### 9.4. Terminal Access

Two layers of shell access, both with zero impact on the immutable host:

* **Host terminal:** `talosctl` — the main Talos interface. No SSH. Provides dashboard, node health, extension logs, and kubeconfig extraction.
* **Cluster terminal:** Ephemeral `kubectl` debug Pods, VS Code Server Pods, or `kubectl exec` into running workloads.

Terminal Pods mount shared workspace volumes, so edits are visible across all connected services.

### 9.5. Migration Path
```
Stage 1: PVC → local-path (single node, immediate)
Stage 2: PVC → MinIO (object storage for large files)
Stage 3: PVC → distributed CSI (multi-node cluster scaling)
```

Data survives Pod restarts, node reboots, and PVC recreation. Data does NOT survive disk wipes or PVC deletion without backups.

---

# PART III — THE SPECS

*Concrete details: hardware, schemas, configs, repo structure, and the open questions we still need to research.*

---

## 10. HARDWARE SPECIFICATION (THE CORE NODE)

To support the Left Brain's continuous retraining pipeline, the chosen hardware for the FabricOS "Core Node" is the **ASUS Ascent GX10-GG0027BN** (Powered by the NVIDIA GB10 Grace Blackwell Superchip).

### 10.1. Asus GX10 vs. NVIDIA Spark DGX

While both systems utilize the same ARM v9.2-A CPU (Grace) and 128GB LPDDR5x Unified Memory, the Asus GX10 was selected over the NVIDIA Spark due to:
1. **The Hacker/Prosumer Advantage:** The GX10 lacks the "Enterprise Tax" (~€1,000 savings) and does not force users into the locked-down DGX OS ecosystem, allowing for custom OS installations like Talos Linux.
2. **Thermal Efficiency:** The GX10 utilizes a custom "QuietFlow" triple-fan/vapor-chamber design, preventing thermal throttling during 4+ hour LLM fine-tuning runs (a known issue in the compact Spark Founders chassis).
3. **Purchasing Strategy:** Utilizing a "Returned & Tested" (B-Ware) unit via Galaxus.de maximizes price-to-performance, securing the highest tier model for under €4,000 while retaining a full 24-month statutory EU warranty.

### 10.2. Storage Constraints & The 4TB Requirement

The GX10 chassis introduces a strict physical limitation: **It features only one (1) M.2 2242 (42mm) NVMe slot.**
* **The Trap:** Buying a 1TB or 2TB model with the intent to upgrade is not feasible. Standard 8TB drives (M.2 2280) will not physically fit, and high-capacity 2242 drives are double-sided, leading to severe thermal throttling against the motherboard.
* **The Standard:** Therefore, the **factory 4TB model is a non-negotiable requirement** to ensure sufficient high-speed PCIe Gen 5 storage for multimodal ROS2/Zenoh logs, LLM weight checkpoints, and NVIDIA NIM caching without bottlenecking the 128GB VRAM.

### 10.3. Operating System Parity

**Talos Linux** natively supports the ARM64 architecture of the Grace CPU. Since it integrates the standard NVIDIA Linux BSP, it provides the deterministic, immutable rootfs required by the Right Brain while fully supporting the GX10's ConnectX-7 (200GbE) networking for future cluster scaling. Talos boots directly into a container runtime — no package manager at the OS layer. The entire FabricOS application stack runs as containers on top.

### 10.4. Core Node Summary

| Spec | Detail |
|------|--------|
| Model | ASUS Ascent GX10-GG0027BN |
| Chip | NVIDIA GB10 Grace Blackwell Superchip |
| CPU | ARM v9.2-A (Grace) |
| Memory | 128GB LPDDR5x Unified |
| Storage | 4TB M.2 2242 PCIe Gen 5 (non-upgradeable) |
| Networking | ConnectX-7 (200GbE) |
| OS | Talos Linux (ARM64) |
| Budget | Under €4,000 (B-Ware via Galaxus.de) |

---

## 11. GIT REPOSITORY STRUCTURE

```
fabric-os/
├── talos/
│   ├── schematic.yaml              # Image Factory extensions
│   ├── machineconfig-control.yaml
│   └── machineconfig-worker.yaml
├── k8s/
│   ├── flux/                       # GitOps bootstrap
│   ├── fabric/                     # System pods
│   │   ├── ui-deployment.yaml
│   │   ├── api-deployment.yaml
│   │   ├── policy-engine-deployment.yaml
│   │   ├── git-reconciler-deployment.yaml
│   │   └── db-statefulset.yaml
│   ├── ai/                         # LLM workloads
│   │   ├── llm-router.yaml
│   │   ├── model-runners.yaml
│   │   └── tool-workers.yaml
│   └── addons/                     # Storage & stateful services
│       ├── local-path-provisioner.yaml
│       ├── longhorn-values.yaml
│       ├── minio-helm-values.yaml
│       ├── karakeep-statefulset.yaml
│       └── openclaw-workspace-pvc.yaml
├── compose/                        # Prototype stack
│   └── docker-compose.yaml
├── linter/                         # Deterministic Linter scripts
├── schemas/                        # Manifest.json + verdict drafts
├── system2/                        # Phi-4 rejection logic prompts
└── docs/
    └── whitepaper.md               # This document
```

---

## 12. PROGRESSION ROADMAP

```
Stage 1: Docker Compose prototype
├── tailscale (host)
├── fabric-ui, fabric-api, llm-router (compose services)
└── local-path-provisioner for storage

Stage 2: Talos + k3s
├── tailscale + fabric-agent (Talos extensions)
├── Fabric services (k3s Deployments)
├── LLM services (k3s Deployments)
└── Longhorn + MinIO for storage scaling

Stage 3: Full GitOps
├── Talos machine config (git)
├── Image Factory schematic (git)
├── Flux-managed Kubernetes manifests (git)
└── LLM workloads (git + PVCs)
```

---

## 13. OPEN QUESTIONS & RESEARCH TRACKER

*Topics requiring active scripting, research, or architectural decisions. Tagged by area: [IDEA], [ARCH], [SPECS].*

### 13.1. Unwritten Schemas (PoC Scoping) [SPECS]

1. **System 2 Rejection Logic:** Scripting the exact prompt/formatting the Phi-4 model uses to issue rejection messages to the Left Brain when a deterministic safety mask is breached.
2. **The Manifest.json Schema:** Defining the JSON structure the Right Brain uses to report hardware telemetry (RAM, GPIO, connected Zenoh nodes) to the Left Brain during a Cold Start.
3. **Perplexity-to-Promotion Workflow:** Mapping the API routing from a Perplexity web-search result into a testable script for the Deterministic Linter.
4. **Talos Machine Config:** Drafting the Talos machine configuration and Image Factory schematic to pin the GB10 NVIDIA drivers for the Asus GX10.
5. **Flux GitOps Bootstrap:** Defining the Flux configuration for automated deployment of Fabric OS and LLM workloads from Git.
6. **System 2 Verdict Schema:** Finalizing the JSON verdict structure (Section 8.3) — which `reason_code` values to define, how `deferred` decisions are handled, and whether Tier 0 bypass requires any signed manifest entry at all.

### 13.2. Architecture Alignment [ARCH]

7. **OpenClaw Identity:** The term "OpenClaw" appears in multiple contexts — as the FabricOS brand name (Section 1), as an agent deployment workload (RWX PVM docs), and possibly as a third-party tool. Clarify: is "OpenClaw" our multi-agent runtime, or is it an external dependency we integrate?
8. **Karakeep as Default App:** Is Karakeep still a planned first-party workload, or was it an example during research? If planned, does it belong in the core FabricOS stack or as a user-installable addon?
9. **`/fabric/bin/` vs Container-Native Promotion:** Biological CI/CD references skills being locked into an immutable `/fabric/bin/` path. But Talos has no mutable host filesystem. Where does a promoted skill actually land? Is it a new container image, a Git commit that Flux reconciles, or a sidecar injected into existing Pods?
10. **seL4 on ARM64 Grace CPU:** The Tri-Brain architecture references seL4 as a Type-1 hypervisor backing System 2. Has seL4 been verified to run on the NVIDIA Grace (ARM v9.2-A) architecture? What's the current state of seL4 on GB10/Grace Blackwell?
11. **MinIO vs Distributed CSI Timing:** When do we move beyond local-path-provisioner? Is MinIO the long-term answer for large storage, or does a distributed CSI driver (Longhorn, Ceph) take over in Stage 3? What triggers the migration?
12. **Longhorn on Single-Node GX10:** Longhorn RWX is designed for multi-node clusters. On a single ASUS GX10, is Longhorn overkill? Does local-path-provisioner + hostPath suffice until Stage 2/3 scaling?

### 13.3. Concept & Vision [IDEA]

13. **Fail-Safe Protocol Definition:** Edge nodes reference a "Fail-Safe Protocol" when the Left Brain drops — defaulting to local safe-stop or autonomous survival. What exactly are the safe-stop states? What triggers autonomous mode vs. hard stop? Needs concrete state machine definition.
14. **Zenoh vs ROS 2 Migration:** The architecture replaces ROS 2/DDS with Zenoh. But existing robotics hardware ships with ROS 2 drivers. What's the compatibility strategy? Zenoh-ROS2 bridge, full rewrite, or dual-protocol support?

---

## 14. LOSSLESS USER INPUTS ARCHIVE

*(A chronological log of all prompts that shaped this architecture, ensuring complete grounding)*

1. `"can we run nemotron on a dgx ?"`
2. `"it seems like in not so long time we will have devices..." [full vision]`
3. `"please be my a chief of staff to build this vision"`
4. `"please the smartest visionary... help me critique..."`
5. `"now that you know my vision, help to envision a complete new execution..."`
6. `"a few. notes to add - a dual flip interface, like a crystal touchball..."`
7. `"help write the draft, whitepaper. and keep Version control..."`
8. `"all semandtic data are on the right left brain, all sits like docker..."`
9. `"lets deploy that same principle of the paper on this chat thread..."`
10. `"so as long as i say anything after the left keyword..."`
11. `"can you save this in memory so i can retrieve this context..."`
12. `"Great regenerate the entire text with one more section..."`
13. `"Great regenerate the entire text with one more section. please include all the texts i input..."`
14. `"Left lets debate the entire paper from the lense of the smartest researcher..."`
15. `"Right - docker and wasm usage combination. We need something in future..."`
16. `"Right - the left brain interface the fluid text vla based interface would have simple commands like / and ."`
17. `"Right - The right interface would be a classic bash, terminal linux like interface, highly deterministic"`
18. `"Right - whenever a cold start the job of the brain system is two fols 1) to understand itself... 2) to understand the hardware it is in..."`
19. `"Left - avocado os vs sel4 architecture and why it evolved out of what market need.gap"`
20. `"Left - can me make the sel4 embedded into he sustem 2, which is the security protocol between left and right brain, we can train a smal phi like model for this?"`
21. `"Right- Promotion happen? Is it a human approval? Or does the Right Brain run a Deterministic Linter (like a CI/CD pipeline) - yes something like this"`
22. `"Left - how would the hardware self audit happen... How can the llm... help to understand and adapt to hardware"`
23. `"Right- Tri-Brain Architecture. By nesting seL4 as the hypervisor/enforcer for the System 2 supervisor..."`
24. `"Right - if linux was the opx of the era, and open claw the new os of era..."`
25. `"Right -our poc use case is on jetson and dgx and esp32s3 and ros enabled. But this should run on envery other raspberry pi and old phones..."`
26. `"Right - the right deterministic kernel terminal like interface, can have plugins like vscode right - the left brain side interface can access a perplexity computer via a plugin api call /"`
27. `"Right - Testing... minici cd pipelines within left brain... stricter cicd for right... combination of docker, drivers, python"`
28. `"Left - help me understand zenoh, yocto avocado base, and how it compare to ros 2 and lora technology"`
29. `"Right - cold start situation, when a new hard comes the new hardware tested libraries goes to the zenoh right brain..."`
30. `"Right- The Configuration: Left Brain: Remote API / Jetson Core. System 2: Local Phi-4-mini... Right Brain: Local GPIO control via Avocado OS."`
31. `"Left - slightly off topic from the paper but related i am planning for a hardware addition,should i go for the gx10 4tb version or the 2tb... gb10 hardware and cuda and nim..."`
32. `"Left - great 4tb sounds like the right choice. Though one question debate and critique the asus gx10 vs the nvdia spark. Price is almost 1k more"`
33. `"-left The Spark isn't just a card; it's a ticket into NVIDIA's enterprise walled garden., research and find me 5 disadvantages..."`
34. `"Can you verify with grounded info, ac Avocado os can run on gx10 asus"`
35. `"- left, ok y ah the extra 2tb makes sense, but i am thinking's to add more ssd 2tb after market later if i need more space. Can you please check if feasible on the asus"`
36. `"Is buying a returned and tested on galaxus . De a good idea has the 24 m warranty etc... ASUS GX10-GG0027BN 4000 GB, 128 GB, NVIDIA Blackwell..."`
37. `"Can we purchase the 1 tb version gx10 asus and upgrade with a 8tb ssd nvme. grounded answer only, i think it has only 1 slot thoroughly check with user reviews"`
38. `"Now give me a full document draft of the whitepaper. All chats Add a section if any topica was discussed but not included in the paper..."`
39. `"Can you draft in a pdf."`
40. `"Export the entire whitepaperr we discussed, and any other content important and not added as a running list at the end. Export as md"`
41. `"can we pickup on writing the fabric OS whitepaper"`
42. `"- Merge/consolidate v2 + the Talos architecture pieces into one unified whitepaper"`
43. `"Lets map these to our later draft, and see whats not already included in latest draft, and add where ever it is aligned... Where its not aligned add those topics to the running open questions tracker"`
44. `"We want to largely have these three broad areas: Idea, architecture, specs. Lets adapt with the new merged paper and create a new version. Lets keep strict version control."`

---

# VERSION HISTORY

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | March 2026 | Initial draft (Vision + Tri-Brain + Hardware) |
| 2.0 | March 22, 2026 | Consolidated: merged v2 + Talos architecture + Fabric Console |
| 2.1 | March 22, 2026 | Merged Biological CI/CD, Data Management, Spatial UI detail. Added 14 open questions. |
| **3.0** | **March 22, 2026** | **Restructured into Idea → Architecture → Specs pillars. Version control enforced.** |
