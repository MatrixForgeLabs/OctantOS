# FAQ

## Is this the source code repository?
No. This is the public architecture and updates repository. Implementation source remains private at this stage.

## Is it really one binary?
Yes. OctantOS is built around a single static multicall binary architecture.

## Why not just run Docker plus OpenClaw?
Containerization alone does not define a complete agent OS threat model. OctantOS is designed around kernel-enforced boundaries, signed capability controls, and audit-first lifecycle governance.

## How is this different from a hardened Linux distro?
OctantOS is not only a hardened base image. It is an integrated agent runtime architecture with capability manifests, service orchestration, policy enforcement, and governance primitives designed as one system.

## When can I try it?
Public updates are posted in this repository. Early access and beta information will be announced as rollout milestones are met.
