# ADR-001 — Scope: This Is a Platform, Not Three Boards

- **Status:** accepted
- **Date:** 2026-09-17

## Context

Three NanoPC-T4 boards (RK3399, 4 GB RAM each) and a dusty MikroTik router.
The temptation is to treat it as a toy: install things manually, `kubectl
apply` by hand, accept that "it's just a homelab".

## Decision

Build it as a platform from day one:

- **GitOps everywhere** — the cluster is a render of a git repo
  (App-of-Apps, ArgoCD, automated sync + prune + selfHeal). No manual
  `kubectl apply` as the source of truth.
- **Reproducible bootstrap** — a single setup script provisions a node
  from an environment file; no per-node snowflakes.
- **Workloads as products** — medical imaging, earth observation, research
  pipelines: each a Helm chart + ArgoCD Application with explicit resource
  budgets, not "let me install this and see".
- **CI/CD on the platform itself** — self-hosted runners (amd64 for speed,
  ARM64 for fidelity) and a local registry feeding the same cluster.
- **Documented decisions** — ADRs and incident reports are part of the
  repo, because the 4 GB constraint makes every choice a real trade-off.

## Consequences

- The constraint (4 GB/node, ARMv8.0) is the design force: it sets the
  resource budgets, the workload selection, and the "what does not fit"
  boundary (see ADR-002, INCIDENT-001).
- Building this way takes longer than a toy install — and that is the
  point: the skills exercised (scoping, GitOps, capacity planning) are the
  same ones a production platform needs.

## Alternatives considered

- "Just install k3s and go" — rejected: nothing reproducible, nothing
  to show, nothing learned.
- "Bigger hardware" — rejected: the constraint is the interesting part.