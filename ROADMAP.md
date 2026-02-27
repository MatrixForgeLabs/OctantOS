# Roadmap

This roadmap tracks OctantOS capability areas and their progression. It is organised by what the system can do, not how it does it.

Progress indicators reflect the maturity of each area as a whole, not a line-item checklist.

---

## Core Runtime

**Status: Operational · Maturity: 85%**

The single-binary runtime, init system, multicall dispatch, and service orchestration are functional. Eight core services boot from cold start in approximately 83 milliseconds. Remaining work focuses on hardening, edge-case resilience, and production deployment tooling.

| Capability | Status |
|---|---|
| Single static binary (multicall architecture) | ✅ Complete |
| PID 1 init with service dependency graph | ✅ Complete |
| 81 built-in utilities (BusyBox-style) | ✅ Complete |
| Signal handling, graceful shutdown | ✅ Complete |
| Service health monitoring and restart | ✅ Complete |
| Deployment packaging (Docker, VM, ISO, Edge) | 🔧 In progress |
| Cross-architecture builds (x86_64 + aarch64) | 📋 Planned |

---

## Security and Isolation

**Status: Operational · Maturity: 85%**

The 10-layer defence stack is architecturally complete. Namespace isolation, seccomp filtering, capability manifests, and the credentials vault are functional. Firecracker microVM integration is actively in progress, adding hardware-level isolation boundaries to the existing kernel-level controls.

| Capability | Status |
|---|---|
| Linux namespace isolation (mount, PID, net, IPC, UTS) | ✅ Complete |
| cgroup v2 resource limits per agent | ✅ Complete |
| seccomp-BPF syscall filtering (default-deny) | ✅ Complete |
| Capability manifest validation and enforcement | ✅ Complete |
| Ed25519 manifest signing and verification | ✅ Complete |
| Encrypted credential vault (AES-256-GCM, scoped tokens) | ✅ Complete |
| Hardware-bound machine identity | ✅ Complete |
| Immutable root filesystem design | ✅ Complete |
| Firecracker microVM agent isolation | ✅ Complete |
| dm-verity verified boot | 📋 Planned |
| Custom LSM kernel module | 📋 Planned (Phase 2) |

---

## Agent Lifecycle

**Status: Operational · Maturity: 75%**

Agents spawn with signed capability manifests, execute within isolated environments, communicate over audited channels, and terminate cleanly. Child agents inherit a strict subset of parent permissions. Remaining work covers advanced lifecycle patterns, fleet-scale management, and deeper integration with microVM boundaries.

| Capability | Status |
|---|---|
| Agent spawning with manifest validation | ✅ Complete |
| Parent-child permission inheritance (intersection) | ✅ Complete |
| Agent identity (AIDs) and lineage tracking | ✅ Complete |
| Workstation construction (namespace + cgroup) | ✅ Complete |
| Graceful retirement and resource cleanup | ✅ Complete |
| Multi-agent orchestration (agents spawning agents) | ✅ Complete |
| Fleet management and monitoring tooling | 📋 Planned |

---

## Platform Services

**Status: Operational · Maturity: 80%**

All eight boot services are functional. Each service listed below starts as part of the init dependency graph and is available to agents and operators within the boot window.

| Service | Purpose | Status |
|---|---|---|
| **Credentials Vault** | Encrypted secret storage, scoped access | ✅ Operational |
| **LLM Router** | Multi-provider routing, failover, policy control | ✅ Operational |
| **Search** | Self-contained metasearch (190+ engines) | ✅ Operational |
| **Semantic Memory** | Tiered context store with local embeddings | ✅ Operational |
| **Governance** | Multi-model parliamentary decision engine | ✅ Operational |
| **Local Inference** | On-device model execution (multiple architectures) | ✅ Operational |
| **Communication** | Embedded homeserver with federation and E2EE | ✅ Operational |
| **Action Ledger** | Tamper-evident hash-chain audit log | ✅ Operational |

---

## Agent Skills and Tooling

**Status: Operational · Maturity: 70%**

Agents execute skills as compiled native binaries rather than interpreted scripts. The skill compilation pipeline, package signing, and a catalog of workflow templates are in place. Ongoing work focuses on the package registry, trust hierarchy, and broader skill ecosystem tooling.

| Capability | Status |
|---|---|
| Workflow-to-binary compilation | ✅ Complete |
| Ed25519 package signing and verification | ✅ Complete |
| Skill catalog (20,000+ workflow templates) | ✅ Complete |
| Existing skill format conversion tooling | ✅ Complete |
| Package registry with HTTP API | ✅ Complete |
| Trust hierarchy (tiered verification) | 📋 Planned |
| Community skill marketplace | 📋 In Progress |

---

## Operator Experience

**Status: Partial · Maturity: 60%**

Operators can manage the system through shell access and remote messaging integrations. The web-based operator console and expanded diagnostics are under active development.

| Capability | Status |
|---|---|
| Interactive shell environment | ✅ Complete |
| Remote management via messaging platforms | ✅ Complete |
| CLI tooling (agent, ledger, vault, package commands) | ✅ Complete |
| Audit and governance visibility | ✅ Complete |
| Web dashboard (real-time monitoring) | 🔧 In progress |
| Approval queue and escalation workflows | 📋 Planned |
| Natural language system management | 📋 Planned |

---

## Federation and Distribution

**Status: Operational · Maturity: 70%**

Cross-machine agent communication uses a federated protocol with end-to-end encryption. Distributed compute mesh capabilities are functional. Remaining work focuses on deployment topology tooling and broader network management.

| Capability | Status |
|---|---|
| Federated inter-agent communication | ✅ Complete |
| End-to-end encryption for agent channels | ✅ Complete |
| Cross-machine agent orchestration | ✅ Complete |
| Distributed compute mesh | ✅ Complete |
| Hub-and-spoke deployment topology | 📋 Planned |
| Network management and observability | 📋 Planned |

---

## Upcoming Focus Areas

These represent the next major capability expansions, roughly in priority order:

**Near-term**
- Firecracker microVM integration completion
- Operator console (web dashboard) buildout
- Package registry and trust hierarchy
- Expanded deployment packaging and installation tooling

**Medium-term**
- Policy simulation and testing tools
- Advanced agent fleet management
- Structured early-access and beta onboarding
- Broader connector and integration ecosystem

**Longer-term**
- Self-evolution capabilities (autonomous system improvement under governance)
- Browser-based demonstration environment
- Mobile and edge deployment targets
- Custom kernel security module (LSM)
- Model fine-tuning orchestration with cryptographic provenance

---

## A Note on This Roadmap

This document describes *what OctantOS can do and will do*. It does not describe implementation internals, architectural wiring, or integration patterns.

For security architecture details, see [`SECURITY.md`](SECURITY.md) and [`docs/security-model.md`](docs/security-model.md).
