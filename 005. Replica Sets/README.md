## ⭐ ReplicaSet

A ReplicaSet is a Kubernetes resource used to ensure that a specified number of identical Pods are running at all times in the cluster. Its main responsibility is to maintain the desired number of Pod replicas. If a Pod crashes or is deleted, the ReplicaSet automatically creates a new Pod to replace it.

ReplicaSet helps provide reliability and availability for applications by making sure the required number of Pods is always maintained.

---

### ⚡ How ReplicaSet Works

A ReplicaSet works by comparing the **desired state** and the **actual state** of Pods.

For example, if the desired state says **3 Pods should be running**, the ReplicaSet continuously checks the cluster. If one Pod fails or is deleted and only **2 Pods remain**, the ReplicaSet detects the difference and immediately creates a new Pod to bring the number back to **3**.

This ensures that applications remain available even when failures occur.

---

### ⚡ Pod Selection Using Labels

ReplicaSet uses **labels and selectors** to identify which Pods it manages.

Each Pod has labels, and the ReplicaSet uses a **selector** to match those labels. Only the Pods with matching labels will be managed by that ReplicaSet.

For example, if a ReplicaSet is configured to manage Pods with the label:

```
app: web
```

Then it will only monitor and control Pods that contain that label.

---

### ⚡ Important Note

In most real-world Kubernetes applications, developers do not create ReplicaSets directly. Instead, they create **Deployments**, and the Deployment automatically creates and manages the ReplicaSet.

The Deployment provides additional features such as **rolling updates, version control, and rollback**, which ReplicaSet alone does not handle.

---

## ⭐ ReplicaSet YAML File Structure

A ReplicaSet in Kubernetes is created using a YAML configuration file. This file defines how many Pods should run and what Pod template should be used to create them. The YAML file contains sections such as **apiVersion, kind, metadata, and spec**, which describe the configuration of the ReplicaSet.

---

### ⚡ apiVersion

The **apiVersion** specifies which Kubernetes API version is used to create the ReplicaSet. For ReplicaSets, the commonly used version is:

```
apiVersion: apps/v1
```

This tells Kubernetes that the resource belongs to the **apps API group**.

---

### ⚡ kind

The **kind** field defines the type of resource you want to create.

```
kind: ReplicaSet
```

This tells Kubernetes that the configuration will create a ReplicaSet.

---

### ⚡ metadata

The **metadata** section contains identifying information about the ReplicaSet.

Example fields include:

* **name** → Name of the ReplicaSet
* **labels** → Key-value pairs used to organize and identify resources

Example:

```
metadata:
  name: web-replicaset
  labels:
    app: web
```

Labels help Kubernetes group related resources together.

---

### ⚡ spec

The **spec** section defines how the ReplicaSet should behave.

---

### ⚡ replicas

The **replicas** field specifies how many Pods should run at the same time.

Example:

```
replicas: 3
```

This means Kubernetes will maintain **3 identical Pods**.

---

### ⚡ selector

The **selector** tells the ReplicaSet which Pods it should manage. It uses labels to identify them.

Example:

```
selector:
  matchLabels:
    project: kube-web
```

This means the ReplicaSet will manage Pods that contain the label **project: kube-web**.

---

### ⚡ template

The **template** defines the Pod configuration that the ReplicaSet will use to create Pods. It includes metadata and spec for the Pods.

---

### ⚡ template → metadata

Inside the template, metadata defines labels for the Pods. These labels must match the selector labels.

Example:

```
template:
  metadata:
    labels:
      project: kube-web
```

---

### ⚡ template → spec → containers

This section defines the containers that will run inside the Pod.

Example fields include:

* **name** → Name of the container
* **image** → Container image to run
* **ports** → Port exposed by the container

Example:

```
spec:
  containers:
    - name: kube-web-container
      image: nginx
      ports:
        - containerPort: 80
```

---

### ⚡ Example ReplicaSet YAML

```
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: kube-web-rs
  labels:
    app: kube-web
spec:
  replicas: 3
  selector:
    matchLabels:
      project: kube-web
  template:
    metadata:
      labels:
        project: kube-web
    spec:
      containers:
        - name: kube-web-container
          image: nginx
          ports:
            - containerPort: 80
```

This configuration tells Kubernetes to create a ReplicaSet that maintains **3 Pods**, each running the **nginx container**, and manages Pods with the label **project: kube-web**.

![demo](../assets/demo007.png)

## ⭐ How ReplicaSet Works (Inside Controller Manager)

