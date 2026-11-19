# Kubernetes Network Isolation with NetworkPolicy

## Project Overview
I built this lab to understand how Kubernetes controls communication between pods across different namespaces and how `NetworkPolicy` can be used to restrict unauthorized access.

Using a local Kind cluster with Calico, I reproduced unrestricted pod communication, applied default-deny policies, and allowed only the traffic required by the application.

## What I Worked On
- Set up a Kind cluster with Calico (`kind-calico.yaml`) to enforce Kubernetes `NetworkPolicy` rules.
- Tested how a client pod in `marketing-web` could access `payments-db` in `prod-payments` before isolation (`workloads.yaml`).
- Applied default-deny ingress and egress policies (`01-default-deny.yaml`).
- Configured specific rules (`02-zero-trust-allow.yaml`) for DNS (port `53`), ingress traffic (port `80`), and application-to-database communication (port `5432`).
- Tested legitimate and unauthorized network connections.

## Issues Encountered and Resolved

| Issue | Root Cause | Resolution |
| :--- | :--- | :--- |
| **Database accessible across namespaces** | Default Kubernetes networking allows pod-to-pod communication. | Applied `default-deny-all` ingress and egress policies. |
| **DNS resolution failed** | Egress policy blocked CoreDNS traffic to `kube-system`. | Allowed `UDP/TCP` port `53` to `kube-dns`. |
| **Ingress could not reach the application** | Ingress traffic from outside the namespace was blocked. | Allowed traffic from `ingress-nginx` on port `80`. |

## Verification Results
After applying the network policies, I performed four tests:

| Test | Result |
| :--- | :--- |
| `payment-api` to `payments-db:5432` | **PASS** |
| `ingress-nginx` to `payment-api` | **PASS** |
| Unauthorized database access (`marketing-web` → `payments-db:5432`) | **BLOCKED** |
| Direct application access bypass (`marketing-web` → `payment-api:80`) | **BLOCKED** |

## What I Learned
- How Kubernetes `NetworkPolicy` controls pod communication.
- Why a policy-enforcing CNI such as Calico is required.
- How default-deny policies affect DNS and ingress traffic.
- How to restrict database access to specific workloads.
- How to validate network isolation using `nslookup` and `nc`.

## Conclusion
This lab gave me practical experience implementing Kubernetes network isolation and troubleshooting connectivity issues caused by restrictive network policies. It also helped me understand how to improve cluster security without disrupting legitimate application communication.
