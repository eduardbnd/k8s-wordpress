# Kubernetes Workshop: WordPress & MariaDB Migration

## 📋 Project Overview
This project demonstrates the migration of a WordPress application (with a MariaDB database) from a `docker-compose` architecture to a highly available, fault-tolerant Kubernetes cluster. The environment is deployed on an AWS EC2 instance using Minikube.

## 🏗 Architecture & Design
- **Database (MariaDB):** Deployed using a `StatefulSet` to ensure a stable network identity, paired with a `PersistentVolumeClaim` (PVC) for reliable data persistence across pod restarts.
- **Application (WordPress):** Deployed via a `Deployment` configured with 2 replicas to ensure High Availability (HA).
- **Networking:** Internal traffic is routed through a `Service` (ClusterIP). External access is managed by an **NGINX Ingress Controller**, routing traffic via the `wordpress.local` domain.
- **Container Registry:** Official Docker images are pulled, re-tagged, and pushed to a private **Amazon ECR** repository to ensure cluster stability and avoid public registry rate limits.

## ⚙️ Configuration Management (Helm)
The entire application stack is packaged into a custom **Helm Chart** (`wordpress-chart`). 
This approach provides:
- Centralized management of application parameters (e.g., replica counts, image tags, credentials) via the `values.yaml` file.
- Automated, single-command deployment of the entire infrastructure.

## 📊 Monitoring & Observability
To monitor the cluster's health and performance, the `kube-prometheus-stack` was installed.
A custom **Grafana** dashboard (using the "Status history" visualization) was created to track the **Uptime** of the WordPress pods based on Prometheus metrics.

## 📦 Workflow: Pulling and Pushing Images
Before deploying the Helm chart, official Docker images for WordPress and MariaDB were pulled, re-tagged, and pushed to a private Amazon ECR repository for stability and to avoid Docker Hub rate limits:

1. **Authenticate Docker to ECR:**
```bash
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin <your_aws_account_id>.dkr.ecr.us-east-1.amazonaws.com
```

2. **Pull official images:**
```bash
docker pull wordpress:latest
docker pull mariadb:10.6.4-focal
```

3. **Tag images for your ECR repository:**
```bash
docker tag wordpress:latest <your_ecr_link>/k8s-wordpress:latest
docker tag mariadb:10.6.4-focal <your_ecr_link>/k8s-mariadb:10.6.4-focal
```

4. **Push to Repository:**
```bash
docker push <your_ecr_link>/k8s-wordpress:latest
docker push <your_ecr_link>/k8s-mariadb:10.6.4-focal
```
## 📁 Configuration (values.yaml)
> **Security Note:** The `values.yaml` file is excluded from this repository via `.gitignore` to protect sensitive information such as AWS Account IDs and database passwords.

To deploy this project, you must create a `values.yaml` file in the chart directory with the following structure:

```yaml
replicaCount: 2

images:
  wordpress: <your_ecr_link>/k8s-wordpress:latest
  mariadb: <your_ecr_link>/k8s-mariadb:10.6.4-focal

ingress:
  host: wordpress.local

database:
  host: db
  name: wordpress
  user: wordpress
  password: <your_password>
  rootPassword: <your_root_password>
```

## 🚀 How to Run the Project

1. **Deploy the application using Helm:**
   ```bash
   helm install my-wordpress ./wordpress-chart
   ```

2. **Configure local access:**
   - Add the following line to your local operating system's `hosts` file:
     ```text
     127.0.0.1 wordpress.local
     ```
   - Forward the Ingress controller port to your local machine:
     ```bash
     kubectl port-forward -n ingress-nginx svc/ingress-nginx-controller 8080:80
     ```

3. **Access the application:**
   Open your web browser and navigate to: `http://wordpress.local:8080`

4. **Access the Monitoring Dashboard (Grafana):**
   - Forward the Grafana port to your local machine:
     ```bash
     kubectl port-forward -n monitoring svc/monitoring-grafana 3000:80
     ```
   - Open your web browser and navigate to: `http://localhost:3000`