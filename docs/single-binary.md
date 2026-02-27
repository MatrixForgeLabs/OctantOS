# Single-Binary Architecture

OctantOS ships as one statically-linked Rust binary that contains the entire OS userspace: init system, service runtime, agent lifecycle manager, operator tools, and 81 built-in utilities. There is nothing else on the system to install, configure, link, or maintain.

---

## Why Single Binary

This is not minimalism for aesthetics. It is a deliberate architectural decision with security, deployment, and operational consequences:

**Reduced attack surface.** One binary means one artifact to verify, sign, and trust. There is no dependency chain of hundreds of packages, each with its own maintainers, release cycles, and vulnerability histories. Supply chain attacks target the weakest link in a dependency graph — this system has no dependency graph at runtime.

**Deterministic deployment.** The binary is the same everywhere it runs. There are no version mismatches between components, no missing shared libraries, no configuration drift between what was tested and what is deployed. What you build is exactly what runs.

**Fast cold start.** Eight core services initialise in approximately 83 milliseconds from a `FROM scratch` container. There is no interpreter startup, no JIT warmup, no dynamic linker resolution, no garbage collector initialisation. Compiled native code starts executing immediately.

**Simplified updates.** Updating the system means replacing one file. The A/B partition scheme swaps the entire root image atomically — there is no partial upgrade state and no possibility of a half-updated system.

---

## Dispatch Model

The binary supports two invocation patterns:

### Subcommand Dispatch

The primary interface for system management and agent operations:

```
octant init              # start as PID 1
octant agent start       # spawn an agent
octant agent list        # list running agents
octant ledger verify     # verify audit chain integrity
octant vault status      # check credential vault health
octant pkg install       # install a signed package
octant doctor            # run system diagnostics
octant shell             # enter interactive operator shell
octant dashboard         # start web-based operator console
```

### Symlink Dispatch

For compatibility and convenience, symlinks resolve back to the binary and invoke the corresponding utility:

```
/bin/ls    →  /octant    # invoked as "ls", dispatches to built-in ls
/bin/cat   →  /octant    # invoked as "cat", dispatches to built-in cat
/bin/grep  →  /octant    # invoked as "grep", dispatches to built-in grep
/bin/ps    →  /octant    # invoked as "ps", dispatches to built-in ps
```

81 utilities are available through this mechanism, covering file operations, text processing, process management, network diagnostics, and system administration. The dispatch overhead is negligible — a single string comparison on the invoked binary name.

This model is directly inspired by BusyBox, which proved that a single multicall binary can replace an entire userspace utility set. OctantOS applies the same principle to a modern agent operating system with significantly broader scope.

---

## Boot Sequence

On startup, the binary initialises as PID 1 and brings up core services in a deterministic dependency order. Each service declares its dependencies, and the init system resolves the graph to ensure correct startup sequencing:

```
ledger     →  (no dependencies — starts first)
vault      →  ledger
comms      →  ledger, vault
router     →  ledger, vault
search     →  ledger
context    →  ledger
quorum     →  ledger
inference  →  ledger
```

Services that share no dependency relationship start concurrently. The full service set reaches operational readiness in approximately 83 milliseconds in optimised environments.

Each service registers a health check. The init system monitors service health and restarts failed services according to configured restart policies. Graceful shutdown propagates termination signals through the dependency graph in reverse order.

---

## Build Characteristics

| Property | Value |
|---|---|
| **Language** | Rust (entire userspace) |
| **Linking** | Fully static (`musl` libc, vendored TLS) |
| **Primary target** | `x86_64-unknown-linux-musl` |
| **Secondary target** | `aarch64-unknown-linux-musl` |
| **Binary size** | ~23MB |
| **Runtime dependencies** | None (Linux kernel only) |
| **Container base** | `FROM scratch` (empty image) |

The binary has zero runtime dependencies beyond the Linux kernel. No shared libraries, no interpreters, no external tools. This means the Docker image is a single layer containing a single file, and the virtual machine image contains the kernel and the binary with nothing in between.

---

## Deployment Targets

The single-binary model enables deployment across every target environment with the same artifact:

| Target | Description |
|---|---|
| **Docker / OCI** | `FROM scratch` image with one file — the smallest possible container |
| **Virtual machine** | Bootable VM image with minimal Linux kernel + binary |
| **Bare metal** | ISO image for direct installation on physical hardware |
| **Edge / embedded** | Cross-compiled ARM64 image for Raspberry Pi, NVIDIA Jetson, and similar devices |
| **Cloud** | AMI, QCOW2, VHD/VHDX, OVA formats for AWS, KVM, Hyper-V, VMware |

The same binary runs in all environments. The only differences are the kernel configuration and the storage layout.

---

## Related Documentation

- [Architecture Overview](architecture-overview.md) — layer model and component map
- [Security Model](security-model.md) — how the single-binary model reduces attack surface
- [Agent Lifecycle](agent-lifecycle.md) — how agents run within the binary's service framework
