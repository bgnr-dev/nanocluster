# ADR-005: The ingress layer — Traefik v3 with the Gateway API

Status: accepted (replaces the ingress-nginx controller)

## Context

- 12 GB of RAM across three ARM64 nodes; ADR-002 caps every workload at
  1 GiB
- The current ingress layer is an ingress-nginx controller with classic
  Ingress objects (argocd.nc, orthanc.nc, titiler.nc)
- The industry is moving to the Gateway API (v1.x GA) — the API the
  tooling and the job market expect in 2026
- Two Gateway-API controllers were considered: Envoy Gateway (CNCF) and
  Traefik v3 (the k3s default)

## Decision

Run **Traefik v3 as the Gateway API controller** (Gateway + HTTPRoute).

Envoy Gateway got the better name — and the bigger bill: ~1.5–2 GiB for
the controller + Envoy-proxy pair. Traefik does the same job in
~100–200 MiB. On this cluster the budget votes.

## Consequences

- Gateway API resources replace the Ingress objects: argocd.nc,
  orthanc.nc, titiler.nc become Gateway + HTTPRoute
- TLS for the `*.nc` zone stays terminated at the gateway (internal CA)
- external-dns gains `--source=gateway-httproute` so the `*.nc` records
  keep flowing from the new resources
- The old ingress-nginx controller is removed once the routes are live
  and verified