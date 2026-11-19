# Kubernetes Probe Flapping & CrashLoopBackOff Triage

## Project Overview

I built this lab to reproduce and troubleshoot a Kubernetes `CrashLoopBackOff` scenario caused by an incorrectly configured liveness probe.

The `payment-api` container was repeatedly restarted because Kubernetes checked its health before the application had completed its startup process. I investigated the pod status and events, updated the health-probe configuration, and verified that the workload could become ready without unnecessary restarts.

I also tested readiness behaviour by temporarily making the application unready and checking that Kubernetes removed the pod from the Service endpoints while leaving the container running.

## Incident Summary

- **Workload:** `payment-api`
- **Namespace:** `probe-lab`
- **Initial status:** `CrashLoopBackOff`
- **Observed restarts:** 13
- **Reported failure:** `Liveness probe failed`
- **Probe error:** `cat: can't open '/tmp/healthy': No such file or directory`

### Root Cause

The original liveness check was failing because its expected health file was not present when the check ran. Kubernetes treated the failed liveness checks as evidence that the container was unhealthy and repeatedly restarted it.


## Remediation

I updated the Deployment manifest in `fixed-app.yaml` to separate the responsibilities of the three Kubernetes health probes.

| Probe | Purpose | Configuration | Behaviour |
|---|---|---|---|
| `startupProbe` | Allows application initialization | `periodSeconds: 5`, `failureThreshold: 12` | Allows up to 60 seconds for startup |
| `readinessProbe` | Determines whether the pod receives traffic | `periodSeconds: 5`, `failureThreshold: 2` | Removes an unready pod from Service endpoints |
| `livenessProbe` | Detects an unhealthy application after startup | `periodSeconds: 10`, `failureThreshold: 3` | Restarts the container after repeated failures |

The startup probe gives the application time to initialize. Readiness controls traffic eligibility, while liveness detects whether the running application has become unhealthy.

## Lab Implementation

### 1. Reproduce the issue

```bash
kubectl create namespace probe-lab
kubectl apply -f broken-app.yaml -n probe-lab
kubectl get pods -n probe-lab -w
kubectl describe pod -l app=payment-api -n probe-lab
```

### 2. Apply the corrected manifest

```bash
kubectl apply -f fixed-app.yaml -n probe-lab
kubectl get pods -n probe-lab
```

### 3. Test readiness and endpoint isolation

I simulated an application readiness failure by removing the readiness file from the running container.

```bash
POD_NAME=$(kubectl get pod -l app=payment-api -n probe-lab -o jsonpath='{.items[0].metadata.name}')

kubectl exec "$POD_NAME" -n probe-lab -- rm /tmp/ready
kubectl get pods -n probe-lab
kubectl get endpoints payment-svc -n probe-lab
```

**Observed result:**

- Pod readiness changed to `0/1`.
- The pod remained in `Running` status.
- The restart count remained at zero.
- The `payment-svc` endpoint list became empty.

I restored readiness and checked the pod and Service endpoints again:

```bash
kubectl exec "$POD_NAME" -n probe-lab -- touch /tmp/ready
sleep 6
kubectl get pods -n probe-lab
kubectl get endpoints payment-svc -n probe-lab
```
### 4. Clean up

```bash
kubectl delete namespace probe-lab
```

## Repository Structure

```text
.
├── broken-app.yaml
├── fixed-app.yaml
├── README.md
```

## What I Practised

- Investigating `CrashLoopBackOff` using `kubectl get pods` and `kubectl describe pod`.
- Reading Kubernetes events to identify liveness probe failures.
- Understanding startup, readiness, and liveness probes.
- Preventing premature container restarts during application initialization.
- Verifying readiness-based Service endpoint isolation without restarting containers.
- Confirming recovery through pod status, restart counts, and Service endpoints.

## Conclusion

This lab gave me practical experience troubleshooting a Kubernetes workload that repeatedly restarted because of health-check configuration. I reproduced the failure, applied a probe-based fix, and verified application recovery and readiness-based endpoint isolation.

