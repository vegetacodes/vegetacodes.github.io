---
title: 'From Crash to Calm: How to Debug and Resolve Redis Pod Issues'
date: 2025-04-25T06:38:35-0400
draft: false
---

## Introduction

In one of our high-traffic deployments, which sits at the intersection of web and AI use cases, we had a Redis cache pod shared between the web application and AI background workers. While this setup wasn’t ideal, it had worked well—until one day, we were paged: Redis was down. When a Redis pod crashes or restarts frequently, it's important to follow a step-by-step guide to diagnose and resolve the root cause effectively.

---

### 🔍 1. Check Pod Status & Events

```bash
kubectl describe pod <pod-name> -n <namespace>
```

Look for:
- **Status**: `OOMKilled`, `CrashLoopBackOff`, etc.
- **Events**: At the bottom — check for warnings, restarts, failed probes.

---

### 📄 2. View Pod Logs

```bash
kubectl logs <pod-name> -n <namespace> --previous
```

Useful to find:
- Crash traces
- Redis-specific errors (`OOM command not allowed`, `MISCONF`, etc.)
- File system or permission issues

---

### 📈 3. Check Resource Limits

```bash
kubectl get pod <pod-name> -o jsonpath="{.spec.containers[*].resources}"
```

- Look for memory limits that may be too low.
- Redis is memory-intensive and may be OOMKilled if usage exceeds limits.

---

### 🧠 4. Log Into the Redis Pod

```bash
kubectl exec -it <pod-name> -n <namespace> -- sh
redis-cli
```

Once inside the Redis CLI, run the following diagnostics:

---

### 🔎 5. Redis Internal Diagnostics

#### ➤ Memory Usage & Key Stats

```bash
INFO MEMORY
INFO KEYSPACE
redis-cli --bigkeys
```

- `INFO MEMORY`: View `used_memory`, fragmentation, overheads.
- `INFO KEYSPACE`: See number of keys in each DB.
- `--bigkeys`: Find largest keys (can reveal memory hogs).

#### ➤ Eviction Policy Check & Fix

Check current policy:
```bash
CONFIG GET maxmemory-policy
```

If it's set to `noeviction`, Redis will crash on memory exhaustion. Update it:

```bash
CONFIG SET maxmemory-policy allkeys-lru
```

> ⚠️ This is a runtime change — to persist, update your Helm chart, ConfigMap, or redis.conf.

Check memory cap:
```bash
CONFIG GET maxmemory
```

Set memory cap (e.g., 512MB):
```bash
CONFIG SET maxmemory 536870912
```

> ⚠️ This is a runtime change — to persist, update your Helm chart, ConfigMap, or redis.conf.

#### ➤ Additional Useful Commands

```bash
INFO clients          # Number of connected clients
INFO commandstats     # Command usage stats
MONITOR               # Live traffic (only use in dev/debug)
```

---

### 🧰 6. Investigate Node & Cluster Health

Sometimes the problem lies outside Redis:
- Run `kubectl top node` / `top pod` to check for high memory or CPU.
- Inspect node logs (`journalctl`, `dmesg`) for kernel-level OOM kills.
- Look for disk pressure or pod evictions.

---

### 📦 7. Persistent Storage (if using AOF or RDB)

- Confirm volume is mounted and writable.
- Check for disk full conditions or slow I/O.

---

### 💾 8. Deployment Config

- Liveness/readiness probes: Are they tuned correctly?
- Volume mounts: Correct paths/permissions?
- Any sidecars affecting Redis behavior?

```yml
# redis-deployment.yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis
spec:
  replicas: 1
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
    spec:
      containers:
      - name: redis
        image: redis:latest
        ports:
        - containerPort: 6379
```

---

## Conclusion

Diagnosing Redis pod crashes requires a methodical approach that covers pod status, logs, resource limits, Redis internal diagnostics, node health, persistent storage, and deployment configurations. This helped me identify and resolve the root cause of Redis crashes in our org and ensuring a more stable cache layer. Key takeaways include:
- Monitor resource limits: Ensure Redis has sufficient memory and CPU resources.
- Configure eviction policies: Set a suitable eviction policy to prevent crashes due to memory exhaustion.
- Check node and cluster health: Identify potential issues outside of Redis.
- Verify persistent storage: Confirm volume mounts and writable permissions.
- Optimize deployment configurations: Tune liveness/readiness probes and volume mounts for optimal performance.
By applying these best practices, you can minimize Redis crashes and ensure a more robust system.

## Resources
- [Redis ConfigMap Configuration](https://kubernetes.io/docs/tutorials/configuration/configure-redis-using-configmap/)
- [Redis Eviction Policies](https://redis.io/docs/latest/develop/reference/eviction/)
- [Redis Enterprise Kubernetes Architecture](https://redis.io/docs/latest/operate/kubernetes/architecture/)
