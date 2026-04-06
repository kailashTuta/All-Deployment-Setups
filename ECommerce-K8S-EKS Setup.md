# 🚀 E-Commerce Application Deployment (CI/CD with EKS)

## 📌 Pipeline Overview

**Pipeline stages in Jenkinsfile:**

```
Code → Dockerfile → Image → Docker Hub
```

**Tools Used:**

* Kubernetes
* Docker
* Jenkins
* Git

**Cluster:**

* AWS EKS

---

## 🔀 Multibranch Pipeline Strategy

* Each branch contains:

  * Application code
  * `Jenkinsfile`
* **Main branch** contains:

  * Jenkinsfile
  * Deployment manifest files

---

# ☁️ Steps to Create EKS Cluster

## 1. Create an EC2 Instance

* Instance type: `t2.large`

---

## 2. Attach IAM Role

* Attach IAM role with **Administrator Access**

---

## 3. Modify `.bashrc`

```bash
vim .bashrc
```

Add:

```bash
export PATH=$PATH:/usr/local/bin/
```

Apply changes:

```bash
source .bashrc
```

---

## 4. Install AWS CLI

```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```

---

## 5. Install eksctl

```bash
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
eksctl version
```

---

## 6. Setup kubectl

```bash
curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.19.6/2021-01-05/bin/linux/amd64/kubectl
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin
kubectl version --short --client
```

---

## 7. Create EKS Cluster

### Create Cluster

```bash
eksctl create cluster \
--name=EKS-1 \
--region=ap-south-1 \
--zones=ap-south-1a,ap-south-1b \
--without-nodegroup
```

---

### Associate IAM OIDC Provider

```bash
eksctl utils associate-iam-oidc-provider \
--region ap-south-1 \
--cluster EKS-1 \
--approve
```

---

### Create Node Group

```bash
eksctl create nodegroup \
--cluster=EKS-1 \
--region=ap-south-1 \
--name=node2 \
--node-type=t3.medium \
--nodes=3 \
--nodes-min=2 \
--nodes-max=4 \
--node-volume-size=20 \
--ssh-access \
--ssh-public-key=<Key_Pair_Name> \
--managed \
--asg-access \
--external-dns-access \
--full-ecr-access \
--appmesh-access \
--alb-ingress-access
```

> ⏳ **Note:** Cluster creation may take several minutes.

---

# ⚙️ Install & Configure Jenkins

## 1. Install Required Packages on EC2

* Jenkins
* Java
* Git
* Docker

---

## 2. Grant Docker Permissions

```bash
chmod 777 /var/run/docker.sock
```

---

## 3. Setup Jenkins

### Install Plugins:

* Docker Pipeline
* Kubernetes
* Kubernetes CLI
* Pipeline Stage View

### Add Credentials:

* Go to **Global Credentials**
* Add Docker credentials
* ID: `Docker-Cred`

---

# ☸️ Kubernetes Configuration

## 🔹 Create Namespace

```bash
kubectl create ns webapps
```

---

## 🔹 Create Service Account

### `serviceaccount.yml`

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: jenkins
  namespace: webapps
```

Apply:

```bash
kubectl create -f serviceaccount.yml
```

---

## 🔹 Create Role

### `role.yml`

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: app-role
  namespace: webapps
rules:
  - apiGroups:
      - ""
      - apps
      - autoscaling
      - batch
      - extensions
      - policy
      - rbac.authorization.k8s.io
    resources:
      - pods
      - componentstatuses
      - configmaps
      - daemonsets
      - deployments
      - events
      - endpoints
      - horizontalpodautoscalers
      - ingress
      - jobs
      - limitranges
      - namespaces
      - nodes
      - persistentvolumes
      - persistentvolumeclaims
      - resourcequotas
      - replicasets
      - replicationcontrollers
      - serviceaccounts
      - services
    verbs:
      - get
      - list
      - watch
      - create
      - update
      - patch
      - delete
```

Apply:

```bash
kubectl create -f role.yml
```

---

## 🔹 Create Role Binding

### `rolebinding.yml`

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: app-rolebinding
  namespace: webapps
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: app-role
subjects:
  - kind: ServiceAccount
    name: jenkins
    namespace: webapps
```

Apply:

```bash
kubectl create -f rolebinding.yml
```

---

## 🔐 Generate Token for Service Account

### `secret.yml`

```yaml
apiVersion: v1
kind: Secret
type: kubernetes.io/service-account-token
metadata:
  name: mysecretname
  annotations:
    kubernetes.io/service-account.name: Jenkins
```

Apply:

```bash
kubectl create -f secret.yml
```

---

### Get Token

```bash
kubectl describe secret <secret-name> -n webapps
```

---

## 🔑 Add Token to Jenkins

* Go to **Credentials**
* Select **Secret Text**
* Paste token
* ID: `k8s-token`

---

## ⚠️ Jenkinsfile Update

* Update **EKS Cluster URL** in main branch `Jenkinsfile`

---

# 🧹 Delete EKS Cluster

```bash
eksctl delete cluster --name EKS-1 --region ap-south-1
```

---

