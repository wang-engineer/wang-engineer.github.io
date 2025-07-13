---
layout: default
title: "2025-07-12: Managing Configuration and Sensitive Data in Kubernetes with ConfigMaps and Secrets"
date: 2025-07-09
permalink: /blogger/posts/2025-07-12-configmap-secret/
tags: [Kubernetes]
---
# Managing Configuration and Sensitive Data in Kubernetes with ConfigMaps and Secrets

Welcome back to our Kubernetes journey! In [our previous post](https://wangmle.dev/blogger/posts/2025-06-07-deployment-yaml/), we explored the power of `Deployment` YAMLs to manage applications in Kubernetes. Now, let’s dive into two essential Kubernetes resources—**ConfigMaps** and **Secrets**—that help you manage configuration and sensitive data like a pro. Whether you're passing app settings or safeguarding database passwords, these tools are your best friends in keeping things organized, secure, and scalable. Let’s get started!

## Introduction: ConfigMaps and Secrets in Kubernetes

In Kubernetes, keeping your application’s configuration and sensitive data separate from your code is a best practice that boosts flexibility and security. That’s where **ConfigMaps** and **Secrets** come in.

- **ConfigMaps** are used to store non-sensitive configuration data, such as environment variables, command-line arguments, or configuration files. They allow you to decouple configuration from your application code, making it easier to tweak settings without rebuilding your Docker images.
- **Secrets** are designed for sensitive information, like passwords, API keys, or TLS certificates. They provide a secure way to store and distribute sensitive data to your containers while keeping it encrypted at rest in Kubernetes.

### Why Use ConfigMaps and Secrets?
- **Portability**: ConfigMaps let you modify app settings without changing the container image, making your apps more portable across environments (e.g., dev, staging, production).
- **Security**: Secrets ensure sensitive data is stored securely and only exposed to authorized pods.
- **Reusability**: Both ConfigMaps and Secrets can be shared across multiple pods or deployments, reducing duplication.
- **Flexibility**: Update configurations or secrets without redeploying your entire application.

### ConfigMap vs. Secret: What’s the Difference?
While both store key-value pairs, their use cases differ:
- **ConfigMaps** are for non-sensitive data, like app settings or feature flags. For example, you might use a ConfigMap to pass a `LOG_LEVEL` or database connection URL (without credentials).
- **Secrets** are for sensitive data, like database passwords or API tokens. Secrets are base64-encoded by default and stored securely in the Kubernetes etcd.

**Example Use Case**: Imagine you’re running a web app in a Docker container. You want to configure the app’s logging level (`LOG_LEVEL=DEBUG`) and database host (`DB_HOST=localhost`) using a ConfigMap. For the database password (`DB_PASSWORD=supersecret`), you’d use a Secret to keep it secure. This separation ensures sensitive data stays protected while non-sensitive settings remain easy to manage.

Let’s explore how to create and use these resources in Kubernetes!

## Creating ConfigMaps and Secrets via Command Line

The ways to create ConfigMaps and Secrets are similar — you can create them using `kubectl` commands in several ways: from literal values, standard input, or files. Below are some practical examples.

### Creating a ConfigMap
Let’s create a ConfigMap for our web app’s settings with `LOG_LEVEL=DEBUG` and `DB_HOST=localhost`.

**From Literal Values**:
```bash
kubectl create configmap app-config --from-literal=LOG_LEVEL=DEBUG --from-literal=DB_HOST=localhost
```

**From Files**:
Suppose we have two files `LOG_LEVEL.txt` and `DB_HOST.txt` with corresponding values as shown below:
- `LOG_LEVEL.txt`:
  ```text
  DEBUG
  ```
- `DB_HOST.txt`:
  ```text
  localhost
  ```

To create a ConfigMap using these files, you can run the following command. By default, the filename `LOG_LEVEL.txt` will become the key. However, the `--from-file` option allows you to specify the key name explicitly (e.g., `LOG_LEVEL=LOG_LEVEL.txt` will create a key called `LOG_LEVEL`). The file’s content becomes the value for that key.
```bash
kubectl create configmap app-config --from-file=LOG_LEVEL=LOG_LEVEL.txt --from-file=DB_HOST=DB_HOST.txt
```

**Output** (view the ConfigMap): This command retrieves the app-config ConfigMap and displays its contents in YAML format. You can see that the keys LOG_LEVEL and DB_HOST were created with the expected values.
```bash
kubectl get configmap app-config -o yaml
```
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  LOG_LEVEL: DEBUG
  DB_HOST: localhost
```

### Creating a Secret
Now, let’s create a Secret for the database password `DB_PASSWORD=supersecret`.

**From Literal Values**:
```bash
kubectl create secret generic db-secret --from-literal=DB_PASSWORD=supersecret
```

**From a File**:
If you have a file named `DB_PASSWORD.txt` with text value 'supersecret'. The command below will create the same secret matching the `--from-literal` output:
```bash
kubectl create secret generic db-secret --from-file=DB_PASSWORD=DB_PASSWORD.txt
```

**Output** (view the Secret):
```bash
kubectl get secret db-secret -o yaml
```
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
data:
  DB_PASSWORD: c3VwZXJzZWNyZXQ=  # base64-encoded "supersecret"
type: Opaque
```

**Note**: Ensure files used with `--from-file` contain only the value (no extra newlines). For Secrets, Kubernetes handles base64 encoding automatically for `--from-file` and `--from-literal`, but you’ll need to encode values manually for YAML files (shown later).

## Using YAML Files to Create ConfigMaps and Secrets

While command-line creation is quick, YAML files offer better control and version control in Git. Let’s define a ConfigMap and Secret using YAML.

### ConfigMap YAML
Create a file named `app-config.yaml`:
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  LOG_LEVEL: DEBUG
  DB_HOST: localhost
```

Apply it:
```bash
kubectl apply -f app-config.yaml
```

### Secret YAML
Create a file named `db-secret.yaml`:
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
data:
  DB_PASSWORD: c3VwZXJzZWNyZXQ=  # base64-encoded "supersecret"
```

Apply it:
```bash
kubectl apply -f db-secret.yaml
```

**Pro Tip**: For Secrets, you need to base64-encode the values yourself in the YAML. Use a command like `echo -n 'supersecret' | base64` to encode `DB_PASSWORD`.

## Using ConfigMaps and Secrets in a Deployment

Now, let’s see how a `Deployment` can consume these resources. You can inject ConfigMaps and Secrets into containers as environment variables or mounted files.

Here’s a sample `deployment.yaml`:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-app
        image: my-app:latest
        env:
        - name: LOG_LEVEL
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: LOG_LEVEL
        - name: DB_HOST
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: DB_HOST
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: DB_PASSWORD
        volumeMounts:
        - name: config-volume
          mountPath: /etc/config
      volumes:
      - name: config-volume
        configMap:
          name: app-config
```

In this example:
- `LOG_LEVEL` and `DB_HOST` are injected as environment variables from the ConfigMap.
- `DB_PASSWORD` is injected from the Secret.
- The `LOG_LEVEL` and `DB_HOST` from the ConfigMap are also mounted as files at `/etc/config/app.properties` in the container. For example, a file named `LOG_LEVEL` with value 'DEBUG' is available int this directory.

## More About Secrets: Data vs. StringData

Secrets support two fields for storing data:
- **data**: Stores base64-encoded values. Kubernetes expects values here to be pre-encoded, and they’re decoded automatically when consumed by pods.
- **stringData**: Allows you to specify non-encoded values. Kubernetes base64-encodes them for you when the Secret is created.

Example with `stringData`:
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
stringData:
  DB_PASSWORD: supersecret  # Not base64-encoded
```

**When to Use**:
- Use `data` when you want full control over encoding or are importing Secrets from external systems.
- Use `stringData` for convenience when defining Secrets manually, as it avoids manual base64 encoding.

**Important**: Secrets are base64-encoded for storage, not encrypted. Use RBAC and encryption at rest (configured at the cluster level) for additional security.

## Editing ConfigMaps and Secrets

To update a ConfigMap or Secret, you can edit them directly with `kubectl`.

**Edit a ConfigMap**:
```bash
kubectl edit configmap app-config
```
This opens the ConfigMap in your default editor (e.g., `vim`). Change `LOG_LEVEL` to `INFO`, save, and exit. The changes are applied immediately.

**Edit a Secret**:
```bash
kubectl edit secret db-secret
```
Edit the base64-encoded `DB_PASSWORD`. You’ll need to re-encode the new value manually (e.g., `echo -n 'newpassword' | base64`).

**Note**: Pods using these resources may need a restart to pick up changes, unless your app is designed to reload configurations dynamically.

## Debugging and Inspecting ConfigMaps and Secrets

Here are some handy commands to manage and troubleshoot:

- **List ConfigMaps and Secrets**:
  ```bash
  kubectl get configmaps
  kubectl get secrets
  ```

- **Describe for Details**:
  ```bash
  kubectl describe configmap app-config
  kubectl describe secret db-secret
  ```

- **View Contents**:
  ```bash
  kubectl get configmap app-config -o yaml
  kubectl get secret db-secret -o yaml
  ```

- **Decode a Secret**:
  ```bash
  echo 'c3VwZXJzZWNyZXQ=' | base64 --decode
  ```
  Output: `supersecret`

- **Check Pod Environment Variables**:
  ```bash
  kubectl exec -it <pod-name> -- env
  ```

- **List Mounted Files**:
  ```bash
  kubectl exec -it <pod-name> -- ls /etc/config
  ```

## Clarifications and Pitfalls to Avoid

1. **Base64 Encoding in Secrets**:
   - When creating Secrets via YAML, ensure values under `data` are base64-encoded without newlines. Use `echo -n 'value' | base64` to avoid newline issues.
   - Example: `echo -n 'supersecret' | base64` outputs `c3VwZXJzZWNyZXQ=`, which is correct.

2. **Secrets Are Not Encrypted by Default**:
   - Base64 is not encryption—it’s just encoding. Enable encryption at rest in your cluster for true security.

3. **Pod Restart After Updates**:
   - Pods don’t automatically reload updated ConfigMaps or Secrets. You may need to restart the pod (`kubectl delete pod <pod-name>`) or use a rolling update.

4. **Immutable ConfigMaps and Secrets**:
   - For high-traffic apps, consider setting `immutable: true` in ConfigMaps or Secrets to prevent accidental updates and improve performance.

5. **File-Based ConfigMaps**:
   - When using `--from-file`, the file name becomes the key in the ConfigMap. Ensure the key name is valid for your app.

6. **RBAC for Secrets**:
   - Restrict access to Secrets using Role-Based Access Control (RBAC) to prevent unauthorized access.

## Summary

ConfigMaps and Secrets are indispensable tools in Kubernetes for managing configuration and sensitive data. ConfigMaps keep your app settings flexible and reusable, while Secrets ensure sensitive data like passwords and keys are handled securely. By using them in your Deployments, you can build scalable, secure, and maintainable applications. Remember to encode Secrets properly, watch out for pod restarts after updates, and leverage RBAC for security.

For more details, check out the [official Kubernetes documentation on ConfigMaps](https://kubernetes.io/docs/concepts/configuration/configmap/) and [Secrets](https://kubernetes.io/docs/concepts/configuration/secret/). Stay tuned for more Kubernetes tips, and happy deploying!