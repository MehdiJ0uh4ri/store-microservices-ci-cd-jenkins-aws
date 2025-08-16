# EKS Jenkins CI/CD Pipeline for Microservices Deployment

This project demonstrates how to set up a **CI/CD pipeline** using **Jenkins**, **AWS EKS (Elastic Kubernetes Service)**, and **Kubernetes** for deploying and managing microservices. The pipeline is designed for an **OnlineBoutique E-Commerce** site with 10 microservices, with Jenkins automating the build and deployment process to AWS EKS clusters.

## Prerequisites

Before starting, ensure you have the following:

1. An AWS account with access to EKS, EC2, IAM, and CloudFormation.
2. A running **EC2 instance** where you'll SSH and configure tools.
3. AWS **IAM User** with required permissions (listed below).
4. **Jenkins** installed on the EC2 instance.
5. **Docker** installed on your EC2 instance.
6. **Kubernetes** and **eksctl** tools installed.

## Setup Instructions

### Step 1: Connect to EC2 Instance via SSH

```bash
ssh -i <your-key.pem> ubuntu@<your-ec2-public-ip>
```

### Step 2: Create an IAM User with Required Permissions

Create a new IAM user with the following policies attached:

* `AmazonEC2FullAccess`
* `AmazonEKSClusterPolicy`
* `AmazonEKSWorkerNodePolicy`
* `AWSCloudFormationFullAccess`
* `IAMFullAccess`

Create a custom policy `eks_policy`:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "eks:*",
            "Resource": "*"
        }
    ]
}
```

### Step 3: Install AWS CLI, kubectl, and eksctl

SSH into your EC2 instance and install the required tools:

#### AWS CLI Installation:

```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
sudo apt install unzip
unzip awscliv2.zip
sudo ./aws/install
```

#### kubectl Installation:

```bash
curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.19.6/2021-01-05/bin/linux/amd64/kubectl
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin
kubectl version --short --client
```

#### eksctl Installation:

```bash
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
eksctl version
```

### Step 4: Configure AWS CLI

Configure the AWS CLI with the IAM user credentials:

```bash
aws configure
```

### Step 5: Create EKS Cluster

Create your EKS cluster and node group with `eksctl`:

```bash
eksctl create cluster --name=EKS-1 --region=ap-south-1 --zones=ap-south-1a,ap-south-1b --without-nodegroup
eksctl utils associate-iam-oidc-provider --region ap-south-1 --cluster EKS-1 --approve
eksctl create nodegroup --cluster=EKS-1 --region=ap-south-1 --name=node2 --node-type=t3.medium --nodes=3 --nodes-min=2 --nodes-max=4 --node-volume-size=20 --ssh-access --ssh-public-key=DevOps --managed --asg-access --external-dns-access --full-ecr-access --appmesh-access --alb-ingress-access
```

### Step 6: Install Jenkins on EC2 Instance

#### Install Java:

```bash
sudo apt install openjdk-17-jre-headless
```

#### Install Jenkins:

```bash
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/" | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install jenkins
```

#### Access Jenkins:

Visit `http://<your-ec2-public-ip>:8080` and complete the setup wizard. Retrieve the initial password from:

```bash
cat /var/lib/jenkins/secrets/initialAdminPassword
```

#### Install Plugins:

* Docker
* Docker Pipeline
* Kubernetes
* Kubernetes CLI

#### Install Docker:

```bash
sudo apt install docker.io -y
sudo chmod 666 /var/run/docker.sock
```

### Step 7: Configure Jenkins

#### Set Up Credentials:

* Docker Hub Credentials (named `docker-cred`)
* GitHub Token (named `git-cred`)
* Kubernetes Service Account Token (named `k8s-token`)

#### Create Kubernetes Service Account, Role, and Binding:

Create a Service Account, Role, and RoleBinding in your EKS cluster. Then store the generated token in Jenkins as a secret (`k8s-token`).

### Step 8: Set Up Multibranch Pipeline

1. **Create a Multibranch Pipeline**: Point Jenkins to your GitHub repository.
2. **Set Up GitHub Webhooks**: Add a webhook to trigger builds when branches are pushed.

### Step 9: Jenkinsfile for Deployment

Add the above `Jenkinsfile` to your repository to handle the deployment of microservices on your EKS cluster:

### Step 10: Deploy Microservices

Jenkins will automatically deploy your microservices to the EKS cluster whenever you push changes to the relevant GitHub branch.

### Conclusion

By following these steps, you've successfully set up a **CI/CD pipeline** using **Jenkins**, **AWS EKS**, and **Kubernetes** to deploy and manage microservices for an **OnlineBoutique E-Commerce** site.

---

### Additional Notes

* Ensure that you clean up your AWS resources (like the EKS cluster) after completing your deployment.
* Consider automating the cleanup process using **AWS CloudFormation**.

