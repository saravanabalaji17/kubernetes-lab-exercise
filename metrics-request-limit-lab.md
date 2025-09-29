
# 🧪 Kubernetes Lab Exercise: Metrics, Requests, Limits 

---

## 🔹 1. Install Metrics Server

```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

👉 On Minikube/Kind, edit the deployment:

```bash
kubectl -n kube-system edit deployment metrics-server
```

Add under container args:

```yaml
        - /metrics-server
        - --kubelet-insecure-tls
        - --kubelet-preferred-address-types=InternalIP
```

Verify:

```bash
kubectl top nodes
kubectl top pods -A
```

---

## 🔹 2. Nginx Pod **without** Requests/Limits

**nginx-no-limits.yaml**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-no-limits
spec:
  containers:
  - name: nginx
    image: nginx
    command: ["sleep", "3600"]   # keep pod alive
```

Deploy:

```bash
kubectl apply -f nginx-no-limits.yaml
kubectl get pod nginx-no-limits
```

---

## 🔹 3. Nginx Pod **with** Requests & Limits

**nginx-limits.yaml**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-limits
spec:
  containers:
  - name: nginx
    image: nginx
    resources:
      requests:
        cpu: "100m"
        memory: "128Mi"
      limits:
        cpu: "200m"
        memory: "256Mi"
    command: ["sleep", "3600"]   # keep pod alive
```

Deploy:

```bash
kubectl apply -f nginx-limits.yaml
kubectl get pod nginx-limits
```

---

## 🔹 4. Increase CPU Usage with `dd`

Exec into pod:

```bash
kubectl exec -it nginx-limits -- bash
```

Run CPU stress (infinite loop with `dd`):

```bash
dd if=/dev/zero of=/dev/null bs=1M &
```

Check metrics:

```bash
kubectl top pod nginx-limits
```

👉 You’ll see CPU capped at **~200m** because of the limit.
Now try the same inside **nginx-no-limits** pod:

```bash
kubectl exec -it nginx-no-limits -- bash
dd if=/dev/zero of=/dev/null bs=1M &
```

👉 CPU usage can grow much higher since there’s no limit.

---

## 🔹 5. Increase Memory Usage with `dd`

Inside **nginx-limits**:

```bash
dd if=/dev/zero of=/dev/shm/testfile bs=10M count=40
```

👉 This tries to allocate ~400MB into RAM (`/dev/shm`).
Since the pod has a **limit of 256Mi**, it will get **OOMKilled**.

Check pod status:

```bash
kubectl get pod nginx-limits
kubectl describe pod nginx-limits | grep -A5 "State"
kubectl get events --sort-by=.metadata.creationTimestamp
```

Now try the same in **nginx-no-limits** pod:

```bash
dd if=/dev/zero of=/dev/shm/testfile bs=10M count=200
```

👉 Pod may survive until node runs out of memory, then the **kubelet OOMKills processes** at the node level.

---

## ✅ Learning Outcomes

* **Metrics-server installed** and `kubectl top` working.
* **Pod without limits** → can consume all CPU & memory until node pressure occurs.
* **Pod with limits** →

  * CPU usage capped (throttling).
  * Memory usage beyond limit → pod is **OOMKilled**.

---
