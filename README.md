# **My Website: CI/CD Pipeline with GitHub Actions and AWS**

This repository contains the source code and CI/CD pipeline for deploying a containerized web application to **AWS EC2** instances in both QA and production (Main) environments. The production environment leverages **Auto Scaling Groups (ASG)** and **Load Balancer** for high availability and scalability.

---

## **Features**
- **CI/CD pipeline using GitHub Actions**:
  - Automatic Docker image build and push to **GitHub Container Registry (GHCR)**.
  - Deployment to **QA** environment using SSH.
  - Deployment to **Production** via Auto Scaling Groups with a User Data Script.
- **Scalable Production Environment**:
  - Instances are automatically managed by AWS Auto Scaling Group.
  - Load Balancer ensures traffic is distributed across healthy instances.
- **High Availability**:
  - New instances automatically pull the latest container image.

---

## **Architecture**

Below is the architecture of the project, showcasing the CI/CD pipeline and deployment setup:

![AwsGithubLBAS](https://github.com/user-attachments/assets/af3f71bf-94ef-4cd1-a202-c1fad25e9a0f)

---

## **Technologies Used**
- **Docker**: Containerize the application.
- **GitHub Actions**: Automate CI/CD pipeline.
- **GitHub Container Registry (GHCR)**: Store and manage Docker images.
- **AWS**: Hosting and infrastructure management.
  - EC2 Instances
  - Auto Scaling Group
  - Application Load Balancer

---

## **Getting Started**

### **Prerequisites**
1. **GitHub Repository**:
   - Create a GitHub repository and clone it to your local machine.
2. **AWS Account**:
   - Ensure you have an active AWS account and access to EC2, Auto Scaling, and Load Balancer services.
3. **Docker**:
   - Install Docker on your local machine.

---

### **Setting Up the Environments**

#### **1. QA Environment**
- Launch two EC2 instances for QA.
- Install Docker on each instance:
  ```bash
  sudo yum update -y
  sudo yum install -y docker
  sudo service docker start
  sudo usermod -aG docker ec2-user
  ```
- Add the public IPs of the instances as <QA_INSTANCE_1> and <QA_INSTANCE_2> secrets in the GitHub repository.
  
#### **2. Production Environment**
- Create a Launch Template in AWS:
 - Include the User Data Script for installing Docker, logging into GHCR, and running the container.
 - Example User Data Script:
```bash
#!/bin/bash
yum update -y
yum install -y docker
service docker start
usermod -aG docker ec2-user
docker login ghcr.io -u <GHCR_ACTOR> -p <GHCR_TOKEN>
docker pull ghcr.io/<github_owner>/<repository>:main
docker run -d --name my-website -p 80:80 ghcr.io/<github_owner>/<repository>:main
```
 - Replace <GHCR_ACTOR> and <GHCR_TOKEN> with your GitHub credentials.
- Create an Auto Scaling Group using the Launch Template.
- Configure an Application Load Balancer:
 - Set up health checks for your instances.


---

### **Pipeline Configuration**

#### **GitHub Secrets**
Add the following secrets to your GitHub repository:
- `GHCR_ACTOR`: Your GitHub username.
- `GHCR_TOKEN`: A GitHub personal access token with `write:packages` and `read:packages` permissions.
- `QA_INSTANCE_1`: Public IP of the first QA instance.
- `QA_INSTANCE_2`: Public IP of the second QA instance.
- `EC2_USER`: The username for your EC2 instances (e.g., `ec2-user`).
- `EC2_KEY`: Your private SSH key for accessing the EC2 instances.
- `MAIN_LB_HOST`: The DNS name of your Load Balancer.

---

### **Pipeline Workflow**

The GitHub Actions pipeline is defined in `.github/workflows/deploy.yml`:

1. **Build and Push Docker Image**:
   - Triggered on a push to `qa` or `main` branches.
   - Builds a Docker image and pushes it to **GitHub Container Registry**.

2. **Deploy to QA**:
   - SSHs into two QA instances and deploys the Docker container.

3. **Deploy to Main**:
   - Updates the Docker image in **GitHub Container Registry**.
   - Auto Scaling Group pulls the latest image when new instances are launched.

---

### **File Structure**

```plaintext
.
├── .github/
│   └── workflows/
│       └── deploy.yml      # GitHub Actions pipeline
├── app/                    # Application source code
│   └── ...                 # Replace with your app's files
├── Dockerfile              # Dockerfile for building the application image
├── README.md               # Project documentation
```
---

## **Usage**

1. **Push to QA Branch**:
   - Push any changes to the `qa` branch to trigger the deployment to QA instances.
   - Example:
     ```bash
     git checkout qa
     git add .
     git commit -m "Deploying to QA"
     git push origin qa
     ```

2. **Push to Main Branch**:
   - Push any changes to the `main` branch to trigger the deployment in production.
   - Example:
     ```bash
     git checkout main
     git add .
     git commit -m "Deploying to Production"
     git push origin main
     ```

---

## **Monitoring and Troubleshooting**

- **AWS Console**:
  - Use the EC2 dashboard to check instance health.
  - Check the Auto Scaling Group for instance scaling activities.
  - Use the Load Balancer dashboard to monitor traffic distribution.
- **GitHub Actions Logs**:
  - View pipeline execution logs in the **Actions** tab of the repository.

---





