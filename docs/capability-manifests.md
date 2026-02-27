# Capability Manifests

A capability manifest is the cryptographically signed contract that defines what an agent is allowed to do. It is not documentation. It is not advisory. It is the enforcement input that directly generates the agent's runtime security policy — namespace boundaries, seccomp filters, resource limits, credential access grants, and network allowlists.

An agent without a valid, signed manifest does not run.

---

## What a Manifest Controls

Each manifest declares the complete set of permissions for an agent across every enforcement dimension:

| Domain | What the manifest specifies |
|---|---|
| **Identity** | Agent identifier, role classification, operator attribution |
| **Filesystem** | Explicit read and write paths — everything outside these paths is invisible |
| **Network** | Allowed destination hosts, ports, and protocols — everything else is unreachable |
| **System calls** | Allowed syscall set for seccomp-BPF — everything else is blocked at the kernel |
| **Processes** | Allowed executable paths — the agent cannot run arbitrary binaries |
| **Resources** | CPU shares, memory ceiling, I/O bandwidth, maximum PID count, network throughput |
| **Credentials** | Named credential grants — the agent receives only these secrets from the vault |
| **Skills** | Approved skill identifiers — the agent can only execute listed skills |
| **Spawning** | Whether the agent can create child agents, maximum spawn depth, maximum descendants |
| **Lifetime** | Maximum time-to-live, automatic termination policy |

Every field is explicit. There are no implicit permissions, no default grants, and no ambient authority. If the manifest does not list it, the agent does not have it.

---

## Validation Rules

Before any agent spawns, the lifecycle manager validates its manifest against a strict set of rules:

1. **Schema compliance** — the manifest must parse successfully and satisfy all structural constraints.
2. **Signature verification** — the Ed25519 signature must verify against a signer in the trusted operator key set.
3. **Role subset check** — if a role is specified, the manifest's permissions must be a strict subset of that role's defaults.
4. **Parent intersection** — if spawned by another agent, all permissions must be a subset of the parent's manifest.
5. **Spawn budget** — the parent's current descendant count must be below its maximum, and the requested spawn depth must be within limits.
6. **Resource availability** — requested resources must fit within available system capacity.
7. **Deny list enforcement** — no permanently denied system calls may appear in the allow set.
8. **Identity uniqueness** — the agent identifier must be globally unique.
9. **Unlimited lifetime approval** — agents requesting no time limit require explicit operator approval.

A manifest that fails any validation rule is rejected. The agent is not spawned. The rejection is recorded in the audit ledger.

---

## Delegation and Inheritance

When Agent A spawns Agent B, the child's effective permissions are computed by strict intersection of the parent and child manifests:

```
effective_permissions = intersect(parent_manifest, child_manifest)
```

Concretely:

- **Filesystem:** child can only access paths that are in *both* parent and child read/write lists
- **Network:** child can only reach destinations in *both* parent and child allowlists
- **System calls:** child can only make calls in *both* parent and child seccomp sets
- **Executables:** child can only run binaries in *both* parent and child exec lists
- **Resources:** child gets the *minimum* of parent and child resource limits
- **Credentials:** child can only access credentials in *both* parent and child grant sets

The result is always equal to or smaller than both inputs. A parent cannot grant its child permissions that the parent does not itself possess. Delegation never escalates privilege.

This holds recursively. If Agent B spawns Agent C, the same intersection applies between B's effective manifest and C's requested manifest. Permissions can only narrow as the spawn chain deepens — they can never widen.

---

## From Manifest to Enforcement

The manifest is not a set of guidelines that the agent is expected to follow. It is a machine-readable specification that deterministically generates kernel-level enforcement:

1. The manifest's filesystem grants become mount namespace bind mounts — paths not listed are not mounted and do not exist in the agent's view of the filesystem.
2. The manifest's network grants become network namespace firewall rules — destinations not listed are unreachable.
3. The manifest's syscall grants become a seccomp-BPF filter — calls not listed are blocked by the kernel and cannot be made regardless of what the agent attempts.
4. The manifest's resource limits become cgroup v2 constraints — the kernel enforces hard ceilings on CPU, memory, I/O, and process count.
5. The manifest's credential grants become vault access tokens — the agent can retrieve only the named secrets and cannot enumerate or access others.

There is no gap between what the manifest says and what the system enforces. The manifest *is* the enforcement policy, expressed in a human-readable, machine-parseable, cryptographically signed format.

---

## Why This Matters

In most agent frameworks, permissions are advisory. An agent might be *told* not to access certain files or make certain API calls, but the enforcement depends on the agent's compliance with those instructions — which, in the case of prompt injection or model misbehaviour, cannot be guaranteed.

In OctantOS, an agent that has been prompt-injected, hallucinating, or actively adversarial still cannot exceed its manifest. The enforcement is in the kernel, below the application layer, below the model, below the prompt. It is not possible to talk your way past a seccomp filter.

---

## Related Documentation

- [Security Model](security-model.md) — how manifests fit into the 10-layer defence stack
- [Agent Lifecycle](agent-lifecycle.md) — how manifests are validated during agent provisioning
- [Architecture Overview](architecture-overview.md) — system context and layer model
