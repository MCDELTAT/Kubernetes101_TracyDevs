# Configuration Steps for Basic Minikube Kubernetes Environment
This document details how to manually set up a basic Kubernetes development environment using Minikube. It was written with Ubuntu 22.04 in mind, but other future versions may work as well. These manual steps are provided as a way to audit the Ansible code to see what I'm doing, and why I'm doing it. Each major section links to the official documentation where it was obtained. This allows the user to easily update the code should things change in the future.

## Building a container with our own code
### Install docker
* From the guide [here][1]
  * `sudo apt update`
  * `sudo apt-get install ca-certificates curl gnupg`
  * `sudo install -m 0755 -d /etc/apt/keyrings`
  * `curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg`
  * `sudo chmod a+r /etc/apt/keyrings/docker.gpg`
  * 
  ```bash
  echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
  ```
  * `sudo apt update`
  * `sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin bash-completion`
  * `sudo docker run hello-world`
  * `sudo usermod -aG docker ubuntu`

## Deploying containers to a Kubernetes cluster
### Install Minikube
* From the guide [here][3]
  * `curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64`
  * `sudo install minikube-linux-amd64 /usr/local/bin/minikube`
  * `minikube start`
* Get kubectl from the instructions [here][4]
  * `curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"`
  * `curl -LO "https://dl.k8s.io/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl.sha256"`
  * `sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl`
  * `echo "source <(kubectl completion bash)" >> ~/.bashrc`
  * `kubectl get pods -A`
  * `kubectl get pods -A --v=7`

### Installing Helm
* From the guide [here][5]
  * `curl https://baltocdn.com/helm/signing.asc | gpg --dearmor | sudo tee /usr/share/keyrings/helm.gpg > /dev/null`
  * `sudo apt-get install apt-transport-https --yes`
  * `echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/helm.gpg] https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list`
  * `sudo apt update`
  * `sudo apt install helm`

[1]: https://docs.docker.com/engine/install/ubuntu/
[3]: https://minikube.sigs.k8s.io/docs/start/
[4]: https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/
[5]: https://helm.sh/docs/intro/install/