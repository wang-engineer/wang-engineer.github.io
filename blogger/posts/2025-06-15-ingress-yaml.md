---
layout: default
title: "2025-06-15: Understanding ingress.yaml in Kubernetes with an example"
tags: [kubernetes]
permalink: /blogger/posts/2025-06-15-ingress-yaml/
---

# Understanding `ingress.yaml` in Kubernetes with an example
> 💬 *“知人者智，自知者明。胜人者有力，自胜者强。(He who knows others is wise; he who knows himself is enlightened. He who conquers others is strong; he who conquers himself is truly powerful.)*  
> — Laozi

## 🧠 Introduction: Why Learn the ingress.yaml?

 In previous posts, you’ve [deployed your Pods with `deployment.yaml`](https://wang-engineer.github.io/blogger/posts/2025-06-07-deployment-yaml/) and made them accessible inside the Kubernetes cluster with [`service.yaml`](https://wang-engineer.github.io/blogger/posts/2025-06-10-service-yaml/). But now you’re wondering, *“How do I let the outside world—aka my users—reach this app?”* Fear not, friend! The `ingress.yaml` is your ticket to exposing your application to the internet in a slick, controlled way. It’s like opening a fancy front door to your Kubernetes house, complete with a doorman checking IDs.

The `ingress.yaml` defines an **Ingress** resource, which manages external HTTP/HTTPS traffic to your Services, typically by routing requests based on hostnames or paths. Think of it as a smart traffic cop directing web requests to the right Service (like the `my-app-service`). Unlike `NodePort` or `LoadBalancer` Services, which can be clunky or costly, Ingress is lightweight, flexible, and works with an Ingress Controller (like NGINX, which routes traffic, unlike the NGINX in our Pod that serves static webpages) to handle the heavy lifting.

Common newbie traps include:
- Thinking a Service of type `ClusterIP` is enough for public access (nope, not without Ingress).
- Assuming you always need a `LoadBalancer` — Ingress can give you more flexibility and save costs.
- Forgetting that Ingress needs a controller to actually work (spoiler alert: yes, you need one).

This post builds on our NGINX app example, showing you an `ingress.yaml` that exposes `my-app-service` externally via `example.com`. We’ll break it down with collapsible sections, keep the YAML indentation at 2 spaces (because YAML is picky), and make you feel like a Kubernetes networking pro by the end. Let’s dive in!

---

## 📄 Example: Full ingress.yaml

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-app-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
    - host: example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: my-app-service
                port:
                  number: 80
```

---

## 🔍 Explanation of Each Section (Expand for details)

<details>
<summary><strong>apiVersion: networking.k8s.io/v1</strong></summary>
<ul>
  <li>Specifies the API version for the Ingress resource.</li>
  <li>`networking.k8s.io/v1` is the stable API for Ingress since Kubernetes 1.19, replacing older `extensions/v1beta1`.</li>
  <li>Ensures compatibility with modern Kubernetes clusters.</li>
</ul>
</details>

<details>
<summary><strong>kind: Ingress</strong></summary>
<ul>
  <li>Declares this YAML as an Ingress resource.</li>
  <li>Ingress manages external HTTP/HTTPS traffic, routing it to Services based on rules like hostnames or paths.</li>
  <li>Requires an Ingress Controller (e.g., NGINX, Traefik) to process these rules.</li>
</ul>
</details>

<details>
<summary><strong>metadata</strong></summary>
<ul>
  <li>`name: my-app-ingress`: The unique name of the Ingress object within the namespace.</li>
  <li>`annotations`: Optional key-value pairs for configuring the Ingress Controller.</li>
  <li>`nginx.ingress.kubernetes.io/rewrite-target: /`: Tells the NGINX Ingress Controller to rewrite incoming URLs to `/`, useful for routing (e.g., stripping prefixes). Adjust based on your app’s needs.</li>
</ul>
</details>

<details>
<summary><strong>spec.ingressClassName</strong></summary>
<ul>
  <li>`ingressClassName: nginx`: Specifies which Ingress Controller should handle this Ingress.</li>
  <li>Matches the name of an `IngressClass` resource, ensuring the NGINX Ingress Controller (not another like Traefik) processes this.</li>
  <li>Think of it as telling Kubernetes, “Hey, NGINX, this one’s for you!”</li>
</ul>
</details>

<details>
<summary><strong>spec.rules</strong></summary>
<ul>
  <li>Defines routing rules for incoming HTTP traffic.</li>
  <li>`host: example.com`: Routes requests for `example.com` to the specified Service.</li>
  <li>If no `host` is set, the rule applies to all incoming requests (less secure, so use with caution).</li>
  <li>Multiple rules can handle different domains (e.g., `api.example.com`, `app.example.com`).</li>
</ul>
</details>

<details>
<summary><strong>spec.rules.http.paths</strong></summary>
<ul>
  <li>Specifies URL path-based routing for the given host.</li>
  <li>`path: /`: Matches requests starting with `/` (e.g., `example.com/`, `example.com/home`).</li>
  <li>`pathType: Prefix`: Indicates the path is a prefix match, so `/` catches all subpaths unless overridden by more specific rules.</li>
  <li>Other `pathType` options include `Exact` (exact URL match) or `ImplementationSpecific` (controller-dependent).</li>
  <li>`backend.service`: Routes matching requests to a Service.</li>
  <li>`name: my-app-service`: Targets the Service from our previous post.</li>
  <li>`port.number: 80`: Sends traffic to port 80 of `my-app-service`, matching its `port` field.</li>
</ul>
</details>

---

## 🧾 What This Ingress Actually Does

This `ingress.yaml` creates a Kubernetes **Ingress** resource named `my-app-ingress` that exposes the `my-app-service` (and thus our NGINX app) to the outside world via the domain `example.com`. It works with an NGINX Ingress Controller to route HTTP traffic from `example.com/` (and all subpaths) to the `my-app-service` Service on port 80, which then load-balances requests across the NGINX Pods.

Here’s the play-by-play:
- A user types `example.com` in their browser.
- The request hits the NGINX Ingress Controller’s external IP (set up via a cloud load balancer or similar).
- The Ingress checks its `rules`: if the request is for `example.com` and starts with `/`, it matches.
- The `rewrite-target` annotation ensures the URL is rewritten to `/` (handy if your app expects root-based paths).
- Traffic is forwarded to `my-app-service:80`, which distributes it to one of the NGINX Pods’ `targetPort: 80`.
- The NGINX Pod serves the default webpage (e.g., “Welcome to Nginx!”).
<!-- ![Workflow Diagram](../images/ingress-service.svg) -->
<div class="mermaid">
graph TD
    A[User Browser<br>example.com] -->|HTTP Request| LB[Load Balancer]
    LB -->|Distributes to| B[NGINX Ingress Controller<br>External IP]

    %% Kubernetes Cluster Subgraph
    subgraph K8sCluster[ ]
        B -->|Matches host: example.com| C[Ingress: my-app-ingress<br>path: /]
        C -->|Routes to| D[Service: my-app-service<br>port: 80]
        D -->|Load Balances| E[Pod 1<br>targetPort: 80]
        D -->|Load Balances| F[Pod 2<br>targetPort: 80]
        D -->|Load Balances| G[Pod 3<br>targetPort: 80]
        E -->|Serves| H[NGINX Webpage]
        F -->|Serves| H
        G -->|Serves| H
    end

    %% Styling nodes
    classDef nodeStyle fill:#69b5ce,stroke:#4287f5,stroke-width:2px,color:#090c0c;
    class A,LB,B,C,D,E,F,G,H nodeStyle;

    %% Styling edges
    linkStyle default stroke:#4287f5,stroke-width:2px;

    %% Styling subgraph border
    classDef clusterStyle fill:none,stroke:#4287f5,stroke-width:4px,stroke-dasharray:5,5;
    class K8sCluster clusterStyle;
</div>

<script type="module">
  import mermaid from 'https://cdn.jsdelivr.net/npm/mermaid@10/dist/mermaid.esm.min.mjs';
  mermaid.initialize({ startOnLoad: true });
</script>

The `ingressClassName: nginx` ensures the NGINX Ingress Controller handles this resource, avoiding confusion if multiple controllers are running. The `pathType: Prefix` makes the rule flexible, catching all URLs under `example.com/`, so `example.com/about` works too. This setup is perfect for public-facing web apps, as it’s secure (HTTPS can be added with TLS settings), scalable (the Ingress Controller and Service handle load balancing), and cost-efficient (no need for a `LoadBalancer` Service per app).

Note: For this to work in production, you need:
- An **Ingress Controller** (e.g., NGINX) installed in your cluster. Without it, the Ingress is just a fancy YAML file collecting digital dust.
- A **DNS record** mapping `example.com` to the Ingress Controller’s external IP.
- Optionally, a TLS certificate for HTTPS (via `spec.tls`), but we’re keeping it simple for now.

This Ingress makes your NGINX app accessible to anyone with the right URL, providing a professional-grade HTTP routing without the headache of managing raw IPs or load balancers manually. It’s like giving your app a shiny billboard on the internet highway!

---

## 🚀 Ingress Command

Before deploying, ensure you have an NGINX Ingress Controller installed. For a quick setup (e.g., in a test cluster):
```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/cloud/deploy.yaml
```

Apply the `ingress.yaml` file to your cluster:

```bash
kubectl apply -f ingress.yaml
```

Verify and troubleshoot the Ingress:

```bash
# Check the Ingress resource
kubectl get ingress

# View detailed information about the Ingress
kubectl describe ingress my-app-ingress

# Get the Ingress Controller’s external IP (look for LoadBalancer IP)
kubectl get -n ingress-nginx service/ingress-nginx-controller

# Test access (replace <EXTERNAL-IP> with the Ingress Controller’s IP or your domain)
# Or you can just open the url in a browser
curl http://example.com/

# For local testing, use port-forward to access the Service behind the Ingress
kubectl port-forward service/my-app-service 8080:80
# Then access http://localhost:8080 in your browser
```

For official Ingress documentation, visit the [Kubernetes Ingress page](https://kubernetes.io/docs/concepts/services-networking/ingress/).
