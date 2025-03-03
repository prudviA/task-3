Deploy Microservices on Your EKS Cluster

Setting Up Amazon Linux for Kubernetes Deployment
Amazon Linux is a good choice because it's lightweight, optimized for AWS, and integrates well with EKS. Follow these steps to set up everything:
Update Amazon Linux & Install Necessary Tools
-------------------------------
sudo yum update -y
sudo yum install aws-cli -y
aws --version
configure AWS credentials
aws configure

You'll be prompted to enter:

AWS Access Key ID
AWS Secret Access Key
Default Region (e.g., us-east-1)
Output format (default: json)
-----------------------------------------------------------------
 Install Docker on Amazon Linux
Since you are using Amazon Linux, first install and start Docker:
sudo yum update -y
sudo yum install -y docker
sudo systemctl start docker
sudo systemctl enable docker
sudo systemctl status docker
sudo usermod -aG docker ec2-user
newgrp docker
sudo cat /etc/group
docker images
docker --version

----------------------------------------------------------------
Install kubectl
kubectl is needed to manage your Kubernetes cluster.

curl -o kubectl https://s3.us-west-2.amazonaws.com/amazon-eks/1.27.0/2023-06-28/bin/linux/amd64/kubectl
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
kubectl version --client
---------------------------------------------------------------------
 Install eksctl

This tool helps to create and manage AWS EKS clusters easily.

curl -sSL "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_Linux_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin/
eksctl version

-----------------------------------------------------------------------
Log in to Docker Hub
docker login
Create a Simple Docker Image
mkdir my-docker-image
cd my-docker-image

nano Dockerfile Paste this inside:

# Use Nginx as base image
FROM nginx:latest

# Copy a simple HTML file (optional)
COPY index.html /usr/share/nginx/html/index.html

# Expose port 80
EXPOSE 80
---------------------------------------------------------------------------
 Create a simple index.html file:
echo "<h1>Hello from my custom Nginx container!</h1>" > index.html

Build the Docker Image
docker build -t your-dockerhub-username/nginx-custom:latest .
docker images

Push the Docker Image to Docker Hub

docker tag your-dockerhub-username/nginx-custom:latest your-dockerhub-username/nginx-custom:latest

Push the image to Docker Hub:
docker push your-dockerhub-username/nginx-custom:latest

---------------------------------------------------------------------------

Create an EKS Cluster
Now, create a Kubernetes cluster using EKS:

eksctl create cluster --name my-cluster --region us-east-1 --nodes 2 --node-type t3.medium
This takes 10-15 minutes to complete.

Once done, verify:
kubectl get nodes

If nodes show STATUS = Ready, your cluster is working!

Check your cluster version using:
aws eks describe-cluster --name my-cluster --query "cluster.version"

Create Deployment & Service YAML Files
You'll need at least two YAML files:


Create a file named deployment.yaml
nano deployment.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-microservice
spec:
  replicas: 2
  selector:
    matchLabels:
      app: my-microservice
  template:
    metadata:
      labels:
        app: my-microservice
    spec:
      containers:
        - name: my-microservice
          image: your-dockerhub-username/your-microservice-image:latest
          ports:
            - containerPort: 80

Replace your-dockerhub-username/your-microservice-image with your actual Docker image.


service.yaml


Create a file named service.yaml:

apiVersion: v1
kind: Service
metadata:
  name: my-microservice-service
spec:
  selector:
    app: my-microservice
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: LoadBalancer



 This will expose your microservice on an AWS LoadBalancer.


 Apply the Kubernetes YAML Files


Now, apply the YAML files to your EKS cluster:

kubectl apply -f deployment.yaml
kubectl apply -f service.yaml

Check if the deployment and service are running:

aws eks update-kubeconfig --region us-east-1 --name my-cluster

kubectl get pods

kubectl get svc

 Get the external IP of the LoadBalancer:




Copy the EXTERNAL-IP (e.g., abc-xyz.elb.amazonaws.com) and test in your browser.


Test Your Microservice

http://your-external-ip
