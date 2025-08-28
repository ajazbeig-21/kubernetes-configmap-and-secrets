# Kubernetes ConfigMap Example

This repository demonstrates how to use Kubernetes ConfigMaps to inject configuration into your pods using both **environment variables** and **volumes**.

## Folder Structure

```
configmap/
  simple-nginx-pod.yaml
  test-config-map.yaml
secrets/
README.md
```

## What is a ConfigMap?

A **ConfigMap** is a Kubernetes object used to store non-confidential configuration data in key-value pairs. ConfigMaps allow you to decouple configuration artifacts from image content to keep containerized applications portable.

## Types of ConfigMap Usage

### 1. Environment Variable-based ConfigMap

You can inject ConfigMap values as environment variables into your containers. This allows your application to access configuration via environment variables.

**Example:**
```yaml
env:
- name: DB_PORT
  valueFrom:
    configMapKeyRef:
      name: test-config-map
      key: db-port
```
This will set the `DB_PORT` environment variable in the container to the value of `db-port` from the ConfigMap named `test-config-map`.

### 2. Volume-based ConfigMap

You can mount a ConfigMap as a file inside your container. Each key in the ConfigMap becomes a file in the specified directory, and the value of the key is the content of the file.

**Example:**
```yaml
volumes:
  - name: db-connection
    configMap:
      name: test-config-map
containers:
  - name: nginx
    ...
    volumeMounts:
      - name: db-connection
        mountPath: /opt
```
This will mount the ConfigMap as files under `/opt` in the container.

## How to Run

1. **Apply the ConfigMap:**

   ```sh
   kubectl apply -f configmap/test-config-map.yaml
   ```

2. **Apply the Pod:**

   ```sh
   kubectl apply -f configmap/simple-nginx-pod.yaml
   ```

3. **Verify the Pod:**

   ```sh
   kubectl get pods
   kubectl describe pod nginx
   ```

4. **Check Mounted ConfigMap in Pod:**

   ```sh
   kubectl exec -it nginx -- ls /opt
   kubectl exec -it nginx -- cat /opt/db-port
   ```

## Notes

- If you use the environment variable method, changes to the ConfigMap will **not** be reflected in running pods. You must restart the pod to pick up changes.
- If you use the volume method, updates to the ConfigMap are automatically updated in the mounted files (with some delay).

## References

- [Kubernetes ConfigMap Documentation](https://kubernetes.io/docs/concepts/configuration/configmap/)