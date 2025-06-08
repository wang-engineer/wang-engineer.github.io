---
layout: default
title: "2025-06-07: understanding deployment manifest in Kubernetes"
tags: [kubernetes, deployment]
permalink: /blogger/posts/2025-06-07-my-first-blog/
---

# Understanding `deployment.yaml` in Kubernetes
> 💬 *“If you can’t explain it simply, you don’t understand it well enough.”*  
> — Albert Einstein

## 🧠 Introduction: Why Learn the `deployment.yaml`?

If you’ve ever opened a `deployment.yaml` file and thought, *“Whoa, what’s going on here?”* — you’re not alone. YAML files in Kubernetes can feel cryptic, especially when you're just getting started with DevOps, MLOps, or deploying machine learning services into production. Even experienced software or ML engineers might glance at a line like `emptyDir: {}` or `livenessProbe:` and think, *“Is this some kind of magic incantation?”*

The truth is: this file is **a blueprint** that tells Kubernetes how to run your application — how many copies (replicas), how much memory or CPU it needs, what to do if something breaks, and even how to keep it connected to the rest of the world. It’s powerful, but because it does so much, it can feel overwhelming.

For example, many people assume:
- That `name: my-app` under `metadata` names the **Pod** — but it actually names the **Deployment**.
- That `resources.limits` is just a suggestion — when in fact, going over that memory limit can get your container **killed**.
- Or that `readinessProbe` is just another way to check if the app is alive — but it’s actually about whether the app is ready to **serve real traffic** (very different from being “just alive”).

This guide breaks down a real-world `deployment.yaml` file into easy-to-understand parts using collapsible sections, so you can explore them at your own pace. Whether you're deploying an NGINX web server, a Python FastAPI app, or a machine learning model behind a REST API, understanding this file gives you control — and confidence — when pushing code into Kubernetes.

So let’s walk through each part of the file and demystify it, line by line. By the end, the only YAML-induced anxiety you’ll have is trying to remember whether it’s 2 spaces or 4. *(It’s 2. Always 2.)*

---

## 📄 Example: Full `deployment.yaml`

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

## 🔍 Explanation of Each Section

<details>
<summary><strong>apiVersion: apps/v1</strong></summary>
<ul>
  <li>Specifies the API version of the Kubernetes resource.</li>
  <li><code>apps/v1</code> is the stable version for Deployments.</li>
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
  <li><code>name</code>: The name of the Deployment object.</li>
  <li>Used for identification within the namespace. This name must be unique within the same namespace and is DNS-compliant (lowercase, numbers, and dashes allowed)</li>
</ul>
</details>

<details>
<summary><strong>spec.replicas</strong></summary>
<ul>
  <li>The number of Pods to run at any given time.</li>
  <li><code>replicas: 3</code>: Tells Kubernetes to maintain 3 replicas (copies) of the Pod at all times. If one Pod crashes or is deleted, Kubernetes automatically creates a new one.</li>
  <li>This is useful for high availability, load balancing, and fault tolerance</li>
</ul>
</details>

<details>
<summary><strong>spec.selector</strong></summary>
<ul>
  <li>Defines how the Deployment finds which Pods to manage.</li>
  <li><code>selector.matchLabels</code>: It matches and manage Pods with the label <code>app: my-app</code>.</li>
  <li>This selector must match the labels in the Pod template below — otherwise the Deployment won’t manage its own Pods.</li>
</ul>
</details>

<details>
<summary><strong>spec.template.metadata.labels</strong></summary>
<ul>
  <li><code>template</code>: This defines the template for creating Pods — the Pod specification that the Deployment will replicate.</li>
  <li><code>Labels</code>: Labels to assign to Pods created by this Deployment.</li>
  <li>Under template.metadata.labels, you define the labels assigned to Pods that are created. These labels must match the <code>selector.matchLabels</code> above.</li>
</ul>
</details>

<details>
<summary><strong>spec.template.spec.containers</strong></summary>
<ul>
  <li>Defines the container(s) in the Pod. This is a list of containers to run inside the Pod. </li>
  <li>A Pod can have one or more containers, though single-container Pods are more common. </li>
  <li><code>name: my-app-container</code>: Logical name for the container. It is used for referencing the container in logging or debugging. Remember: this is not the name of the Pod or Deployment — just the container</li>
  <li><code>image: nginx:1.25</code>: This tells Kubernetes to pull the container image nginx:1.25 from the Docker Hub (by default).</li>
  <li><code>ports</code>: This defines the ports exposed by the container, i.e., ports the application listens on internally.</li>
  <li><code>containerPort: 80</code>: In this case, NGINX is configured to serve HTTP traffic on port 80.</li>
  <li>To make it available externally, you'd define a <code>Service</code> object which maps an external port to this <code>containerPort</code>.</li>

</ul>
</details>

<details>
<summary><strong>spec.template.spec.containers.resources</strong></summary>
<ul>
  <li>Resource management: This block is part of the container configuration and tells Kubernetes how to allocate and enforce compute resources (CPU and memory) for the container</li>
  <li><strong>requests</strong>: Minimum resources the container is guaranteed. Reserve these resources for the container even if it's not using them fully at the moment</li>
  <li><code>cpu: "250m"</code>: "250m" means 250 millicores = 0.25 of a single CPU core. If a node has 4 cores, Kubernetes can fit up to 16 such containers if no other usage.</li>
  <li><code>memory: "64Mi"</code>: Kubernetes ensures the node has at least 64Mi available before scheduling the container.</li>
  <li><strong>limits</strong>: Maximum resources the container is allowed to use.</li>
  <li><code>cpu: "500m"</code>: The container can use up to 0.5 CPU cores. If it tries to exceed that, the kernel throttles the CPU usage.</li>
  <li><code>memory: "128Mi"</code>: If the container tries to use more than 128Mi, it is killed with an OOM (Out Of Memory) error.</li>
</ul>
</details>

<details>
<summary><strong>spec.template.spec.containers.livenessProbe</strong></summary>
<ul>
  <li>Tells Kubernetes how to check if the app is still running.</li>
  <li>If this probe fails repeatedly, the Pod is restarted.</li>
  <li><code>httpGet</code>: Use an HTTP GET request as the probe method.</li>
  <li><code>path: /</code>: Perform the GET request on the root path (/). You can customize this for /healthz, /status, etc.</li>
  <li><code>port: 80</code>: Use port 80 inside the container for the check.</li>
  <li><code>initialDelaySeconds: 15</code>: Wait 15 seconds after the container starts before beginning checks (gives the app time to start).</li>
  <li><code>periodSeconds: 20</code>: After the first check, perform this probe every 20 seconds.</li>
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
  <li><code>volumeMounts</code>: Defines where in the container the volume is mounted.</li>
  <li><code>name: html-volume</code>: Refers to the Pod-level volume defined under volumes:. The names must match exactly.</li>
  <li><code>mountPath: /usr/share/nginx/html</code>: This is the target path inside the container.</li>
</ul>
</details>

<details>
<summary><strong>spec.template.spec.volumes</strong></summary>
<ul>
  <li><code>volumes</code>: Defines the actual volume resource (e.g., <code>emptyDir</code>, <code>configMap</code>, etc.). It is a list of named volumes that the Pod can use.</li>
  <li><code>name: html-volume</code>: This is the name of the volume. It must match what's used in the container’s volumeMounts</li>
  <li><code>emptyDir: {}</code>: This tells Kubernetes to use an emptyDir volume — a built-in ephemeral volume type.</li>
</ul>
</details>

---

## ✅ Deployment Command
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

