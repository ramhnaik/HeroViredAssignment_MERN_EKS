To deploy a MERN application using Azure Kubernetes Service (AKS), follow these steps:

### Step 1: Set Up Azure CLI and AKS

1. **Install Azure CLI:**
   ```sh
   curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
   ```

2. **Log in to Azure:**
   ```sh
   az login
   ```

3. **Create a Resource Group:**
   ```sh
   az group create --name myResourceGroup --location eastus
   ```

4. **Create an AKS Cluster:**
   ```sh
   az aks create --resource-group myResourceGroup --name myAKSCluster --node-count 1 --enable-addons monitoring --generate-ssh-keys
   ```

5. **Get AKS Credentials:**
   ```sh
   az aks get-credentials --resource-group myResourceGroup --name myAKSCluster
   ```

### Step 2: Prepare the MERN Application

1. **Clone the Repository:**
   ```sh
   git clone https://github.com/UnpredictablePrashant/SampleMERNwithMicroservices.git
   cd SampleMERNwithMicroservices
   ```

2. **Create Dockerfiles for Frontend and Backend:**
   - `Dockerfile` for Backend:
     ```dockerfile
     FROM node:14
     WORKDIR /app
     COPY package*.json ./
     RUN npm install
     COPY . .
     EXPOSE 5000
     CMD ["npm", "start"]
     ```

   - `Dockerfile` for Frontend:
     ```dockerfile
     FROM node:14
     WORKDIR /app
     COPY package*.json ./
     RUN npm install
     COPY . .
     EXPOSE 3000
     CMD ["npm", "start"]
     ```

3. **Build Docker Images:**
   ```sh
   docker build -t mern-backend:latest -f backend/Dockerfile .
   docker build -t mern-frontend:latest -f frontend/Dockerfile .
   ```

4. **Push Docker Images to Azure Container Registry (ACR):**
   - Create ACR:
     ```sh
     az acr create --resource-group myResourceGroup --name myACR --sku Basic
     ```

   - Log in to ACR:
     ```sh
     az acr login --name myACR
     ```

   - Tag and Push Images:
     ```sh
     docker tag mern-backend:latest myacr.azurecr.io/mern-backend:latest
     docker tag mern-frontend:latest myacr.azurecr.io/mern-frontend:latest
     docker push myacr.azurecr.io/mern-backend:latest
     docker push myacr.azurecr.io/mern-frontend:latest
     ```

### Step 3: Deploy to AKS

1. **Create Kubernetes Deployment and Service Files:**
   - `backend-deployment.yaml`:
     ```yaml
     apiVersion: apps/v1
     kind: Deployment
     metadata:
       name: backend
     spec:
       replicas: 1
       selector:
         matchLabels:
           app: backend
       template:
         metadata:
           labels:
             app: backend
         spec:
           containers:
           - name: backend
             image: myacr.azurecr.io/mern-backend:latest
             ports:
             - containerPort: 5000
     ```

   - `backend-service.yaml`:
     ```yaml
     apiVersion: v1
     kind: Service
     metadata:
       name: backend
     spec:
       selector:
         app: backend
       ports:
         - protocol: TCP
           port: 5000
           targetPort: 5000
       type: LoadBalancer
     ```

   - `frontend-deployment.yaml`:
     ```yaml
     apiVersion: apps/v1
     kind: Deployment
     metadata:
       name: frontend
     spec:
       replicas: 1
       selector:
         matchLabels:
           app: frontend
       template:
         metadata:
           labels:
             app: frontend
         spec:
           containers:
           - name: frontend
             image: myacr.azurecr.io/mern-frontend:latest
             ports:
             - containerPort: 3000
     ```

   - `frontend-service.yaml`:
     ```yaml
     apiVersion: v1
     kind: Service
     metadata:
       name: frontend
     spec:
       selector:
         app: frontend
       ports:
         - protocol: TCP
           port: 3000
           targetPort: 3000
       type: LoadBalancer
     ```

2. **Deploy to AKS:**
   ```sh
   kubectl apply -f backend-deployment.yaml
   kubectl apply -f backend-service.yaml
   kubectl apply -f frontend-deployment.yaml
   kubectl apply -f frontend-service.yaml
   ```

### Step 4: Documentation

1. **Create Documentation:**
   - Document each step with commands and screenshots.
   - Save the documentation in a file named `deployment_guide.md`.

2. **Push Documentation to GitHub:**
   ```sh
   git add deployment_guide.md
   git commit -m "Add deployment guide for MERN application on AKS"
   git push origin main
   ```

### Step 5: Validate Deployment

1. **Get Service IPs:**
   ```sh
   kubectl get services
   ```

2. **Access the Application:**
   - Use the external IPs of the frontend and backend services to access the application.
