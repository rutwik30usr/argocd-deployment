# argocd-deployment

below is a complete real-world DevOps project pipeline using

GitHub → Jenkins → Docker → ECR → EKS → ArgoCD

This architecture is very commonly used in companies and also asked in DevOps interviews.

DevOps CI/CD Project Architecture
4

Pipeline flow:

Developer
   │
   ▼
GitHub (Application Code)
   │
   ▼
Jenkins Pipeline
   │
   ├── Build Docker Image
   ├── Push Image to AWS ECR
   │
   ▼
Update Kubernetes Manifest in Git
   │
   ▼
ArgoCD detects change
   │
   ▼
Deploy to EKS Cluster
1️⃣ Project Structure

Repository example:

devops-project
│
├── app
│   ├── app.py
│   ├── requirements.txt
│   └── Dockerfile
│
├── k8s
│   ├── deployment.yaml
│   └── service.yaml
│
└── Jenkinsfile
2️⃣ Sample Application

app/app.py

from flask import Flask
app = Flask(__name__)

@app.route("/")
def home():
    return "Hello from DevOps CI/CD Pipeline!"

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)
3️⃣ Requirements File

requirements.txt

flask
4️⃣ Dockerfile

Dockerfile

FROM python:3.9

WORKDIR /app

COPY . .

RUN pip install -r requirements.txt

EXPOSE 5000

CMD ["python","app.py"]
5️⃣ Kubernetes Deployment

k8s/deployment.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: devops-demo
spec:
  replicas: 2
  selector:
    matchLabels:
      app: devops-demo
  template:
    metadata:
      labels:
        app: devops-demo
    spec:
      containers:
      - name: devops-demo
        image: <AWS_ACCOUNT_ID>.dkr.ecr.<REGION>.amazonaws.com/devops-demo:latest
        ports:
        - containerPort: 5000
6️⃣ Kubernetes Service

k8s/service.yaml

apiVersion: v1
kind: Service
metadata:
  name: devops-service
spec:
  type: LoadBalancer
  selector:
    app: devops-demo
  ports:
  - port: 80
    targetPort: 5000
7️⃣ Create AWS ECR Repository
aws ecr create-repository \
--repository-name devops-demo \
--region ap-south-1

Login to ECR

aws ecr get-login-password \
--region ap-south-1 | docker login \
--username AWS \
--password-stdin <AWS_ACCOUNT_ID>.dkr.ecr.ap-south-1.amazonaws.com
8️⃣ Jenkins Pipeline

Jenkinsfile

pipeline {
    agent any

    environment {
        AWS_REGION = "ap-south-1"
        ECR_REPO = "<AWS_ACCOUNT_ID>.dkr.ecr.ap-south-1.amazonaws.com/devops-demo"
        IMAGE_TAG = "latest"
    }

    stages {

        stage('Clone Repo') {
            steps {
                git 'https://github.com/yourusername/devops-project.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $ECR_REPO:$IMAGE_TAG .'
            }
        }

        stage('Login to ECR') {
            steps {
                sh '''
                aws ecr get-login-password --region $AWS_REGION | docker login \
                --username AWS \
                --password-stdin $ECR_REPO
                '''
            }
        }

        stage('Push Docker Image') {
            steps {
                sh 'docker push $ECR_REPO:$IMAGE_TAG'
            }
        }

    }
}
9️⃣ Create EKS Cluster

Using eksctl

eksctl create cluster \
--name devops-cluster \
--region ap-south-1

Check nodes

kubectl get nodes
🔟 Install ArgoCD on EKS
kubectl create namespace argocd
kubectl apply -n argocd \
-f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

Check pods

kubectl get pods -n argocd
1️⃣1️⃣ Access ArgoCD
kubectl port-forward svc/argocd-server -n argocd 8080:443

Open

https://localhost:8080

Password

kubectl -n argocd get secret argocd-initial-admin-secret \
-o jsonpath="{.data.password}" | base64 -d
1️⃣2️⃣ Create ArgoCD Application
argocd app create devops-demo \
--repo https://github.com/yourusername/devops-project.git \
--path k8s \
--dest-server https://kubernetes.default.svc \
--dest-namespace default

Deploy

argocd app sync devops-demo
1️⃣3️⃣ Verify Deployment
kubectl get pods
kubectl get svc

You will get LoadBalancer URL

Open browser:

http://<EXTERNAL-IP>
🔥 How CI/CD Works
Step 1

Developer pushes code to GitHub

Step 2

Jenkins pipeline triggers

Step 3

Docker image built

Step 4

Image pushed to AWS ECR

Step 5

Update Kubernetes manifest in Git

Step 6

ArgoCD detects change

Step 7

ArgoCD deploys to EKS

Real DevOps Tools Used
Stage	Tool
Source Code	GitHub
CI	Jenkins
Containerization	Docker
Registry	AWS ECR
Orchestration	AWS EKS
Deployment	ArgoCD
