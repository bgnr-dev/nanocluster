# NanoCluster

A portable Kubernetes platform on three NanoPC-T4 boards — RK3399, ARMv8,
**4 GB of RAM each**. GitOps, self-hosted CI/CD and real production-style
workloads, all inside that constraint.

> This repo grows with a public building-log series: each episode ships
> the artifacts it talks about. What you see now is the foundation.

## The hardware (episode 1)

- 3× [NanoPC-T4](https://wiki.friendlyelec.com/wiki/index.php/NanoPC-T4) (Rockchip RK3399, 4 GB LPDDR3, ARMv8.0)
- 1 dusty MikroTik router acting as LAN DNS
- 1 NAS (amd64) next to it — registry mirror + the fast CI runner

## The scope (episode 2, [ADR-001](docs/decisions/ADR-001-scope.md))

This is not "three boards with k3s". This is a platform:

- **GitOps** — the cluster is a render of this repo (App-of-Apps, ArgoCD,
  automate + prune + selfHeal)
- **Reproducible** — one bootstrap script, one environment file, zero
  snowflakes
- **Workloads as products** — each with a Helm chart, an ArgoCD
  Application and an explicit resource budget
- **CI/CD on the platform** — self-hosted runners + a local registry
- **Decisions in the repo** — ADRs and incidents, because on 12 GB every
  choice is a real trade-off

## The roadmap (repo artifacts appear with each episode)

| # | Episode | Status |
|---|---------|--------|
| 1 | The hardware | ✅ shipped |
| 2 | The scope — repo public | ✅ shipped |
| 3+ | The build continues | coming — each episode adds its own artifacts |

(Every link and artifact appears here at the same time the episode
publishes — nothing spoils the story ahead of its post.)

## Layout (final, as it will grow)

```
argocd/            # App-of-Apps root + child Applications
modules/apps/*/    # one Helm chart per workload
docs/
  decisions/       # ADRs
  incidents/       # what did not fit and why
scripts/           # bootstrap + tooling
```