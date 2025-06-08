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
<summary><strong>template.metadata.labels</strong></summary>
<ul>
  <li>Labels to assign to Pods created by this Deployment.</li>
</ul>
</details>

<details>
<summary><strong>containers</strong></summary>
<ul>
  <li>Defines the container(s) in the Pod.</li>
  <li><code>name</code>: Logical name for the container.</li>
  <li><code>image</code>: Docker image to use.</li>
  <li><code>ports</code>: Exposed ports.</li>
</ul>
</details>

<details>
<summary><strong>resources</strong></summary>
<ul>
  <li>Resource management:</li>
  <li><strong>requests</strong>: Minimum resources the container is guaranteed.</li>
  <li><strong>limits</strong>: Maximum resources the container can use.</li>
</ul>
</details>

<details>
<summary><strong>livenessProbe</strong></summary>
<ul>
  <li>Tells Kubernetes how to check if the app is still running.</li>
  <li>If this probe fails repeatedly, the Pod is restarted.</li>
</ul>
</details>

<details>
<summary><strong>readinessProbe</strong></summary>
<ul>
  <li>Determines if the app is ready to receive traffic.</li>
  <li>If it fails, the Pod is removed from service endpoints.</li>
</ul>
</details>

<details>
<summary><strong>volumeMounts and volumes</strong></summary>
<ul>
  <li><code>volumeMounts</code>: Defines where in the container the volume is mounted.</li>
  <li><code>volumes</code>: Defines the actual volume resource (e.g., <code>emptyDir</code>, <code>configMap</code>, etc.).</li>
</ul>
</details>

---

## ✅ Deployment Command

```bash
kubectl apply -f deployment.yaml
```
