# OctantOS

### The agent security problem is architectural. So is the fix.

Current autonomous agent platforms give AI models a shell prompt and call it infrastructure. They store credentials in plaintext `.env` files. They run every agent under the same user, on the same filesystem, with the same network access. They protect you with *text-based instructions* and hope the model follows them.

Then they act surprised when agents get prompt-injected, credentials get exfiltrated, and entire deployments get compromised.

**OctantOS is a Linux-based operating system built from first principles for autonomous AI agent operations.** Not a framework bolted onto an existing OS. Not a Docker wrapper. Not another chatbot scaffold. A complete, security-first operating system where agents are first-class kernel-visible entities, isolated by default, and permitted only by explicit cryptographic grant.

One static Rust binary. Entire OS userspace. No shell. No coreutils. No interpreter. No attack surface you didn't ask for.

---

## By The Numbers

| | |
|---|---|
| **Binary size** | ~23MB — statically linked, vendored TLS, `FROM scratch` deployable |
| **Boot time** | ~83ms — 8 core services running from cold start |
| **Codebase** | 125,000+ lines of first-party Rust across 50 packages |
| **Search engines** | 190+ — fully self-contained, zero external API leakage |
| **LLM providers** | 7 — intelligent routing, automatic failover, policy-driven selection |
| **Built-in utilities** | 81 — multicall binary, BusyBox-style dispatch |
| **Security layers** | 10 — kernel-enforced, independent, defence-in-depth |
| **Agent isolation** | Firecracker microVM + Linux namespaces + seccomp-BPF |
| **Development time** | ~3 months |
| **Developers** | 1 |

That last line is not a typo.

---

## Architecture

After the Linux kernel transfers control to the `/octant` binary, there is no other software on the system. No shell, no package manager, no interpreter, no service manager. Every capability is compiled into the binary and dispatched through subcommands or symlinks.

```
┌──────────────────────────────────────────────────────────────┐
│  OPERATOR LAYER                                              │
│  Console dashboard · Policy controls · Remote management     │
│  Matrix/Telegram/Signal integration · MatrixShell            │
├──────────────────────────────────────────────────────────────┤
│  AGENT LAYER                                                 │
│  Lifecycle manager · Capability manifests · Compiled skills  │
│  Inter-agent comms (Matrix) · Tamper-evident action ledger   │
├──────────────────────────────────────────────────────────────┤
│  SERVICES LAYER                                              │
│  ProviderForge (LLM routing)    · King of Context (memory)   │
│  searchrs (190+ engines)        · Credentials Vault          │
│  llama.rs (local inference)     · Quorum (governance)        │
│  MatrixMesh (federation)        · BinaryCircuits (compiler)  │
├──────────────────────────────────────────────────────────────┤
│  RUNTIME LAYER                                               │
│  PID 1 init · Service graph · Multicall dispatch             │
│  Package manager · Namespace/cgroup construction             │
├──────────────────────────────────────────────────────────────┤
│  KERNEL LAYER                                                │
│  Linux namespaces · cgroups v2 · seccomp-BPF                 │
│  Firecracker microVMs · eBPF · dm-verity · LSM (Phase 2)    │
└──────────────────────────────────────────────────────────────┘
```

Every layer operates independently. The system is designed under the assumption that any individual layer may be compromised — and the remaining layers maintain containment regardless.

---

## 10-Layer Defence Stack

Most platforms have a security *feature*. OctantOS has a security *architecture*.

| Layer | Mechanism | What It Prevents |
|-------|-----------|-----------------|
| **1. Hardware Identity** | Ed25519 machine fingerprint, optional TPM | Unauthorised cloning, license migration |
| **2. Verified Boot** | dm-verity on squashfs root | Tampered OS binary, rootkit persistence |
| **3. Immutable Root** | Read-only squashfs, A/B atomic updates | Runtime OS modification by any process |
| **4. Memory Safety** | Pure Rust userspace, no `unsafe` in app code | Buffer overflows, use-after-free, data races |
| **5. Agent Identity** | Unique AIDs, lineage tracking, attribution | Identity confusion, unattributed actions |
| **6. Namespace Isolation** | Mount, PID, Net, IPC, UTS per agent | Cross-agent filesystem/process/network access |
| **7. Syscall Filtering** | Per-agent seccomp-BPF, default-deny | Kernel exploitation, prohibited operations |
| **8. Encrypted Secrets** | AES-256-GCM vault, Argon2id KDF, scoped tokens | Credential theft, plaintext exposure |
| **9. Tamper-Evident Audit** | SHA-256 hash-chain ledger, per-agent partitions | Log tampering, evidence destruction |
| **10. Governance** | Multi-model parliamentary voting, human gates | Unilateral agent decisions on critical actions |

Prompt injection is dangerous only when it crosses into unrestricted execution. In OctantOS, **there is no unrestricted execution.**

---

## The Problem With Everyone Else

This is not a theoretical comparison. These are architectural realities.

