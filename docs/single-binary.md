# Single-Binary Architecture

OctantOS ships as one static multicall binary that contains the runtime, service entrypoints, and operational commands.

## Dispatch Model

The executable supports both:

- Subcommand invocation (`octant <command>`)
- Symlink-based multicall dispatch (`/bin/<tool> -> /octant`)

This model is inspired by proven multicall patterns (for example BusyBox), applied to a modern agent operating system.

## Why Single Binary

1. Minimal deployment surface and dependency complexity
2. Consistent behavior across environments
3. Unified build and release process
4. Faster cold-start operational profile

## Boot Sequence (Public Summary)

On startup, core services initialize in deterministic dependency order:

- ledger
- vault
- matrix
- router
- search
- context
- quorum
- inference

Observed startup target: ~83ms for full service readiness in optimized environments.
