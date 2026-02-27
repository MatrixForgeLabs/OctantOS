# OctantOS

> One static binary. Entire agent operating system.

## At A Glance

- **~23MB** statically linked Rust binary for the full OS runtime
- **8 core services in ~83ms** from `FROM scratch`
- **125k+ lines of first-party Rust**
- **50 Rust packages** in one workspace (dozens of focused crates)
- **10-layer defense stack** with kernel-enforced isolation
- **Firecracker microVM isolation** integrated into agent execution paths
- **190+ self-contained search engines** with no external API query leakage
- **7 LLM providers** with intelligent routing, fallback, and policy control
- **81 multicall utilities** shipped in the same binary
- **Built by 1 developer in ~3 months**

## Architecture Stack

```text
+------------------------------------------------------------------+
| Operator Layer                                                    |
|  Shell, admin workflows, policy controls, observability          |
+------------------------------------------------------------------+
| Agent Layer                                                       |
|  Runtime, lifecycle, capability checks, skill execution           |
+------------------------------------------------------------------+
| Services Layer                                                    |
|  Matrix, Vault, Router, Search, Context, Quorum, Inference       |
+------------------------------------------------------------------+
| Runtime Layer                                                     |
|  Single multicall binary, init/dispatch, package/service control  |
+------------------------------------------------------------------+
| Kernel Layer                                                      |
|  Namespaces, cgroups, seccomp-BPF, eBPF/LSM enforcement           |
+------------------------------------------------------------------+
```

## What Makes This Different

1. Security is foundational, not bolted on after launch.
2. Agents are default-deny by design, not trust-first.
3. Capability manifests are signed, validated, and enforced.
4. Isolation is Linux-kernel enforced, not prompt-level.
5. Entire platform ships as one deployable binary.
6. Search is self-contained; query intent is not leaked to third-party API providers.
7. Multi-provider LLM routing avoids lock-in and handles failover.
8. Every action can be traced through an auditable ledger.
9. Architecture is service-complete (identity, memory, comms, governance), not just tool wrappers.
10. The system is built for autonomous-agent operations, not chatbot demos.

## Why Not Just Use OpenClaw?

| Dimension | OpenClaw | OctantOS |
|---|---|---|
| Security posture | Primarily app-layer controls | 10-layer defense, kernel-enforced boundaries |
| Capability control | Limited or ad hoc | Signed capability manifests, default-deny execution |
| Isolation model | Shared process trust assumptions | Per-agent namespace/cgroup/seccomp isolation |
| Auditability | Partial runtime logs | Action-ledger model with tamper-evident chain semantics |
| Search behavior | External API dependency common | 190+ self-contained engines, no external search API leakage |
| Deployment model | Multi-component stack | Single static binary, minimal runtime assumptions |
| LLM routing | Provider-specific bias likely | 7-provider routing with fallback and policy-driven selection |
| Production hardening focus | Feature velocity first | Threat-model-driven from first principles |

## Repository Scope

This public repository intentionally contains **no implementation source code**.

It exists to document architecture, security model, roadmap, and release progression.

## Documentation

- [`docs/architecture-overview.md`](docs/architecture-overview.md)
- [`docs/security-model.md`](docs/security-model.md)
- [`docs/capability-manifests.md`](docs/capability-manifests.md)
- [`docs/agent-lifecycle.md`](docs/agent-lifecycle.md)
- [`docs/single-binary.md`](docs/single-binary.md)

## Public Updates

- [`ROADMAP.md`](ROADMAP.md)
- [`CHANGELOG.md`](CHANGELOG.md)
- [`FAQ.md`](FAQ.md)
- [`SECURITY.md`](SECURITY.md)
- [`CONTRIBUTING.md`](CONTRIBUTING.md)
