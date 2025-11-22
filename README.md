In this repo I have used github webhooks in which i have used link for hitting the post metthod is which i derived from ngrok by command ngrok http 80 then it given some link that link need to be updated everytime when it runs so we need to change webhook link which generated from ngrok http 80
In jenkins which is running locally which has docker and all other dependencies installed already
so in jenkins we need to set
credencials of docker in credentials
first create some pipe line in which set pipeline from scm which is present in the git repo itself from there it self it will run the automation also select GitHub hook trigger for GITScm polling for sure to run repo on push event

first go to docker 
then start jenkins container 
then login into it by username abhishek and password is also be same 
in powershell  
ngrok http 8080
then try to push something 
webhook we have configured already
Readme.md from docker file

rm -f /etc/apt/sources.list.d/kubernetes.list

apt-get update

apt-get install -y apt-transport-https ca-certificates curl

curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key \
  | gpg --dearmor -o /usr/share/keyrings/kubernetes-archive-keyring.gpg

echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] \
https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /" \
  > /etc/apt/sources.list.d/kubernetes.list

apt-get update
apt-get install -y kubelet kubeadm kubectl
apt-mark hold kubelet kubeadm kubectl
<!-- to verify -->
kubectl version --client
kubeadm version
kubelet --version

# to install nodejs
curl -fsSL https://deb.nodesource.com/setup_lts.x | bash -
apt-get install -y nodejs
docker compose.yaml for run 
version: '3.9'

services:
  jenkins:
    build: .
    container_name: jenkins-ubuntu
    ports:
      - "8080:8080"
      - "3000:3000"    # Your React/Node app visible on Windows
    volumes:
      - jenkins_home:/var/lib/jenkins
      - /var/run/docker.sock:/var/run/docker.sock
    restart: unless-stopped
    user: root  # important to avoid permission issues initially

volumes:
  jenkins_home:

Dockerfile

FROM ubuntu:22.04

USER root

# Prevent interactive issues
ENV DEBIAN_FRONTEND=noninteractive

# -----------------------------
# Install basic dependencies
# -----------------------------
RUN apt-get update && apt-get install -y \
    curl \
    wget \
    git \
    ca-certificates \
    gnupg \
    lsb-release \
    unzip \
    zip \
    make \
    sudo \
    apt-transport-https \
    software-properties-common

# -----------------------------
# Install Java (required for Jenkins)
# -----------------------------
RUN apt-get install -y openjdk-17-jdk

# -----------------------------
# Install Jenkins
# -----------------------------
RUN curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | tee \
    /usr/share/keyrings/jenkins-keyring.asc > /dev/null

RUN echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
    https://pkg.jenkins.io/debian-stable binary/ | \
    tee /etc/apt/sources.list.d/jenkins.list > /dev/null

RUN apt-get update && apt-get install -y jenkins

# -----------------------------
# Install Docker CLI inside container
# -----------------------------
RUN curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add - && \
    add-apt-repository \
    "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
    $(lsb_release -cs) stable"

RUN apt-get update && apt-get install -y docker-ce-cli

# -----------------------------
# Create Jenkins user & fix permissions
# -----------------------------
# Add Jenkins to sudo group
RUN usermod -aG sudo jenkins

# Give passwordless sudo
RUN echo "jenkins ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers

# Create docker group and add Jenkins user
RUN groupadd -f docker && usermod -aG docker jenkins

# -----------------------------
# Allow Jenkins to use Docker Socket
# -----------------------------
VOLUME ["/var/run/docker.sock"]

# -----------------------------
# Expose Jenkins port
# -----------------------------
EXPOSE 8080

# Start Jenkins
CMD ["/usr/bin/jenkins"]
