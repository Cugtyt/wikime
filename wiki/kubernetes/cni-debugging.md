---
title: CNI Debugging
topic: kubernetes
sources:
  - notes/2026/04-w15.md
last_updated: 2026-04-07
---

# CNI Debugging

Resolved staging cluster CNI instability caused by MTU mismatch across nodes. Flannel pods were crashing with OOMKilled errors, blocking deploys for the platform, payments, and onboarding teams. Fixed by forcing Flannel to use a consistent interface (`--iface=eth0`), restoring cluster reliability.

## Problem

Flannel pods in staging kept crashing with OOMKilled errors. Root cause: MTU settings were mismatched between nodes, causing packet fragmentation and memory pressure.

## Resolution

Set `--iface=eth0` explicitly in the Flannel configuration. This forces a consistent network interface across all nodes, avoiding MTU negotiation problems.

## Runbook

When troubleshooting CNI issues, check in this order:

1. **MTU settings** — ensure consistency across all nodes
2. **iptables rules** — verify rules aren't dropping or mangling pod traffic
3. **Pod CIDR allocation** — confirm non-overlapping ranges and correct routing

## Key Takeaways

- MTU mismatches between nodes are a common cause of CNI instability
- Explicitly setting `--iface` on Flannel avoids interface ambiguity
- Always check MTU first — it's the most common and least obvious cause

## Related

- [Monitoring Onboarding](monitoring-onboarding.md) — operational-support: Grafana dashboards were used during the CNI investigation to identify affected pods
