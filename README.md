<img width="1366" height="768" alt="Screenshot (9)" src="https://github.com/user-attachments/assets/5a33dd30-88e3-4d7e-b9dc-9319ddccd6fe" /># 🔐 Jenkins + Trivy + SBOM + Private GKE CI/CD Pipeline  
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
A[Local System 🧑‍💻] -- SSH Tunnel 9090 --> B[Jenkins Master VM 🟦]
B -- kubectl deploy --> C[GKE Private Cluster 🟩]
C --> D[Python App Pod + ClusterIP SVC 🔐]
B -->|Trivy Scan| E[Artifact + SBOM 📄]

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



<img width="1366" height="768" alt="Screenshot (8)" src="https://github.com/user-attachments/assets/3ae034ff-423f-4a13-9a5e-316b6be107c7" />


<img width="1366" height="768" alt="Screenshot (9)" src="https://github.com/user-attachments/assets/8e6c7533-40c5-4886-a8c7-cc8000bbeee0" />


<img width="1366" height="768" alt="Screenshot (10)" src="https://github.com/user-attachments/assets/719b7522-5d0f-463e-91c3-d0f3c9471214" />


<img width="1366" height="768" alt="Screenshot (11)" src="https://github.com/user-attachments/assets/f43c0e00-931b-4317-90b0-21e782c7d233" />


<img width="1366" height="768" alt="Screenshot (12)" src="https://github.com/user-attachments/assets/f54f415d-e4be-4633-91da-6327c21ecef0" />


<img width="1366" height="768" alt="Screenshot (13)" src="https://github.com/user-attachments/assets/6f17bc18-b875-44a2-8e7a-cb4e0c4112cb" />


<img width="1366" height="768" alt="Screenshot (13)" src="https://github.com/user-attachments/assets/c1a41e65-ba7e-4853-9b69-8b3de65ed303" />


<img width="1366" height="768" alt="Screenshot (15)" src="https://github.com/user-attachments/assets/223d13d0-f641-44f1-963f-1653abd6de7c" />


<img width="1366" height="768" alt="Screenshot (16)" src="https://github.com/user-attachments/assets/751340e3-74c1-432a-aec0-e3b6abf110fc" />


<img width="1366" height="768" alt="Screenshot (17)" src="https://github.com/user-attachments/assets/d9f4ee71-97a4-4fb9-8489-bf5cd749b2a6" />


<img width="1366" height="768" alt="Screenshot (18)" src="https://github.com/user-attachments/assets/8709a1ca-588f-4b7a-b20f-528e3173a82c" />


<img width="1366" height="768" alt="Screenshot (19)" src="https://github.com/user-attachments/assets/97b0c462-1a51-46b3-b79d-1e14056f7fb9" />


<img width="1366" height="768" alt="Screenshot (20)" src="https://github.com/user-attachments/assets/d5b31b8a-6186-4f23-99aa-5f24e3d79ca5" />


<img width="1366" height="768" alt="Screenshot (21)" src="https://github.com/user-attachments/assets/1341a47b-9bfc-44ea-8b6f-e6132e5298cb" />


<img width="1366" height="768" alt="Screenshot (22)" src="https://github.com/user-attachments/assets/3da71cc3-2c38-49e8-a908-080c798e6f73" />


<img width="1366" height="768" alt="Screenshot (23)" src="https://github.com/user-attachments/assets/4f01cba9-ffa7-4eb9-8d63-1b76fa8cc8c1" />


<img width="1366" height="768" alt="Screenshot (24)" src="https://github.com/user-attachments/assets/6ec6494b-f072-49e7-892d-a96ef9cbc747" />


<img width="1366" height="768" alt="Screenshot (25)" src="https://github.com/user-attachments/assets/83012b45-2634-4124-b9dd-9289d9b8fb08" />


<img width="1366" height="768" alt="Screenshot (26)" src="https://github.com/user-attachments/assets/83d51662-8e7d-43ea-9e1a-3cd4d9d57d81" />


<img width="1366" height="768" alt="Screenshot (27)" src="https://github.com/user-attachments/assets/ba0745b1-3ccf-4efe-8674-248f4f98b6e9" />



