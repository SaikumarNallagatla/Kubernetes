Here's a guide to set up your EC2 instance for Kubernetes with Minikube, designed for GitHub and VS Code:

Setting up Kubernetes with Minikube on AWS EC2 (Ubuntu)
This guide helps you set up a single-node Kubernetes cluster using Minikube on an AWS EC2 Ubuntu instance (free tier compatible). It's designed to be easily copy-pasted and followed.

# 1. Connect to your EC2 Instance
First, connect to your EC2 instance using SSH.

Using VS Code Terminal:
Open VS Code.

Go to Terminal > New Terminal.

Navigate to the directory where your EC2 key pair (.pem file) is stored.

Run the SSH command:

```Bash
ssh -i your-key-pair.pem ubuntu@your-ec2-public-ip
```
Replace your-key-pair.pem with the name of your key pair file.
Replace your-ec2-public-ip with the Public IPv4 DNS or Public IPv4 address of your EC2 instance.

# 2. Update and Upgrade your EC2 Instance
It's always a good idea to update and upgrade your system.

```Bash

sudo apt update -y
sudo apt upgrade -y
```

# 3. Install Docker
Minikube uses Docker as a driver to run the Kubernetes cluster.


Update your system and install necessary packages:

```Bash
sudo apt update
sudo apt install ca-certificates curl gnupg lsb-release -y
```

This ensures your system is up to date and has the tools needed to add the key and repository.

Add Docker's official GPG key:

```Bash
sudo mkdir -m 0755 -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```
This downloads the GPG key and stores it securely on your system.

Add the Docker repository to your APT sources:

```Bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```
This tells your system where to find the Docker packages.

After you've run these commands, you can then try your original installation commands:

```Bash
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io -y
```

Add your user to the docker group (important for running Docker commands without sudo):

```Bash
sudo usermod -aG docker ubuntu
```
Apply the group changes (you might need to log out and back in, or simply start a new terminal session):

```Bash
newgrp docker
```
Verify Docker installation:

```Bash
docker run hello-world
```
You should see a message confirming Docker is working.

# 4. Install Kubectl
kubectl is the command-line tool for running commands against Kubernetes clusters.

Download kubectl:
```Bash

curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
```
Install kubectl:
```Bash

sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
```
Verify kubectl installation:
```Bash

kubectl version --client
```
# 5. Install Minikube
Minikube helps you run a single-node Kubernetes cluster locally.

Download Minikube:
```Bash

curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
```
Install Minikube:
```Bash

sudo install minikube-linux-amd64 /usr/local/bin/minikube
```
Verify Minikube installation:
```Bash

minikube version
```
# 6. Start Minikube
Now, start your Kubernetes cluster using Minikube with the Docker driver.

```Bash

minikube start --driver=docker
```
This command might take a few minutes as it downloads necessary components.

Check Minikube status:
```Bash

minikube status
```
You should see host: Running, kubelet: Running, apiserver: Running, and kubeconfig: Configured.

# 7. Verify Kubernetes Cluster
Once Minikube is running, you can use kubectl to interact with your cluster.

Get cluster information:
```Bash

kubectl cluster-info
```
List all pods in the kube-system namespace:
```Bash

kubectl get pods -A
```
# 8. Stopping and Deleting Minikube
Stop Minikube:
```Bash

minikube stop
```
Delete Minikube cluster:
```Bash

minikube delete
```
Happy Learning!