In Kubernetes, the **ReplicaSet controller** runs inside the **Controller Manager**. Its main responsibility is to ensure that the number of running Pods always matches the desired state defined in the ReplicaSet YAML file.

When a ReplicaSet is created, Kubernetes reads the **replicaset.yml** configuration. Inside the `spec` section, the ReplicaSet controller checks the desired number of Pods.

For example, if the YAML file contains:

```yaml
spec:
  replicas: 3
```

This means the **desired state** is to run **3 Pods**.

---

### ⚡ Finding Which Pods Belong to the ReplicaSet

In a Kubernetes cluster there may be many Pods running. The ReplicaSet must determine which Pods belong to it. To identify its Pods, the ReplicaSet uses **labels and selectors**.

In the YAML file, the selector is defined like this:

```yaml
selector:
  matchLabels:
    app: kube-web-app
```

This means the ReplicaSet will manage Pods that have the label:

```
app: kube-web-app
```

The ReplicaSet controller sends this label information to the **API Server** and asks whether any Pods exist with this label.

---

### ⚡ Checking the Actual State

The API Server communicates with the Worker Nodes through the kubelet and checks whether any Pods with the label **app: kube-web-app** are running.

If kubelet finds no Pods with this label, it reports the **actual state as 0 Pods** back to the API Server.

The API Server then sends this information to the ReplicaSet controller.

---

### ⚡ Comparing Desired State and Actual State

Now the ReplicaSet controller compares the values:

* Desired state = 3 Pods
* Actual state = 0 Pods

The controller calculates:

```
3 - 0 = 3
```

This means **3 Pods need to be created**.

The ReplicaSet controller then informs the **API Server** that three Pods must be created.

---

### ⚡ Using the Pod Template

The API Server asks how the Pods should be created and which container image should be used. The ReplicaSet controller provides the **Pod template** defined in the YAML file.

Example template:

```yaml
template:
  metadata:
    labels:
      app: kube-web-app
  spec:
    containers:
      - name: kube-web
        image: itisameerkhan/kube-web-backend:v1
        ports:
          - containerPort: 8080
```

This template defines the container name, image, and port configuration.

---

### ⚡ Pod Creation

The API Server sends this template to the **kubelet** running on Worker Nodes. The kubelet then communicates with the container runtime and creates the Pods using the provided container image.

As a result, **3 Pods are created and started**.

---

### ⚡ Continuous Monitoring

The ReplicaSet controller never stops monitoring the cluster. It continuously checks the desired state and the actual state.

If one Pod crashes and only **2 Pods remain**, the ReplicaSet controller again detects the difference and creates a new Pod to maintain **3 Pods**.

This continuous monitoring ensures that the application always runs with the correct number of Pods.

---

## ⭐ Self-Healing in Kubernetes

Self-healing is a core feature of Kubernetes that ensures applications continue running even when containers or Pods fail. Kubernetes constantly monitors the system and automatically replaces failed components so that the desired state defined in the configuration is always maintained.

The self-healing process mainly involves the **ReplicaSet controller, API Server, kubelet, and container runtime** working together.

---

### ⚡ Detecting a Pod Failure

Suppose a ReplicaSet configuration specifies that **3 Pods should always be running**. This desired state is stored in **etcd** and managed by the ReplicaSet controller inside the Controller Manager.

If one Pod crashes or gets deleted, the kubelet on the Worker Node detects that the container is no longer running. The kubelet reports this status to the **API Server**, which updates the cluster state.

---

### ⚡ Comparing Desired State and Actual State

The ReplicaSet controller continuously checks the cluster state through the API Server.

It compares:

* **Desired state** → 3 Pods should be running
* **Actual state** → Only 2 Pods are running

The controller detects that one Pod is missing.

---

### ⚡ Creating a New Pod

After detecting the difference, the ReplicaSet controller instructs the API Server to create a new Pod to restore the desired state.

The ReplicaSet provides the **Pod template** defined in the YAML file, which includes the container image, container name, and port configuration.

The API Server sends this information to the **kubelet** on a Worker Node.

---

### ⚡ Starting the Container

The kubelet receives the Pod specification and instructs the **container runtime** to create and start a new container using the specified image.

Once the container starts successfully, the Pod becomes active and the cluster returns to the desired state of **3 running Pods**.

---

### ⚡ Continuous Monitoring

Kubernetes continuously repeats this process. If a Pod crashes again or a node fails, Kubernetes automatically creates new Pods to replace the failed ones.

This automatic detection and recovery mechanism is known as **self-healing**, which helps maintain application availability without manual intervention.

