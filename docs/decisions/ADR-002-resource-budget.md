# ADR-002: The 1 GiB Budget

Status: accepted

## Context

The cluster is three NanoPC-T4 boards — 4 GB of RAM each, 12 GB total,
no swap pressure worth betting the design on. RAM is the dominant
constraint, and every workload on the platform is sized against it.

## Decision

**No workload on this cluster may request or limit more than 1 GiB of
RAM.**

That single number is the whole capacity plan. Workloads are chosen for
the budget — never is the budget bent for a workload. If a service needs
more than 1 GiB, it does not get squeezed in; it gets a dedicated machine.

## Consequences

- One number instead of a capacity spreadsheet — every component fits a
  mental model instantly.
- The budget is self-enforcing: a workload that cannot fit under 1 GiB
  reveals itself during evaluation, not during an OOMKill at 3 a.m.
- Some workloads simply do not belong on this platform. Saying no is a
  feature, not a failure.