## ⭐ Problem When Updating Application Using ReplicaSet

Suppose you have a calculator application running in Kubernetes with **3 Pods**, and all of them are running **version v1** of the application. Users access the calculator through these Pods, and everything works normally.

| Pod   | Version |
| ----- | ------- |
| pod-1 | v1      |
| pod-2 | v1      |
| pod-3 | v1      |

Now imagine you want to update the application to **version v2**.

---

### ⚡ Updating Using ReplicaSet

If the application is managed using only a **ReplicaSet**, updating the application version is not straightforward. To update the application, you would typically need to delete the existing ReplicaSet that runs version v1 and then create a new ReplicaSet that runs version v2.

The steps would look like this:

* Delete the ReplicaSet running **version v1**
* Create a new ReplicaSet running **version v2**

---

### ⚡ What Happens During This Update

When the ReplicaSet running version v1 is deleted, Kubernetes removes all the Pods created by that ReplicaSet.

| Pod   | Version |
| ----- | ------- |
| pod-1 | Deleted |
| pod-2 | Deleted |
| pod-3 | Deleted |

At this moment, no Pods are running in the cluster.

Because the application is no longer running, users trying to access the calculator will experience **downtime**.

After that, when the new ReplicaSet for version v2 is created, Kubernetes starts new Pods.

| Pod   | Version |
| ----- | ------- |
| pod-1 | v2      |
| pod-2 | v2      |
| pod-3 | v2      |

Once these Pods start running, the application becomes available again.

---

### ⚡ Problem with This Approach

This update process creates a temporary period where the application is not running.

* All old Pods are deleted first
* New Pods are created afterward
* During this time the application is unavailable

This results in **downtime for users**.

---

## ⭐ Deployment in Kubernetes

A Deployment in Kubernetes is a resource used to manage the lifecycle of applications running in Pods. It allows you to define how many Pods should run, which container image should be used, and how updates to the application should be performed.

Instead of directly creating Pods or ReplicaSets, developers usually create a Deployment. The Deployment automatically creates and manages a ReplicaSet, and the ReplicaSet manages the Pods. This layered structure makes application management easier and more reliable.

## ⭐ Rolling Update Strategy in Kubernetes

Rolling Update is a deployment strategy in Kubernetes used to update an application to a new version without stopping the service. Instead of deleting all old Pods and creating new ones at once, Kubernetes gradually replaces the old Pods with new Pods running the updated version. This ensures that the application remains available to users during the update process.

When a new version of an application is deployed, Kubernetes starts creating new Pods with the updated image while slowly terminating the old Pods. Because some Pods remain active throughout the process, users can continue accessing the application without experiencing downtime.

![demo](../assets/demo001.gif)

## ⭐ History Storage in Kubernetes Deployment

In Kubernetes, a Deployment keeps the history of previous versions of an application using ReplicaSets. Every time the application is updated with a new version, Kubernetes creates a new ReplicaSet while keeping the old ReplicaSets stored as part of the deployment history.

This allows Kubernetes to roll back to a previous version if the new version fails or causes problems.

![demo](../assets/demo002.gif)

### ⚡ Creating a ReplicaSet 

```yml
apiVersion: apps/v1
kind: ReplicaSet 
metadata:
  name: node-backend-rs
  labels:
    app: node-backend-rs-label
spec: 
  replicas: 3
  selector: 
    matchLabels:
      app: kube-web-backend
  template:
    metadata:
      labels:
        app: kube-web-backend
    spec: 
      containers:
        - name: kube-web-backend-container
          image: itisameerkhan/kube-web-backend:v1
          ports:
            - containerPort: 8080


---
apiVersion: v1
kind: Service 
metadata:
  name: node-backend-service 
  labels:
    app: node-backend-service-labels
spec: 
  type: NodePort
  selector: 
    app: kube-web-backend
  ports: 
    - protocol: TCP
      port: 80
      targetPort: 8080
      nodePort: 30080
```

![demo](../assets/demo015.png)


```cmd
kubectl apply -f replicaSet.yml
```

```cmd
minikube service node-backend-service --url
```

#### watch pods 

```
kubectl get pods --watch
```

#### Delete the replica set 

```
kubectl delete rs node-backend-rs
```

### ⚡ Changing the image to v2

```yml
apiVersion: apps/v1
kind: ReplicaSet 
metadata:
  name: node-backend-rs
  labels:
    app: node-backend-rs-label
spec: 
  replicas: 3
  selector: 
    matchLabels:
      app: kube-web-backend
  template:
    metadata:
      labels:
        app: kube-web-backend
    spec: 
      containers:
        - name: kube-web-backend-container
          image: itisameerkhan/kube-web-backend:v2
          ports:
            - containerPort: 8080


---
apiVersion: v1
kind: Service 
metadata:
  name: node-backend-service 
  labels:
    app: node-backend-service-labels
spec: 
  type: NodePort
  selector: 
    app: kube-web-backend
  ports: 
    - protocol: TCP
      port: 80
      targetPort: 8080
      nodePort: 30080
```

