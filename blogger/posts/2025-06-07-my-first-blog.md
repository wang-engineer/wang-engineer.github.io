---
layout: default
title: "2025-06-07: understanding deployment manifest in Kubernetes"
tags: [kubernetes, deployment, machine-learning]
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

<div style="list-style-type: disc; margin: 1em 0; padding-left: 2em;">
<details>
<summary><strong>apiVersion: apps/v1</strong></summary>

- Specifies the API version of the Kubernetes resource.
- `apps/v1` is the stable version for Deployments.

</details>
</div>

<details>
<summary><strong>kind: Deployment</strong></summary>

- Declares that this YAML defines a Deployment resource.

</details>

<details>
<summary><strong>metadata</strong></summary>

- `name`: The name of the Deployment object.
- Used for identification within the namespace.

</details>

<details>
<summary><strong>spec.replicas</strong></summary>

- The number of Pods to run at any given time.

</details>

<details>
<summary><strong>spec.selector</strong></summary>

- Defines how the Deployment finds which Pods to manage.
- It matches Pods with the label `app: my-app`.

</details>

<details>
<summary><strong>template.metadata.labels</strong></summary>

- Labels to assign to Pods created by this Deployment.

</details>

<details>
<summary><strong>containers</strong></summary>

Defines the container(s) in the Pod.

- `name`: Logical name for the container.
- `image`: Docker image to use.
- `ports`: Exposed ports.

</details>

<details>
<summary><strong>resources</strong></summary>

Resource management:

- **requests**: Minimum resources the container is guaranteed.
- **limits**: Maximum resources the container can use.

</details>

<details>
<summary><strong>livenessProbe</strong></summary>

- Tells Kubernetes how to check if the app is still running.
- If this probe fails repeatedly, the Pod is restarted.

</details>

<details>
<summary><strong>readinessProbe</strong></summary>

- Determines if the app is ready to receive traffic.
- If it fails, the Pod is removed from service endpoints.

</details>

<details>
<summary><strong>volumeMounts and volumes</strong></summary>

- `volumeMounts` defines where in the container the volume is mounted.
- `volumes` defines the actual volume resource (e.g., `emptyDir`, `configMap`, etc.).

</details>

---

## ✅ Deployment Command

```bash
kubectl apply -f deployment.yaml
```
