# Capability Manifests

Capability manifests are the contract that defines what an agent is allowed to do.

## Manifest Structure (Conceptual)

- Agent identity metadata
- Signature metadata
- Tool and command capabilities
- Network permissions
- Filesystem access scopes
- Resource limits (cpu, memory, runtime)
- Optional parent-child inheritance references

## Validation Rules

1. Manifest must parse and satisfy schema constraints.
2. Signature must verify against trusted identities.
3. Requested capabilities must be explicit, never implicit.
4. Parent-child manifests must intersect to least privilege.
5. Runtime policy generation must be deterministic from manifest input.

## Intersection Algorithm (High-Level)

When an agent spawns a child workload:

- Parent capabilities form the maximum possible envelope.
- Child manifest requests a subset.
- Effective policy = strict intersection(parent, child).
- Any capability outside parent scope is rejected.

Result: delegation without privilege escalation.

## Why This Matters

The manifest is not documentation. It is enforcement input that drives runtime policy boundaries.
