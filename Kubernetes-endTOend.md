
## Kubernetes Cluster Deployment and Container Orchestration on AWS
- Build Kubernetes cluster on AWS from Scratch with Minkube
- Setup and Managed Docker Containers for Django and React Applications into Kubernetes Pods
- Managed Deployment, Replication, Autohealing, auto scaling for Kubernetes Cluster.
- Managed network and services with Host IP allocation through Proxy on AWS EC2 and Route53
Achievement: Reduced Downtime by 75% on Production Environments


### Installing minikube (Local Kubernetes)
```bash
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube && rm minikube-linux-amd64
```

To Start minikube it require a Docker as Driver (or a virtual machine), So we have to install Docker 
### Install using the apt repository
#### Set up Docker's apt repository.

```bash
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```

#### Install the Docker packages
```bash
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

#### Verify that the Docker Engine installation is successful by running the hello-world image.
```bash
sudo docker run hello-world
```

#### Giving Perminssion to Docker by adding docker to present User Group and new group docker
```bash
sudo usermod -aG docker $USER && newgrp docker
```

Now You can start minikube

#### Starting minikube
```bash
minikube start --driver=docker
```

#### Installing kubectl --classic 
- Kubectl is a command-line tool used to manage and interact with Kubernetes clusters for deploying and monitoring applications
```bash
sudo snap install kubectl --classic
```

#### List all the Pods across all namespaces
```bash
$ kubectl get po -A

$ minikube status
# checks the status of the Minikube cluster, displaying information about the cluster's components such as the VM, Kubernetes, and any running services.

$ kubectl get node
#retrieves information about the nodes in the Kubernetes cluster, listing their status, roles, and other relevant details.
```



Clone the Project repo
```bash
$ git clone https://github.com/manishkr04/Django_Todo_Web_App.git
```
Build the Docker image and Containerise it 
``` bash
docker buiild -t manishkr4u/django-todo:latest .
docker run -d -p 8000:8000 manishkr4u/django-todo:latest  
# manishkr4u is the username of Docker Hub where images is managed
```

Check if the container(TodoApp) is running or not
```bash
curl -L http://IP-address:8000
# if this return an output i.e the App is Running
```

Creating Kubernetes Pods

- Generally kubernetes Pods pull images from Docker Hub, so we have to push the image to Docker Hub
```bash
docker images
docker push manishkr4u/django-todo
```

- Now Create docker Files
```bash
mkdir k8s # to segregate and manage Kubernetes Files
cd k8s/
vim pod.yaml # file to create Pod

apiVersion: v1

kind: Pod

metadata:

  name: todo-pod

spec:

  containers:

  - name: todo-app

    image: manishkr4u/django-todo:latest

    ports:

    - containerPort: 8000

# save and Exit

# check if there is running container on 8000 port because we have to create Pod , if it is running the kill that docker container 
docker ps
docker kill <Container-ID>

kubectl apply -f pod.yaml # command to create POD

#check if the pod is running 
kubectl get pods
# You can see that single pod is running name todo-pod
```

App Deployment 

```bash 
#create a kubernetes deployment file 
vim deployment.yaml

apiVersion: apps/v1

kind: Deployment

metadata:

  name: todo-deployment

  labels:

    app: todo-app

spec:

  replicas: 3

  selector:

    matchLabels:

      app: todo-app

  template:

    metadata:

      labels:

        app: todo-app

    spec:

      containers:

      - name: todo-app

        image: manishkr4u/django-todo:latest

        ports:

        - containerPort: 8000

# save and exit 

kubectl app -f deployment.yaml

kubectl get deployments

kubectl get pods # you can see three running pods because in deployment.yaml file we set 3 replicas for auto healing and auto scalling 

# basically the pod created by pod.yaml file can be deleted or manipulated but pods created using deployment.yaml file cannot be manipulated as it supports Replication of pods , Autohealing and auto scalling. 


# TO see the running pods along with its associated IP 
kubectl get pods -o wide


# When trying to access the aap with associated IP 
curl -L http://IP-address:8000
# this will give an error , Filed to Connect to "Ip-address" port 8000 after 3077 ms : No route to host

# Cause: The pod is running inside the Kubernetes cluster, but there is no service exposing the pod to the outside world.
```

Create a kubernetes service to expose the pod outside the cluster. 
- You can create a service with type NodePort or LoadBalancer.

```bash
vim service.yaml # NodePort Type service

apiVersion: v1
kind: Service
metadata:
  name: todo-service
spec:
  type: NodePort
  selector:
    app: todo-app
  ports:
      # By default and for convenience, the `targetPort` is set to the same value as the `port` field.
    - port: 80
      targetPort: 8000
      # Optional field
      # By default and for convenience, the Kubernetes control plane will allocate a port from a range (default: 30000-32767)
      nodePort: 30007

# save and Exit

kubectl apply -f service.yaml
#create or update resources (like services) in the Kubernetes cluste 

kubectl get svc 
# Lists all the services running in the Kubernetes cluster along with their details such as IPs and exposed ports. 

minikube service todo-service --url
 # Retrieves the URL to access the todo-service running in Minikube, allowing you to interact with the service from your local machine.

#Now you can Curl the url you have received
curl -L http://192.168.49.2:30007
