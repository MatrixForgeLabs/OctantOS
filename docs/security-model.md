# Security Model

## Core Thesis

Current autonomous agent platforms are architecturally unsafe because they bolt security onto systems designed for human operation. They give AI models access to shells, filesystems, credentials, and network interfaces, then attempt to constrain behaviour through prompt-level instructions, application-layer sandboxes, or runtime configuration flags.

This is not a security model. This is hope.

OctantOS makes security the design constraint, not a feature. The fundamental shift: in traditional systems, processes are trusted by default and restricted by exception. In OctantOS, agents are **untrusted by default** and permitted only by **explicit cryptographic grant**.

---

## 10-Layer Defence Stack

Every layer operates independently. The system is designed under the assumption that any individual layer may fail — and the remaining layers maintain containment regardless.

### Layer 1: Hardware Identity

Machine-level identity using cryptographic fingerprinting. Prevents unauthorised cloning, license migration, and deployment on untrusted hardware. Optional TPM integration provides hardware-rooted trust anchors.

### Layer 2: Verified Boot

The root filesystem is hash-verified at mount time. Any modification to the OS binary or root partition — whether by an attacker, a misconfigured process, or a compromised agent — causes boot verification to fail. Rootkit persistence is structurally prevented.

### Layer 3: Immutable Root Filesystem

The root partition is inherently read-only. No process, including root, can modify the operating system at runtime. The only writable storage is the data partition. OS updates replace the entire root image atomically using an A/B partition scheme — there is no in-place patching.

### Layer 4: Memory Safety

The entire userspace is compiled from Rust with no `unsafe` blocks in application code. This eliminates buffer overflows, use-after-free vulnerabilities, null pointer dereferences, and data races as attack categories. The binary itself is hardened against memory corruption before any runtime security mechanism engages.

### Layer 5: Agent Identity

Every agent is a kernel-visible entity with a unique identifier. Every process in the system is attributable to a specific agent or to the system itself. There is no identity confusion between human actions and agent actions. Lineage tracking maintains a full chain from any agent back to the human who authorised its creation.

### Layer 6: Namespace Isolation

Each agent executes in its own set of Linux namespaces:

- **Mount namespace** — agents see only files explicitly granted in their manifest
- **PID namespace** — agents see only their own processes
- **Network namespace** — agents can only reach explicitly allowed destinations
- **IPC namespace** — no shared memory between agents
- **UTS namespace** — agents believe they are running on separate machines

An agent cannot observe, access, or interfere with any other agent's resources. As far as each agent is concerned, it is the only workload on the system.

### Layer 7: Syscall Filtering

Each agent receives a custom seccomp-BPF filter derived from its signed capability manifest. The filter operates at the kernel boundary:

- **Default-deny** — only explicitly allowed system calls pass through
- **Hardcoded deny list** — certain operations (module loading, reboot, ptrace) are permanently blocked regardless of manifest content
- **Irremovable** — once installed, the filter cannot be modified or removed by the filtered process

No amount of prompt injection, context manipulation, or model persuasion can cause an agent to execute a system call that its filter does not permit. The enforcement is in the kernel, not in application code.

### Layer 8: Encrypted Secrets

Credentials are stored in an encrypted vault using authenticated encryption with a key derivation function resistant to brute-force attacks. Access is scoped — agents receive only the credentials matching their manifest grants. There are no plaintext configuration files, no shared credential stores, and no environment variables containing secrets.

Token-scoped access means a compromised agent gains access to its own credentials only. It cannot read, enumerate, or infer the existence of credentials belonging to other agents.

### Layer 9: Tamper-Evident Audit

Every agent action is recorded in an append-only ledger using a hash-chain structure. Each entry includes the cryptographic hash of the previous entry. Deleting or modifying any record breaks the chain in a way that is computationally infeasible to repair without detection.

The ledger records semantic events (what the agent did and why) rather than raw system call traces. This makes the audit trail human-queryable and forensically useful, not just a compliance checkbox.

### Layer 10: Governance

Critical decisions pass through a multi-model parliamentary decision engine. No single model makes unilateral high-stakes decisions. The governance system supports structured proposals, debate, amendments, and weighted voting across independent models.

Escalation workflows enforce human approval gates for operations that exceed agent authority. The system enforces the approval boundary — it is not a suggestion that depends on the model's compliance.

---

## Threat Model

### Prompt Injection

**Attack:** Malicious input causes the LLM to generate harmful commands.

**Why it's contained:** Injected instructions execute inside a namespace with only the permissions defined in the agent's manifest. The agent cannot access files outside its mount namespace, reach network destinations not on its allowlist, make system calls not in its seccomp filter, or read credentials scoped to other agents. Critical actions trigger governance escalation. Every action — including injected ones — is recorded in the tamper-evident ledger.

