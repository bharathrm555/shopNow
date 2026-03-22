# ShopNow MERN Stack - Deployment Guide

This project contains a MERN (MongoDB, Express, React, Node) application with a fully automated Kubernetes deployment using Helm and Jenkins.

## 1. Local Deployment & Testing (What we did)

To prepare the application for Kubernetes, several changes were made to the original codebase to ensure seamless service discovery and correct path routing.

### Key Changes and Key Decision Points

#### **Dockerfiles (Frontend & Admin)**
- **Problem**: The original build process used a hardcoded `/aryan` and `/aryan-admin` as `PUBLIC_URL`. This caused "connection reset" or "blank page" errors when accessing via `localhost`.
- **Solution**: Changed `ARG USER_NAME` to default to an empty string (`""`).
- **Why**: This ensures that when the application is built for local testing or simple Kubernetes deployments, it is served at the root (`/`) of the domain.
- **Reference**: [frontend/Dockerfile](file:///home/bharathrm/Local_projects/shopNow/frontend/Dockerfile) and [admin/Dockerfile](file:///home/bharathrm/Local_projects/shopNow/admin/Dockerfile).

#### **Nginx Configuration (Admin)**
- **Problem**: The Admin Nginx config was trying to proxy `/admin/` requests to the backend, but the backend only handles `/api/` routes.
- **Solution**: Updated `admin/nginx/default.conf` to proxy `/api/` to `http://backend-service:5000/api/`.
- **Connection Flow**: Browser -> Admin Service (Port 80) -> Admin Nginx -> Backend Service (Port 5000) -> Node.js API -> MongoDB.

#### **Helm Chart**
- **Decision**: Created a Helm chart in `charts/shopnow` to manage all components together.
- **Why**: Helm allows for versioned, reproducible deployments and separates configuration (`values.yaml`) from the actual manifests.
- **Components**: MongoDB, Backend, Frontend, and Admin.

---

## 2. Jenkins CI/CD Setup Guide

Follow these steps to deploy this application using Jenkins on your local machine.

### Prerequisites
1. **Jenkins Instance**: Running on your local machine (e.g., [http://localhost:8080](http://localhost:8080)).
2. **Tools**: Ensure `docker`, `kubectl`, `helm`, and `kind` are installed on the Jenkins server (or the host if using the agent).
3. **Kubeconfig**: Jenkins needs access to your KIND cluster. Usually found at `~/.kube/config`.

### Step-by-Step Jenkins Pipeline Setup

1. **Create New Item**: 
   - Open Jenkins -> New Item.
   - Enter name: `shopnow-pipeline` -> Select **Pipeline** -> OK.
2. **Configure Pipeline**:
   - Scroll down to the **Pipeline** section.
   - Definition: **Pipeline script from SCM**.
   - SCM: **Git**.
   - Repository URL: Path to this local folder (or your GitHub repo).
   - Branch Specifier: `*/main` (or your active branch).
   - Script Path: `Jenkinsfile`.
3. **Credentials**:
   - If deploying to a local KIND cluster from the same machine, Jenkins usually doesn't need extra credentials as long as it can access the `~/.kube/config` and the `docker` socket.
   - **Important**: Ensure the Jenkins user has permission to run `docker` commands (`sudo usermod -aG docker jenkins`).
4. **Environment Variables**:
   - The `Jenkinsfile` already sets up the `NAMESPACE` and `KUBECONFIG` environment. Ensure the path to your kubeconfig is correct in the `Jenkinsfile`.

### Connection Flow in Jenkins
1. **Build**: Jenkins builds Docker images for all components.
2. **Load**: Jenkins uses `kind load docker-image` to push images into the KIND cluster.
3. **Deploy**: Jenkins runs `helm upgrade --install` to update the applications.

---

## 3. Verification Post-Deployment

After the Jenkins pipeline finishes, verify the deployment:

```bash
# Check if pods are running
kubectl get pods -n shopnow

# Forward ports to access the UIs
kubectl port-forward svc/frontend-service -n shopnow 8085:80
kubectl port-forward svc/admin-service -n shopnow 8081:80
```

- **Frontend**: [http://localhost:8085](http://localhost:8085)
- **Admin**: [http://localhost:8081](http://localhost:8081)
- **API Health**: [http://localhost:5000/api/health](http://localhost:5000/api/health) (if forwarded)