---

## ⭐ Creating a ReplicaSet

```yml
# pod.yml
apiVersion: v1
kind: Pod
metadata: 
  name: kube-web
  labels: 
    app: kube-web-app
spec: 
  containers:
    - name: kube-web
      image: itisameerkhan/kube-web-backend:v1
      ports:
        - containerPort: 8080
```

```yml
# replica-set.yml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: kube-web-rs
  labels:
    app: kube-web-rs-l
spec:
  replicas: 3
  selector: 
    matchLabels:
      app: kube-web-app
  template: 
    metadata:
      labels:
        app: kube-web-app
    spec: 
      containers: 
        - name: kube-web
          image: itisameerkhan/kube-web-backend:v1
          ports:
            - containerPort: 8080
```


### ⚡ Check replica set

```bash
kubectl get rs
```

```cmd
NAME          DESIRED   CURRENT   READY   AGE
kube-web-rs   3         3         3       16s
```

### ⚡Checking the Pods 

```bash
kubectl get pods
``` 

```bash
NAME                READY   STATUS    RESTARTS   AGE
kube-web            1/1     Running   0          52m
kube-web-rs-ccz92   1/1     Running   0          2m47s
kube-web-rs-d89j8   1/1     Running   0          2m47s
```

### ⚡ Deleting one pod

```bash
kubectl delete pod kube-web-rs-ccz92
```

```cmd
pod "kube-web-rs-ccz92" deleted from default namespace
```

### ⚡ Checking the pods

```bash
kubectl get pods
```

```cmd
NAME                READY   STATUS    RESTARTS   AGE
kube-web            1/1     Running   0          54m
kube-web-rs-d89j8   1/1     Running   0          4m34s
kube-web-rs-x8s5b   1/1     Running   0          43s
```

### ⚡ Deleting all 3 pods 

```cmd
kubectl delete pod kube-web kube-web-rs-d89j8 kube-web-rs-x8s5b
```
### ⚡ Checking the pods 

```bash
kubectl get pods
```

```
NAME                READY   STATUS    RESTARTS   AGE
kube-web-rs-khkqk   1/1     Running   0          2m8s
kube-web-rs-n56x5   1/1     Running   0          2m8s
kube-web-rs-zs8sl   1/1     Running   0          2m8s
```

### ⚡ Get ReplicaSet

```bash
kubectl get rs
```

```
NAME          DESIRED   CURRENT   READY   AGE
kube-web-rs   3         3         3       10m
```

### ⚡ Get FullDetails about one ReplicaSet

```bash
kubectl describe rs kube-web-rs
```

```
Name:         kube-web-rs
Namespace:    default
Selector:     app=kube-web-app
Labels:       app=kube-web-rs-l
Annotations:  <none>
Replicas:     3 current / 3 desired
Pods Status:  3 Running / 0 Waiting / 0 Succeeded / 0 Failed
Pod Template:
  Labels:  app=kube-web-app
  Containers:
   kube-web:
    Image:         itisameerkhan/kube-web-backend:v1
    Port:          8080/TCP
    Host Port:     0/TCP
    Environment:   <none>
    Mounts:        <none>
  Volumes:         <none>
  Node-Selectors:  <none>
  Tolerations:     <none>
Events:
  Type    Reason            Age    From                   Message
  ----    ------            ----   ----                   -------
  Normal  SuccessfulCreate  11m    replicaset-controller  Created pod: kube-web-rs-ccz92
  Normal  SuccessfulCreate  11m    replicaset-controller  Created pod: kube-web-rs-d89j8
  Normal  SuccessfulCreate  7m10s  replicaset-controller  Created pod: kube-web-rs-x8s5b
  Normal  SuccessfulCreate  4m     replicaset-controller  Created pod: kube-web-rs-zs8sl
  Normal  SuccessfulCreate  4m     replicaset-controller  Created pod: kube-web-rs-khkqk
  Normal  SuccessfulCreate  4m     replicaset-controller  Created pod: kube-web-rs-n56x5
```

### ⚡ Updating the replicas in ReplicaSet

```yml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: kube-web-rs
  labels:
    app: kube-web-rs-l
spec:
  replicas: 5
  selector: 
    matchLabels:
      app: kube-web-app
  template: 
    metadata:
      labels:
        app: kube-web-app
    spec: 
      containers: 
        - name: kube-web
          image: itisameerkhan/kube-web-backend:v1
          ports:
            - containerPort: 8080
```

```
kubectl apply -f replica-set.yml
```

