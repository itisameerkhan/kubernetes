![demo](https://i.pinimg.com/originals/d7/36/99/d73699ac8cdeb9db3719848280a4be3a.gif)

## ⭐ Containers

Containers are a lightweight way to package and run applications along with all the dependencies they need. Instead of installing software directly on a server, the application and its environment are bundled together inside a container so that it runs consistently across different systems.

A container includes the application code, runtime, libraries, and system tools required for the application to work. Because everything the application needs is packaged together, the container behaves the same way in development, testing, and production environments.

Containers run on top of a container runtime such as Docker or containerd. Unlike virtual machines, containers share the host operating system kernel, which makes them faster to start and more efficient in terms of resource usage.

---

### ⚡ Why Containers Are Used

Containers solve many problems related to application deployment and environment differences.

* Applications run the same everywhere
* Faster startup compared to virtual machines
* Efficient use of system resources
* Easy to scale applications
* Simplifies deployment and updates

---

### ⚡ Containers vs Virtual Machines

| Feature        | Containers           | Virtual Machines            |
| -------------- | -------------------- | --------------------------- |
| Startup Time   | Seconds              | Minutes                     |
| Resource Usage | Lightweight          | Heavy                       |
| OS Requirement | Share host OS kernel | Each VM has its own OS      |
| Performance    | High                 | Lower due to virtualization |
| Isolation      | Process-level        | Full OS isolation           |

---

### ⚡ Role of Containers in Kubernetes

Kubernetes uses containers to run applications inside Pods. A Pod can contain one or more containers that share networking and storage. Kubernetes then manages these containers by scheduling them across nodes, scaling them when needed, and restarting them if they fail.

Containers are the fundamental unit that Kubernetes orchestrates to build scalable and reliable applications.
