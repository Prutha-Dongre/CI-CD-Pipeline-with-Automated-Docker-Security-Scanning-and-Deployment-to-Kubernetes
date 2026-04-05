# Secure CI/CD Pipeline with Docker, Trivy & Kubernetes

## Project Overview

This project demonstrates a complete **DevSecOps CI/CD pipeline** where Docker images are scanned for vulnerabilities before being deployed to Kubernetes.
Only secure images are allowed to proceed to deployment.

---

## Tech Stack

* Jenkins (CI/CD)
* Docker (Containerization)
* Kubernetes (Minikube)
* Trivy (Security Scanner)
* AWS EC2

---

## GitHub Repository

You can find the complete source code here:
[🔗 View Source Code](https://github.com/Prutha-Dongre/my-devsecops-app)

---

## Security Gating Logic

The pipeline integrates **Trivy** to enforce security:

* Docker image is scanned before deployment
* Pipeline fails if **CRITICAL vulnerabilities** are detected
* Pipeline proceeds only if no critical vulnerabilities are found

```bash
trivy image --severity CRITICAL --exit-code 1 <image>
```

 This ensures insecure images are blocked from deployment.

---

## Detailed Implementation Steps

### 🟢 Step 1: Create Jenkins Master Server

* Launch EC2 instance (t3.micro)
* Install Java

```bash
sudo apt update
sudo apt install fontconfig openjdk-21-jre
java -version
```

* Install Jenkins

```
sudo wget -O /etc/apt/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2026.key
echo "deb [signed-by=/etc/apt/keyrings/jenkins-keyring.asc]" \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt update
sudo apt install jenkins
sudo systemctl start jenkins
sudo systemctl enable jenkins
sudo systemctl status jenkins
```

* Access Jenkins:

```
http://<EC2-IP>:8080
```

---

### 🟢 Step 2: Install Required Plugins

Go to **Manage Jenkins → Plugins**

Install:

* Git Plugin
* Pipeline Plugin
* Docker Pipeline Plugin
* SSH Build Agents Plugin

---

### 🟢 Step 3: Create Jenkins Agent (Node Server)

* Launch EC2 instance (c7i-flex.large)

Install required tools:

#### Install Java

```bash
sudo apt update
sudo apt install fontconfig openjdk-21-jre
java -version
```

#### Install Docker

```bash
sudo apt install docker.io -y
sudo systemctl start docker
sudo usermod -aG docker ubuntu
```

#### Install Trivy

```bash
sudo apt install wget apt-transport-https gnupg lsb-release -y

wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -

echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main | \
sudo tee /etc/apt/sources.list.d/trivy.list

sudo apt update
sudo apt install trivy -y
```

#### Install kubectl

```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

chmod +x kubectl
sudo mv kubectl /usr/local/bin/
```

#### Install Minikube

```bash
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64

sudo install minikube-linux-amd64 /usr/local/bin/minikube
```

---

### 🟢 Step 4: Start Minikube

```bash
minikube start --driver=docker --cpus=2 --memory=3000mb
kubectl get nodes
```

---

### 🟢 Step 5: Add Node in Jenkins

* Go to **Manage Jenkins → Nodes → New Node**
* Configure:

  * Name: agent-node
  * Labels: docker-agent
  * Launch method: SSH
* Add SSH credentials (.pem key)

---

### 🟢 Step 6: Increase Storage Volume for node server

* Go to AWS EC2 → Volumes(jenkins node server)
* Modify volume from **8GB → 20GB**

---

### 🟢 Step 7: Extend Disk

```bash
sudo growpart /dev/nvme0n1 1
sudo resize2fs /dev/nvme0n1p1

df -h
```

---

### 🟢 Step 8: Restart Node & Bring Online

```bash
sudo systemctl restart jenkins
```

* In Jenkins UI → Bring node online

---

### 🟢 Step 9: Create Pipeline Job

* Click **New Item → Pipeline**
* Add Jenkinsfile

---

### 🟢 Step 10: Create Docker Hub Access Token

* Go to Docker Hub → Security
* Create token (Read & Write)

---

### 🟢 Step 11: Add Credentials in Jenkins

* Manage Jenkins → Credentials

Add:

* Username: Docker Hub username
* Password: Access token
* ID: `docker-creds`

---

### 🟢 Step 12: Run Pipeline

Click **Build Now**

Pipeline stages:

* Clone Code
* Build Docker Image
* Scan using Trivy
* Push to Docker Hub
* Deploy to Kubernetes

---

### 🟢 Step 13: Access Application

```bash
kubectl port-forward service/myapp-service 8080:80
```

Open new terminal on your laptop (not EC2)

Run:
```
ssh -i your-key.pem -L 8080:localhost:8080 ubuntu@3.101.123.177
```

Open in browser:

```
http://localhost:8080
```

What will happen

- Your laptop connects to EC2
- EC2 connects to Kubernetes
- Kubernetes shows your app

Simple flow
```
Laptop → SSH → EC2 → Port-forward → App
```

---

## ☸️ Kubernetes Configuration

### deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: pruthadongre09/myapp:v1
        ports:
        - containerPort: 3000
```

---

### service.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  type: NodePort
  selector:
    app: myapp
  ports:
    - port: 80
      targetPort: 3000
      nodePort: 30007
```

---

## Scan Report Sample

Include screenshot from Jenkins showing:

* Image name
* Vulnerabilities = 0
* Secrets = 0

---

## Project Structure

```
my-devsecops-app/
├── app.js
├── Dockerfile
├── Jenkinsfile
├── package.json
└── k8s/
    ├── deployment.yaml
    └── service.yaml
```

---

## Final Output

* Jenkins Pipeline → SUCCESS
* Docker Image → Built & Pushed
* Trivy Scan → 0 Vulnerabilities
* Kubernetes → Deployment Running
* Application → Accessible in browser

---

## Key Learnings

* Implemented secure CI/CD pipeline
* Integrated security scanning (Trivy)
* Automated deployment to Kubernetes
* Understood DevSecOps workflow

---

## Conclusion

This project demonstrates a **secure DevSecOps pipeline** ensuring only vulnerability-free Docker images are deployed to Kubernetes, improving overall system security and reliability.
