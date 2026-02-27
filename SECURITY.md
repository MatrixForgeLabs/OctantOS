# Security Policy

OctantOS exists because agent security is broken. We take security reports as seriously as we take the problem we're solving.

---

## Reporting a Vulnerability

**Do not disclose vulnerabilities publicly before coordinated remediation.**

Report all security issues privately to: **`security@matrixforgelabs.com`**

### What to Include

- **Affected component** — specify which layer, service, or subsystem is impacted (e.g., namespace isolation, credential vault, manifest validation, ledger integrity, seccomp policy, package signing)
- **Reproduction steps** — clear, minimal instructions to trigger the issue
- **Impact assessment** — what an attacker gains, which defence layers are bypassed, and the blast radius
- **Proof of concept** — if available, include code, logs, or traces demonstrating the vulnerability
- **Environment details** — deployment target (Docker, VM, bare metal), architecture (x86_64, aarch64), and OctantOS version or commit hash

### What We Consider In Scope

This repository is documentation-only, but our security scope covers the full OctantOS system:

| Area | Examples |
|------|----------|
| **Agent isolation** | Namespace escapes, cgroup bypasses, cross-agent data access |
| **Credential security** | Vault key extraction, token scope violations, plaintext credential exposure |
| **Manifest enforcement** | Privilege escalation beyond manifest grants, signature verification bypass |
| **Ledger integrity** | Hash-chain breaks, audit log tampering, event suppression |
| **Syscall filtering** | seccomp-BPF filter escapes, deny_always circumvention |
| **Package signing** | Ed25519 signature bypass, unsigned package installation |
| **Boot integrity** | dm-verity circumvention, immutable root modification |
| **Inter-agent communication** | Matrix protocol abuse, unauthorised room access, message injection |
| **Governance bypass** | Quorum decision circumvention, escalation policy evasion |
| **Supply chain** | Build reproducibility failures, dependency integrity issues |
| **Architecture design** | Logical flaws in the security model documented in this repository |

### What Is Out of Scope

- Vulnerabilities in upstream dependencies (Linux kernel, musl libc) unless OctantOS's usage creates a novel attack vector
- Social engineering attacks against MatrixForge Labs personnel
- Denial of service through expected resource consumption within cgroup limits
- Issues requiring physical access to hardware where full disk encryption is enabled
- Theoretical attacks with no demonstrated path to exploitation

---

## Response Commitments

| Stage | Target |
|-------|--------|
| Initial acknowledgement | Within **48 hours** |
| Severity classification | Within **3 business days** |
| Status updates during active triage | At least **weekly** |
| Fix or mitigation deployed | Based on severity (see below) |

### Severity-Based Response

| Severity | Definition | Fix Target |
|----------|-----------|------------|
| **Critical** | Agent isolation bypass, credential vault compromise, or ledger tampering — an attacker can escape containment or destroy audit integrity | **72 hours** |
| **High** | Privilege escalation within a single agent boundary, manifest validation bypass, or unsigned package installation | **7 days** |
| **Medium** | Information disclosure within authorised scope, partial seccomp filter weakness, or governance policy inconsistency | **30 days** |
| **Low** | Documentation inconsistency, non-exploitable edge case, or defence-in-depth improvement opportunity | **Next release** |

---

## Security Architecture Overview

OctantOS enforces security at the infrastructure level through a 10-layer defence stack. Each layer operates independently — the system is designed under the assumption that any individual layer may be compromised.

