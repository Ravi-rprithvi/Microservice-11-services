# 11 Microservice CI/CD Pipeline

## Overview

This project showcases the implementation of a CI/CD pipeline for an e-commerce website composed of 11 microservices. The deployment platform is AWS EKS (Elastic Kubernetes Service).

![Screenshot from 2024-07-28 17-27-11](https://github.com/user-attachments/assets/dd93bf34-14e8-4b10-b06f-b2401a40e8eb)

## Microservices

1. Ad Sense Service
2. Cart Service
3. Checkout Service
4. Currency Service
5. Email Service
6. Frontend Service
7. External Frontend (for load balancing)
8. Payment Service
9. Product Catalogue Service
10. Recommendation Service

## Benefits of Microservices

- **Scalability:** Independent scaling of services.
- **Resilience:** Service failures do not impact the entire system.
- **Development Speed:** Parallel development and quicker releases.
- **Technology Diversity:** Use of the best-suited technology for each service.
- **Isolation and Maintenance:** Easier to understand, develop, test, and maintain.

## Project Components and Pipeline

### CI/CD Pipeline Using Jenkins

- **Jenkinsfiles:** Define steps for building, testing, and deploying each service.
- **Docker:** Containerization for consistent environments.
- **AWS EKS:** Managed Kubernetes service for deployment.


### Phase 1: Infrastructure Setup

1. **Create AWS IAM User**
   - Attach necessary policies: `AmazonEC2FullAccess`, `AmazonEKS_CNI_Policy`, etc.
   - Create an inline policy for EKS access.
   - Download `credentials.csv`.

2. **Launch Virtual Machine using AWS EC2**
   - **Instance Type:** `t2.large`
   - **AMI:** Ubuntu Server 20.04 LTS
   - Configure security groups.

3. **Install AWS CLI, EKSCTL & KUBECTL on VM Server**
   - **AWSCLI:**
     ```sh
     curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
     sudo apt install unzip
     unzip awscliv2.zip
     sudo ./aws/install
     aws configure
     ```
   - **KUBECTL:**
     ```sh
     curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.19.6/2021-01-05/bin/linux/amd64/kubectl
     chmod +x ./kubectl
     sudo mv ./kubectl /usr/local/bin
     kubectl version --short --client
     ```
   - **EKSCTL:**
     ```sh
     curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
     sudo mv /tmp/eksctl /usr/local/bin
     eksctl version
     ```

4. **Installing Jenkins on Ubuntu**
   ```sh
   #!/bin/bash
   sudo apt install openjdk-17-jre-headless -y
   sudo wget -O /usr/share/keyrings/jenkins-keyring.asc https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
   echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/ | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
   sudo apt-get update
   sudo apt-get install jenkins -y
   ```
   
5. **Create EKS Cluster **
   ```sh
   eksctl create cluster --name=EKS-1 --region=ap-south-1 --zones=ap-south-1a,ap-south-1b --without-nodegroup
   eksctl utils associate-iam-oidc-provider --region ap-south-1 --cluster EKS-1 --approve
   eksctl create nodegroup --cluster=EKS-1 --region=ap-south-1 --name=node2 --node-type=t3.medium --nodes=3 --nodes-min=2 --nodes-max=4 --node-volume-size=20 --ssh-access --ssh-public-key=DevOps --managed --asg-access --external-dns-access --full-ecr-access --appmesh-access --alb-ingress-access
   ```
   
## Phase 2: Multi-branch Pipeline Setup

### Install Jenkins Plugins
1. **Docker**
2. **Docker Pipeline**
3. **Kubernetes**
4. **Kubernetes CLI**
5. **Multibranch Scan Webhook Trigger**

### Create Credentials
1. **Docker Credentials:** `docker-cred`
2. **GitHub Credentials:** `git-cred`

### Configure Multibranch Pipeline
1. Add repository URL and GitHub credentials.
2. Configure webhook trigger with the generated URL.

---

## Phase 3: Continuous Deployment

### Create Kubernetes Resources
1. service account
   ```sh
   apiVersion: v1
   kind: ServiceAccount
   metadata:
   name: jenkins
   namespace: webapps
   ```

2. Role
   ```sh
   apiVersion: rbac.authorization.k8s.io/v1
   kind: Role
   metadata:
   name: app-role
   namespace: webapps
   rules:
    - apiGroups: [""]
      resources: ["pods", "services"]
      verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
   ```
3. Role Binding
   ```sh
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
4. Secret
   ```sh
   apiVersion: v1
   kind: Secret
   metadata:
      name: mysecretname
      annotations:
         kubernetes.io/service-account.name: jenkins
   ```
### Create Jenkinsfile 
```sh
pipeline {
  agent any
  stages {
    stage('Deploy To Kubernetes') {
      steps {
        withKubeCredentials(kubectlCredentials: [[caCertificate: '', clusterName: 'EKS-1', contextName: '', credentialsId: 'k8-token', namespace: 'webapps', serverUrl: 'https://B7C7C20487B2624AAB0AD54DF1469566.yl4.ap-south-1.eks.amazonaws.com']]) {
          sh "kubectl apply -f deployment-service.yml"
        }
      }
    }
    stage('Verify Deployment') {
      steps {
        withKubeCredentials(kubectlCredentials: [[caCertificate: '', clusterName: 'EKS-1', contextName: '', credentialsId: 'k8-token', namespace: 'webapps', serverUrl: 'https://B7C7C20487B2624AAB0AD54DF1469566.yl4.ap-south-1.eks.amazonaws.com']]) {
          sh "kubectl get svc -n webapps"
        }
      }
    }
  }
}
```

## Results

### Pipeline
- Automated CI/CD pipeline for 11 microservices.

### Application
- Scalable and resilient e-commerce website on AWS EKS.

### Console Output
- Continuous feedback from Jenkins builds.

## Project Impact

### Improved Efficiency
- Reduced time and effort for manual deployments.

### Enhanced Reliability
- Automated testing and deployment for early issue detection.

### Scalability
- Efficient handling of varying loads.

### Maintainability
- Simplified maintenance and updates.

## Conclusion
The successful deployment of the e-commerce website using a microservices architecture and an automated CI/CD pipeline on AWS EKS exemplifies the advantages of modern software development practices. This project not only achieves the goals of scalability, reliability, and efficiency but also sets a strong foundation for future enhancements and growth. The methodologies and technologies employed in this project can serve as a blueprint for similar projects aiming to leverage microservices and CI/CD pipelines for continuous innovation and delivery.




   
