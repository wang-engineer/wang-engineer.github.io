---
layout: default
title: "2025-06-10: Understanding service manifest in Kubernetes with an example"
tags: [kubernetes]
permalink: /blogger/posts/2025-06-10-service-yaml/
---

# Understanding `service.yaml` in Kubernetes with an example
> 💬 *“上善若水。水善利万物而不争，处众人之所恶，故几于道 。(The highest good is like water. Water benefits all things without contention. It dwells in places that others disdain. Thus, it is close to the Dao (Way).)”*  
> — Laozi

## Introduction: Why Learn the service.yaml?

If you’ve ever deployed an application in Kubernetes using a `deployment.yaml` and wondered, *“How do I actually access this app?”* — you’re in the right place. The `service.yaml` is the key to making your application reachable, whether it’s for other services inside the cluster or, with additional configuration, for users outside the cluster. Without a Service, your Pods are like hidden treasures — running perfectly but inaccessible.

The `service.yaml` file in Kubernetes acts as a **networking abstraction**. It defines how your application is exposed, typically within the cluster or externally via additional resources like an Ingress. It’s the bridge between your Pods (managed by a Deployment) and the rest of the world. However, it’s easy to get confused by terms like `ClusterIP`, `NodePort`, or `LoadBalancer`, or wonder why your app isn’t reachable even though the Pods are running fine.

Common misconceptions include:
- Thinking a Service automatically makes your app accessible on the internet (it depends on the Service type!).
- Assuming `targetPort` and `port` are the same thing (they’re not — one’s for the Pod, the other’s for the Service).
- Believing a Service creates Pods (it doesn’t — it just routes traffic to them).

This guide breaks down a `service.yaml` file that complements the `deployment.yaml` from our previous post ([Understanding deployment.yaml in Kubernetes with an example](/blogger/posts/2025-06-07-deployment-yaml/)). We’ll use collapsible sections to explain each part clearly, focusing on a Service that exposes an NGINX application running on port 80 for internal cluster access. By the end, you’ll understand how to make your app accessible and why each line in the YAML matters. And yes, it’s still 2 spaces for indentation in YAML. Always 2.

---

## Example: Full service.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-app-service
spec:
  selector:
    app: my-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: ClusterIP
