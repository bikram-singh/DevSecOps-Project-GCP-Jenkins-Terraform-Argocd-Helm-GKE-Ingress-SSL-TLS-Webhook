# DevSecOps CI/CD Pipeline on GCP with Jenkins, Terraform, ArgoCD, Helm, and GKE

![CI/CD Pipeline Architecture](CI-CD-Pipeline%20Architecture%20Overview.png)

## 📋 Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Tech Stack](#tech-stack)
- [Project Structure](#project-structure)
- [Prerequisites](#prerequisites)
- [Setup Instructions](#setup-instructions)
- [Jenkins Pipeline](#jenkins-pipeline)
- [Infrastructure as Code (Terraform)](#infrastructure-as-code-terraform)
- [Helm Deployment](#helm-deployment)
- [Security Scanning](#security-scanning)
- [CI/CD Workflow](#cicd-workflow)
- [Troubleshooting](#troubleshooting)
- [Contributing](#contributing)
- [Author](#author)
- [License](#license)

---

## 🚀 Overview

This project demonstrates a **complete DevSecOps pipeline** on Google Cloud Platform (GCP) using enterprise-grade tools and best practices. The pipeline integrates:

- **Jenkins** for CI/CD orchestration
- **Terraform** for Infrastructure as Code
- **Google Kubernetes Engine (GKE)** for container orchestration
- **Helm** for Kubernetes package management
- **ArgoCD** for GitOps continuous delivery
- **Security scanning tools** (Trivy, SonarQube, OWASP Dependency Check)
- **Monitoring** with Prometheus & Grafana
- **SSL/TLS** with Ingress and Cert-Manager

Each commit to the repository automatically triggers a complete pipeline that:
1. Scans source code for vulnerabilities
2. Performs static code analysis
3. Builds and pushes Docker images
4. Scans container images for vulnerabilities
5. Updates Helm charts
6. Deploys to GKE using GitOps

---

## 🏗️ Architecture

The architecture consists of three main components:

### 1. **Source Code Management**
   - GitHub repository containing application code and IaC
   - Jenkinsfile for pipeline definition
   - Helm charts for Kubernetes deployments

### 2. **CI/CD Pipeline (Jenkins)**
   - Automated build on every commit
   - Security scanning (Trivy, SonarQube, OWASP)
   - Docker image building and pushing to Artifact Registry
   - Helm chart updates for GitOps

### 3. **Infrastructure Layer**
   - GCP VPC and subnets
   - GKE cluster with auto-scaling node pools
   - Firewall rules for network security
   - Google Artifact Registry for container images

### 4. **Deployment Layer**
   - ArgoCD for GitOps-based continuous deployment
   - Helm for templated Kubernetes deployments
   - Ingress controller with SSL/TLS certificates

---

## 💻 Tech Stack

| Component | Purpose | Version |
|-----------|---------|---------|
| **Jenkins** | CI/CD Pipeline Orchestration | Latest |
| **Terraform** | Infrastructure as Code | v6.26.0 |
| **Kubernetes (GKE)** | Container Orchestration | 1.31.6-gke.1020000 |
| **Helm** | Kubernetes Package Manager | v3.x |
| **Docker** | Container Runtime | Latest |
| **GCP Services** | Cloud Platform | - |
| **Trivy** | Container & Filesystem Scanning | Latest |
| **SonarQube** | Code Quality & Security Analysis | Latest |
| **OWASP** | Dependency Vulnerability Scanning | Latest |
| **Nginx** | Web Server | Alpine |

---

## 📂 Project Structure

```
📦 DevSecOps-Project-GCP-Jenkins-Terraform-Argocd-Helm-GKE-Ingress-SSL-TLS-Webhook
│
├── 📂 app/                                    # Application Source Code
│   ├── 📄 index.html                         # Web application UI
│   ├── 🐳 Dockerfile                         # Docker image definition
│   └── 📘 README.md                          # Application documentation
│
├── 📂 helm/                                  # Helm Charts for Kubernetes Deployment
│   ├── 📄 Chart.yaml                         # Helm chart metadata
│   ├── 📄 values.yaml                        # Default configuration values
│   └── 📂 templates/
│       ├── 📄 deployment.yaml                # Kubernetes Deployment manifest
│       └── 📄 service.yaml                   # Kubernetes Service manifest
│
├── 📂 gke-terraform/                         # Terraform IaC for GKE Provisioning
│   ├── 📄 main.tf                            # GKE cluster, VPC, firewall configs
│   ├── 📄 provider.tf                        # GCP provider configuration
│   └── 📄 variable.tf                        # Terraform variables (project, region, zone)
│
├── 📂 jenkins/                               # Jenkins Pipeline Definitions
│   └── 📄 Jenkinsfile-ci-gke                 # Main CI/CD and Infra pipeline
│
├── 📄 Jenkinsfile                            # Jenkins pipeline (if used from root)
├── 📄 DevSecOps_Jenkins_GCP_Medium_Blog.md   # Detailed setup guide
├── 📄 CI-CD-Pipeline Architecture Overview.png # Architecture diagram
├── 📄 devsecops-project-111-b292bfc51bf0.json # GCP service account key
├── 📄 GitHub-Token-PAT.txt                   # GitHub Personal Access Token
└── 📘 README.md                              # This file
```

---

## 📋 Prerequisites

Before you begin, ensure you have the following:

### 1. **GCP Setup**
   - Active GCP account with billing enabled
   - GCP project created (`devsecops-project-111`)
   - Service account with appropriate permissions
   - Service account key JSON file downloaded

### 2. **Local Tools**
   - `gcloud` CLI installed and configured
   - `terraform` CLI v1.0+
   - `kubectl` CLI installed
   - `helm` v3.x installed
   - `docker` installed (if building locally)
   - `git` installed

### 3. **Jenkins Infrastructure**
   - RHEL 8 VM on GCP (4 vCPU, 8GB RAM, 30GB storage)
   - Jenkins 2.387+ installed and running
   - Jenkins plugins installed:
     - Docker Pipeline
     - GitHub Integration
     - Kubernetes
     - SonarQube Scanner
     - Trivy
     - OWASP Dependency-Check

### 4. **GCP Services**
   - Artifact Registry API enabled
   - Compute Engine API enabled
   - Kubernetes Engine API enabled
   - Firewall rules configured

### 5. **Credentials & Access**
   - GitHub Personal Access Token (PAT)
   - GCP Service Account Key JSON
   - Jenkins credentials configured for GitHub and GCP

---

## 🔧 Setup Instructions

### Step 1: Clone the Repository

```bash
git clone https://github.com/bikram-singh/DevSecOps-Project-GCP-Jenkins-Terraform-Argocd-Helm-GKE-Ingress-SSL-TLS-Webhook.git
cd DevSecOps-Project-GCP-Jenkins-Terraform-Argocd-Helm-GKE-Ingress-SSL-TLS-Webhook
```

### Step 2: Configure GCP

```bash
# Set your GCP project
gcloud config set project devsecops-project-111
gcloud config set compute/region us-central1
gcloud config set compute/zone us-central1-a

# Authenticate
gcloud auth login
```

### Step 3: Setup Jenkins VM on GCP

```bash
# Create RHEL 8 VM
gcloud compute instances create jenkins-server \
  --image-family=rhel-8 \
  --image-project=rhel-cloud \
  --machine-type=e2-standard-4 \
  --zone=us-central1-a \
  --boot-disk-size=30GB \
  --network=devsecops-vpc \
  --subnet=devsecops-subnet

# SSH into the instance
gcloud compute ssh jenkins-server --zone=us-central1-a
```

### Step 4: Install Jenkins and Tools

Run the installation script on the Jenkins VM:

```bash
#!/bin/bash
set -e

# Update system packages
sudo dnf upgrade -y

# Install Java 17
sudo dnf install java-17-openjdk java-17-openjdk-devel -y

# Add Jenkins repository
sudo curl -fsSL -o /etc/yum.repos.d/jenkins.repo \
    https://pkg.jenkins.io/redhat-stable/jenkins.repo

# Import GPG key
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key

# Install Jenkins
sudo dnf install jenkins -y

# Start Jenkins
sudo systemctl enable jenkins
sudo systemctl start jenkins

# Configure firewall
sudo firewall-cmd --permanent --add-port=8080/tcp
sudo firewall-cmd --reload

# Install Docker
sudo dnf remove podman -y
sudo dnf config-manager --add-repo=https://download.docker.com/linux/centos/docker-ce.repo
sudo dnf install docker-ce docker-ce-cli containerd.io -y
sudo systemctl enable --now docker
sudo usermod -aG docker jenkins

# Install Trivy
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
echo "deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main" | sudo tee -a /etc/apt/sources.list.d/trivy.list

# Install Terraform
curl -fsSL https://apt.releases.hashicorp.com/gpg | sudo apt-key add -
echo "deb [arch=amd64] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt-get update && sudo apt-get install terraform -y

echo "✅ Installation complete!"
```

### Step 5: Configure Jenkins Credentials

1. Access Jenkins at `http://<JENKINS_IP>:8080`
2. Log in with the admin password
3. Navigate to **Manage Jenkins** → **Manage Credentials**
4. Add the following credentials:
   - **GCP Service Account** (ID: `gcp-service-account`)
   - **GCP Project ID** (ID: `gcp-project-id`)
   - **GitHub Token** (ID: `github-token`)
   - **SonarQube Token** (ID: `Sonar-token`)

### Step 6: Create Jenkins Pipeline

1. Create a new **Pipeline** job in Jenkins
2. Configure the pipeline to use the GitHub repository
3. Set the Jenkinsfile path to `jenkins/Jenkinsfile-ci-gke`
4. Save and test the pipeline

### Step 7: Initialize Terraform

```bash
cd gke-terraform
terraform init
terraform plan
```

### Step 8: Deploy GKE Cluster

**Using Jenkins Pipeline:**
1. Run the Jenkins job with parameters:
   - `PIPELINE`: `infra`
   - `ACTION`: `apply`

**Or manually with Terraform:**
```bash
terraform apply -auto-approve
```

### Step 9: Configure kubectl

```bash
# Get GKE cluster credentials
gcloud container clusters get-credentials terraform-gke-cluster \
  --region us-central1

# Verify connection
kubectl get nodes
```

### Step 10: Setup ArgoCD (Optional but Recommended)

```bash
# Create namespace
kubectl create namespace argocd

# Install ArgoCD
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Expose ArgoCD (port forward)
kubectl port-forward -n argocd svc/argocd-server 8443:443

# Get initial admin password
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```

---

## 🔄 Jenkins Pipeline

### Pipeline Stages

The Jenkins pipeline supports two modes:

#### **CI/CD Pipeline (Application Build & Deploy)**

1. **Checkout Repo** - Clone the GitHub repository
2. **Authenticate with GCP** - Setup GCP credentials
3. **Security Scan - Trivy FS** - Scan filesystem for vulnerabilities
4. **SonarQube Analysis** - Perform code quality analysis
5. **Quality Gate** - Wait for SonarQube approval
6. **OWASP Dependency Check** - Scan dependencies for vulnerabilities
7. **Build & Push Docker Image** - Build and push to Artifact Registry
8. **Security Scan - Trivy Image** - Scan the built Docker image
9. **Update Helm Chart values.yaml** - Update image tag in Helm chart
10. **CD Trigger** - ArgoCD automatically syncs and deploys

#### **Infrastructure Pipeline (Terraform)**

1. **Terraform Init** - Initialize Terraform
2. **Terraform Plan/Apply/Destroy** - Apply or destroy infrastructure based on ACTION parameter

### Pipeline Parameters

| Parameter | Type | Values | Purpose |
|-----------|------|--------|---------|
| `PIPELINE` | Choice | `ci`, `infra` | Select which pipeline to run |
| `ACTION` | Choice | `apply`, `destroy` | For infra pipeline: create or delete resources |

### Environment Variables

```
GOOGLE_APPLICATION_CREDENTIALS = GCP Service Account JSON
GOOGLE_CLOUD_PROJECT = GCP Project ID
IMAGE_TAG = 1.0.${BUILD_NUMBER}
SCANNER_HOME = SonarQube scanner path
```

---

## 🏗️ Infrastructure as Code (Terraform)

### GCP Resources Created

The Terraform configuration provisions:

1. **VPC Network**
   - Name: `custom-vpc`
   - Custom CIDR: `10.10.0.0/16`

2. **Subnet**
   - Name: `custom-subnetwork`
   - Region: `us-central1`

3. **Firewall Rules**
   - Internal communication (10.10.0.0/16)
   - External access (SSH, RDP, ICMP)
   - GKE communication (443, 10250, 15017)

4. **GKE Cluster**
   - Name: `terraform-gke-cluster`
   - Version: 1.31.6-gke.1020000
   - Network: Custom VPC
   - Auto-scaling enabled

5. **Node Pool**
   - Machine type: `e2-medium`
   - Initial nodes: 1
   - Disk size: 15GB

### Terraform Variables

```hcl
variable "project" {
  default = "devsecops-project-111"
}

variable "region" {
  default = "us-central1"
}

variable "zone" {
  default = "us-central1-a"
}

variable "K8s_version" {
  default = "1.31.6-gke.1020000"
}
```

### Running Terraform Commands

```bash
# Initialize Terraform
terraform init

# Plan changes
terraform plan

# Apply changes
terraform apply --auto-approve

# Destroy infrastructure
terraform destroy --auto-approve
```

---

## 📦 Helm Deployment

### Helm Chart Structure

```yaml
name: helm
version: 0.1.0
appVersion: 1.16.0
type: application
```

### Helm Configuration

**values.yaml:**
```yaml
test:
  deploymentName: hello-deployment
  image: us-central1-docker.pkg.dev/devsecops-project-111/docker-repo/hello:1.0.30
  replicas: 2
  serviceName: hello-service
  servicetype: NodePort
```

### Helm Templates

#### **Deployment Template** (`deployment.yaml`)
- Defines Kubernetes Deployment
- 2 replicas with auto-scaling
- Readiness and liveness probes
- Container port: 80

#### **Service Template** (`service.yaml`)
- Service type: NodePort
- Exposes port 80
- Selector: `app: test`

### Deploying with Helm

```bash
# Add Helm repository (if using external repos)
helm repo add <repo-name> <repo-url>

# Install Helm chart
helm install hello helm/ -n default

# Upgrade Helm chart
helm upgrade hello helm/ -n default

# Uninstall Helm chart
helm uninstall hello -n default

# List releases
helm list
```

### Verifying Deployment

```bash
# Check Deployment
kubectl get deployments
kubectl describe deployment hello-deployment

# Check Pods
kubectl get pods
kubectl logs -f <pod-name>

# Check Service
kubectl get svc
kubectl describe svc hello-service

# Access application
kubectl port-forward svc/hello-service 8000:80
# Visit http://localhost:8000
```

---

## 🔒 Security Scanning

### 1. **Trivy - Filesystem Scanning**

Scans application source code for vulnerable dependencies:

```bash
trivy fs app/
```

**Output:** List of CVEs found in dependencies

### 2. **SonarQube - Code Quality Analysis**

Performs static code analysis for:
- Code smells
- Bugs
- Vulnerabilities
- Security hotspots
- Code coverage

```bash
sonar-scanner \
  -Dsonar.projectKey=app \
  -Dsonar.projectName=app \
  -Dsonar.sources=./app
```

### 3. **OWASP Dependency Check**

Scans dependencies against the NVD (National Vulnerability Database):

```bash
dependency-check.sh --scan ./app --disableYarnAudit --disableNodeAudit
```

### 4. **Trivy - Container Image Scanning**

Scans the built Docker image for vulnerabilities:

```bash
trivy image us-central1-docker.pkg.dev/devsecops-project-111/docker-repo/hello:1.0.30
```

---

## 🔄 CI/CD Workflow

### Typical Workflow

1. **Developer commits code** to the GitHub repository
2. **GitHub webhook triggers** Jenkins pipeline
3. **Pipeline executes:**
   - Checkout latest code
   - Run security scans (Trivy FS, SonarQube, OWASP)
   - Build Docker image
   - Scan Docker image (Trivy)
   - Push image to Artifact Registry
   - Update Helm chart with new image tag
   - Commit updated chart back to GitHub
4. **ArgoCD detects** the updated Helm chart
5. **ArgoCD deploys** the new version to GKE
6. **Monitoring & logging** track the deployment

### GitHub Webhook Setup

1. Go to GitHub repository settings
2. Navigate to **Webhooks**
3. Add new webhook:
   - **Payload URL:** `http://<JENKINS_IP>:8080/github-webhook/`
   - **Content type:** `application/json`
   - **Events:** `Push events`
4. Save and test

---

## 🚀 Application

The application is a simple web server running Nginx with a custom HTML page (GCP Cloud HUB).

### Dockerfile

```dockerfile
FROM nginx:alpine
RUN rm -rf /usr/share/nginx/html/*
COPY index.html /usr/share/nginx/html/index.html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

### Building Locally

```bash
cd app
docker build -t hello:latest .
docker run -p 8080:80 hello:latest
# Visit http://localhost:8080
```

---

## 🔧 Troubleshooting

### Jenkins Pipeline Fails

**Issue:** Pipeline job fails to start
**Solution:**
- Check Jenkins logs: `/var/lib/jenkins/logs/jenkins.log`
- Verify credentials are configured correctly
- Check GitHub webhook is reachable

### Terraform Apply Fails

**Issue:** `Error: Error creating GKE cluster`
**Solution:**
```bash
# Enable required GCP APIs
gcloud services enable container.googleapis.com
gcloud services enable compute.googleapis.com
gcloud services enable artifactregistry.googleapis.com
```

### Pod CrashLoopBackOff

**Issue:** Kubernetes pods keep crashing
**Solution:**
```bash
# Check pod logs
kubectl logs <pod-name>

# Describe pod for events
kubectl describe pod <pod-name>

# Check resource limits
kubectl top pods
```

### Docker Push Fails

**Issue:** `unauthorized: authentication required`
**Solution:**
```bash
# Authenticate Docker with GCP
gcloud auth configure-docker us-central1-docker.pkg.dev

# Or use service account
cat SERVICE_ACCOUNT_KEY.json | docker login -u _json_key --password-stdin https://us-central1-docker.pkg.dev
```

### Cannot Connect to GKE

**Issue:** `Unable to connect to the server`
**Solution:**
```bash
# Refresh GKE credentials
gcloud container clusters get-credentials terraform-gke-cluster --region us-central1

# Verify kubectl config
kubectl config get-contexts
kubectl cluster-info
```

---

## 📊 Monitoring & Logging

### Prometheus (Metrics Collection)

Installed in Kubernetes cluster to collect metrics from:
- GKE nodes
- Pod CPU/Memory usage
- Application metrics

### Grafana (Visualization)

Provides dashboards for:
- Cluster health
- Application performance
- Resource utilization
- Custom application metrics

### Setup Prometheus & Grafana

```bash
# Add Prometheus Helm repository
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

# Install Prometheus
helm install prometheus prometheus-community/kube-prometheus-stack -n monitoring --create-namespace

# Access Grafana
kubectl port-forward -n monitoring svc/prometheus-grafana 3000:80
# Default credentials: admin / prom-operator
```

---

## 🌐 Ingress & SSL/TLS

### Ingress Controller Setup

```bash
# Install Nginx Ingress Controller
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm install ingress-nginx ingress-nginx/ingress-nginx -n ingress-nginx --create-namespace
```

### Cert-Manager for SSL/TLS

```bash
# Install Cert-Manager
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.14.0/cert-manager.yaml

# Create ClusterIssuer for Let's Encrypt
kubectl apply -f - <<EOF
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: admin@example.com
    privateKeySecretRef:
      name: letsencrypt-prod
    solvers:
    - http01:
        ingress:
          class: nginx
EOF
```

### Create Ingress Resource

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
  annotations:
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
spec:
  ingressClassName: nginx
  tls:
  - hosts:
    - app.example.com
    secretName: app-tls
  rules:
  - host: app.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: hello-service
            port:
              number: 80
```

---

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add some amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

### Code Standards

- Follow Google Cloud best practices
- Use proper Terraform formatting
- Include comments in complex sections
- Test changes before committing

---

## 👤 Author

**Bikram Singh**
- Email: bikram23march@gmail.com
- GitHub: [@bikram-singh](https://github.com/bikram-singh)
- Medium: [@bikram23march](https://medium.com/@bikram23march)

---

## 📄 License

This project is licensed under the MIT License - see the LICENSE file for details.

---

## 📚 Additional Resources

### Official Documentation

- [GKE Documentation](https://cloud.google.com/kubernetes-engine/docs)
- [Terraform GCP Provider](https://registry.terraform.io/providers/hashicorp/google/latest/docs)
- [Helm Documentation](https://helm.sh/docs/)
- [Jenkins Documentation](https://www.jenkins.io/doc/)
- [ArgoCD Documentation](https://argo-cd.readthedocs.io/)

### Related Articles

- [Building a DevSecOps Pipeline with Jenkins (Medium)](https://medium.com/@bikram23march)
- [GitHub Actions DevSecOps Pipeline on GCP](https://medium.com/@bikram23march/building-a-complete-devsecops-ci-cd-pipeline-on-gcp-with-github-actions-terraform-argocd-8e98a4ce8099)

### Tools

- [Trivy Scanner](https://github.com/aquasecurity/trivy)
- [SonarQube](https://www.sonarqube.org/)
- [OWASP Dependency-Check](https://owasp.org/www-project-dependency-check/)
- [Prometheus](https://prometheus.io/)
- [Grafana](https://grafana.com/)

---

## ⚠️ Important Notes

1. **Security**: Ensure credentials and secrets are stored securely in Jenkins
2. **Costs**: Monitor GCP billing as GKE cluster incurs charges
3. **Cleanup**: Use `terraform destroy` to remove resources when not in use
4. **Backups**: Maintain backups of critical configurations
5. **Updates**: Keep Jenkins, Kubernetes, and tools updated for security patches

---

## 🎯 Next Steps

1. Clone this repository
2. Follow the Setup Instructions
3. Configure Jenkins with credentials
4. Deploy the GKE infrastructure
5. Trigger the first pipeline run
6. Monitor the deployment
7. Integrate monitoring and logging
8. Customize for your application

---

**Happy DevSecOps! 🚀**

For questions or issues, please open a GitHub issue or contact the author.

---

*Last Updated: May 2026*
*Version: 1.0.0*

