# Kubernetes Secret Example

This example demonstrates how to use Kubernetes Secrets to securely inject sensitive data (like API keys or passwords) into your pods using both **environment variables** and **volumes**.

---

## What is a Kubernetes Secret?

A **Kubernetes Secret** is an object that lets you store and manage sensitive information, such as passwords, OAuth tokens, and SSH keys. Storing confidential information in a Secret is safer and more flexible than putting it verbatim in a Pod definition or a container image.

**Key Features:**
- Secrets are base64-encoded and stored in etcd (the Kubernetes backing store).
- Secrets can be mounted as files or exposed as environment variables.
- Access to Secrets can be controlled via RBAC.

---

## Files in This Example

- `secret.yaml`: Defines a Secret containing an API key.
- `pod.yaml`: Defines a Pod that consumes the Secret as both an environment variable and a mounted volume.

---

## How to Create and Use a Secret

### 1. Create the Secret

The `secret.yaml` file contains:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: map-key-secret
type: Opaque
data:
  API_KEY: MTIzNDU2
```

- The `data` field must contain base64-encoded values.
- For example, to encode `123456`:
  ```sh
  echo -n "123456" | base64
  # Output: MTIzNDU2
  ```

Apply the secret:
```sh
kubectl apply -f secret.yaml
```

---

### 2. Use the Secret in a Pod

The `pod.yaml` file demonstrates two ways to use the Secret:

#### a. As an Environment Variable

```yaml
env:
- name: API_KEY
  valueFrom:
    secretKeyRef:
      name: map-key-secret
      key: API_KEY
```
- This sets the `API_KEY` environment variable in the container to the decoded value from the Secret.

#### b. As a Mounted Volume

```yaml
volumeMounts:
  - name: secret-volume
    mountPath: /etc/secret-volume
    readOnly: true
volumes:
  - name: secret-volume
    secret:
      secretName: map-key-secret
```
- This mounts the Secret as a file at `/etc/secret-volume/API_KEY` inside the container.

Apply the pod:
```sh
kubectl apply -f pod.yaml
```

---

## Verifying the Secret in the Pod

1. **Check the environment variable:**
   ```sh
   kubectl exec -it nginx -- printenv | grep API_KEY
   ```
   Output should show: `API_KEY=123456`

2. **Check the mounted file:**
   ```sh
   kubectl exec -it nginx -- cat /etc/secret-volume/API_KEY
   ```
   Output should show: `123456`

---

## Interview Tips

- **Why use Secrets?**  
  To avoid hardcoding sensitive data in pod specs or images and to control access to sensitive information.
- **How are Secrets different from ConfigMaps?**  
  Secrets are intended for sensitive data and are base64-encoded; ConfigMaps are for non-sensitive config.
- **How can you consume a Secret in a pod?**  
  As environment variables or as files via volumes.
- **Are Secrets encrypted at rest?**  
  By default, they are only base64-encoded. Encryption at rest requires additional configuration in Kubernetes.
- **How do you update a Secret?**  
  Use `kubectl apply` or `kubectl edit secret <name>`. Pods consuming the Secret as env vars need to be restarted to pick up changes; mounted secrets update automatically (with a short delay).

---

## References

- [Kubernetes Secrets Documentation](https://kubernetes.io/docs/concepts/configuration/secret/)