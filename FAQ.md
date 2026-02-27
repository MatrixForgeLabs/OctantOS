# Frequently Asked Questions

---

### Is this the source code repository?

No. This repository contains architecture documentation, security model details, roadmap, and project updates. The OctantOS implementation source is under active closed development.

Select components will be open-sourced as the architecture stabilises. If you're interested in early access or contributing, see [`CONTRIBUTING.md`](CONTRIBUTING.md).

---

### Is it really one binary?

Yes. After the Linux kernel hands off control, the entire OctantOS userspace is a single statically-linked Rust binary at `/octant`. There is no shell, no coreutils, no interpreter, no package manager, no service manager — nothing else on the system.

The binary uses multicall dispatch (similar to BusyBox): it can be invoked directly via subcommands (`octant agent start`, `octant ledger verify`) or through symlinks that resolve back to the same binary (`ls`, `cat`, `grep`, `ps` — 81 utilities total).

Eight core services — credential vault, LLM router, search, semantic memory, governance engine, local inference, federated communication, and action ledger — all boot from this single binary in approximately 83 milliseconds.

---

### Why not just use Docker plus an existing agent framework?

Docker solves process isolation. It does not solve agent isolation, and the distinction matters.

Containerisation gives you a filesystem boundary and a network namespace. It does not give you signed capability manifests that cryptographically limit what an agent can do. It does not give you per-agent seccomp filters derived from those manifests. It does not give you a tamper-evident audit ledger where deleting a record breaks a hash chain. It does not give you multi-model governance that requires parliamentary consensus before critical actions execute.

Most agent frameworks assume a trusted execution environment and rely on prompt-level instructions for safety. OctantOS assumes every agent is compromised from the moment it spawns and enforces every constraint at the kernel boundary where no amount of prompt injection can override the policy.

OctantOS can run *inside* Docker for development and testing. But the security model is designed for environments where Docker's trust assumptions are insufficient — which, for autonomous agents with real credentials and real network access, is most production environments.

---

### How is this different from a hardened Linux distribution?

A hardened distro gives you a more secure base to run traditional software on. OctantOS is not that.

OctantOS is an integrated agent runtime where the operating system, the agent lifecycle manager, the credential vault, the LLM router, the governance engine, the audit system, and the communication layer are all one system designed together. Hardened distros secure the platform *underneath* your application. OctantOS *is* the application and the platform.

The practical difference: on a hardened distro, you still need to build agent isolation, capability enforcement, credential management, audit logging, and governance yourself — or trust a framework to do it for you at the application layer. On OctantOS, these are kernel-enforced system primitives that exist before any agent code runs.

---

### How does agent isolation actually work?

Every agent runs in its own set of Linux namespaces — separate mount, PID, network, IPC, and UTS namespaces. An agent literally cannot see another agent's files, processes, network connections, or shared memory. As far as each agent is concerned, it is the only thing running on the machine.

On top of namespace isolation, each agent gets a custom seccomp-BPF filter derived from its signed capability manifest. Only explicitly allowed system calls pass through. The filter is applied at the kernel level and cannot be removed or modified after installation.

Resource consumption is hard-limited per agent via cgroup v2 controls — CPU, memory, I/O, PIDs, and network bandwidth. A misbehaving agent cannot starve the system or other agents.

Firecracker microVM integration (in progress) adds hardware-level isolation on top of the kernel-level controls, giving each agent its own virtual machine boundary.

---

### What happens if an agent gets prompt-injected?

The injected instructions execute inside a namespace with only the permissions defined in that agent's manifest. The agent cannot access files outside its mount namespace, cannot reach network destinations not on its allowlist, cannot make system calls not in its seccomp filter, and cannot read credentials scoped to other agents.

Critical actions trigger governance escalation through the Quorum engine, which requires consensus from multiple independent models and, for high-stakes operations, explicit human approval. A single compromised model cannot unilaterally execute dangerous actions.

Every action — including the injected ones — is recorded in the tamper-evident action ledger. You can forensically reconstruct exactly what happened and verify that no records were altered or deleted.

Prompt injection is dangerous when it crosses into unrestricted execution. OctantOS is designed so that unrestricted execution does not exist.

---

### What LLM providers are supported?

OctantOS routes requests across multiple providers with automatic failover, rate-limit awareness, and policy-driven model selection. The router selects models based on task complexity, cost, latency, and historical performance — without requiring manual configuration.

If a provider is rate-limited or unavailable, the router fails over to alternative providers and API keys automatically. Your agents do not stop working because one provider has an outage.

---

### Why Rust?

Memory safety is not optional for security-critical infrastructure. Buffer overflows, use-after-free vulnerabilities, null pointer dereferences, and data races are entire categories of exploits that do not exist in safe Rust. The OctantOS userspace contains no `unsafe` blocks in application code.

Beyond safety, Rust enables the single-binary static compilation model. The entire OS userspace compiles to one artifact with no runtime dependencies — no interpreter, no garbage collector, no dynamic linker. This eliminates entire classes of supply chain and dependency-based attacks.

Performance is a secondary benefit that matters more than you'd expect: 83ms boot times for eight services are a direct consequence of compiled native code with zero startup overhead.

---

### Can I self-host this?

That is the entire point. OctantOS is designed for deployment on your infrastructure — Docker containers for development, virtual machines for production, bare metal for maximum security posture, and edge devices for distributed mesh topologies.

Nothing in OctantOS requires phoning home to a cloud service. Search is self-contained. Inference can run locally. Credentials never leave the vault. The system is architecturally designed to operate without external dependencies beyond the LLM providers you choose to configure.

---

### When can I try it?

OctantOS is in active development. Progress updates are posted in this repository and on [octant-os.com](https://octant-os.com).

Early access and beta programme details will be announced as rollout milestones are reached. If you want to be notified, watch this repository or follow project updates on the website.

---

### I found a security issue. What do I do?

Do not open a public issue. Report it privately to `security@matrixforge.com`. See [`SECURITY.md`](SECURITY.md) for full details on scope, severity classification, response commitments, and safe harbour protections.

---

### How do I get involved?

See [`CONTRIBUTING.md`](CONTRIBUTING.md) for current opportunities. Even without public source code, there are ways to participate — architecture review, documentation feedback, security research, and early testing are all valuable contributions.
