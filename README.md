# Automatic Deployment Project with GitHub Actions and AWS EC2

This project uses GitHub Actions to automate the deployment of a web application on an AWS EC2 instance. The integration is designed to copy the code to the EC2 instance and deploy it using Docker Compose every time a push is made to the `main` branch.

## Project Architecture

![GitActionsAWS drawio](https://github.com/user-attachments/assets/36895de7-4a8d-4638-8568-9b801d181257)

## 🎥 Demonstration Video

To see a demonstration of the project's functionality, check out the following video:
[![Watch Demonstration](https://img.youtube.com/vi/TtX_Bu5HPEo/0.jpg)](https://www.youtube.com/watch?v=TtX_Bu5HPEo&ab_channel=RichardMacas)

## Features

- Automated deployment to AWS EC2 upon changes in the `main` branch.
- Docker Compose for building and running the application.
- Secure configuration using GitHub secrets.

## Repository Structure

```
.
├── Dockerfile               # Defines the Docker image for the application
├── docker-compose.yml       # Docker Compose configuration for deployment
├── app.js                   # Application source code
├── index.html               # Main webpage of the application
├── styles.css               # Application styling
└── .github/workflows
    └── deploy.yml           # GitHub Actions configuration for deployment
```

## Prerequisites

1. **EC2 Instance:** Ensure you have a properly configured EC2 instance accessible via SSH.

2. **Docker and Docker Compose Setup:**
   - Install Docker on your EC2 instance by following the [official documentation](https://docs.docker.com/engine/install/).
   - Install Docker Compose as per the [official guide](https://docs.docker.com/compose/install/).

3. **GitHub Secrets Configuration:**
   Go to `Settings > Secrets and variables > Actions` in your repository and add the following secrets:

   - `EC2_HOST`: Public IP address of your EC2 instance.
   - `EC2_USER`: SSH username (e.g., `ubuntu`).
   - `EC2_KEY`: SSH private key for accessing the instance.

## Workflow File Explanation (`.github/workflows/deploy.yml`)

```yaml
name: Deploy to EC2

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Copy files to EC2
        uses: appleboy/scp-action@v0.1.7
        with:
          host: ${{ secrets.EC2_HOST }}
          username: ${{ secrets.EC2_USER }}
          key: ${{ secrets.EC2_KEY }}
          source: "."
          target: "~/my-website"

      - name: Deploy with Docker Compose
        uses: appleboy/ssh-action@v0.1.7
        with:
          host: ${{ secrets.EC2_HOST }}
          username: ${{ secrets.EC2_USER }}
          key: ${{ secrets.EC2_KEY }}
          script: |
            cd ~/my-website
            docker-compose up --build -d
```

### Workflow Steps

1. **Checkout Code:** Pulls the latest code from the repository.
2. **Copy Files to EC2:** Uses the `scp-action` to securely transfer files to the EC2 instance.
3. **Deploy with Docker Compose:** Connects to the EC2 instance via SSH and executes Docker Compose to build and deploy the application.

## Deployment Steps

1. **Clone this repository locally:**
   ```bash
   git clone <repository URL>
   ```

2. **Set up your EC2 instance:**
   - Install Docker and Docker Compose.
   - Ensure the application port (e.g., 80 or 3000) is open in the EC2 instance’s security group.

3. **Add GitHub secrets to your repository.**

4. **Push changes to the `main` branch:**
   ```bash
   git add .
   git commit -m "Project update"
   git push origin main
   ```

5. **Verify the deployment:**
   Access your EC2 instance’s public IP address to ensure the application is running.

## System Requirements

- EC2 instance running Ubuntu or similar.
- Docker version 20.10 or newer.
- Docker Compose version 1.29 or newer.

## Notes

- Ensure your SSH private key does not have additional spaces or invalid characters.
- For debugging GitHub Actions, check the workflow logs under the `Actions` tab in your repository.

## Contributions

Contributions are welcome. If you encounter an issue or have suggestions to improve the project, feel free to create an issue or submit a pull request.

## License

This project is licensed under the MIT License. See the `LICENSE` file for more details.
