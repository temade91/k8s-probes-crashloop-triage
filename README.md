# Kubernetes Production SRE Triage Runbooks

Hands-on production incident reproduction, root cause analysis (RCA), and remediation runbooks executed on local Kind clusters.

| # | Incident Scenario | Root Cause & Architecture | Runbook & Manifests |
| :--- | :--- | :--- | :--- |
| **01** | **Pod CrashLoopBackOff & Premature Traffic Routing** | Aggressive `livenessProbe` killing slow-starting containers and missing `startupProbe` / `readinessProbe` separation. | [`01-probes-crashloop/`](./01-probes-crashloop/) |
| **02** | **Cross-Namespace Database Exposure & DNS/Ingress Blackout** | Default all-allow flat pod networking mitigated via Calico CNI, `default-deny-all` `NetworkPolicy`, and targeted whitelists (`UDP/TCP 53`, `TCP 80`, `TCP 5432`). | [`02-network-isolation/`](./02-network-isolation/) |
