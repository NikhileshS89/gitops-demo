steps to do this project you can refer the pdf or the cmds here
--------------
Step 1: Prepare your Local Directory
Open a terminal (PowerShell) and create your project structure.

PowerShell

mkdir gitops-demo
cd gitops-demo
mkdir -p helm/nginx/templates
Step 2: Create the Helm Files
You need to create the files exactly as defined in your lab. Use Notepad or VS Code to create these three files inside the gitops-demo folder:

File 1: helm/nginx/Chart.yaml

YAML

apiVersion: v2
name: nginx
description: A Helm chart for Kubernetes
version: 0.1.0
appVersion: "1.25"
File 2: helm/nginx/values.yaml

YAML

replicaCount: 3
image:
  repository: nginx
  tag: "1.25"
File 3: helm/nginx/templates/deployment.yaml

YAML

apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      app: {{ .Release.Name }}
  template:
    metadata:
      labels:
        app: {{ .Release.Name }}
    spec:
      containers:
      - name: nginx
        image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
        ports:
        - containerPort: 80
Step 3: Push to GitHub
Argo CD needs to "read" these files from the cloud.

Go to GitHub and create a Public repository named gitops-demo.

Run these commands in your local folder:

PowerShell

git init
git add .
git commit -m "Initial Helm chart"
git branch -M main
git remote add origin https://github.com/<YOUR_GITHUB_USERNAME>/gitops-demo.git
git push -u origin main
Step 4: Create the Argo Application (argo-app.yaml)
This file tells Argo CD: "Watch my GitHub repo and deploy what's inside to Minikube."

Create argo-app.yaml in your root gitops-demo folder:

YAML

apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: nginx-demo
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/<YOUR_GITHUB_USERNAME>/gitops-demo
    targetRevision: HEAD
    path: helm/nginx
  destination:
    server: https://kubernetes.default.svc
    namespace: default
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
Apply it:

PowerShell

kubectl apply -f argo-app.yaml
Step 5: Observe in the GUI
Go back to https://localhost:8081.

You will see a new card named nginx-demo.

Click it. You should see a tree diagram showing the Application connecting to a Deployment, which connects to 3 Pods.

Step 6: Test Self-Healing (The GitOps "Magic")
To prove Git is the owner, try to break the cluster manually:

Manual Delete: ```powershell kubectl delete pod -l app=nginx-demo

*Watch the GUI: Argo CD will immediately spin up new pods.*
Manual Scale (Drift):

PowerShell

kubectl scale deployment nginx-demo --replicas=1
Watch the GUI: Within seconds, Argo CD will see the "Out of Sync" status and scale it back up to 3 replicas automatically.

Step 7: Update via Git (The Correct Way)
If you want to change your app (e.g., change 3 replicas to 2):

Edit helm/nginx/values.yaml and change replicaCount: 2.

Commit and Push:

PowerShell

git add .
git commit -m "Scale down to 2"
git push
Refresh the Argo CD GUI (or wait for the auto-poll). The cluster will update itself to 2 pods.