| Attack Vector | Typical Agent Platforms | OctantOS |
|---|---|---|
| Prompt injection → shell | Full shell access via agent runtime | Namespace + seccomp filtered execution in microVM |
| Malicious skills/plugins | npm packages with system access | Compiled signed Rust binaries, no interpreter |
| Credentials | Plaintext `.env` files, shared across agents | AES-256-GCM vault, per-agent scoped tokens |
| Agent-to-agent lateral movement | Shared filesystem, shared user, shared memory | Separate namespaces, separate identity, Matrix-only IPC |
| Supply chain | npm/pip dependency chains with arbitrary code | Single signed binary + Ed25519 signed packages |
| Audit trail | Partial runtime logs, trivially deletable | SHA-256 hash-chain — deletion breaks the chain |
| Search queries | External API calls leak user intent | 190+ self-contained engines, nothing leaves the box |
| OS integrity | User can write to everything | Immutable root, dm-verity verified at boot |
| Resource exhaustion | No per-agent limits | cgroup v2 hard limits — CPU, memory, I/O, PIDs |
| Privilege escalation | Standard Linux user model | Signed manifests + seccomp + namespaces + Firecracker |

The question is not whether your agents will be targeted. It's whether your infrastructure can contain a compromised agent without taking down everything else.

---

## Key Capabilities

### Agent-Native Architecture
Agents are not processes bolted onto a general-purpose OS. They are kernel-visible entities with unique identities, cryptographically signed capability manifests, and lifecycle management from spawn to retirement. Child agents inherit a strict subset of their parent's permissions — never more.

### Intelligent LLM Routing (ProviderForge)
7 providers. Automatic failover across providers and API keys. Policy-driven model selection based on task complexity, cost, latency, and historical performance. Your agent picks the cheapest model for simple tasks and the most capable for complex reasoning. Zero configuration required.

### Semantic Memory (King of Context)
Five-tier memory system running entirely locally — from sub-millisecond DashMap retrieval through BERT embeddings to Neo4j relationship graphs. No cloud memory dependencies. Your agents remember context without sending it to someone else's servers.

### Self-Contained Search (searchrs)
190+ search engines compiled into the binary. Every search happens locally. No API keys leaking queries. No third-party services observing what your agents are looking for.

### Compiled Skills (BinaryCircuits)
Agent skills compile to native Rust binaries. Not interpreted scripts. Not npm packages. Not YAML workflows executed by a runtime. You cannot prompt-inject a compiled binary.

### Multi-Model Governance (Quorum)
Critical decisions go through a parliamentary vote of multiple AI models. No single model makes unilateral high-stakes decisions. Escalation workflows enforce human approval gates. The system enforces the boundary, not a system prompt.

### Federated Communication (MatrixMesh)
Agents communicate over the Matrix protocol — the same decentralised, end-to-end encrypted protocol used by governments and defence organisations. Cross-machine agent orchestration uses federation, not proprietary APIs.

### Tamper-Evident Auditing (Action Ledger)
Every agent action is recorded in a SHA-256 hash-chain. Each entry includes the hash of the previous entry. Delete or modify any record and the chain breaks. You can cryptographically prove what happened — and that nothing was removed.

---

## Repository Scope

This repository documents the OctantOS architecture, security model, roadmap, and release progression.

**Implementation source code is not included in this repository.** OctantOS is under active closed development with plans to open-source select components as the architecture stabilises.

### Architecture Documentation

- [`docs/architecture-overview.md`](docs/architecture-overview.md) — Layer model, component map, filesystem layout
- [`docs/security-model.md`](docs/security-model.md) — 10-layer defence stack, threat model, design principles
- [`docs/capability-manifests.md`](docs/capability-manifests.md) — Manifest schema, validation rules, intersection algorithm
- [`docs/agent-lifecycle.md`](docs/agent-lifecycle.md) — Spawning, isolation, communication, retirement
- [`docs/single-binary.md`](docs/single-binary.md) — Multicall dispatch, workspace structure, build pipeline

### Project Updates

- [`ROADMAP.md`](ROADMAP.md) — Development phases and progress
- [`CHANGELOG.md`](CHANGELOG.md) — Version history and release notes
- [`FAQ.md`](FAQ.md) — Common questions answered directly
- [`SECURITY.md`](SECURITY.md) — Vulnerability disclosure policy and security architecture
- [`CONTRIBUTING.md`](CONTRIBUTING.md) — How to get involved

---

## Status

OctantOS is in active development. The core runtime, agent lifecycle, credential vault, LLM routing, search infrastructure, semantic memory, governance engine, local inference, and operator console are operational. Firecracker microVM integration is in progress.

For detailed progress, see the [Roadmap](ROADMAP.md).

---

## Contact

- **Security:** `security@matrixforgelabs.com` (see [`SECURITY.md`](SECURITY.md))
- **General:** [Discussions](https://github.com/MatrixForgeLabs/OctantOS/discussions) · [Issues](https://github.com/MatrixForgeLabs/OctantOS/issues)
- **Website:** [octant-os.com](https://octant-os.com)

---

<p align="center"><sub>Built with Rust. Secured by architecture. Operated by agents.</sub></p>
