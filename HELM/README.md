# Helm Installation and Getting Started Guide

## Introduction to Helm

Helm is a package manager for Kubernetes that simplifies the deployment and management of applications on Kubernetes clusters. It allows you to define, install, and upgrade even the most complex Kubernetes applications using charts, which are collections of pre-configured Kubernetes resources.

## Prerequisites

- A running Kubernetes cluster
- `kubectl` installed and configured to communicate with your cluster

## Installing Helm

### Step 1: Download Helm Binary

For Windows, you can download the Helm binary from the official Helm releases page:

```sh
curl -LO https://get.helm.sh/helm-v3.15.3-windows-amd64.zip
```
### Step 2: Extract and Move Binary
```sh
unzip helm-v3.15.3-windows-amd64.zip
```
# Create the bin directory if it doesn't exist and move the helm.exe binary
```sh
mkdir -p ~/bin
mv windows-amd64/helm.exe ~/bin/helm.exe
```
# Add ~/bin to PATH if it's not already included
```sh
echo 'export PATH=$PATH:/c/Users/mhono/bin' >> ~/.bashrc
```
# Reload the .bashrc file to apply changes
```sh
source ~/.bashrc
```
# Verify the installation
```sh
helm version
```
## Getting Started with Helm

### Step 4: Add a Chart Repository

Add the official Helm eks charts repository:

```sh
helm repo add eks https://aws.github.io/eks-charts

```

### Step 5: Update Repositories

Update the repositories to get the latest charts:

```sh
helm repo update eks
```

### Step 6: Create a Domain in AWS Route 53

1. **Open Route 53 Console:**
   - Navigate to the [Route 53 Console](https://console.aws.amazon.com/route53/home).
   
2. **Create a Hosted Zone:**
   - Click on "Create hosted zone".
   - Enter the domain name you want to use (e.g., `hilltop.com`).
   - Choose the type as "Public hosted zone".
   - Click on "Create".

3. **Note the Nameservers:**
   - After creating the hosted zone, note the nameservers provided by Route 53.
   - Update your domain registrar with these nameservers to point your domain to Route 53.

### Step 7: Request an SSL Certificate using AWS Certificate Manager (ACM)

1. **Open ACM Console:**
   - Navigate to the [ACM Console](https://console.aws.amazon.com/acm/home).
   
2. **Request a Certificate:**
   - Click on "Request a certificate".
   - Choose "Request a public certificate".
   - Enter your domain name (e.g., `hilltop.com`).
   - Optionally, add additional names like `www.hilltop.com`.
   - Click "Next".

3. **Validate the Domain:**
   - Choose DNS validation (recommended).
   - ACM will provide DNS records that you need to add to your Route 53 hosted zone.
   - After adding the DNS records, ACM will validate your domain and issue the certificate.

### Step 8: Set Up the IAM Role and Policy for ALB

1. **deploy the relevant RBAC roles and role bindings as required by the AWS ALB Ingress controller:**
```sh
curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.7.2/docs/install/iam_policy.json
```
2. **Next, create an IAM policy named ALBIngressControllerIAMPolicy to allow the ALB Ingress controller to make AWS API calls on your behalf. Record the Policy.Arn in the command output, you will need it in the next step:**
```bash
aws iam create-policy \
    --policy-name AWSLoadBalancerControllerIAMPolicy \
    --policy-document file://iam_policy.json
```

2: **Install eksctl: eksctl is a command-line tool for creating and managing Kubernetes clusters on Amazon EKS.**
```sh
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_Windows_amd64.zip" -o "eksctl.zip"
unzip eksctl.zip -d /tmp
```
```sh
mkdir -p /c/tools
mv /tmp/eksctl.exe /c/tools/
```
```sh
echo 'export PATH=$PATH:/c/tools' >> ~/.bashrc
source ~/.bashrc
eksctl version
```
3. **Associate IAM OIDC Provider:**
```sh
eksctl utils associate-iam-oidc-provider --region=us-west-2 --cluster=eks-hilltop-dev --approve
```
3. **Next, create a Kubernetes service account and an IAM role (for the pod running the AWS ALB Ingress controller) by substituting $PolicyARN with the recorded value from the previous step:**
```bash
eksctl create iamserviceaccount \
  --cluster=eks-hilltop-dev \
  --namespace=kube-system \
  --name=aws-load-balancer-controller \
  --role-name AmazonEKSLoadBalancerControllerRole \
  --attach-policy-arn=arn:aws:iam::905418026355:policy/AWSLoadBalancerControllerIAMPolicy \
  --approve
```

4. **Step 2: Install AWS Load Balancer Controller**
```sh
helm repo add eks https://aws.github.io/eks-charts
```
5. **Update your local repo to make sure that you have the most recent charts.**
```sh
helm repo update eks
```
6. **Deploy deploy the AWS ALB Ingress controller:**
```sh
helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
  -n kube-system \
  --set clusterName=eks-hilltop-dev \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller 
```
Step 7: **Verify that the controller is installed**
```sh
kubectl get deployment -n kube-system aws-load-balancer-controller
```
### Step 9: Install the AWS Load Balancer Controller

1. **Add the EKS Helm Repository:**
   ```bash
   helm repo update
   ```
2. **Create a Helm chart named `hilltop`:**

```bash
helm create hilltop
```

#### 9 Modify the Deployment YAML

### `values.yaml`

### `templates/ingress.yaml`

### `templates/service.yaml`

### Deploy the Application:
```bash
helm upgrade --install hilltop . -f values.yaml
```

### Notes:

- Ensure you replace placeholders like `YOUR_ACCOUNT_ID`, `REGION`, and `CERTIFICATE_ID` with actual values.
- The AWS Load Balancer Controller will automatically create an ALB when the Ingress resource is applied. You don't need to create the ALB manually.


