# Spring Boot Application Deployment with Kubernetes (kubeadm), GHCR, Ingress, Helm, and GitHub Actions

## Project Overview

This project demonstrates the full DevOps lifecycle:

* Creating and building a Spring Boot application.
* Dockerizing the application.
* Pushing the Docker image to GitHub Container Registry (GHCR).
* Setting up a Kubernetes cluster using `kubeadm` on two Linux machines.
* Deploying the application to Kubernetes.
* Exposing the application using an Ingress Controller (NGINX).
* Automating build and push processes using GitHub Actions.
* Attempting Continuous Deployment (CD) from GitHub Actions to Kubernetes.

---

## Architecture

```
+----------------+        +-----------------+        +-------------------+
|   GitHub Repo  |  --->  |   GitHub Actions |  --->  | GitHub Container   |
| (Spring Boot)  |        | (CI Workflow)    |        | Registry (GHCR)    |
+----------------+        +-----------------+        +-------------------+
                                                            |
                                                            v
+---------------------------+
| Kubernetes Cluster (kubeadm) |
| (Deployed Spring Boot App)  |
| (Exposed via Ingress)       |
+---------------------------+
```

---

## Project Steps

### 1. Create a Spring Boot Application

* Simple REST endpoint (`/hello`) returning a "Hello World" message.

### 2. Build and Dockerize the Application

```bash
./mvnw clean package
docker build -t ghcr.io/<your-github-username>/project1:latest .
```

### 3. Push Docker Image to GHCR

```bash
echo $GITHUB_TOKEN | docker login ghcr.io -u <your-github-username> --password-stdin
docker push ghcr.io/<your-github-username>/project1:latest
```

### 4. Set Up Kubernetes Cluster

* Set up two Linux machines.
* Verified connectivity (`ping` test).
* Installed `kubeadm`, `kubelet`, and `kubectl`.
* Initialized Kubernetes master node.
* Joined worker nodes to cluster.

### 5. Deploy Application to Kubernetes

* Created Kubernetes Deployment and Service manifests.
* Deployed Spring Boot application and exposed internally via `ClusterIP`.

### 6. Set Up Ingress Controller

* Installed NGINX Ingress Controller for bare-metal setup.
* Created an Ingress resource mapping host `springboot.local` to the service.

### 7. Access Application

* Updated local `/etc/hosts` to map `springboot.local` to cluster node IP.
* Accessed the application via:

  ```bash
  curl http://springboot.local:<NodePort>/hello
  ```

### 8. Create and Deploy Helm Chart

* Packaged the application as a Helm chart.
* Installed and deployed via Helm.

---

## Automation with GitHub Actions

### Continuous Integration (CI)

* **Workflow** triggered on push to `main`.
* Actions performed:

  * Checkout code.
  * Build Spring Boot project.
  * Build Docker image.
  * Push Docker image to GHCR.

Example `.github/workflows/ci.yaml`:

```yaml
name: Build and Push Docker Image

on:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Code
        uses: actions/checkout@v4

      - name: Set up JDK 17
        uses: actions/setup-java@v4
        with:
          distribution: 'temurin'
          java-version: 17

      - name: Build Application
        run: ./mvnw clean package

      - name: Log in to GHCR
        run: echo ${{ secrets.GITHUB_TOKEN }} | docker login ghcr.io -u ${{ github.actor }} --password-stdin

      - name: Build and Push Docker Image
        run: |
          docker build -t ghcr.io/${{ github.repository }}/project1:latest .
          docker push ghcr.io/${{ github.repository }}/project1:latest
```

✅ **Result:** Successful automated build and push on every code commit.

---

### Continuous Deployment (CD) Attempt

* Tried automating Kubernetes deployment using GitHub Actions.
* Challenge: Kubernetes cluster was in a private network behind a bastion host.
* GitHub Action runner could not access the cluster directly.
* **Deployment failed** due to network isolation.

> **Future Fix:** Set up a self-hosted GitHub Action Runner inside the network or use VPN/SSH tunneling.

---

## Project Status

| Task                            | Status                                    |
| :------------------------------ | :---------------------------------------- |
| Create Spring Boot App          | ✅ Completed                               |
| Build and Dockerize App         | ✅ Completed                               |
| Push Image to GHCR              | ✅ Completed                               |
| Set up Kubernetes Cluster       | ✅ Completed                               |
| Deploy App to Kubernetes        | ✅ Completed                               |
| Expose App via Ingress          | ✅ Completed                               |
| Create Helm Chart               | ✅ Completed                               |
| Automate CI with GitHub Actions | ✅ Completed                               |
| Automate CD                     | ⚠️ Attempted (Blocked by private network) |

---

## Technologies Used

* Java 17 / Spring Boot
* Docker
* GitHub Container Registry (GHCR)
* Kubernetes (kubeadm)
* NGINX Ingress Controller
* Helm
* GitHub Actions

---

## Author

**\[Your Name Here]**
[GitHub Profile](https://github.com/<your-github-username>)

---

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

# 📣 Notes

> This project can be further enhanced by setting up a **self-hosted runner** inside the Kubernetes network or using **ArgoCD** for GitOps deployment in future phases.