```

---

## Explanation of Each Section (Expand for details)

<details>
<summary><strong>apiVersion: v1</strong></summary>
<ul>
  <li>Specifies the API version of the Kubernetes resource.</li>
  <li>For Services, `v1` is the stable API version, as Services are a core Kubernetes resource.</li>
</ul>
</details>

<details>
<summary><strong>kind: Service</strong></summary>
<ul>
  <li>Declares that this YAML defines a Service resource.</li>
  <li>A Service is responsible for enabling network access to a set of Pods, typically managed by a Deployment.</li>
</ul>
</details>

<details>
<summary><strong>metadata</strong></summary>
<ul>
  <li>`name: my-app-service`: The name of the Service object, unique within the namespace.</li>
  <li>This name is DNS-compliant (lowercase, numbers, and dashes allowed) and is used to reference the Service in the cluster, e.g., for DNS resolution or routing.</li>
</ul>
</details>

<details>
<summary><strong>spec.selector</strong></summary>
<ul>
  <li>Defines which Pods this Service will route traffic to.</li>
  <li>`selector: app: my-app`: Matches Pods with the label `app: my-app`, which corresponds to the labels defined in the `deployment.yaml` (`spec.template.metadata.labels`).</li>
  <li>If the selector doesn’t match any Pods, the Service won’t route traffic anywhere, and you’ll get connection errors.</li>
</ul>
</details>

<details>
<summary><strong>spec.ports</strong></summary>
<ul>
  <li>Defines the port configuration for the Service, including how traffic is routed to the Pods.</li>
  <li>`protocol: TCP`: Specifies the protocol for traffic (TCP is default and most common; UDP is also supported).</li>
  <li>`port: 80`: The port the Service exposes within the cluster. Other services or clients in the cluster can access the Service at this port (e.g., via `my-app-service:80`).</li>
  <li>`targetPort: 80`: The port on the Pod where traffic is sent. This matches the `containerPort` defined in the `deployment.yaml` (port 80 for NGINX).</li>
</ul>
</details>

<details>
<summary><strong>spec.type</strong></summary>
<ul>
  <li>Defines the type of Service, which determines how it’s exposed.</li>
  <li>`type: ClusterIP`: The default Service type, which exposes the Service only within the cluster via a cluster-internal IP address. It’s not accessible externally unless paired with an Ingress or other external routing mechanism.</li>
  <li>Other common types include:
    <ul>
      <li>`NodePort`: Exposes the Service on a specific port of each node in the cluster.</li>
      <li>`LoadBalancer`: Exposes the Service via a cloud provider’s load balancer (e.g., AWS ELB, GCP Cloud Load Balancer).</li>
      <li>`ExternalName`: Maps the Service to an external DNS name without creating a proxy.</li>
    </ul>
  </li>
  <li>In this example, `ClusterIP` is chosen for internal cluster communication, ideal for services that don’t need direct external access but may be accessed by other applications within the cluster or via an Ingress for external traffic.</li>
</ul>
</details>

---

## What This Service Actually Does

This `service.yaml` creates a **Kubernetes Service** named `my-app-service` that makes the NGINX application (from the `deployment.yaml`) accessible within the Kubernetes cluster. It connects to the Pods managed by the `my-app` Deployment, identified by the label `app: my-app`. The Service ensures that traffic sent to it is distributed across the three replicas of the NGINX Pods, providing **load balancing** and fault tolerance. If one Pod fails, the Service automatically routes traffic to the remaining healthy Pods.

The Service is of type `ClusterIP`, meaning it is assigned a cluster-internal IP address and is only accessible within the Kubernetes cluster. Other services or Pods in the cluster can reach this Service by using its DNS name, `my-app-service`, on port `80` (e.g., `http://my-app-service:80`). The `ports` configuration maps the Service’s port `80` to the Pods’ `targetPort: 80`, ensuring traffic reaches the NGINX container’s HTTP server. The `protocol: TCP` confirms that the Service handles standard TCP traffic, suitable for HTTP.

Since `ClusterIP` Services are not exposed externally, you cannot access this Service directly from outside the cluster (e.g., from a browser). To enable external access, you would need additional resources, such as:
- An **Ingress** resource, which provides HTTP routing and can map a domain name (e.g., `example.com`) to this Service.
- A proxy like `kubectl port-forward` for testing, which forwards local traffic to the Service’s cluster IP.

This setup is ideal for internal microservices architectures, where applications need to communicate within the cluster without being exposed to the public internet. For example, this NGINX Service might serve static content for another internal application, such as a frontend or API service. In production, `ClusterIP` is the most common Service type, as it provides a stable internal endpoint and pairs well with Ingress for controlled external access.

In summary, this Service enables internal cluster access to the NGINX application, leveraging Kubernetes’ built-in load balancing to distribute traffic across the Deployment’s Pods. It’s a secure and efficient way to make your application available to other cluster components.

---

## Service Command

Once you’ve written and saved your `service.yaml` file, you can apply it to your Kubernetes cluster with the following command:

```bash
kubectl apply -f service.yaml
```

To verify and check the status of the Service:

```bash
# Check the status of the Service
kubectl get services

# View detailed information about the Service
kubectl describe service my-app-service

# Test access within the cluster (e.g., from another Pod)
kubectl run -it --rm --image=curlimages/curl curl -- curl http://my-app-service:80

# For local testing, use port-forward to access the Service
kubectl port-forward service/my-app-service 8080:80
# Then access http://localhost:8080 in your browser
```

For official instructions about Service YAML, go to the [Kubernetes official site](https://kubernetes.io/docs/concepts/services-networking/service/).