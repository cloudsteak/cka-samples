# Storage management in Kubernetes

This document provides an overview of storage management in Kubernetes, focusing on the implementation of storage classes and dynamic volume provisioning.

## Implement storage classes and dynamic volume provisioning

In Kubernetes, persistent storage is managed through the use of Persistent Volumes (PVs) and Persistent Volume Claims (PVCs). To automate the provisioning of volumes, Kubernetes uses StorageClasses. These allow users to request storage without needing to manually create the underlying PersistentVolumes.

### Key components

- **PersistentVolume (PV)**: A cluster resource that represents a piece of storage.
- **PersistentVolumeClaim (PVC)**: A user request for storage that is matched with a PV.
- **StorageClass**: A resource that defines a class of storage and the provisioner used to dynamically create volumes.

### Benefits of dynamic provisioning

Dynamic provisioning allows PVCs to automatically trigger the creation of the required Persistent Volume by referencing a StorageClass. When a PVC is created with a storageClassName, Kubernetes uses the StorageClass to provision a volume dynamically.

- Eliminates the need for pre-provisioning PVs.
- Supports multiple types of storage backends.
- Offers flexibility via parameters and reclaim policies.

### Common provisioners

- `kubernetes.io/aws-ebs` – AWS Elastic Block Store
- `kubernetes.io/gce-pd` – Google Cloud Persistent Disk
- `kubernetes.io/no-provisioner` – For static volumes
- `rancher.io/local-path` – LocalPath provisioner for local development (used in kind or minikube)

### LocalPath provisioner (for kind)

For kind clusters, dynamic provisioning can be enabled using the [LocalPath provisioner](https://github.com/rancher/local-path-provisioner), which provisions volumes on the local disk of each node.

This provisioner needs to be installed separately and set as the default StorageClass for proper testing.

### Goal

Understand how to:

- Install a StorageClass (LocalPath for kind)
- Use it to dynamically provision volumes
- Verify the lifecycle of PV/PVC/Pod bindings

### Official Kubernetes documentation

- [Persistent Volumes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)
- [Persistent Volume Claims](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#persistentvolumeclaims)
- [Storage Classes](https://kubernetes.io/docs/concepts/storage/storage-classes/)
- [Dynamic Volume Provisioning](https://kubernetes.io/docs/concepts/storage/dynamic-provisioning/)
- [Local Path Provisioner (GitHub)](https://github.com/rancher/local-path-provisioner)

### Prerequisites

- A running Kind cluster
- `kubectl` installed and configured
- Optional: `local-path-provisioner` deployed in the cluster
- Working directory: `01-storage/01-dynamic-volume-provisioning`

### Step-by-step practice using Kind and local-path-provisioner

Kind does not support cloud-native dynamic provisioning (e.g., EBS, GCE PD), but it supports simulation via `local-path-provisioner`.

#### Step 1: Deploy the local-path-provisioner

```bash
kubectl apply -f https://raw.githubusercontent.com/rancher/local-path-provisioner/master/deploy/local-path-storage.yaml
```

This creates a StorageClass named `local-path`.

#### Step 2: Set local-path as the default StorageClass (optional)

```bash
kubectl patch storageclass local-path -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
```


#### Step 3: Create a PersistentVolumeClaim

Check the pvc manifest in `pvc-1gb.yaml`

Apply it:

```bash
kubectl apply -f pvc-1gb.yaml
```

#### Step 4: Create a pod that uses the PVC

Check the pod manifest in `pod-1-pvc-1gb.yaml`

Apply it:

```bash
kubectl apply -f pod-1-pvc-1gb.yaml
```

#### Step 5: Verify resources

```bash
kubectl get pvc
kubectl get pv
kubectl get pod pod-1-pvc
kubectl describe pvc pvc-1gb
```

#### Step 6: Test volume mount inside the container

```bash
kubectl exec -it pod-1-pvc -- sh
```

Inside the pod:

```sh
echo "hello cka" > /data/test.txt
cat /data/test.txt
```

#### Step 7: Create a second pod using the same PVC

Check the pod manifest in `pod-2-pvc-1gb.yaml`
Apply it:

```bash
kubectl apply -f pod-2-pvc-1gb.yaml
```

#### Step 8: Verify the second pod

```bash
kubectl get pod pod-2-pvc
kubectl exec -it pod-2-pvc -- sh
```

Inside the second pod:

```sh
cat /data/test.txt
```

This should show the same content as in the first pod.

#### Step 9: Clean up

```bash
kubectl delete -f pod-2-pvc-1gb.yaml
kubectl delete -f pod-1-pvc-1gb.yaml
kubectl delete -f pvc-1gb.yaml
```

### Key observations

- A PersistentVolume is automatically provisioned for the PVC.
- The volume is bound and mounted into the container.
- Files written to `/data` in the container persist for the pod’s lifetime.

### Kind versus cloud-based Kubernetes clusters

This section highlights key differences when working with storage classes and dynamic provisioning in Kind versus production-grade Kubernetes clusters such as Amazon EKS, Google GKE, or Azure AKS.

#### 1. Provisioner

**Kind:**
- Uses `hostPath` under the hood, typically via `local-path-provisioner`.
- Volumes are directories on the host machine (e.g., within Docker container filesystem).
- The provisioner is not cloud-native; no real persistent storage is provisioned outside the local node.

**Real cluster:**
- Uses cloud-native CSI drivers (e.g., `ebs.csi.aws.com`, `disk.csi.azure.com`, `pd.csi.storage.gke.io`).
- Provisioned volumes are real block storage devices in the cloud.
- Volumes persist independently of the node or pod lifecycle.

---

#### 2. Storage persistence and behavior

**Kind:**
- Volumes are ephemeral and tied to the container running the Kind node.
- Restarting the Kind cluster or Docker container may lead to data loss unless special volume mappings are configured.

**Real cluster:**
- Persistent Volumes are truly persistent and survive pod restarts, node failures, and even cluster upgrades (depending on reclaim policy).

---

#### 3. Access modes

**Kind:**
- `ReadWriteOnce` is supported for pods scheduled to the same node.
- Multi-pod access using the same PVC may fail if pods are scheduled on different nodes, depending on hostPath limitations.

**Real cluster:**
- Real storage backends can enforce and support `ReadWriteOnce`, `ReadOnlyMany`, or `ReadWriteMany` according to backend capabilities.
- Shared access is properly enforced at the infrastructure level.

---

#### 4. Reclaim policies and lifecycle

**Kind:**
- `Retain` and `Delete` policies can be defined but often do not behave as expected, because the backing storage is not persistent.

**Real cluster:**
- `Delete` reclaims and deletes the cloud disk.
- `Retain` detaches the disk but leaves it available for manual recovery or reassignment.

---

#### 5. Performance and scalability

**Kind:**
- Not suitable for performance or scale testing.
- Ideal for functional and configuration validation.

**Real cluster:**
- Designed for production workloads.
- Storage behavior and performance metrics reflect real-world expectations.

---

#### Summary

| Feature                | Kind (local-path)                  | Real Cluster (Cloud CSI)         |
|------------------------|------------------------------------|----------------------------------|
| Provisioner type       | HostPath via local-path            | Cloud-native CSI driver          |
| Persistent storage     | No (host-level only)               | Yes                              |
| StorageClass behavior  | Simulated                          | Fully supported                  |
| AccessMode enforcement | Limited                            | Enforced                         |
| ReclaimPolicy behavior | Incomplete                         | Fully supported                  |
| Use case               | Local testing                      | Production-grade deployments     |

