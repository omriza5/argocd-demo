# Currency Converter - GitOps with ArgoCD

Full-stack currency converter application deployed using GitOps with ArgoCD on Kubernetes.

**Tech Stack:**

- Backend: Flask (Python) - Port 8000
- Frontend: Vite + React - Port 80
- Deployment: Kubernetes + ArgoCD

---

## 📋 Prerequisites

### macOS

```bash
# Install Homebrew
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Install required tools
brew install --cask docker        # Docker Desktop (includes Kubernetes)
brew install kubectl              # Kubernetes CLI
brew install minikube            # Local Kubernetes cluster
brew install git                 # Version control
```

### Windows

```powershell
# Open PowerShell as Administrator

# Install Chocolatey
Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))

# Install required tools
choco install docker-desktop -y   # Docker Desktop (includes Kubernetes)
choco install kubernetes-cli -y   # kubectl
choco install minikube -y        # Local Kubernetes cluster
choco install git -y             # Version control

# Restart terminal after installation
```

---

## 🚀 Quick Start

### 1. Start Kubernetes Cluster

```bash
# Start Minikube
minikube start --cpus=4 --memory=8192

# Verify it's running
kubectl get nodes
```

### 2. Install ArgoCD

```bash
# Create namespace and install ArgoCD
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Wait for ArgoCD to be ready (takes 2-3 minutes)
kubectl wait --for=condition=ready pod -l app.kubernetes.io/name=argocd-server -n argocd --timeout=300s
```

### 3. Access ArgoCD Dashboard

```bash
# Port forward (keep this terminal open)
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

**Open browser:** https://localhost:8080

**Get password:**

```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 --decode && echo
```

**Login:**

- Username: `admin`
- Password: (from command above)

### 4. Deploy Applications

```bash
# Clone this repo
git clone https://github.com/omriza5/argocd-demo.git
cd argocd-demo

# Deploy backend and frontend
kubectl apply -f applications/application-server.yaml
kubectl apply -f applications/application-client.yaml

# Wait for pods to be ready
kubectl get pods -n myapp -w
```

Press `Ctrl+C` when all pods show `Running` status.

### 5. Access the Application

**Frontend:**

```bash
kubectl port-forward svc/myapp-client -n myapp 3000:80
```

Open: http://localhost:3000

**Backend API:**

```bash
kubectl port-forward svc/myapp-server -n myapp 8000:8000
```

Test: http://localhost:8000

---

## 🔧 Local Development (Without Kubernetes)

### Backend

```bash
# Clone and setup
git clone https://github.com/omriza5/converter-be.git
cd converter-be

# Mac/Windows with Python 3
python -m venv venv
source venv/bin/activate          # Mac
.\venv\Scripts\activate           # Windows

pip install -r requirements.txt
python app.py
```

Runs on: http://localhost:8000

### Frontend

```bash
# Clone and setup
git clone https://github.com/omriza5/converter-fe.git
cd converter-fe

npm install
npm run dev
```

Runs on: http://localhost:5173

---

## 🐳 Docker (Alternative)

```bash
# Build images
docker build -t omriza5/cr-server:latest ./converter-be
docker build -t omriza5/cr-client:latest ./converter-fe

# Run containers
docker run -d -p 8000:8000 omriza5/cr-server:latest
docker run -d -p 3000:80 omriza5/cr-client:latest
```

---

## 🔍 Troubleshooting

### Check Pod Status

```bash
kubectl get pods -n myapp
kubectl logs -l app=myapp-server -n myapp
kubectl logs -l app=myapp-client -n myapp
```

### Restart Deployment

```bash
kubectl rollout restart deployment myapp-server -n myapp
kubectl rollout restart deployment myapp-client -n myapp
```

### Image Pull Issues (Mac M1/M2)

```bash
# Build multi-architecture images
docker buildx build --platform linux/amd64,linux/arm64 -t omriza5/cr-server:latest --push .
```

### Port Already in Use

```bash
# Mac
lsof -ti:8080 | xargs kill -9

# Windows
netstat -ano | findstr :8080
taskkill /PID <PID> /F
```

---

## 🧹 Cleanup

```bash
# Delete applications
kubectl delete -f applications/

# Delete namespace
kubectl delete namespace myapp

# Uninstall ArgoCD
kubectl delete namespace argocd

# Stop Minikube
minikube stop
minikube delete
```

---

## 📚 Useful Commands

```bash
# View all resources
kubectl get all -n myapp

# Check ArgoCD apps
kubectl get applications -n argocd

# View logs
kubectl logs -f <pod-name> -n myapp

# Restart pods
kubectl delete pod <pod-name> -n myapp
```

---

## 📁 Repositories

- **GitOps**: https://github.com/omriza5/argocd-demo
- **Backend**: https://github.com/omriza5/converter-be
- **Frontend**: https://github.com/omriza5/converter-fe

---

**That's it! Your app is now running with GitOps! 🚀**
