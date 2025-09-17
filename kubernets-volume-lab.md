

* `emptyDir`
* `hostPath`
* `NFS`

---

# 🔹 Lab Exercise 1: `emptyDir` Volume

📌 Use case: Temporary scratch space (data lost when Pod is deleted).

**YAML – Pod with emptyDir**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: emptydir-pod
spec:
  containers:
  - name: app
    image: busybox
    command: ["/bin/sh", "-c", "sleep 3600"]
    volumeMounts:
    - name: cache-volume
      mountPath: /tmp/cache
  volumes:
  - name: cache-volume
    emptyDir: {}
```

✅ **Steps**

1. Apply:

   ```bash
   kubectl apply -f emptydir-pod.yaml
   ```
2. Exec into pod:

   ```bash
   kubectl exec -it emptydir-pod -- sh
   ```
3. Inside pod:

   ```bash
   cd /tmp/cache
   echo "hello from emptyDir" > test.txt
   cat test.txt
   ```
4. Delete the pod and recreate it → file will be gone (ephemeral).

---

# 🔹 Lab Exercise 2: `hostPath` Volume

📌 Use case: Access **node filesystem** inside a pod.
⚠️ Not recommended in production (node-coupling), but good for lab.

**YAML – Pod with hostPath**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: hostpath-pod
spec:
  containers:
  - name: app
    image: busybox
    command: ["/bin/sh", "-c", "sleep 3600"]
    volumeMounts:
    - name: host-volume
      mountPath: /mnt/hostdata
  volumes:
  - name: host-volume
    hostPath:
      path: /data/hostshare
      type: DirectoryOrCreate
```

✅ **Steps**

1. On the Kubernetes node (VM/physical host), create test directory:

   ```bash
   sudo mkdir -p /data/hostshare
   sudo chmod 777 /data/hostshare
   ```
2. Apply:

   ```bash
   kubectl apply -f hostpath-pod.yaml
   ```
3. Exec into pod:

   ```bash
   kubectl exec -it hostpath-pod -- sh
   ```
4. Inside pod, create file:

   ```bash
   echo "hello from hostPath" > /mnt/hostdata/test.txt
   ```
5. Check on node:

   ```bash
   cat /data/hostshare/test.txt
   ```

   → You’ll see the file.

---

# 🔹 Lab Exercise 3: `NFS` Volume

📌 Use case: **Shared storage** across pods.
(All pods mounting same NFS share can read/write the same files).

### Step 1: Setup NFS Server (on VM or your host)

```bash
sudo apt update && sudo apt install -y nfs-kernel-server
sudo mkdir -p /srv/nfs/kubedata
sudo chmod 777 /srv/nfs/kubedata
echo "/srv/nfs/kubedata *(rw,sync,no_subtree_check,no_root_squash)" | sudo tee -a /etc/exports
sudo exportfs -rav
```

Check NFS server IP:

```bash
hostname -I
```

Suppose it’s `192.168.1.100`.

---

### Step 2: Create Pod with NFS Volume

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nfs-pod
spec:
  containers:
  - name: app
    image: busybox
    command: ["/bin/sh", "-c", "sleep 3600"]
    volumeMounts:
    - name: nfs-volume
      mountPath: /mnt/nfsdata
  volumes:
  - name: nfs-volume
    nfs:
      server: 192.168.1.100   # replace with your NFS server IP
      path: /srv/nfs/kubedata
