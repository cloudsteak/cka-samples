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

Save this manifest as `pvc.yaml`:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-1gb
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: local-path
```

Apply it:

```bash
kubectl apply -f pvc.yaml
```

#### Step 4: Create a pod that uses the PVC

Save this manifest as `pod.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-pvc
spec:
  containers:
    - name: app
      image: busybox
      command: ["sleep", "3600"]
      volumeMounts:
        - mountPath: "/data"
          name: storage
  volumes:
    - name: storage
      persistentVolumeClaim:
        claimName: pvc-1gb
```

Apply it:

```bash
kubectl apply -f pod.yaml
```

#### Step 5: Verify resources

```bash
kubectl get pvc
kubectl get pv
kubectl get pod pod-pvc
kubectl describe pvc pvc-1gb
```

#### Step 6: Test volume mount inside the container

```bash
kubectl exec -it pod-pvc -- sh
```

Inside the pod:

```sh
echo "hello cka" > /data/test.txt
cat /data/test.txt
```

### Key observations

- A PersistentVolume is automatically provisioned for the PVC.
- The volume is bound and mounted into the container.
- Files written to `/data` in the container persist for the pod’s lifetime.

### Kind versus cloud-based Kubernetes clusters

In a Kind cluster:

- Dynamic provisioning is simulated using hostPath via local-path-provisioner.
- No real cloud disk resources are provisioned.

In a real Kubernetes cluster (e.g., AWS, Azure, GCP):

- The StorageClass uses a CSI driver to provision real block storage (e.g., EBS, Azure Disk).
- Volumes are managed by cloud infrastructure and persist independently of the container lifecycle.
