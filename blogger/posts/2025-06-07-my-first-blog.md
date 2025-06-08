---
layout: default
title: "2025-06-07: understanding deployment manifest in Kubernetes"
tags: [kubernetes, deployment]
permalink: /blogger/posts/2025-06-07-my-first-blog/
---

# Understanding `deployment.yaml` in Kubernetes

This guide explains the key components of a sample Kubernetes Deployment YAML file using collapsible sections. This format is styled to look good on GitHub Pages.

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
  <li>Used for identification within the namespace.</li>
</ul>
</details>

<details>
<summary><strong>spec.replicas</strong></summary>
<ul>
  <li>The number of Pods to run at any given time.</li>
</ul>
</details>

<details>
<summary><strong>spec.selector</strong></summary>
<ul>
  <li>Defines how the Deployment finds which Pods to manage.</li>
  <li>It matches Pods with the label <code>app: my-app</code>.</li>
</ul>
</details>

<details>
<summary><strong>spec.template.metadata.labels</strong></summary>
<ul>
  <li>Labels to assign to Pods created by this Deployment.</li>
</ul>
</details>

<details>
<summary><strong>spec.template.spec.containers</strong></summary>
<ul>
  <li>Defines the container(s) in the Pod.</li>
  <li><code>name</code>: Logical name for the container.</li>
  <li><code>image</code>: Docker image to use.</li>
  <li><code>ports</code>: Exposed ports.</li>
</ul>
</details>

<details>
<summary><strong>spec.template.spec.containers.resources</strong></summary>
<ul>
  <li>Resource management:</li>
  <li><strong>requests</strong>: Minimum resources the container is guaranteed.</li>
  <li><strong>limits</strong>: Maximum resources the container can use.</li>
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
  <li>If it fails, the Pod is removed from service endpoints.</li>
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

```bash
kubectl apply -f deployment.yaml
```