```

✅ **Steps**

1. Apply:

   ```bash
   kubectl apply -f nfs-pod.yaml
   ```

2. Exec into pod:

   ```bash
   kubectl exec -it nfs-pod -- sh
   ```

3. Inside pod:

   ```bash
   echo "hello from NFS" > /mnt/nfsdata/test.txt
   ```

4. Check on NFS server:

   ```bash
   cat /srv/nfs/kubedata/test.txt
   ```

   → File should exist.

5. Deploy another pod with the same NFS mount → it should **see the same file**.

---

📌 **Comparison**

| Volume Type | Storage Location | Persistence                | Use Case                          |
| ----------- | ---------------- | -------------------------- | --------------------------------- |
| emptyDir    | Node (ephemeral) | Lost when Pod deleted      | Scratch/temp space                |
| hostPath    | Node filesystem  | Persists if node survives  | Node-specific data (logs, config) |
| NFS         | Remote server    | Persists beyond pods/nodes | Shared storage between pods       |

---

---

# 🧪 Lab 1: `emptyDir` with Pod

📌 **Objective**: Learn ephemeral storage inside a pod.

**YAML File: `emptydir-pod.yaml`**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: emptydir-pod
spec:
  containers:
  - name: app
    image: busybox
    command: ["/bin/sh", "-c", "sleep 3600"]
    volumeMounts:
    - name: cache-volume
      mountPath: /tmp/cache
  volumes:
  - name: cache-volume
    emptyDir: {}
```

**Steps**

1. Apply pod:

   ```bash
   kubectl apply -f emptydir-pod.yaml
   ```
2. Enter pod:

   ```bash
   kubectl exec -it emptydir-pod -- sh
   ```
3. Inside pod:

   ```bash
   echo "hello emptyDir" > /tmp/cache/test.txt
   cat /tmp/cache/test.txt
   ```
4. Delete pod & recreate → file will be gone.

---

# 🧪 Lab 2: `hostPath` with Pod

📌 **Objective**: Mount **node directory** into pod.

**Prep on Node**

```bash
sudo mkdir -p /data/hostshare
sudo chmod 777 /data/hostshare
```

**YAML File: `hostpath-pod.yaml`**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: hostpath-pod
spec:
  containers:
  - name: app
    image: busybox
    command: ["/bin/sh", "-c", "sleep 3600"]
    volumeMounts:
    - name: host-volume
      mountPath: /mnt/hostdata
  volumes:
  - name: host-volume
    hostPath:
      path: /data/hostshare
      type: DirectoryOrCreate
```

**Steps**

1. Apply pod:

   ```bash
   kubectl apply -f hostpath-pod.yaml
   ```
2. Enter pod:

   ```bash
   kubectl exec -it hostpath-pod -- sh
   ```
3. Inside pod:

   ```bash
   echo "hello hostPath" > /mnt/hostdata/test.txt
   ```
4. On node, verify:

   ```bash
   cat /data/hostshare/test.txt
   ```

   → File should exist.

---

# 🧪 Lab 3: `NFS` with Pod

📌 **Objective**: Use **shared NFS storage** across pods.

**Prep: Setup NFS Server (outside cluster or on one node)**

```bash
sudo apt update && sudo apt install -y nfs-kernel-server
sudo mkdir -p /srv/nfs/kubedata
sudo chmod 777 /srv/nfs/kubedata
echo "/srv/nfs/kubedata *(rw,sync,no_subtree_check,no_root_squash)" | sudo tee -a /etc/exports
sudo exportfs -rav
```

Check server IP:

```bash
hostname -I
```

Suppose → `192.168.1.100`

**YAML File: `nfs-pod.yaml`**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nfs-pod
spec:
  containers:
  - name: app
    image: busybox
    command: ["/bin/sh", "-c", "sleep 3600"]
    volumeMounts:
    - name: nfs-volume
      mountPath: /mnt/nfsdata
  volumes:
  - name: nfs-volume
    nfs:
      server: 192.168.1.100   # replace with your NFS server IP
      path: /srv/nfs/kubedata
```

**Steps**

1. Apply pod:

   ```bash
   kubectl apply -f nfs-pod.yaml
   ```
2. Enter pod:

   ```bash
   kubectl exec -it nfs-pod -- sh
   ```
3. Inside pod:

   ```bash
   echo "hello NFS" > /mnt/nfsdata/test.txt
   ```
4. On NFS server:

   ```bash
   cat /srv/nfs/kubedata/test.txt
   ```

   → File should exist.
5. Start another pod with same NFS mount → verify it can read `test.txt`.

---

✅ Now you have **three separate labs**:

1. `emptyDir` → ephemeral pod storage
2. `hostPath` → node’s local storage
3. `NFS` → shared persistent storage

---

