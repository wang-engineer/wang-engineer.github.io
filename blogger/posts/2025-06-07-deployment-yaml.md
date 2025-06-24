---
layout: default
title: "2025-06-07: understanding deployment manifest in Kubernetes with an example"
tags: [kubernetes, deployment]
permalink: /blogger/posts/2025-06-07-deployment-yaml/
---

# Understanding `deployment.yaml` in Kubernetes with an example
> 💬 *“大道甚夷，而人好径。(The great Dao is perfectly level, yet people love to take narrow, winding paths.)”*  
> — Laozi

## Introduction: Why Learn the `deployment.yaml`?

If you’ve ever opened a `deployment.yaml` file and thought, *“Whoa, what’s going on here?”* — you’re not alone. YAML files in Kubernetes can feel cryptic, especially when you're just getting started with DevOps, MLOps, or deploying machine learning services into production. Even experienced software or ML engineers might glance at a line like `emptyDir: {}` or `livenessProbe:` and think, *“Is this some kind of magic incantation?”*

The truth is: this file is **a blueprint** that tells Kubernetes how to run your application — how many copies (replicas), how much memory or CPU it needs, what to do if something breaks, and even how to keep it connected to the rest of the world. It’s powerful, but because it does so much, it can feel overwhelming.

For example, many people assume:
- That `name: my-app` under `metadata` names the **Pod** — but it actually names the **Deployment**.
- That `resources.limits` is just a suggestion — when in fact, going over that memory limit can get your container **killed**.
- Or that `readinessProbe` is just another way to check if the app is alive — but it’s actually about whether the app is ready to **serve real traffic** (very different from being “just alive”).

This guide breaks down a real-world `deployment.yaml` file into easy-to-understand parts using collapsible sections, so you can explore them at your own pace. Whether you're deploying an NGINX web server, a Python FastAPI app, or a machine learning model behind a REST API, understanding this file gives you control — and confidence — when pushing code into Kubernetes.

So let’s walk through each part of the file and demystify it, line by line. By the end, the only YAML-induced anxiety you’ll have is trying to remember whether it’s 2 spaces or 4. *(It’s 2. Always 2.)*

---

## Example: Full `deployment.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
        - name: my-app-container
          image: nginx:1.25
          ports:
            - containerPort: 80
          resources:
            requests:
              cpu: "250m"
              memory: "64Mi"
            limits:
              cpu: "500m"
              memory: "128Mi"
          livenessProbe:
            httpGet:
              path: /
              port: 80
            initialDelaySeconds: 15
            periodSeconds: 20
          readinessProbe:
            httpGet:
              path: /
              port: 80
            initialDelaySeconds: 5
            periodSeconds: 10
          volumeMounts:
            - name: html-volume
              mountPath: /usr/share/nginx/html
      volumes:
        - name: html-volume
          emptyDir: {}
