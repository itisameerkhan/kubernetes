## ⭐ Pod

A Pod is the smallest deployable unit in Kubernetes. It represents a running instance of an application inside the cluster. A Pod can contain **one or more containers**, and these containers work together as a single unit.

Each Pod is assigned a **unique IP address** inside the Kubernetes cluster. This IP address allows the Pod to communicate with other Pods and services in the cluster.

Pods also have their own **network and storage environment**. The containers inside the Pod share this environment, which means they use the same network and storage resources.

---

### ⚡ Containers Inside a Pod

A Pod can contain multiple containers that are closely related and need to work together. Since these containers run inside the same Pod, they **share the same network and storage**.

Because they share the same network namespace, containers inside the Pod can communicate with each other using **localhost** instead of separate IP addresses. This means they do not need individual IP addresses to talk to each other.

For example, if a Pod has two containers:

* One container runs the main application
* Another container runs a logging service

Both containers can communicate internally using localhost because they share the same network.

---

### ⚡ Pod-to-Pod Communication

When two different Pods need to communicate with each other, they must use the **Pod IP address** because each Pod has its own unique IP.

For example:

* Pod A has IP `10.0.1.5`
* Pod B has IP `10.0.1.8`

If Pod A needs to communicate with Pod B, it must use Pod B’s IP address.

In Kubernetes, multiple Pods can be created to run multiple instances of the same application. Each Pod will have its own unique IP address, but the containers inside the same Pod will always share the same network and storage environment.

![demo](../assets/demo006.png)