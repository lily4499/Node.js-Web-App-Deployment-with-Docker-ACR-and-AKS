#  Node.js Web App Deployment with Docker, ACR, and AKS

---




This project demonstrates how to build a Node.js web application, dockerize it, push the image to **Azure Container Registry (ACR)**, and deploy it to **Azure Kubernetes Service (AKS)**.

---

## 🌍 Real-World Scenario

You’re a DevOps engineer at a startup. Developers build a basic Node.js app and you need to:

- Dockerize the app
- Push the image to ACR
- Deploy the app to AKS
- Maintain code in GitHub

---

## 🧱 Tech Stack

- Node.js + Express
- Docker
- Azure CLI
- Azure Container Registry (ACR)
- Azure Kubernetes Service (AKS)
- Git + GitHub

---

## 📁 Project Structure

```bash
nodejs-webapp/
├── public/
│   ├── index.html
│   └── about.html
├── server.js
├── package.json
├── .gitignore
├── .dockerignore
├── Dockerfile
├── deployment.yaml
```
---
## file-setup.py

```python
import os

# Base directory
base_path = "/home/lilia/VIDEOS/nodejs-webapp"

# File structure and contents
files = {
    "public/index.html": "<h1>Welcome to Node.js Web App</h1>",
    "public/about.html": "<h1>About Page</h1>",
    "server.js": """const express = require("express");
const app = express();
const path = require("path");

app.use(express.static("public"));

app.get("/", (req, res) => res.sendFile(path.join(__dirname, "public/index.html")));
app.get("/about", (req, res) => res.sendFile(path.join(__dirname, "public/about.html")));

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => console.log(`Server running on port ${PORT}`));
""",
    "package.json": """{
  "name": "nodejs-webapp",
  "version": "1.0.0",
  "main": "server.js",
  "scripts": {
    "start": "node server.js"
  },
  "dependencies": {
    "express": "^4.18.2"
  },
  "devDependencies": {
    "mocha": "^10.0.0"
  }
}
""",
    ".gitignore": "node_modules\n.env\n",
    ".dockerignore": "node_modules\n.env\n",
    "Dockerfile": """FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install --production
COPY . .
EXPOSE 3000
CMD ["node", "server.js"]
""",
    "deployment.yaml": """apiVersion: apps/v1
kind: Deployment
metadata:
  name: nodejs-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nodejs-app
  template:
    metadata:
      labels:
        app: nodejs-app
    spec:
      containers:
        - name: nodejs-app
          image: liliacr.azurecr.io/nodejswebapp:v1
          ports:
            - containerPort: 3000
---
apiVersion: v1
kind: Service
metadata:
  name: nodejs-service
spec:
  selector:
    app: nodejs-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 3000
  type: LoadBalancer
"""
}

# Create directories and write files
for filepath, content in files.items():
    full_path = os.path.join(base_path, filepath)
    os.makedirs(os.path.dirname(full_path), exist_ok=True)
    with open(full_path, "w") as f:
        f.write(content)

"✅ All files have been created successfully in /home/lilia/VIDEOS/nodejs-webapp/"


```

---

## 🧰 Step-by-Step Instructions

### ✅ 1. Initialize Project

```bash
npm init -y
```

### ✅ 2. Install Dependencies

```bash
npm install express
npm install --save-dev mocha
```

---

### ✅ 3. Create App and Static HTML Files

#### 📄 server.js

```js
const express = require("express");
const app = express();
const path = require("path");

app.use(express.static("public"));

app.get("/", (req, res) => res.sendFile(path.join(__dirname, "public/index.html")));
app.get("/about", (req, res) => res.sendFile(path.join(__dirname, "public/about.html")));

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => console.log(`Server running on port ${PORT}`));
```

#### 📄 public/index.html

```html
<h1>Welcome to Node.js Web App</h1>
```

#### 📄 public/about.html

```html
<h1>About Page</h1>
```

---

### ✅ 4. Run App Locally

```bash
node server.js
```

Visit: `http://localhost:3000` or `/about`

---

### ✅ 5. Dockerize the App

#### 📄 Dockerfile

```Dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install --production
COPY . .
EXPOSE 3000
CMD ["node", "server.js"]
```

#### 📄 .dockerignore

```
node_modules
.env
```

---

### ✅ 6. Build Docker Image

```bash
docker build -t lily4499/nodejswebapp:latest .
```

---

### ✅ 7. Push Docker Image to ACR

#### 🔐 Login to Azure

```bash
az login
```