---

## ⭐ creating a deployment 

![demo](../assets/demo016.png)

```yml
apiVersion: apps/v1
kind: Deployment 
metadata: 
  name: node-backend-deployment
  labels:
    app: node-backend-deployment-label
spec: 
  replicas: 3
  strategy: 
    type: RollingUpdate
  selector:
    matchLabels:
      app: kube-web-backend 
  template:
    metadata:
      labels:
        app: kube-web-backend
    spec: 
      containers:  
        - name: kube-web-backend-container
          image: itisameeerkhan/kube-web-backend:v1
          ports: 
            - containerPort: 8080

---
apiVersion: v1 
kind: Service 
metadata:
  name: node-backend-service 
  labels: 
    app: node-backend-service-label
spec: 
  type: NodePort
  selector: 
       app: kube-web-backend
  ports: 
    - protocol: TCP
      port: 80
      targetPort: 8080
      nodePort: 30080
```

```bash
# output
node-backend-deployment-6465674fc5-tsgl7   0/1     ContainerCreating        0          0s
node-backend-deployment-6465674fc5-rzg2g   0/1     ContainerCreating        0          0s
node-backend-deployment-6465674fc5-xcdvx   0/1     ContainerCreating        0          0s  
node-backend-deployment-6465674fc5-tsgl7   1/1     Running                  0          2s
node-backend-deployment-6465674fc5-xcdvx   1/1     Running                  0          2s
node-backend-deployment-6465674fc5-rzg2g   1/1     Running                  0          2s
```

```cmd
kubectl apply -f deployment.yml
```

#### Watch

```
kubectl get pods --watch
```

#### url from minikube

```
minikube service node-backend-service --url
```

### ⚡ Now im changing v1 to v2 

```yml
apiVersion: apps/v1
kind: Deployment 
metadata: 
  name: node-backend-deployment
  labels:
    app: node-backend-deployment-label
spec: 
  replicas: 3
  strategy: 
    type: RollingUpdate
  selector:
    matchLabels:
      app: kube-web-backend 
  template:
    metadata:
      labels:
        app: kube-web-backend
    spec: 
      containers:  
        - name: kube-web-backend-container
          image: itisameeerkhan/kube-web-backend:v2
          ports: 
            - containerPort: 8080

---
apiVersion: v1 
kind: Service 
metadata:
  name: node-backend-service 
  labels: 
    app: node-backend-service-label
spec: 
  type: NodePort
  selector: 
       app: kube-web-backend
  ports: 
    - protocol: TCP
      port: 80
      targetPort: 8080
      nodePort: 30080
```

### after updating the version run this command 

```
kubectl apply -f deployment.yml
```

#### output 

```
node-backend-deployment-6465674fc5-jjksc   0/1     Pending       0          0s
node-backend-deployment-6465674fc5-jjksc   0/1     Pending       0          0s
node-backend-deployment-6465674fc5-jjksc   0/1     ContainerCreating   0          0s
node-backend-deployment-6465674fc5-jjksc   1/1     Running             0          1s
node-backend-deployment-779bf76c94-tgz2c   1/1     Terminating         0          25s
node-backend-deployment-779bf76c94-tgz2c   1/1     Terminating         0          25s
node-backend-deployment-6465674fc5-f7h4g   0/1     Pending             0          0s
node-backend-deployment-6465674fc5-f7h4g   0/1     Pending             0          0s
node-backend-deployment-6465674fc5-f7h4g   0/1     ContainerCreating   0          0s
node-backend-deployment-6465674fc5-f7h4g   1/1     Running             0          2s
node-backend-deployment-779bf76c94-xpmdb   1/1     Terminating         0          23s
node-backend-deployment-6465674fc5-q8s58   0/1     Pending             0          0s       
node-backend-deployment-779bf76c94-xpmdb   1/1     Terminating         0          23s      
node-backend-deployment-6465674fc5-q8s58   0/1     Pending             0          0s
node-backend-deployment-6465674fc5-q8s58   0/1     ContainerCreating   0          0s
node-backend-deployment-6465674fc5-q8s58   1/1     Running             0          1s
node-backend-deployment-779bf76c94-57h5m   1/1     Terminating         0          27s
node-backend-deployment-779bf76c94-57h5m   1/1     Terminating         0          27s      
```

### ⚡ Checking the History

```cmd
kubectl rollout history deployment node-backend-deployment
```

### ⚡ Checking how much replica set created 

```cmd
kubectl get rs
```

```
NAME                                 DESIRED   CURRENT   READY   AGE
node-backend-deployment-6465674fc5   3         3         3       3m1s
node-backend-deployment-779bf76c94   0         0         0       3m38s
```

### ⚡ Rollback (one version back)

```cmd
kubectl rollout undo deployment node-backend-deployment
```

```
kubectl get rs
```

```
NAME                                 DESIRED   CURRENT   READY   AGE
node-backend-deployment-6465674fc5   0         0         0       2m21s
node-backend-deployment-779bf76c94   3         3         3       2m58s
```

