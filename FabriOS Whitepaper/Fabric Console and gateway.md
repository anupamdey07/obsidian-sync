Export this as an md file

# Fabric OS Architecture Skeleton

Talos provides the immutable host with Tailscale connectivity and a minimal node console extension. Kubernetes (or k3s/Docker Compose) runs the main Fabric OS application layer. LLM workloads live as separate Pods. Tailscale provides private access from mobile devices.

## Core Layers

```
Phone/Laptop (Tailscale) 
    ↓
Talos Node (Tailscale extension + fabric-agent extension)
    ↓ (node console checks service health)
Kubernetes Services (fabric-ui, fabric-api, git-reconciler)
    ↓ (orchestration requests)
LLM Pods (llm-router → model-runners)
```

## Talos Extension Layer

**Purpose**: Stable host entrypoint, recovery console, node health dashboard.

**Components**:
```
tailscale                    # Official extension: private network access
fabric-agent (custom ext)    # Node console: health/status/fallback UI
```

**fabric-agent responsibilities**:
- Node boot state, Talos health, disk/network status
- Tailscale connectivity status
- Fabric backend service health checks
- Actions: reconnect, reload config, request upgrade, "Open Fabric UI"
- Break-glass recovery API

## Kubernetes System Pods (Fabric OS Core)

**Purpose**: Main product platform - UI, API, orchestration, GitOps.

| Component | Controller | Purpose |
|-----------|------------|---------|
| `fabric-ui` | Deployment | Two-sided left/right brain fluid UI |
| `fabric-api` | Deployment | Central orchestration API |
| `git-reconciler` | Deployment/CronJob | Fetches desired state from Git |
| `policy-engine` | Deployment | System 2 approval logic |
| `fabric-db` | StatefulSet | Persistent app state |

## LLM Workload Pods

**Purpose**: Model inference, routing, tool execution.

| Component | Controller | Purpose |
|-----------|------------|---------|
| `llm-router` | Deployment | Routes requests to model backends |
| `model-runner-*` | Deployment/StatefulSet | Chat/completion models |
| `embedding-service` | Deployment | Embedding models |
| `tool-worker-*` | Deployment/Job | Long-running tool tasks |

## Access Flow

```
1. Phone → Tailscale → fabric-node.tailnet.ts.net (node console)
2. Node console → checks fabric-ui service health  
3. ✓ Healthy → redirect to fabric-ui.tailnet.ts.net (main UI)
4. ✗ Unhealthy → stay on node console (rescue mode)
5. Fabric UI → fabric-api → llm-router → model pods
```

## Service Discovery

Node console reaches Fabric services via stable Kubernetes Service DNS:
```
fabric-ui.fabric.svc.cluster.local:8080
fabric-api.fabric.svc.cluster.local:3000  
git-reconciler.fabric.svc.cluster.local:9000
llm-router.ai.svc.cluster.local:8000
```

## Progression Roadmap

```
Stage 1: Docker Compose prototype
├── tailscale (host)
├── fabric-ui, fabric-api, llm-router (compose services)

Stage 2: Talos + k3s  
├── tailscale + fabric-agent (Talos extensions)
├── Fabric services (k3s Deployments)
└── LLM services (k3s Deployments)

Stage 3: Full GitOps
├── Talos machine config (git)
├── Image Factory schematic (git) 
├── Flux-managed Kubernetes manifests (git)
└── LLM workloads (git + PVCs)
```

## Key Principles

1. **Host layer**: reachability + recovery (Talos extensions)
2. **App layer**: product behavior (Kubernetes Services/Deployments)  
3. **Workload layer**: models + compute (dedicated Pods)
4. **Git as source of truth**: configs, manifests, desired state
5. **Tailscale everywhere**: private access without public exposure

## Git Repository Structure

```
fabric-os/
├── talos/
│   ├── schematic.yaml          # Image Factory extensions
│   ├── machineconfig-control.yaml
│   └── machineconfig-worker.yaml
├── k8s/
│   ├── flux/                   # GitOps bootstrap
│   ├── fabric/                 # System pods
│   │   ├── ui-deployment.yaml
│   │   ├── api-deployment.yaml
│   │   └── db-statefulset.yaml
│   └── ai/                     # LLM workloads
│       ├── llm-router.yaml
│       └── model-runners.yaml
└── compose/                    # Prototype stack
    └── docker-compose.yaml
```

This skeleton separates concerns cleanly: immutable host (Talos), mutable app (Kubernetes), heavy compute (LLM Pods), private transport (Tailscale).

Citations:
[1] selected_image_4260137130203372782.jpg https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/images/160990054/ee167d32-41a2-4a31-8edb-015eb5d82ae3/selected_image_4260137130203372782.jpg
