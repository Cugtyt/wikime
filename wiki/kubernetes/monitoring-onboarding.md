---
title: Monitoring Onboarding
topic: kubernetes
sources:
  - notes/2026/04-w15.md
last_updated: 2026-04-08
---

# Monitoring Onboarding

Onboarded Jake to the Kubernetes monitoring stack — Grafana dashboards, alerting rules, and escalation paths. Created a repeatable walkthrough that can be used for future team members joining the platform team.

## Problem

New team members had no structured introduction to our monitoring infrastructure. Jake needed to get up to speed on how we observe and alert on cluster health.

## What was covered

- Grafana dashboard layout: cluster overview, node health, pod resource usage
- Alerting rules: what triggers pages vs. warnings
- Escalation paths: who to contact for which alerts

## Key Takeaways

- A structured monitoring walkthrough saves ~2 days of self-discovery for new team members
- Grafana dashboards should be self-documenting — add descriptions to panels

## Related

- [CNI Debugging](cni-debugging.md) — operational-support: monitoring stack was used during the CNI investigation
