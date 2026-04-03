---
title: CNI Debugging
topic: kubernetes
sources:
  - notes/2026/04-w1.md
last_updated: 2026-04-03
---

# CNI Debugging

Container Network Interface (CNI) issues can cause pod communication failures and crashes in Kubernetes clusters. This article covers common debugging steps and a specific Flannel OOMKilled resolution.

## Flannel OOMKilled / MTU Mismatch

Flannel pods in the staging cluster were crashing with OOMKilled errors, traced to mismatched MTU settings between nodes. The fix was to set the network interface explicitly:

```
--iface=eth0
```

This forces Flannel to use a consistent interface across nodes, avoiding MTU negotiation problems.

## CNI Debugging Runbook

When troubleshooting CNI issues, check the following in order:

1. **MTU settings** — ensure consistency across all nodes
2. **iptables rules** — verify rules aren't dropping or mangling pod traffic
3. **Pod CIDR allocation** — confirm non-overlapping ranges and correct routing

## Key Takeaways

- MTU mismatches between nodes are a common cause of CNI instability
- Explicitly setting `--iface` on Flannel avoids interface ambiguity
- A systematic check of MTU, iptables, and CIDR allocation covers most CNI issues

## Related

- [Kubernetes Monitoring Stack](../onboarding/kubernetes-monitoring.md) — monitoring stack for cluster observability