**Architectural insight:** Prompt injection is dangerous only when it crosses into unrestricted execution. OctantOS ensures there is no unrestricted execution.

### Malicious Skill Execution

**Attack:** A skill binary contains malicious code (backdoor, data exfiltration, resource abuse).

**Why it's contained:** Skills execute under the agent's identity with the agent's manifest constraints. They inherit namespace restrictions, seccomp filters, and resource limits. They cannot modify the OS (immutable root), cannot access other agents' resources (namespace isolation), and cannot operate unobserved (ledger audit).

**Architectural insight:** Skills are compiled native binaries, not interpreted scripts. There is no interpreter to exploit, no dependency chain to poison, no `eval()` to inject. The attack surface for skill-based compromise is categorically smaller than script-based alternatives.

### Agent-to-Agent Lateral Movement

**Attack:** Compromised Agent A attempts to access, influence, or compromise Agent B.

**Why it's contained:** Agents exist in separate namespaces with no shared filesystem, no shared memory, no shared network namespace, and no shared process visibility. The only communication channel between agents is a mediated, auditable messaging layer with room-based access control. Agent A cannot discover Agent B's existence through the operating system — only through explicitly permitted communication channels.

### Privilege Escalation

**Attack:** An agent attempts to gain more permissions than its manifest allows.

**Why it's contained:** The manifest is signed and immutable — the agent cannot modify its own constraints. The seccomp filter is kernel-enforced and irremovable after installation. Namespace boundaries are kernel-enforced. Any privilege escalation attempt that involves prohibited system calls is blocked at the kernel boundary before it executes, and the attempt is logged.

### Data Exfiltration

**Attack:** A compromised agent attempts to send sensitive data to an external server.

**Why it's contained:** Network namespace restrictions limit the agent to explicitly allowed destinations. Mount namespace restrictions limit the agent to explicitly granted files. Credential scoping limits the agent to its own secrets. All network connections are logged in the action ledger.

### Resource Exhaustion

**Attack:** An agent consumes all system resources to deny service to other agents or the operator.

**Why it's contained:** Per-agent resource limits are enforced by the kernel via cgroup controls. CPU, memory, I/O throughput, process count, and network bandwidth are all hard-limited. The kernel enforces these limits regardless of what the agent does.

### Supply Chain Compromise

**Attack:** Malicious code enters the system through a dependency, package, or update.

**Why it's contained:** The OS is a single signed binary — one artifact to verify, not thousands of packages. Third-party packages use cryptographic signature verification; unsigned packages do not install. There is no general-purpose package manager with arbitrary code execution capabilities. Build reproducibility ensures that the same source produces the same binary with the same hash.

---

## Comparison With Typical Agent Frameworks

| Attack Vector | Typical Agent Frameworks | OctantOS |
|---|---|---|
| **Trust model** | Agent trusted by default, restrictions added reactively | Agent untrusted by default, permitted only by cryptographic grant |
| **Execution isolation** | Shared process space, app-layer sandboxing | Per-agent kernel namespaces + seccomp-BPF + cgroups |
| **Permission model** | Runtime config flags, environment variables | Signed capability manifests with intersection-based delegation |
| **Credential storage** | Plaintext files, shared environment variables | Encrypted vault with per-agent scoped access |
| **Audit guarantees** | Application logs, deletable, no integrity proof | Hash-chain ledger with tamper-evident semantics |
| **Skill/plugin safety** | Interpreted scripts, npm/pip dependency chains | Compiled signed binaries, no interpreter, no dependency chain |
| **Search privacy** | External API calls expose query intent | Self-contained engines, zero external leakage |
| **OS integrity** | Mutable filesystem, writable by application code | Immutable root, verified boot, atomic updates |
| **Resource control** | No per-agent limits, shared resource pool | cgroup v2 hard limits per agent |
| **Governance** | None, or single-model decision making | Multi-model parliamentary consensus with human gates |

---

## Design Principles

1. **Never trust agent output.** Always validate through policy gates.
2. **Never store secrets in plaintext.** Always use the vault.
3. **Never grant more permissions than needed.** Manifests should be minimal.
4. **Never allow unsigned artifacts.** Manifests, packages, skills, policies — all signed.
5. **Never break the audit chain.** Every action is recorded.
6. **Assume compromise.** Design every layer as if every other layer has failed.
7. **Fail closed.** On any error, deny. Never fail open.
8. **Log first, act second.** The ledger entry is written before the action executes.

---

## Vulnerability Reporting

See [`../SECURITY.md`](../SECURITY.md) for coordinated vulnerability disclosure, severity classification, response commitments, and safe harbour protections.
