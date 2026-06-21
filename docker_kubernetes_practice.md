# Docker & Kubernetes: Practice Questions and Solutions

This document focuses on containerization with Docker and orchestration with Kubernetes, testing your practical knowledge of building, deploying, and managing containers.

---

## Scenario 1: Optimizing Docker Image Size
**Question:**
You have created a Dockerfile for a Node.js application. The resulting Docker image is over 1GB, which is slowing down deployments and increasing storage costs. What practical steps can you take in your Dockerfile to drastically reduce the image size?

**Solution:**
Large images are usually caused by including unnecessary build tools and cache files.
1.  **Use a Smaller Base Image:** Instead of using `node:latest` (which is based on a full Debian distribution), use a specialized slim image like `node:alpine` or `node:slim`. Alpine Linux is exceptionally small.
2.  **Multi-Stage Builds:** If your application requires build tools (like `npm install` building C++ dependencies or compiling TypeScript), use a multi-stage build. 
    *   *Stage 1 (Builder):* Use a heavier image to install all dependencies and compile the code.
    *   *Stage 2 (Production):* Use a minimal base image, and only `COPY` the final compiled artifacts and the `node_modules` required for production from Stage 1. The build tools are left behind.
3.  **Use .dockerignore:** Ensure you have a `.dockerignore` file that excludes the local `.git` directory, local `node_modules`, and `README` files so they aren't unnecessarily copied into the image context.
4.  **Clean up caches:** Run commands like `npm cache clean --force` or `rm -rf /var/cache/apk/*` in the same `RUN` layer where you install packages to prevent cache files from inflating the layer size.

---

## Scenario 2: Data Persistence in Containers
**Question:**
You are running a MySQL database inside a Docker container. You stop the container, remove it, and start a new one using the same image. You discover that all your database records are gone. Why did this happen, and how do you fix it?

**Solution:**
By default, data created inside a container is stored on a writable container layer. When the container is deleted, the writable layer is also deleted.
1.  **Use Docker Volumes:** To persist data beyond the lifecycle of a single container, you must map a directory on the host machine to a directory inside the container.
2.  **Implementation:** When running the database container, use the `-v` or `--mount` flag.
    ```bash
    docker run -d --name mydb -v mysql-data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=secret mysql:8.0
    ```
    This creates a named Docker volume `mysql-data` and mounts it to `/var/lib/mysql` inside the container. If you delete the `mydb` container and start a new one pointing to the same `mysql-data` volume, your records will still be there.

---

## Scenario 3: Kubernetes Pod CrashLoopBackOff
**Question:**
You deployed a new microservice to a Kubernetes cluster. When you run `kubectl get pods`, the status of your pod shows `CrashLoopBackOff`. How do you practically troubleshoot and resolve this?

**Solution:**
`CrashLoopBackOff` means the application inside the container is repeatedly starting, failing, and exiting, causing Kubernetes to continually restart it.
1.  **Check Pod Logs:** The first step is always to check the application logs to see why it's crashing.
    ```bash
    kubectl logs <pod-name>
    ```
    If the pod restarted recently, check the logs of the previous instance:
    ```bash
    kubectl logs <pod-name> --previous
    ```
2.  **Check Pod Events:** If the logs are empty (e.g., the container couldn't even start), check the Kubernetes events related to the pod.
    ```bash
    kubectl describe pod <pod-name>
    ```
    Look at the "Events" section at the bottom. This might reveal issues like `OOMKilled` (Out of Memory), misconfigured liveness/readiness probes, or inability to pull the Docker image (`ImagePullBackOff`).
3.  **Common Causes:** 
    *   The application requires an environment variable that wasn't provided in the Deployment manifest.
    *   The application is trying to bind to a port it doesn't have permission to use.
    *   The container command completes immediately (e.g., it runs a script and exits instead of running a persistent server process).

---

## Scenario 4: Managing Kubernetes Secrets securely
**Question:**
Your application requires an API key to communicate with a third-party payment gateway. You have defined this API key as plain text in your Kubernetes Deployment YAML file as an environment variable. Why is this bad practice, and how should you do it correctly in Kubernetes?

**Solution:**
Putting secrets in plain text in a Deployment YAML means anyone with access to the source code repository or the Kubernetes API can read the sensitive key.
1.  **Create a Kubernetes Secret:** Store the API key as a distinct `Secret` object in Kubernetes.
    ```bash
    kubectl create secret generic payment-secret --from-literal=api-key=YOUR_SUPER_SECRET_KEY
    ```
2.  **Reference in Deployment:** Update your Deployment YAML to inject the value from the Secret into the pod as an environment variable.
    ```yaml
    env:
      - name: PAYMENT_API_KEY
        valueFrom:
          secretKeyRef:
            name: payment-secret
            key: api-key
    ```
3.  *(Advanced)* **External Secrets:** For enterprise environments, don't create Secrets manually. Use tools like "External Secrets Operator" to automatically sync secrets from AWS Secrets Manager or HashiCorp Vault directly into Kubernetes Secret objects.
