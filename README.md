1. Project Overview
This POC demonstrates the end-to-end deployment of the Brain Tasks Application using Docker, AWS ECR, Amazon EKS, AWS CodeBuild, AWS CodePipeline, GitHub, and CloudWatch.

Technology Stack
•	GitHub – Source Code Management
•	Docker – Containerization
•	Amazon ECR – Docker Image Registry
•	Amazon EKS – Kubernetes Platform
•	AWS CodeBuild – CI Build
•	AWS CodePipeline – CI/CD Pipeline
•	Kubernetes – Application Deployment
•	Amazon CloudWatch – Monitoring and Logs

2. Application Setup
Clone the repository:
git clone https://github.com/Vennilavanguvi/Brain-Tasks-App.git
cd Brain-Tasks-App
The application was configured to run on port 3000.
Application validation:
curl http://localhost:3000

3. Dockerization
A Dockerfile was created to containerize the application.

Docker File :
FROM nginx:alpine
COPY dist/ /usr/share/nginx/html/
EXPOSE 3000
CMD ["nginx", "-g", "daemon off;"]

Build the Docker image:
docker build -t brain-tasks-app:latest .

Verify the image: docker images

Run the container: docker run -d -p 3000:3000 brain-tasks-app:latest
Validate: curl http://localhost:3000

4. Amazon ECR
An Amazon ECR repository was created to store the Docker image.

Authenticate Docker with ECR:
aws ecr get-login-password --region <AWS_REGION> | \
docker login --username AWS --password-stdin <AWS_ACCOUNT_ID>.dkr.ecr.<AWS_REGION>.amazonaws.com

Tag the image:
docker tag brain-tasks-app:latest \
<AWS_ACCOUNT_ID>.dkr.ecr.<AWS_REGION>.amazonaws.com/brain-tasks-app:latest

Push the image:
docker push \
<AWS_ACCOUNT_ID>.dkr.ecr.<AWS_REGION>.amazonaws.com/brain-tasks-app:latest
The image was successfully stored in Amazon ECR.

5. Amazon EKS
An Amazon EKS cluster was created for Kubernetes deployment.

Verify the cluster:
aws eks update-kubeconfig \
--region <AWS_REGION> \
--name <EKS_CLUSTER_NAME>

Check cluster connectivity:
kubectl get nodes

Check cluster status:
aws eks describe-cluster \
--name <EKS_CLUSTER_NAME> \
--region <AWS_REGION> \
--query "cluster.status"

Expected status:
ACTIVE

6. Kubernetes Deployment
The Kubernetes deployment file was created as deployment.yaml.

Deployment.yaml file:
apiVersion: apps/v1
kind: Deployment
metadata:
  name: brain-tasks-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: brain-tasks-app
  template:
    metadata:
      labels:
        app: brain-tasks-app
    spec:
      containers:
        - name: brain-tasks-app
          image: <ECR_IMAGE_URI>:latest
          ports:
            - containerPort: 3000

The Kubernetes service was created as service.yaml.

service.yaml:
apiVersion: v1
kind: Service
metadata:
  name: brain-tasks-service
spec:
  type: LoadBalancer
  selector:
    app: brain-tasks-app
  ports:
    - port: 80
      targetPort: 3000

Deploy the application:
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml

Verify deployment:
kubectl get deployments
kubectl get pods
kubectl get services
The application was exposed through the Kubernetes LoadBalancer service.

7. AWS CodeBuild

An AWS CodeBuild project was created with:
•	Source: GitHub repository
•	Environment: AWS managed Linux image
•	Build configuration: buildspec.yml
•	Output: Docker image pushed to Amazon ECR

buildspec.yml:
version: 0.2

phases:
  build:
    commands:
      - "echo Validating EKS manifest"
      - "test -f eks-manifest.yaml"
      - "echo Build stage completed successfully"
  build:
    commands:
      - echo Building Docker image
      - docker build -t $IMAGE_REPO_NAME:$IMAGE_TAG .

  post_build:
    commands:
      - echo Tagging Docker image
      - docker tag $IMAGE_REPO_NAME:$IMAGE_TAG $AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com/$IMAGE_REPO_NAME:$IMAGE_TAG
      - echo Pushing Docker image
      - docker push $AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com/$IMAGE_REPO_NAME:$IMAGE_TAG
The CodeBuild project was executed successfully and the Docker image was pushed to ECR.

8. GitHub Version Control
Git was used to maintain and push the project source/configuration files.
git init
git add .
git commit -m "Initial DevOps implementation"
git branch -M main
git remote add origin https://github.com/Franklin-Moses/Brain-Tasks-App.git
git push -u origin main
The repository contains the required deployment files such as:
Dockerfile
buildspec.yml
deployment.yaml
service.yaml
README.md

9. AWS CodePipeline
An AWS CodePipeline was configured with the following stages:
GitHub
   ↓
AWS CodeBuild
   ↓
Amazon ECR
   ↓
Amazon EKS
   ↓
Application
Pipeline Stages
Source
•	Provider: GitHub
•	Repository: Brain-Tasks-App
•	Branch: main
Build
•	Provider: AWS CodeBuild
•	Docker image is built
•	Image is pushed to Amazon ECR
Deploy
•	Target: Amazon EKS
•	Kubernetes deployment/service are applied
•	New application version is deployed to the cluster
The CodePipeline execution was completed successfully.

10. CloudWatch Monitoring
Amazon CloudWatch was used for monitoring and troubleshooting.
The following logs were monitored:
•	AWS CodeBuild logs
•	AWS CodePipeline execution information
•	Application/container logs where configured
Useful commands for Kubernetes application logs:
kubectl get pods
kubectl logs <POD_NAME>
CloudWatch Logs were checked to verify successful build and deployment activities.

11. Validation
The following checks were completed successfully:
Component	Validation
GitHub	Source code pushed
Application	Running on port 3000
Docker	Image built successfully
Docker Container	Application accessible
ECR	Image pushed successfully
EKS	Cluster status ACTIVE
Kubernetes	Pods running
Service	LoadBalancer created
CodeBuild	Build completed successfully
CodePipeline	Pipeline execution completed
CloudWatch	Logs available

12. Conclusion
The Brain Tasks Application was successfully containerized using Docker and deployed on Amazon EKS. The complete CI/CD workflow was implemented using GitHub, AWS CodeBuild, Amazon ECR, and AWS CodePipeline. Kubernetes was used for application deployment and service exposure, while CloudWatch was used for monitoring and log verification.
This POC demonstrates an end-to-end GitHub → CodePipeline → CodeBuild → ECR → EKS → Application deployment workflow.
