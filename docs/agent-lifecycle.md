# Agent Lifecycle

OctantOS treats agents as managed operating system workloads with explicit lifecycle control. An agent is not a process that happens to run an LLM — it is a first-class kernel-visible entity with a unique identity, a signed permission contract, an isolated execution environment, a dedicated audit trail, and a defined end-of-life procedure.

---

## Lifecycle States

Every agent passes through a defined sequence of states. Transitions are deterministic and audited.

```
┌────────────┐    ┌────────────┐    ┌────────────┐    ┌────────────┐
│ Provisioned│───▸│ Validated  │───▸│  Isolated  │───▸│  Running   │
└────────────┘    └────────────┘    └────────────┘    └────────────┘
                        │                                   │
                        │ manifest                          │
                        │ rejected                     ┌────┴─────┐
                        ▼                              ▼          ▼
                  ┌────────────┐               ┌──────────┐ ┌──────────┐
                  │  Rejected  │               │ Suspended│ │Terminated│
                  └────────────┘               └──────────┘ └──────────┘
                                                                │
                                                                ▼
                                                          ┌──────────┐
                                                          │ Archived │
                                                          └──────────┘
```

### Provisioned

The agent has been requested — either by an operator, by another agent, or by a system automation. A manifest has been submitted. Resources have been tentatively reserved. No execution has occurred.

### Validated

The manifest has passed all validation checks: schema compliance, signature verification, role subset constraints, parent permission intersection, spawn budget limits, resource availability, deny list enforcement, and identity uniqueness. The agent is approved to proceed.

If validation fails at any point, the agent transitions to **Rejected**. The rejection reason is recorded in the audit ledger. No execution environment is created.

### Isolated

The agent's execution environment has been constructed. Namespaces (mount, PID, network, IPC, UTS) are created. cgroup limits are applied. The seccomp-BPF filter derived from the manifest is installed. Filesystem bind mounts are configured. Network rules are in place. Credential tokens are provisioned. The environment is fully constrained before any agent code runs.

### Running

The agent is actively executing within its isolated environment. All actions are monitored against manifest permissions and logged to the action ledger. Resource consumption is tracked against cgroup limits.

From this state, the agent may be:
- **Suspended** — execution paused, state preserved, resources retained
- **Throttled** — resource limits tightened in response to policy or system pressure
- **Terminated** — execution ended, cleanup initiated

### Suspended

Execution is paused. The agent's state is preserved. Resources remain allocated but idle. An operator or policy trigger can resume the agent to Running or terminate it. Suspension events are logged.

### Terminated

Execution has ended — either by the agent completing its task, by operator intervention, by TTL expiry, by policy violation, or by parent agent decision.

### Archived

The final state. Active execution context has been fully removed. Audit records are committed and sealed. The agent's lifecycle is complete.

---

## Spawning Agents from Agents

Agents can spawn child agents, creating hierarchical task delegation. This is a core capability for complex autonomous workflows — a research agent might spawn specialist sub-agents for different information sources, then synthesise their results.

Spawning is governed by strict rules:

- The parent must have spawning permission in its manifest.
- The child's manifest is validated against all standard rules.
- The child's effective permissions are the **intersection** of the parent's manifest and the child's requested manifest — never more than the parent has.
- The parent's spawn depth and descendant count are checked against its limits.
- The child's identity includes lineage metadata linking it to its parent and ultimately to the authorising human.

A parent cannot create a child with more privilege than itself. A child cannot create a grandchild with more privilege than the child has. Permissions can only narrow as delegation depth increases.

If a parent is terminated, its policy determines what happens to its children — immediate termination, graceful wind-down, or orphan adoption by the system.

---

## Communication Between Agents

Agents do not share filesystems, memory, process tables, or network namespaces. The operating system provides no covert channels between agent execution environments.

All inter-agent communication passes through a mediated messaging layer with:
- **Room-based access control** — agents can only communicate through explicitly permitted channels
- **End-to-end encryption** — message content is protected in transit
- **Structured event types** — lifecycle events, task assignments, escalation requests, and status updates use defined schemas
- **Full audit trail** — every message is attributable and logged

An agent that has been compromised cannot use operating system primitives to reach other agents. The only communication path is through the mediated layer, which enforces access policy independently of the agent's intentions.

---

## End-of-Life Guarantees

When an agent terminates, the system enforces a deterministic cleanup sequence:

1. **Execution halt** — all agent processes within the namespace are signalled and terminated.
2. **Credential revocation** — all vault access tokens issued to the agent are invalidated. Ephemeral credentials are destroyed. Zeroize-on-drop semantics ensure credentials do not persist in memory.
3. **Resource release** — cgroup allocations are freed. Namespace structures are torn down. Filesystem bind mounts are removed.
4. **Audit finalisation** — final lifecycle events are committed to the action ledger. The agent's audit partition is sealed.
5. **Artifact retention** — work products explicitly marked for retention are preserved. Everything else is removed.

After cleanup, no trace of the agent's execution environment remains in the operating system's active state. The only evidence of the agent's existence is its sealed audit record in the ledger.

---

## Why This Matters

Most agent frameworks treat agents as application-layer constructs — objects in memory managed by a runtime, sharing the same process space, filesystem, and credentials. When something goes wrong, the blast radius is the entire application.

OctantOS treats agents as operating system workloads with the same isolation guarantees you would expect between separate virtual machines — but with the performance characteristics and management capabilities of native processes. A compromised agent is contained. A misbehaving agent is throttled or terminated. A completed agent is cleanly retired with cryptographic proof of everything it did.

---

## Related Documentation

- [Capability Manifests](capability-manifests.md) — the permission contract that governs each agent
- [Security Model](security-model.md) — how lifecycle controls fit into the defence stack
- [Architecture Overview](architecture-overview.md) — system context and layer model