```

---

## Explanation of Each Section (Expand for details)

<details>
<summary><strong>apiVersion: apps/v1</strong></summary>
<ul>
  <li>Specifies the API version of the Kubernetes resource.</li>
  <li>`apps/v1` is the stable version for Deployments.</li>
</ul>
</details>

<details>
<summary><strong>kind: Deployment</strong></summary>
<ul>
  <li>Declares that this YAML defines a Deployment resource.</li>
</ul>
</details>

<details>
<summary><strong>metadata</strong></summary>
<ul>
  <li>`name`: The name of the Deployment object.</li>
  <li>Used for identification within the namespace. This name must be unique within the same namespace and is DNS-compliant (lowercase, numbers, and dashes allowed)</li>
</ul>
</details>

<details>
<summary><strong>spec.replicas</strong></summary>
<ul>
  <li>The number of Pods to run at any given time.</li>
  <li>`replicas: 3`: Tells Kubernetes to maintain 3 replicas (copies) of the Pod at all times. If one Pod crashes or is deleted, Kubernetes automatically creates a new one.</li>
  <li>This is useful for high availability, load balancing, and fault tolerance</li>
</ul>
</details>

<details>
<summary><strong>spec.selector</strong></summary>
<ul>
  <li>Defines how the Deployment finds which Pods to manage.</li>
  <li>`selector.matchLabels`: It matches and manage Pods with the label `app: my-app`.</li>
  <li>This selector must match the labels in the Pod template below — otherwise the Deployment won’t manage its own Pods.</li>
</ul>
</details>

<details>
<summary><strong>spec.template.metadata.labels</strong></summary>
<ul>
  <li>`template`: This defines the template for creating Pods — the Pod specification that the Deployment will replicate.</li>
  <li>`Labels`: Labels to assign to Pods created by this Deployment.</li>
  <li>Under template.metadata.labels, you define the labels assigned to Pods that are created. These labels must match the `selector.matchLabels` above.</li>
</ul>
</details>

<details>
<summary><strong>spec.template.spec.containers</strong></summary>
<ul>
  <li>Defines the container(s) in the Pod. This is a list of containers to run inside the Pod. </li>
  <li>A Pod can have one or more containers, though single-container Pods are more common. </li>
  <li>`name: my-app-container`: Logical name for the container. It is used for referencing the container in logging or debugging. Remember: this is not the name of the Pod or Deployment — just the container</li>
  <li>`image: nginx:1.25`: This tells Kubernetes to pull the container image nginx:1.25 from the Docker Hub (by default).</li>
  <li>`ports`: This defines the ports exposed by the container, i.e., ports the application listens on internally.</li>
  <li>`containerPort: 80`: In this case, NGINX is configured to serve HTTP traffic on port 80.</li>
  <li>To make it available externally, you'd define a `Service` object which maps an external port to this `containerPort`.</li>

</ul>
</details>

<details>
<summary><strong>spec.template.spec.containers.resources</strong></summary>
<ul>
  <li>Resource management: This block is part of the container configuration and tells Kubernetes how to allocate and enforce compute resources (CPU and memory) for the container</li>
  <li><strong>requests</strong>: Minimum resources the container is guaranteed. Reserve these resources for the container even if it's not using them fully at the moment</li>
  <li>`cpu: "250m"`: "250m" means 250 millicores = 0.25 of a single CPU core. If a node has 4 cores, Kubernetes can fit up to 16 such containers if no other usage.</li>
  <li>`memory: "64Mi"`: Kubernetes ensures the node has at least 64Mi available before scheduling the container.</li>
  <li><strong>limits</strong>: Maximum resources the container is allowed to use.</li>
  <li>`cpu: "500m"`: The container can use up to 0.5 CPU cores. If it tries to exceed that, the kernel throttles the CPU usage.</li>
  <li>`memory: "128Mi"`: If the container tries to use more than 128Mi, it is killed with an OOM (Out Of Memory) error.</li>
</ul>
</details>

<details>
<summary><strong>spec.template.spec.containers.livenessProbe</strong></summary>
<ul>
  <li>Tells Kubernetes how to check if the app is still running.</li>
  <li>If this probe fails repeatedly, the Pod is restarted.</li>
  <li>`httpGet`: Use an HTTP GET request as the probe method.</li>
  <li>`path: /`: Perform the GET request on the root path (/). You can customize this for /healthz, /status, etc.</li>
  <li>`port: 80`: Use port 80 inside the container for the check.</li>
  <li>`initialDelaySeconds: 15`: Wait 15 seconds after the container starts before beginning checks (gives the app time to start).</li>
  <li>`periodSeconds: 20`: After the first check, perform this probe every 20 seconds.</li>
</ul>
</details>

<details>
<summary><strong>spec.template.spec.containers.readinessProbe</strong></summary>
<ul>
  <li>Determines if the app is ready to receive traffic.</li>
  <li>The livenessProbe is used to detect whether the application is stuck or dead. If the check fails, Kubernetes will restart the container.</li>
  <li>In contrast, the readinessProbe checks whether the application is ready to serve traffic. If this check fails, the container is temporarily removed from the load balancer and will not receive any traffic until it passes again.</li>
</ul>
</details>

<details>
<summary><strong>spec.template.spec.containers.volumeMounts</strong></summary>
<ul>
  <li>`volumeMounts`: Defines where in the container the volume is mounted.</li>
  <li>`name: html-volume`: Refers to the Pod-level volume defined under volumes:. The names must match exactly.</li>
  <li>`mountPath: /usr/share/nginx/html`: This is the target path inside the container.</li>
</ul>
</details>

<details>
<summary><strong>spec.template.spec.volumes</strong></summary>
<ul>
  <li>`volumes`: Defines the actual volume resource (e.g., `emptyDir`, `configMap`, etc.). It is a list of named volumes that the Pod can use.</li>
  <li>`name: html-volume`: This is the name of the volume. It must match what's used in the container’s `volumeMounts`</li>
  <li>`emptyDir: {}`: This tells Kubernetes to use an emptyDir volume — a built-in ephemeral volume type.</li>
</ul>
</details>

---

## What This Deployment Actually Does

Now that we’ve explored each section of the YAML file line by line, let’s take a step back and talk through what this whole file accomplishes — not in fragments, but as a story.

This deployment manifest creates a **Kubernetes Deployment** named `my-app`, which acts as a controller to manage a set of identical Pods. It specifies that Kubernetes should always keep **three replicas** of the application running. That means if one Pod fails, crashes, or is terminated (for example, due to a node failure), Kubernetes will immediately spin up a new one to maintain the desired state. This design ensures high availability and fault tolerance, which are essential for any production-grade application.

Inside each of these Pods is a single container based on the `nginx:1.25` image — a popular web server. This container listens on **port 80**, which is the standard port for HTTP traffic. If you were to expose this Deployment using a Kubernetes `Service`, traffic from outside the cluster could be routed to this port, allowing users to access the web server from their browsers or other clients.

The Deployment also defines **resource requests and limits** for the container. Each instance is guaranteed 250 millicores of CPU and 64Mi of memory, meaning the scheduler will only place it on a node with at least that much available. However, the container is not allowed to consume more than 500 millicores and 128Mi. If it exceeds the memory limit, Kubernetes will terminate it with an out-of-memory (OOM) error. These constraints prevent the container from consuming excessive resources, helping to maintain cluster stability — especially when multiple workloads share the same nodes.

<details>
<summary><strong>🩺 Liveness and Readiness Probes (click to expand)</strong></summary>

To monitor container health, the manifest includes two types of probes: `livenessProbe` and `readinessProbe`.

- The **liveness probe** checks whether the application is still running properly. It performs an HTTP GET request to the root path `/` on port 80. If the application fails this check repeatedly, Kubernetes assumes it's unhealthy and will restart the container automatically.

- The **readiness probe** also sends an HTTP GET to `/`, but its purpose is slightly different. It checks whether the application is ready to **serve user traffic**. If this probe fails, the container stays alive, but it is removed from the pool of endpoints behind any Kubernetes `Service`, meaning it won't receive traffic until it passes the check again.

This distinction between *liveness* (is it running?) and *readiness* (can it serve?) is crucial for smooth, resilient deployments.

</details>

Another important feature in this manifest is the **ephemeral volume**. Each container mounts a volume named `html-volume` at the path `/usr/share/nginx/html`, which is where NGINX typically serves static files. The backing volume is an `emptyDir`, meaning it's a temporary storage area that exists only for the lifetime of the Pod. It’s wiped clean if the Pod is deleted or restarted. This can be useful for caching, staging content dynamically, or running test workloads that don’t require long-term data persistence.

Altogether, this Deployment manifest represents a compact but powerful configuration that introduces many core Kubernetes concepts: desired state management, replication, resource allocation, container health checks, and volume mounting. Even though it deploys a simple web server, it lays the foundation for understanding how Kubernetes orchestrates and safeguards workloads in real-world environments.

---

## Deployment Command
Once you’ve written and saved your `deployment.yaml` file, you can apply it to your Kubernetes cluster with the following command:

```bash
kubectl apply -f deployment.yaml
```
To verify and check status of the deployment, pod, containers:

```bash
# Check the status of the Deployment
kubectl get deployments

# View the status of the Pods
kubectl get pods

# Describe the Deployment in detail
kubectl describe deployment my-app

# Stream logs from a container
kubectl logs -f <pod-name>
```
For official instructions about Deployment YAML, go to the [Kubernetes official site](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/).
