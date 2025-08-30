# Kubernetes ConfigMap Example

This guide demonstrates how to use Kubernetes ConfigMaps to inject non-sensitive configuration data into your pods using both **environment variables** and **volumes**.

---

## What is a Kubernetes ConfigMap?

A **ConfigMap** is a Kubernetes object used to store non-confidential configuration data as key-value pairs. ConfigMaps help you decouple configuration from container images, making your applications more portable and easier to manage.

**Key Points:**
- ConfigMaps are for non-sensitive, environment-specific configuration.
- They can be consumed as environment variables or mounted as files inside pods.
- ConfigMaps allow you to update configuration without rebuilding container images.

---

## Files in This Example

- `test-config-map.yaml`: Defines a ConfigMap containing configuration data (e.g., database port).
- `simple-nginx-pod.yaml`: Defines a Pod that consumes the ConfigMap as both an environment variable and a mounted volume.

---

## How to Run This Example

### 1. Create the ConfigMap

**File:** `test-config-map.yaml`
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: test-config-map
data:
  db-port: "9999"
```

Apply the ConfigMap:
```sh
kubectl apply -f test-config-map.yaml
```

---

### 2. Create the Pod

**File:** `simple-nginx-pod.yaml`
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx
    volumeMounts:
      - name: db-connection
        mountPath: /opt
    ports:
    - containerPort: 80
    # Example for env variable usage (uncomment to use):
    # env:
    # - name: DB_PORT
    #   valueFrom:
    #     configMapKeyRef:
    #       name: test-config-map
    #       key: db-port
  volumes:
    - name: db-connection
      configMap:
        name: test-config-map
```

Apply the Pod:
```sh
kubectl apply -f simple-nginx-pod.yaml
```

---

### 3. Verify the Pod and ConfigMap

- **Check Pod Status:**
  ```sh
  kubectl get pods
  kubectl describe pod nginx
  ```

- **Check the Mounted File:**
  ```sh
  kubectl exec -it nginx -- ls /opt
  kubectl exec -it nginx -- cat /opt/db-port
  ```
  Output should show: `9999`

- **(Optional) Check Environment Variable:**
  If you uncomment the `env` section in the pod YAML, you can verify:
  ```sh
  kubectl exec -it nginx -- printenv | grep DB_PORT
  ```
  Output should show: `DB_PORT=9999`

---

## Notes

- **Environment variable-based ConfigMaps:**  
  Changes to the ConfigMap are **not** reflected in running pods. You must restart the pod to pick up changes.
- **Volume-based ConfigMaps:**  
  Updates to the ConfigMap are automatically reflected in the mounted files (with a short delay).
- **ConfigMaps are not for sensitive data.** Use Secrets for confidential information.

---

## Interview Preparation: Common Questions

- **What is a ConfigMap and why is it used?**  
  To externalize non-sensitive configuration from container images and manage it centrally.

- **How can you consume a ConfigMap in a pod?**  
  As environment variables or as files via volumes.

- **What happens if you update a ConfigMap?**  
  - For env vars: running pods are **not** updated; restart required.
  - For volumes: files are updated automatically (with a short delay).

- **Can ConfigMaps be used for sensitive data?**  
  No, use Secrets for sensitive data.

- **How do you create a ConfigMap from literal values or files?**  
  ```sh
  kubectl create configmap my-config --from-literal=key1=value1 --from-file=app.properties
  ```

- **Difference between ConfigMap and Secret?**  
  ConfigMaps are for non-sensitive data; Secrets are for sensitive data and are base64-encoded.

---

## References

- [Kubernetes ConfigMap Documentation](https://kubernetes.io/docs/concepts/configuration/configmap/)