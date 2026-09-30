# ADR-003: Dual CI/CD runners

- **Status:** accepted
- **Date:** September 2026

## Context

The product is an ARM64 Kubernetes platform with a hard memory budget.
The pipeline that builds *for* the cluster should run *on* the cluster —
otherwise the images are built on a foreign architecture and the "real
test" never happens. At the same time, iterating an amd64 build from a
slow container-in-container daemon is wasteful when a fast native box
sits next door.

## Decision

Two self-hosted Gitea Actions runners, one for each world:

| Runner | Machine | Architecture | Execution | Label |
|--------|---------|--------------|-----------|-------|
| native | the desktop (amd64, 48 GB) | amd64 | host docker.sock — native, seconds | `tardis,self-hosted` |
| cluster | the nanocluster (ARM64, 3×4 GB) | arm64 | dind-rootless inside the cluster — the real test | `nanocluster,self-hosted,arm64` |

- **Jobs are named by architecture** (`amd64` / `arm64`), `runs-on` picks
  the machine label (`tardis` / `nanocluster`) — the log and the runner
  both tell you which world you are in.
- Both runners push to the **local registry** — one build per arch, a
  single shared target the cluster can pull from.
- The registry endpoint comes from a workflow variable
  (`vars.REGISTRY`), never from a hardcoded address in the workflow.

## Consequences

- Every merge produces a native amd64 and a native arm64 image of the
  same source — the registry holds both, so the cluster pulls the one
  that matches its nodes.
- The dind runner carries a real cost (a daemon inside the pod); the
  config that keeps it honest (insecure registry, labels) lives in the
  chart, not in `kubectl edit` history.
- Two runners mean two places to register; the plan (`tardis` and
  `nanocluster` labels) is the contract — jobs fail loudly with
  "no matching runner" instead of silently running on the wrong arch.