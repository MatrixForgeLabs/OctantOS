# Architecture Overview

OctantOS is a Linux-based operating system where a single static Rust binary provides the entire userspace. After the kernel transfers control to the `/octant` binary, there is no other software on the system. No shell, no coreutils, no package manager, no interpreter, no service manager.

Every capability the system provides — from agent lifecycle management to LLM routing to credential storage to search infrastructure — is compiled into this single binary and dispatched through subcommands or symlinks.

---

## Layer Model

The architecture is organised into five layers with hard boundaries between them. Each layer has a clear responsibility and communicates with adjacent layers through defined interfaces.

```
┌──────────────────────────────────────────────────────────────┐
│  OPERATOR LAYER                                              │
│  Interactive shell · Web dashboard · Remote management       │
│  Policy configuration · Audit review · Approval workflows    │
├──────────────────────────────────────────────────────────────┤
│  AGENT LAYER                                                 │
│  Lifecycle manager · Capability enforcement · Skill dispatch │
│  Inter-agent communication · Action ledger · Lineage control │
├──────────────────────────────────────────────────────────────┤
│  SERVICES LAYER                                              │
│  Credential Vault     · LLM Router       · Metasearch       │
│  Semantic Memory      · Governance Engine · Local Inference  │
│  Federated Comms      · Skill Compiler   · Compute Mesh     │
├──────────────────────────────────────────────────────────────┤
│  RUNTIME LAYER                                               │
│  PID 1 init · Service dependency graph · Multicall dispatch  │
│  Namespace/cgroup construction · Package management          │
├──────────────────────────────────────────────────────────────┤
│  KERNEL LAYER                                                │
│  Linux namespaces · cgroups v2 · seccomp-BPF                 │
│  Firecracker microVMs · eBPF · dm-verity · LSM (planned)    │
└──────────────────────────────────────────────────────────────┘
```

### Operator Layer

The human interface. Operators interact with the system through an interactive shell, a web-based dashboard, and remote management integrations with messaging platforms. Policy controls, approval workflows, and audit review happen at this layer. Operators can observe, intervene, and override agent behaviour without entering the agent's execution environment.

### Agent Layer

Where autonomous workloads live. Each agent is a managed operating system entity with a unique identity, a signed capability manifest, an isolated execution environment, and a dedicated audit trail. The lifecycle manager handles provisioning, validation, isolation, monitoring, and retirement. Agents communicate with each other through mediated, auditable channels — never through shared memory, shared filesystems, or direct process interaction.

### Services Layer

Infrastructure that agents and operators consume. Nine core service domains provide the capabilities that make autonomous operation possible: secret management, intelligent LLM routing, privacy-preserving search, semantic memory with local embeddings, multi-model governance, on-device inference, federated communication, skill compilation, and distributed compute coordination. All services boot from the single binary as part of the init dependency graph.

### Runtime Layer

The system foundation. PID 1 initialisation, service dependency resolution, multicall binary dispatch, namespace and cgroup construction for agent workstations, and package management for third-party software. This layer translates the declarative intent of capability manifests into concrete kernel-level enforcement.

### Kernel Layer

Where enforcement lives. Linux namespaces provide isolation boundaries. cgroups v2 enforce resource limits. seccomp-BPF filters restrict system call access. Firecracker microVMs add hardware-level isolation. dm-verity verifies root filesystem integrity. A custom Linux Security Module (planned) will add kernel-level enforcement of agent identity and policy.

---

## Component Map

The binary is organised as a Cargo workspace with focused crates. Each crate has a single responsibility. The public names are listed here; internal structure and dependencies are not documented externally.

| Crate | Responsibility |
|---|---|
| `octant-bin` | Multicall entrypoint, subcommand and symlink dispatch |
| `octant-init` | PID 1, service dependency graph, signal handling |
| `octant-agent` | Agent runtime, lifecycle management, orchestration |
| `octant-manifest` | Capability manifest parsing, validation, signing |
| `octant-isolation` | Namespace, cgroup, and seccomp-BPF construction |
| `octant-vault` | Encrypted credential storage, scoped token access |
| `octant-router` | Multi-provider LLM routing, failover, policy selection |
| `octant-search` | Self-contained metasearch and indexing infrastructure |
| `octant-context` | Tiered semantic memory and retrieval |
| `octant-ledger` | Append-only hash-chain action logging and verification |
| `octant-quorum` | Multi-model governance, proposals, voting, escalation |
| `octant-infer` | Local model inference across multiple architectures |
| `octant-comms` | Federated inter-agent and inter-machine communication |
| `octant-compile` | Skill compilation pipeline |
| `octant-mesh` | Distributed compute coordination |
| `octant-pkg` | Package management and signed package verification |
| `octant-shell` | Interactive operator shell environment |
| `octant-dashboard` | Web-based operator console |
| `octant-coreutils` | Built-in system utilities (81 commands) |
| `octant-gateway` | Multi-protocol external communication |
| `octant-license` | Hardware-bound identity and licensing |

---

## Filesystem Layout

OctantOS uses a minimal, predictable filesystem structure. The root partition is immutable. All mutable state lives under `/data` or standard writable paths.

```
/octant                         # the binary — that's it
/etc/octant/                    # configuration, policy definitions
/etc/octant/manifests/          # signed capability manifests
/var/lib/octant/                # persistent state (ledger, indices, keys)
/var/lib/octant/vault/          # encrypted credential store
/var/lib/octant/ledger/         # action audit records
/var/log/octant/                # operational logs
/opt/octant/skills/             # installed skill artifacts
/opt/octant/packages/           # third-party signed packages
/data/models/                   # local inference model weights
```

On a `FROM scratch` Docker deployment, the image contains exactly one file: `/octant`. Configuration, state, and model storage are mounted or generated at first run.

---

## Design Priorities

1. **Deterministic startup** — services initialise in a defined dependency order with no race conditions or timing assumptions.
2. **Security below application logic** — enforcement happens at the kernel boundary, not in application code that an agent could influence.
3. **Capability-constrained autonomy** — agents operate with the maximum freedom their manifest permits and zero freedom beyond it.
4. **Auditable operations** — every action is attributable, traceable, and verifiable through cryptographic evidence.
5. **Single-artefact deployment** — one binary, one verification target, one thing to sign, ship, and trust.

---

## Related Documentation

- [Security Model](security-model.md) — 10-layer defence stack, threat model, design principles
- [Capability Manifests](capability-manifests.md) — manifest structure, validation, delegation
- [Agent Lifecycle](agent-lifecycle.md) — states, transitions, isolation, retirement
- [Single-Binary Architecture](single-binary.md) — dispatch model, boot sequence, build approach