## Or use Azure Service Principal (SP) Interacts with ACR and AKS
A Service Principal (SP) is an identity used to authenticate and authorize applications or automation scripts to access Azure resources like:
🧱 Azure Container Registry (ACR) — to push/pull container images
☸️ Azure Kubernetes Service (AKS) — to manage or deploy apps via kubectl

a - Create Azure Service Principal
📄 azure/create-sp.sh
```bash
#!/bin/bash

SP_NAME="acr-aks-sp"
ACR_NAME="liliacr"
SUBSCRIPTION_ID=$(az account show --query id -o tsv)

az ad sp create-for-rbac \
  --name $SP_NAME \
  --role="Contributor" \
  --scopes="/subscriptions/$SUBSCRIPTION_ID" \
  --sdk-auth > sp-credentials.json

echo "✅ Service Principal created. Credentials saved in sp-credentials.json"

```
💡 This sp-credentials.json can be used for CI/CD secrets.


b - Set Environment Variables Using SP
📄 azure/set-env.sh
```bash
#!/bin/bash

# Load credentials
export AZURE_CLIENT_ID="<your-sp-client-id>"
export AZURE_CLIENT_SECRET="<your-sp-client-secret>"
export AZURE_TENANT_ID="<your-tenant-id>"
export AZURE_SUBSCRIPTION_ID="<your-subscription-id>"

# Login using Service Principal
az login --service-principal \
  --username "$AZURE_CLIENT_ID" \
  --password "$AZURE_CLIENT_SECRET" \
  --tenant "$AZURE_TENANT_ID"

az account set --subscription "$AZURE_SUBSCRIPTION_ID"

```

#### 🏗️ Create Resource Group and ACR

```bash
az group create --name lili-Group --location eastus
az acr create --resource-group lili-Group --name liliacr --sku Standard --location eastus
az acr login --name liliacr
```

#### 🔖 Tag & Push Image
When using a Service Principal to push or pull images from ACR:
🛠️ Required Role: AcrPush or Contributor
✅ You must assign the Service Principal a role that allows access to the ACR.

```bash
az role assignment create \
  --assignee <SP_CLIENT_ID> \
  --scope $(az acr show --name liliacr --query id -o tsv) \
  --role AcrPush

docker tag lily4499/nodejswebapp:latest liliacr.azurecr.io/nodejswebapp:v1
docker push liliacr.azurecr.io/nodejswebapp:v1
```

#### 📦 View Image in ACR

```bash
az acr repository list --name liliacr --output table
az acr repository show-tags --name liliacr --repository nodejswebapp --output table
```

---

### ✅ 8. Create AKS Cluster
When using the SP to authenticate to Azure and deploy to AKS with kubectl:
🛠️ Required Role: Contributor on AKS resource group
✅ This lets the SP fetch AKS credentials and interact with the cluster.

```bash
az group create --name lili-Group --location eastus
az aks create --resource-group lili-Group --name liliCluster --node-count 2 --generate-ssh-keys --enable-addons monitoring
az aks get-credentials --resource-group lili-Group --name liliCluster --overwrite-existing
az aks update -n liliCluster -g lili-Group --attach-acr liliacr
```

---

### ✅ 9. Create Kubernetes Deployment

#### 📄 deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nodejs-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nodejs-app
  template:
    metadata:
      labels:
        app: nodejs-app
    spec:
      containers:
        - name: nodejs-app
          image: liliacr.azurecr.io/nodejswebapp:v1
          ports:
            - containerPort: 3000
---
apiVersion: v1
kind: Service
metadata:
  name: nodejs-service
spec:
  selector:
    app: nodejs-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 3000
  type: LoadBalancer
```

#### 📦 Apply Deployment

```bash
kubectl apply -f deployment.yaml
kubectl get svc
```

> Access the app using the external IP shown from `kubectl get svc`.

---

### ✅ 10. Create GitHub Repo

```bash
gh repo create nodejs-webapp --public --source=. --remote=origin
```

---

### ✅ 11. Push Code to GitHub

```bash
echo "node_modules\n.env" > .gitignore
git init
git add .
git commit -m "Initial commit"
git remote add origin https://github.com/<your-username>/nodejs-webapp.git
git push -u origin main
```

---

## 🧹 Clean Up

```bash
kubectl delete -f deployment.yaml
az group delete --name lili-Group --yes --no-wait
```

---

## ✅ Summary

- 🎯 App built with Express
- 📦 Dockerized and pushed to ACR
- ☁️ Deployed on AKS using `kubectl`
- 🔁 Maintained in GitHub

---

