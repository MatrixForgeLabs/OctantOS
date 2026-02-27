# Security Model

OctantOS assumes agents are untrusted by default and enforces boundaries at the OS/kernel level.

## 10-Layer Defense Stack

1. Hardware-bound identity and machine attestation
2. Verified boot and startup integrity checks
3. Immutable runtime assumptions for critical components
4. Memory-safe implementation baseline (Rust)
5. Capability-manifest validation and signature checks
6. Namespace isolation per agent execution context
7. Syscall filtering via seccomp-BPF policies
8. Resource governance via cgroups and limits
9. Encrypted credential vault and scoped secret access
10. Tamper-evident action ledger and policy audit trail

## Threat Model Focus

- Prompt injection against autonomous agents
- Lateral movement between agent workloads
- Secret exfiltration and credential abuse
- Unbounded tool execution
- Silent policy bypass and non-auditable actions

## Competitor Security Comparison (Public Summary)

| Category | Typical Agent Framework | OctantOS |
|---|---|---|
| Trust model | Agent trusted, checks added later | Agent untrusted, default-deny from start |
| Isolation | App-level sandboxing and conventions | Kernel-enforced namespaces + seccomp + cgroups |
| Permissions | Runtime flags/config gates | Signed capability manifests with explicit intersections |
| Credential handling | Environment variables/common stores | Encrypted vault with scoped retrieval paths |
| Auditability | Logs without strong guarantees | Ledger model with tamper-evident semantics |

## Security Disclosure

See [`../SECURITY.md`](../SECURITY.md) for coordinated vulnerability reporting.