#### output
 
```
replicaset.apps/kube-web-rs configured
```

```
kubectl describe rs kube-web-rs
```

```
Name:         kube-web-rs
Namespace:    default
Selector:     app=kube-web-app
Labels:       app=kube-web-rs-l
Annotations:  <none>
Replicas:     5 current / 5 desired
Pods Status:  5 Running / 0 Waiting / 0 Succeeded / 0 Failed
Pod Template:
  Labels:  app=kube-web-app
  Containers:
   kube-web:
    Image:         itisameerkhan/kube-web-backend:v1
    Port:          8080/TCP
    Host Port:     0/TCP
    Environment:   <none>
    Mounts:        <none>
  Volumes:         <none>
  Node-Selectors:  <none>
  Tolerations:     <none>
Events:
  Type    Reason            Age    From                   Message
  ----    ------            ----   ----                   -------
  Normal  SuccessfulCreate  14m    replicaset-controller  Created pod: kube-web-rs-ccz92
  Normal  SuccessfulCreate  14m    replicaset-controller  Created pod: kube-web-rs-d89j8
  Normal  SuccessfulCreate  11m    replicaset-controller  Created pod: kube-web-rs-x8s5b
  Normal  SuccessfulCreate  7m55s  replicaset-controller  Created pod: kube-web-rs-zs8sl
  Normal  SuccessfulCreate  7m55s  replicaset-controller  Created pod: kube-web-rs-khkqk
  Normal  SuccessfulCreate  7m55s  replicaset-controller  Created pod: kube-web-rs-n56x5
  Normal  SuccessfulCreate  47s    replicaset-controller  Created pod: kube-web-rs-sw4vm
  Normal  SuccessfulCreate  47s    replicaset-controller  Created pod: kube-web-rs-44jc8
```

```
NAME                READY   STATUS    RESTARTS   AGE
kube-web-rs-44jc8   1/1     Running   0          88s
kube-web-rs-khkqk   1/1     Running   0          8m36s
kube-web-rs-n56x5   1/1     Running   0          8m36s
kube-web-rs-sw4vm   1/1     Running   0          88s
kube-web-rs-zs8sl   1/1     Running   0          8m36s
```

### ⚡ Updating the replica without changing the file 

```
kubectl scale rs kube-web-rs --replicas=7
```

#### output
```
replicaset.apps/kube-web-rs scaled
```

```
kubectl get pods
```

```
NAME                READY   STATUS    RESTARTS   AGE
kube-web-rs-259ld   1/1     Running   0          29s
kube-web-rs-44jc8   1/1     Running   0          3m27s
kube-web-rs-78lmj   1/1     Running   0          29s
kube-web-rs-khkqk   1/1     Running   0          10m
kube-web-rs-kj9f8   1/1     Running   0          29s
kube-web-rs-n56x5   1/1     Running   0          10m
kube-web-rs-sw4vm   1/1     Running   0          3m27s
kube-web-rs-vfdtd   1/1     Running   0          29s
kube-web-rs-xtqxm   1/1     Running   0          29s
kube-web-rs-zs8sl   1/1     Running   0          10m
```

```
kubectl describe rs kube-web-rs
```

