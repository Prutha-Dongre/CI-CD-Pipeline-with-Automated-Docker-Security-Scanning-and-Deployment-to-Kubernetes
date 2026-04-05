# CI-CD-Pipeline-with-Automated-Docker-Security-Scanning-and-Deployment-to-Kubernetes

## 📌 Project Overview

This project demonstrates a **DevSecOps CI/CD pipeline** where Docker images are scanned for vulnerabilities before being deployed to Kubernetes.
Only secure images are allowed to proceed to deployment.

---

## ⚙️ Tech Stack

* Jenkins (CI/CD)
* Docker (Containerization)
* Kubernetes (Minikube)
* Trivy (Security Scanner)
* AWS EC2

---

## 🔐 Security Gating Logic

The pipeline integrates **Trivy** to enforce security:

* Docker image is scanned before deployment
* Pipeline fails if **CRITICAL vulnerabilities** are detected
* Pipeline proceeds only if no critical vulnerabilities are found

```bash
trivy image --severity CRITICAL --exit-code 1 <image>
```

👉 This ensures insecure images are blocked from deployment.

---

## 🔄 CI/CD Workflow (Step-by-Step)

### 1️⃣ Clone Code

Jenkins pulls code from GitHub repository

```bash
git clone <repo-url>
```

---

### 2️⃣ Build Docker Image

```bash
docker build -t <image-name>:<tag> .
```

---

### 3️⃣ Security Scan (Trivy)

```bash
trivy image --severity CRITICAL --exit-code 1 <image-name>:<tag>
```

✔ If vulnerabilities found → ❌ Pipeline FAIL
✔ If clean → ✅ Continue

---

### 4️⃣ Push Image to DockerHub

```bash
docker push <image-name>:<tag>
```

---

### 5️⃣ Deploy to Kubernetes

```bash
kubectl apply -f k8s/
```

---

### 6️⃣ Verify Deployment

```bash
kubectl get pods
kubectl get svc
```

---

## ☸️ Kubernetes Configuration

* Deployment created using `deployment.yaml`
* Service exposed using `NodePort`
* Application runs inside Kubernetes cluster

---

## 🌐 Accessing the Application

### 🔹 Option Used (Recommended for Demo)

```bash
kubectl port-forward service/myapp-service 8080:80
```

Open in browser:

```
http://localhost:8080
```

---

### 🔹 Alternative (NodePort)

```
http://<EC2-IP>:30007
```

⚠ Requires proper network configuration

---

## 📂 Project Structure

```
my-devsecops-app/
├── app.js
├── Dockerfile
├── Jenkinsfile
├── package.json
├── README.md
└── k8s/
    ├── deployment.yaml
    └── service.yaml
```

---

## 📸 Scan Report Sample

Include a screenshot from Jenkins console showing:

* Image name
* Vulnerabilities = 0
* Secrets = 0

---

## ✅ Final Output

* ✔ Jenkins Pipeline → SUCCESS
* ✔ Docker Image → Built & Pushed
* ✔ Trivy Scan → 0 Vulnerabilities
* ✔ Kubernetes → Deployment Running
* ✔ Application → Accessible in browser

---

## 🧠 Key Learnings

* Implemented secure CI/CD pipeline (DevSecOps)
* Automated vulnerability scanning using Trivy
* Integrated Docker with Jenkins pipeline
* Deployed application on Kubernetes
* Enforced security gating before deployment

---

## 🚀 Future Improvements

* Use Kubernetes Ingress for production access
* Deploy on AWS EKS
* Use Helm charts for better management
* Integrate SonarQube for code quality analysis

---

## 📌 Conclusion

This project demonstrates how to build a **secure and automated CI/CD pipeline** where only vulnerability-free Docker images are deployed to Kubernetes, ensuring safer production environments.
