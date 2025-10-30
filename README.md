# 🔐 Jenkins + Trivy + SBOM + Private GKE CI/CD Pipeline  
_A secure modern DevSecOps pipeline built by Dhiraj_

### ✅ What this Project Does
This repository implements an **end-to-end CI/CD pipeline** using:
- **Jenkins** (running on GCP VM)
- **Trivy** → Vulnerability Scanning + SBOM Generation
- **GKE (Private cluster)** → Deployment stage
- **Container Security Best Practices**
- **Zero Public Exposure** using:
  - **Private Pod access**
  - **Bastion SSH Tunnel**
  - **ClusterIP only**
  - **Firewall IP restrictions**

Your deployed app is only accessible from:
✅ Jenkins master VM  
✅ Your local system (via SSH tunneling)  
❌ Not reachable publicly

---

## 🏗️ Architecture

```mermaid
flowchart LR

A(Developer-Local-System)
B(Jenkins-Master-VM)
C(GKE-Private-Cluster)
D(Python-App-Pod)
E(Container-Registry)
F(Trivy-SBOM)

A -->|SSH-Tunnel-9090| B
B -->|Deploy-via-kubectl| C
C --> D
B -->|Push-Image| E
E -->|Scan| F


📦 Tech Stack
Component	Tech
CI	Jenkins Freestyle + Jenkinsfile
Security	Trivy (Vulnerability Scan + SBOM)
Git	GitHub
Container	Docker
Deployment	Private GKE Cluster
Private Access	SSH Bastion + Port-forward
Monitoring	kubectl logs


📁 Folder Structure
.
├── Python-App/
│   ├── app.py
│   ├── requirements.txt
│   ├── Dockerfile
│   └── k8s/
│       ├── deployment.yaml
│       └── service-clusterip.yaml
└── Jenkinsfile


🚀 CI/CD Workflow
Stage	Action
1️⃣ SCM Checkout	Pull latest code from GitHub
2️⃣ Build Image	Docker build + Tag
3️⃣ Security Scan	Trivy vulnerability scan
4️⃣ SBOM Export	Trivy --format cyclonedx
5️⃣ Push Image	To Artifact Registry
6️⃣ Deploy	kubectl apply to GKE
✅ Result	App runs privately inside cluster


🔧 Jenkins Prerequisites

✅ Plugins installed:
Kubernetes CLI
Docker
Git
Credentials Binding

✅ Credentials required:

    ID	              Type	                Used for
gcp-sa-key	       Secret File	       GCP Authentication
docker-auth	    Username/Password	      Artifact Registry
github-creds	     GitHub Token	          SCM pull/push


🐳 Docker Build (Manual Test)
docker build -t python-app:latest .
docker run -p 5000:5000 python-app:latest


🔒 Private-Only Access (No Public Exposure)
✅ Setup SSH Tunnel from Local → Jenkins VM → GKE
ssh -i ~/.ssh/jenkins-master -L 9090:127.0.0.1:9090 dhiraj_shivade1@<Jenkins-VM-EXTERNAL-IP>

Keep SSH session running!

Now open in browser:
http://localhost:9090


✅ Only YOU can see the application
✅ Not accessible on the internet

🧪 Validate Private App
curl http://localhost:9090


Output should be:
hello from private-gke app


✅ Security Hardening Used
Security Feature	Status
Private GKE cluster	✅
Only ClusterIP service	✅
No ingress / LB exposed	✅
SSH Tunnel access only	✅
SBOM for compliance	✅
Image vulnerability scanning	✅
📌 Future Enhancements

✅ Add Network Policies
✅ Workload Identity instead of SA key
✅ Role-based pod access for additional users
✅ Monitoring with Prometheus + Grafana

🧑‍💻 Author
Dhiraj
DevOps | Cloud | Security Engineer
📌 GitHub: https://github.com/Dhiraj-Shivade

📜 License
This project is licensed under the MIT License.

⭐ If this project helped you — please star the repo! ⭐
git clone https://github.com/Dhiraj-Shivade/Jenkins-Trivy-SBOM-GKE-CI-CD.git
cd Jenkins-Trivy-SBOM-GKE-CI-CD





