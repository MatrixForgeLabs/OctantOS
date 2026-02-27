# Changelog

All notable public-facing changes to OctantOS are documented in this file.

This changelog covers the public repository only. Internal development milestones are tracked separately.

---

## 2026-02-27 — Public Repository Launch

### Added

- **Public architecture documentation**
  - [`docs/architecture-overview.md`](docs/architecture-overview.md) — layer model, component map, filesystem layout, design priorities
  - [`docs/security-model.md`](docs/security-model.md) — 10-layer defence stack, threat model, attack containment analysis, design principles
  - [`docs/capability-manifests.md`](docs/capability-manifests.md) — manifest structure, validation rules, intersection-based delegation, enforcement pipeline
  - [`docs/agent-lifecycle.md`](docs/agent-lifecycle.md) — lifecycle states, spawning hierarchy, communication model, end-of-life guarantees
  - [`docs/single-binary.md`](docs/single-binary.md) — dispatch model, boot sequence, build characteristics, deployment targets

- **Project documentation**
  - [`README.md`](README.md) — project overview, architecture stack, differentiation, and comparison analysis
  - [`ROADMAP.md`](ROADMAP.md) — capability-based progress tracking across all system areas
  - [`FAQ.md`](FAQ.md) — technical questions answered with substance
  - [`SECURITY.md`](SECURITY.md) — vulnerability disclosure policy, severity classification, response commitments, safe harbour, and security architecture overview
  - [`CONTRIBUTING.md`](CONTRIBUTING.md) — contribution pathways for the current development stage

- **Repository infrastructure**
  - GitHub issue templates and discussion categories
  - Mermaid-ready diagrams in architecture documents

### Context

This repository launches as the public-facing documentation and community hub for OctantOS. Implementation source code remains in private development. The system is operational with core runtime, agent lifecycle, credential vault, LLM routing, search infrastructure, semantic memory, governance engine, local inference, federated communication, and action ledger all functional.