```
┌─────────────────────────────────────────────────────────────────┐
│  Layer 10  │  Governance          │  Multi-model parliamentary  │
│            │  (Quorum)            │  voting, human approval     │
├────────────┼──────────────────────┼─────────────────────────────┤
│  Layer 9   │  Tamper-Evident      │  SHA-256 hash-chain, per-   │
│            │  Audit Ledger        │  agent partitions           │
├────────────┼──────────────────────┼─────────────────────────────┤
│  Layer 8   │  Encrypted Secrets   │  AES-256-GCM vault,         │
│            │  (Credentials Vault) │  Argon2id KDF, scoped       │
├────────────┼──────────────────────┼─────────────────────────────┤
│  Layer 7   │  Syscall Filtering   │  Per-agent seccomp-BPF,     │
│            │  (seccomp-BPF)       │  default-deny, irremovable  │
├────────────┼──────────────────────┼─────────────────────────────┤
│  Layer 6   │  Namespace Isolation │  Mount, PID, Net, IPC, UTS  │
│            │                      │  per agent                  │
├────────────┼──────────────────────┼─────────────────────────────┤
│  Layer 5   │  Agent Identity      │  Unique AIDs, lineage       │
│            │  (AIDs)              │  tracking, attribution      │
├────────────┼──────────────────────┼─────────────────────────────┤
│  Layer 4   │  Memory Safety       │  Pure Rust userspace,       │
│            │  (Rust)              │  no unsafe in app code      │
├────────────┼──────────────────────┼─────────────────────────────┤
│  Layer 3   │  Immutable Root      │  squashfs, read-only,       │
│            │                      │  A/B atomic updates         │
├────────────┼──────────────────────┼─────────────────────────────┤
│  Layer 2   │  Verified Boot       │  dm-verity hash             │
│            │                      │  verification at mount      │
├────────────┼──────────────────────┼─────────────────────────────┤
│  Layer 1   │  Hardware Identity   │  Ed25519 machine            │
│            │  (LicenseForge)      │  fingerprint, TPM optional  │
└─────────────────────────────────────────────────────────────────┘
```

### Core Design Principle

> In traditional systems, processes are trusted by default and restricted by exception.
> In OctantOS, agents are **untrusted by default** and permitted only by **explicit cryptographic grant**.

Text-based instructions are not guardrails. Prompt-level protections are not security boundaries. OctantOS enforces every constraint at the kernel level, where no amount of prompt injection, context manipulation, or model persuasion can override the policy.

### Defence-in-Depth Approach

Every threat is addressed by multiple independent layers. If an attacker compromises one layer, subsequent layers maintain containment:

- **Prompt injection** → Namespace isolation + syscall filtering + scoped credentials + governance escalation
- **Malicious skill execution** → Immutable root + manifest constraints + namespace inheritance + ledger audit
- **Lateral movement between agents** → Separate namespaces + separate identity + Matrix-only communication + access control
- **Privilege escalation** → Signed immutable manifests + kernel-enforced seccomp + namespace boundaries + audit logging
- **Supply chain attack** → Single signed binary + Ed25519 package signing + no arbitrary package managers + build reproducibility
- **Data exfiltration** → Network namespace allowlists + scoped credentials + mount namespace restrictions + ledger monitoring

---

## Development Security Principles

These principles govern all OctantOS development and we hold ourselves accountable to them:

1. **Never trust agent output.** Always validate through policy gates.
2. **Never store secrets in plaintext.** Always use the vault.
3. **Never grant more permissions than needed.** Manifests should be minimal.
4. **Never allow unsigned artifacts.** Manifests, packages, skills, policies — all signed.
5. **Never break the audit chain.** Every action is recorded.
6. **Assume compromise.** Design every layer as if every other layer has failed.
7. **Fail closed.** On any error, deny. Never fail open.
8. **Log first, act second.** The ledger entry is written before the action executes.

---

## Safe Harbour

MatrixForge Labs considers good-faith security research to be authorised conduct. We will not pursue legal action against researchers who:

- Make a genuine effort to avoid privacy violations, data destruction, and service disruption
- Only interact with accounts they own or have explicit permission to test
- Report vulnerabilities through the process described above
- Allow reasonable time for remediation before any public disclosure

We will work with researchers to understand and validate reports, and we will credit researchers (with their permission) in any public advisory resulting from their findings.

---

## PGP Key

A PGP key for encrypted vulnerability reports will be published here when available. In the interim, use `security@matrixforgelabs.com` with sensitive details minimised in the initial report — we will establish a secure channel for full disclosure.

---

## Contact

- **Security reports:** `security@matrixforgelabs.com`
- **General enquiries:** Open a [Discussion](https://github.com/MatrixForgeLabs/OctantOS/discussions) or [Issue](https://github.com/MatrixForgeLabs/OctantOS/issues)
- **Website:** [octant-os.com](https://octant-os.com)
