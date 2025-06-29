# 🚀 Simple DevOps App — Docker + GitHub Actions + Azure ACR + ACI

This project demonstrates a complete CI/CD pipeline that containerizes a simple Python web application (Flask-based), builds and pushes the Docker image to **Azure Container Registry (ACR)**, and deploys it automatically to **Azure Container Instance (ACI)** using **GitHub Actions**.

---

## 🌐 Live Demo

Once deployed, access the app via:

http://<aci-dns-name>.<region>.azurecontainer.io


---

## 📦 Features

- ✅ Dockerized Flask web application
- ✅ Fully automated CI/CD with GitHub Actions
- ✅ Container registry integration using Azure Container Registry (ACR)
- ✅ Deployment using Azure Container Instance (ACI)
- ✅ Secure authentication via GitHub secrets
- ✅ Configurable, scalable & cloud-native

---

## 📁 Project Structure

<pre>
  ├── app.py # Flask application
├── Dockerfile # Docker build instructions
├── requirements.txt # Python dependencies
├── .github
│ └── workflows
│ └── ci-cd.yml # GitHub Actions workflow file
└── README.md # This file
</pre>


---

## 🐳 Docker

### Dockerfile

```Dockerfile
FROM python:3.9-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

CMD ["python", "app.py"]

Build Docker Image Locally
docker build -t simple-devops-app .



☁️ Azure Resources Used
Resource	Purpose
ACR (Azure Container Registry)	Stores the Docker images
ACI (Azure Container Instance)	Hosts the containerized web app
Resource Group	Logical container for all Azure resources

🔐 GitHub Secrets Required
Add these to your GitHub repo → Settings → Secrets:

Secret Name	Description
AZURE_CREDENTIALS	Output of az ad sp create-for-rbac ... in JSON
ACR_USERNAME	Azure Container Registry username
ACR_PASSWORD	Azure Container Registry password
ACR_SERVER	Example: flaskappactionacr.azurecr.io

You can generate AZURE_CREDENTIALS via:
az ad sp create-for-rbac --name github-deployer --role contributor \
  --scopes /subscriptions/<SUBSCRIPTION_ID>/resourceGroups/<RESOURCE_GROUP> \
  --sdk-auth

⚙️ GitHub Actions Workflow
Path: .github/workflows/ci-cd.yml

name: CI/CD - Docker + ACR + ACI

on:
  push:
    branches: [main]

env:
  RESOURCE_GROUP: "Flask-App-Action-RG"
  IMAGE_NAME: "simple-devops-app"
  ACR_NAME: "flaskappactionacr"
  ACI_NAME: "flaskappactionaci"
  LOCATION: "centralus"

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v2

      - name: Azure Login
        uses: azure/login@v1
        with:
          creds: ${{ secrets.AZURE_CREDENTIALS }}

      - name: Build Docker image
        run: |
          docker build -t $IMAGE_NAME .

      - name: Tag Docker image
        run: |
          docker tag $IMAGE_NAME:latest $ACR_NAME.azurecr.io/$IMAGE_NAME:latest

      - name: Login to Azure Container Registry
        uses: azure/docker-login@v1
        with:
          login-server: ${{ env.ACR_NAME }}.azurecr.io
          username: ${{ secrets.ACR_USERNAME }}
          password: ${{ secrets.ACR_PASSWORD }}

      - name: Push Docker image to ACR
        run: |
          docker push $ACR_NAME.azurecr.io/$IMAGE_NAME:latest

      - name: Deploy to Azure Container Instance (ACI)
        run: |
          az container create \
            --resource-group $RESOURCE_GROUP \
            --name $ACI_NAME \
            --image $ACR_NAME.azurecr.io/$IMAGE_NAME:latest \
            --cpu 1 \
            --memory 1 \
            --registry-login-server $ACR_NAME.azurecr.io \
            --registry-username ${{ secrets.ACR_USERNAME }} \
            --registry-password ${{ secrets.ACR_PASSWORD }} \
            --dns-name-label $ACI_NAME-af1n \
            --ports 80 \
            --os-type Linux \
            --location $LOCATION \
            --restart-policy Always


🚀 Deploy Manually (Optional)
You can test deploy manually if needed:

az login
az acr login --name flaskappactionacr

az container create \
  --resource-group Flask-App-Action-RG \
  --name flaskappactionaci \
  --image flaskappactionacr.azurecr.io/simple-devops-app:latest \
  --cpu 1 --memory 1 \
  --registry-login-server flaskappactionacr.azurecr.io \
  --registry-username <your-username> \
  --registry-password <your-password> \
  --dns-name-label flaskappactionaci-af1n \
  --ports 80 \
  --location centralus \
  --restart-policy Always



🧠 Learning Outcomes
By completing this project, you’ve learned:

✅ How to containerize a Python app using Docker

✅ How to push images to Azure Container Registry

✅ How to deploy containers with Azure CLI to ACI

✅ How to configure a complete GitHub Actions CI/CD pipeline

✅ How to manage secrets and automation securely

📌 Future Improvements
Use Terraform to provision ACR & ACI

Add unit testing before build stage

Add Slack/Email notifications on deploy

Switch deployment to Azure Kubernetes Service (AKS)

📄 License
MIT License. See LICENSE file for details.

🧑‍💻 Author
Af1nZR – GitHub | DevOps Engineer | Azure Cloud Engineer ☁️

---

Let me know if you'd like to customize it for a Django app, Node.js app, or include Terraform too.
