Here's a structured README for your Docker Java Kubernetes project, enhanced with emojis for clarity and engagement:

---

# 🚀 Docker Java Kubernetes Project

## 📥 Install Minikube

### 1. Install `kubectl`
```bash
curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.20.4/2021-04-12/bin/linux/amd64/kubectl
chmod +x ./kubectl
mkdir -p $HOME/bin
cp ./kubectl $HOME/bin/kubectl
export PATH=$HOME/bin:$PATH
echo 'export PATH=$HOME/bin:$PATH' >> ~/.bashrc
source $HOME/.bashrc
kubectl version --short --client
```

### 2. Install Docker
```bash
yum install docker -y
systemctl start docker
systemctl enable docker
```
- For detailed Minikube installation, refer to the [Minikube Documentation](https://minikube.sigs.k8s.io/docs/start/).

---

## 🛠️ Install EKS Setup

### Step 1: Launch EC2 Instance
- **Instance Type**: `t2.medium`

### Step 2: Create IAM Role
- **Policy**: Admin policy for EKS cluster
- **Attach**: Role to EC2 instance

### Step 3: Install `kubectl`
```bash
curl -o kubectl https://amazon-eks.s3-us-west-2.amazonaws.com/1.14.6/2019-08-22/bin/linux/amd64/kubectl
chmod +x ./kubectl
mkdir -p $HOME/bin
cp ./kubectl $HOME/bin/kubectl
export PATH=$HOME/bin:$PATH
echo 'export PATH=$HOME/bin:$PATH' >> ~/.bashrc
source $HOME/.bashrc
kubectl version --short --client
```

### Step 4: Install `eksctl`
```bash
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/bin
eksctl version
```

### Step 5: Create Master Cluster
```bash
eksctl create cluster --name=eksdemo \
                  --region=us-west-1 \
                  --zones=us-west-1b,us-west-1c \
                  --without-nodegroup 
```

### Step 6: Associate IAM OIDC Provider
```bash
eksctl utils associate-iam-oidc-provider \
    --region us-west-1 \
    --cluster eksdemo \
    --approve 
```
- This allows the service to connect with EKS.

### Step 7: Create Worker Node Group
```bash
eksctl create nodegroup --cluster=eksdemo \
                   --region=us-west-1 \
                   --name=eksdemo-ng-public \
                   --node-type=t2.medium \
                   --nodes=2 \
                   --nodes-min=2 \
                   --nodes-max=4 \
                   --node-volume-size=10 \
                   --ssh-access \
                   --ssh-public-key=Praveen-test \
                   --managed \
                   --asg-access \
                   --external-dns-access \
                   --full-ecr-access \
                   --appmesh-access \
                   --alb-ingress-access
```

### Optional Cleanup Commands
```bash
eksctl delete nodegroup --cluster=eksdemo --region=us-east-1 --name=eksdemo-ng-public
eksctl delete cluster --name=eksdemo --region=us-west-1
```

---

## 🔧 Hands-On

### Deploying Java Applications with Docker and Kubernetes

1. **Build Each Project**
   ```bash
   mvn clean install -DskipTests
   ```

2. **Create Docker Hub Account**

3. **Build the Docker Images Locally**
   ```bash
   docker build -t vishalgunjalswe/shopfront:latest .
   docker build -t vishalgunjalswe/productcatalogue:latest .
   docker build -t vishalgunjalswe/stockmanager:latest .
   ```

4. **Push Images to Docker Hub**
   ```bash
   docker push vishalgunjalswe/shopfront:latest 
   docker push vishalgunjalswe/productcatalogue:latest
   docker push vishalgunjalswe/stockmanager:latest
   ```

5. **Create Pods in Kubernetes**
   ```bash
   kubectl apply -f shopfront-service.yaml
   kubectl apply -f productcatalogue-service.yaml
   kubectl apply -f stockmanager-service.yaml
   ```

6. **Access Services via Minikube**
   ```bash
   minikube service shopfront
   minikube service productcatalogue
   minikube service stockmanager
   ```

7. **Open URLs in Browser**
   - **Order to Build and Deploy**:
     - `shopfront` → `productcatalogue` → `stockmanager`
   - **Endpoints**:
     - Product Endpoint: `/products`
     - Stock Endpoint: `/stocks`