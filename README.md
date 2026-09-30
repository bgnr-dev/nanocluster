# NanoCluster

A portable Kubernetes platform on three NanoPC-T4 boards — RK3399, ARMv8,
**4 GB of RAM each**. GitOps, self-hosted CI/CD and real production-style
workloads, all inside that constraint.

> This repo grows with a public building-log series: each episode ships
> the artifacts it talks about. What you see now is the foundation.

## The hardware (episode 1)

- 3× [NanoPC-T4](https://wiki.friendlyelec.com/wiki/index.php/NanoPC-T4) (Rockchip RK3399, 4 GB LPDDR3, ARMv8.0)
- 1 dusty MikroTik routing the cluster (10.42.42.1) + a newer one
  hosting the LAN DNS for the `*.nc` zone (192.168.1.1)
- 1 desktop (amd64 — the TARDIS, 48 GB) next to it — registry mirror
  + the fast CI runner

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

## The 4 GB wall (episode 3, [ADR-002](docs/decisions/ADR-002-resource-budget.md))

The whole design document is one number: **no workload exceeds 1 GiB of RAM**.

- One budget instead of a capacity spreadsheet — every component fits a
  mental model
- Workloads are chosen for the budget, never the other way around
- Saying no is a feature: what cannot fit under 1 GiB gets a dedicated
  machine, not a squeeze

## The GitOps foundation (episode 4, `argocd/`)

The cluster is a render of this repo — pushed commits are deployments.

- [`argocd/apps.yaml`](argocd/apps.yaml) — the App-of-Apps root: it
  watches the repo and reconciles the child Applications
- sync + prune + selfHeal — drift is impossible by construction
- reproducible by default: delete the Applications and git recreates
  them from the same commit

## The networking (episode 5, [ADR-005](docs/decisions/ADR-005-ingress-gateway-api.md))

Every service gets a `*.nc` name — the LAN gateway's DNS hosts the zone,
external-dns writes the records. Traffic is routed by the Gateway API:

- the **Gateway** (capital G) is the Kubernetes resource that routes
  traffic: `Gateway → HTTPRoute → Service`
- the **gateway** (lowercase) is the MikroTik that resolves the names —
  a newer box at 192.168.1.1, while the older one (10.42.42.1) is where
  the cluster's nodes plug in — same word, two different jobs
- [Traefik v3](modules/apps/traefik/) runs it: ~100–200 MiB, against the
  ~1.5 GiB an Envoy Gateway would ask for on 12 GB

## The roadmap (repo artifacts appear with each episode)

| # | Episode | Status |
|---|---------|--------|
| 1 | The hardware | ✅ shipped |
| 2 | The scope — repo public | ✅ shipped |
| 3 | The 4 GB wall | ✅ shipped |
| 4 | The GitOps foundation | ✅ shipped |
| 5 | The networking | ✅ shipped |
| 6 | CI/CD — the dual runner | ✅ shipped |
| 7+ | The build continues | coming — each episode adds its own artifacts |

(Every link and artifact appears here at the same time the episode
publishes — nothing spoils the story ahead of its post.)

A GitLab migration is on my mind — free self-hosted CI in the same
resource style — but that's the future's music, not this season's.

## Layout (final, as it will grow)

```
argocd/            # App-of-Apps root + child Applications
modules/apps/*/    # one Helm chart per workload
docs/
  decisions/       # ADRs
  incidents/       # what did not fit and why
scripts/           # bootstrap + tooling
```