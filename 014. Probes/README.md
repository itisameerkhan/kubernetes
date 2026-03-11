## ⭐ What is a Probe in Kubernetes?

A Probe in Kubernetes is a mechanism used by Kubernetes to check the health and status of containers running inside a Pod. It helps Kubernetes determine whether an application is running correctly, whether it is ready to receive traffic, or whether it needs to be restarted.

Kubernetes periodically performs these checks on containers. If the probe detects a problem, Kubernetes can take automatic actions such as restarting the container or stopping traffic to it.

Probes help Kubernetes maintain high availability and reliability of applications.

| Probe           | Purpose                                           |
| --------------- | ------------------------------------------------- |
| Liveness Probe  | Checks if the container is alive                  |
| Readiness Probe | Checks if the container is ready to serve traffic |
| Startup Probe   | Checks if the application has finished starting   |

```yml
livenessProbe:
    httpGet: 
        path: /health
        port: 80
    initialDelaySeconds: 10
    periodSeconds: 5
    failureThreshold: 3
```

## ⭐ What is a Readiness Probe in Kubernetes?

A Readiness Probe in Kubernetes is used to check whether a container is ready to accept incoming traffic. Even if a container is running, it may not yet be ready to handle requests. The readiness probe ensures that traffic is sent only to pods that are fully prepared.

When the readiness probe fails, Kubernetes does not restart the container. Instead, it simply removes the pod from the service load balancer, so it temporarily stops receiving requests. Once the application becomes ready again and the probe succeeds, Kubernetes adds the pod back to the service and it starts receiving traffic again.

This mechanism helps prevent users from sending requests to an application that is still starting or temporarily unavailable.

```yml
readinessProbe:
    httpGet:
        path: /health
        port: 80
    initialDelaySeconds: 5
    periodSeconds: 5
```

## ⭐ What is a Startup Probe in Kubernetes?

A Startup Probe in Kubernetes is used to check whether an application inside a container has successfully started. Some applications take a long time to initialize, such as databases or applications that load large configurations. During this startup time, Kubernetes might think the container is unhealthy and restart it unnecessarily.

The Startup Probe prevents this problem by giving the application enough time to start before Kubernetes begins running other health checks like liveness and readiness probes.

Until the startup probe succeeds, Kubernetes will not run the liveness or readiness probes.

```yml
startupProbe: 
    httpGet:
        path: /health
        port: 80
    failureThreshold: 30
    periodSeconds: 5
```