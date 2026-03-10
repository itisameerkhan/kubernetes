## ⭐ Persistent Volume in Kubernetes

A Persistent Volume (PV) in Kubernetes is a storage resource in the cluster that allows data to persist even if Pods are deleted or recreated. Normally, when a Pod is removed, any data stored inside the container is also lost. Persistent Volumes solve this problem by providing storage that exists independently of the Pod lifecycle.

Persistent Volumes are managed by Kubernetes and can be backed by different types of storage systems such as local disks, network storage, or cloud storage services.

### ⚡ Where Persistent Volume Stores Data in Kubernetes

A Persistent Volume (PV) stores application data in an external storage system or physical disk, not inside Kubernetes itself. Kubernetes only manages the reference to the storage, while the actual files and data are stored in the underlying storage infrastructure.

| Storage Type                  | Where Data is Stored         |
| ----------------------------- | ---------------------------- |
| **Local Persistent Volume**   | Disk of the worker node      |
| **NFS (Network File System)** | External NFS storage server  |
| **AWS EBS**                   | Amazon Elastic Block Storage |
| **Azure Disk**                | Azure managed disk           |
| **Google Persistent Disk**    | Google Cloud storage disk    |
| **Ceph / GlusterFS**          | Distributed storage cluster  |

## ⭐ Types of Persistent Volume Access Modes

| Access Mode | Full Form        | Meaning                                               |
| ----------- | ---------------- | ----------------------------------------------------- |
| RWO         | ReadWriteOnce    | One node can mount the volume as read-write           |
| ROX         | ReadOnlyMany     | Multiple nodes can mount the volume as read-only      |
| RWX         | ReadWriteMany    | Multiple nodes can mount the volume as read-write     |
| RWOP        | ReadWriteOncePod | Only one pod in the entire cluster can use the volume |


## ⭐ Example YML file 

```yml
apiVersion: v1
kind: PersistentVolume
metadata: 
    name: demo-pv
    labels: 
        app: demo-pv-label
spec: 
    capacity: 
        storage: 1Gi
    accessMode: 
        - ReadWriteOnce
    hostPath: 
        path: /data/test 
```

![demo](../assets/demo021.png)

## ⭐ What is a PersistentVolumeClaim (PVC)?

A PersistentVolumeClaim (PVC) is a request made by a Pod for storage in a Kubernetes cluster. Instead of directly using a Persistent Volume (PV), applications request storage through a PVC. Kubernetes then automatically finds a suitable PV and binds it to the PVC.

In simple terms, PVC is like asking Kubernetes for storage, and Kubernetes provides a matching Persistent Volume that satisfies the request.

This abstraction allows developers to request storage without worrying about the underlying storage system (local disk, cloud disk, NFS, etc.).

### ⚡ Example YML for PersistentVolumeClaim (PVC)

```yml
apiVersion: v1
kind: PersistentVolumeClaim
metadata: 
    name: demo-pvc
    labels: 
        app: demo-pvc-label
spec:
    accessMode: 
        - ReadWriteOnce
    resources: 
        requests: 
            storage: 1Gi
```

## ⭐ Question: If PVC Requests 1Gi but Only a 2Gi PV Exists, What Happens?

In Kubernetes, a **PersistentVolumeClaim (PVC)** requests a specific amount of storage. Kubernetes will try to find a **PersistentVolume (PV)** that satisfies the request. The important rule is that the PV must have **equal or greater capacity than the requested storage**.

So if a PVC requests **1Gi** and the only available PV has **2Gi**, Kubernetes **will bind the PVC to that 2Gi PV** because it satisfies the requirement.

However, the PVC will still **request only 1Gi**, even though the underlying PV has more capacity.


### ⚡ Important Kubernetes Rule

| Condition             | Result     |
| --------------------- | ---------- |
| PV size = PVC request | Bind       |
| PV size > PVC request | Bind       |
| PV size < PVC request | No binding |

---

## ⭐ What is a StorageClass in Kubernetes?

A StorageClass in Kubernetes defines how Persistent Volumes (PV) should be created automatically. It provides a way to dynamically provision storage instead of manually creating Persistent Volumes.

Before StorageClass existed, administrators had to manually create PVs and developers would create PVCs to bind to them. With StorageClass, when a PVC requests storage, Kubernetes can automatically create the required PV using the specified storage backend.

In simple terms, StorageClass acts like a storage template or blueprint that tells Kubernetes how to create storage dynamically.