```
NAME                READY   STATUS    RESTARTS   AGE
kube-web-rs-259ld   1/1     Running   0          29s
kube-web-rs-44jc8   1/1     Running   0          3m27s
kube-web-rs-78lmj   1/1     Running   0          29s
kube-web-rs-khkqk   1/1     Running   0          10m
kube-web-rs-kj9f8   1/1     Running   0          29s
kube-web-rs-n56x5   1/1     Running   0          10m
kube-web-rs-sw4vm   1/1     Running   0          3m27s
kube-web-rs-vfdtd   1/1     Running   0          29s
kube-web-rs-xtqxm   1/1     Running   0          29s
kube-web-rs-zs8sl   1/1     Running   0          10m
PS C:\Users\itisa\OneDrive\Desktop\kubernetes-demo> kubectl describe rs kube-web-rs
Name:         kube-web-rs
Namespace:    default
Selector:     app=kube-web-app
Labels:       app=kube-web-rs-l
Annotations:  <none>
Replicas:     10 current / 10 desired
Pods Status:  10 Running / 0 Waiting / 0 Succeeded / 0 Failed
Pod Template:
  Labels:  app=kube-web-app
  Containers:
   kube-web:
    Image:         itisameerkhan/kube-web-backend:v1
    Port:          8080/TCP
    Host Port:     0/TCP
    Environment:   <none>
    Mounts:        <none>
  Volumes:         <none>
  Node-Selectors:  <none>
  Tolerations:     <none>
Events:
  Type    Reason            Age                From                   Message
  ----    ------            ----               ----                   -------
  Normal  SuccessfulCreate  17m                replicaset-controller  Created pod: kube-web-rs-ccz92
  Normal  SuccessfulCreate  17m                replicaset-controller  Created pod: kube-web-rs-d89j8
  Normal  SuccessfulCreate  14m                replicaset-controller  Created pod: kube-web-rs-x8s5b
  Normal  SuccessfulCreate  10m                replicaset-controller  Created pod: kube-web-rs-zs8sl
  Normal  SuccessfulCreate  10m                replicaset-controller  Created pod: kube-web-rs-khkqk
  Normal  SuccessfulCreate  10m                replicaset-controller  Created pod: kube-web-rs-n56x5
  Normal  SuccessfulCreate  3m47s              replicaset-controller  Created pod: kube-web-rs-sw4vm
  Normal  SuccessfulCreate  3m47s              replicaset-controller  Created pod: kube-web-rs-44jc8
  Normal  SuccessfulCreate  49s                replicaset-controller  Created pod: kube-web-rs-78lmj
  Normal  SuccessfulCreate  49s (x4 over 49s)  replicaset-controller  (combined from similar events): Created pod: kube-web-rs-kj9f8
```

## ⭐ Use of ReplicaSet

A ReplicaSet in Kubernetes is used to ensure that a specified number of identical Pods are always running in the cluster. Its main purpose is to maintain the desired number of Pod replicas and automatically replace any Pods that fail or are deleted.

When you define a ReplicaSet, you specify the desired number of Pods using the **replicas** field in the YAML configuration. The ReplicaSet controller constantly checks the cluster and compares the **desired state** with the **actual state**.

For example, if the ReplicaSet configuration says that **3 Pods should be running**, Kubernetes continuously monitors the cluster. If one Pod crashes or gets deleted and only **2 Pods remain**, the ReplicaSet automatically creates a new Pod to bring the total back to **3**. This helps maintain application availability.

ReplicaSet also uses **labels and selectors** to identify which Pods it should manage. It looks for Pods that match the specified labels and ensures that the required number of those Pods are always running.

---

## ⭐ Increasing or Decreasing Replica Count in a ReplicaSet

In Kubernetes, the number of Pods managed by a ReplicaSet can be changed by modifying the **replica count**. This process is called **scaling**. Scaling can be done in two ways: **Imperative** and **Declarative**.

---

### ⚡ Imperative Method

In the imperative approach, you directly give a command to Kubernetes to change the number of replicas. This method is quick and is usually used from the command line.

For example, if you want to increase the number of replicas to **5**, you can run:

```bash
kubectl scale replicaset kube-web-rs --replicas=5
```

If you want to reduce the number of replicas to **2**, you can run:

```bash
kubectl scale replicaset kube-web-rs --replicas=2
```

Kubernetes immediately updates the ReplicaSet and creates or removes Pods to match the new replica count.

---

### ⚡ Declarative Method

In the declarative approach, you modify the **ReplicaSet YAML file** and update the value of the `replicas` field.

Example:

```yaml
spec:
  replicas: 5
```

After modifying the file, apply the changes using:

```bash
kubectl apply -f replicaset.yaml
```

Kubernetes reads the updated configuration and adjusts the number of Pods to match the new desired state.

---

## ⭐ Role of Label Selectors in a ReplicaSet

Label selectors are used by a ReplicaSet to identify which Pods it should manage. In a Kubernetes cluster there can be many Pods running, so the ReplicaSet needs a way to determine which Pods belong to it. This is done using labels and selectors.

A label is a key-value pair attached to a Pod, while a label selector is used by the ReplicaSet to search for Pods that have matching labels. The ReplicaSet continuously checks the cluster and looks for Pods whose labels match the selector defined in the ReplicaSet configuration.

For example, if the ReplicaSet selector is defined as:

```yaml
selector:
  matchLabels:
    app: kube-web-app
```

The ReplicaSet will only manage Pods that contain the label:

```yaml
labels:
  app: kube-web-app
```

If a Pod with this label is deleted or crashes, the ReplicaSet detects that the number of Pods has decreased and automatically creates a new Pod to maintain the desired number of replicas.

In simple terms, label selectors allow the ReplicaSet to **find, monitor, and manage the correct set of Pods** inside the Kubernetes cluster.
