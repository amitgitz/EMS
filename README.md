# Django Employee Management System
Employee Management System (EMS) Deployment Using Kubernetes MiniKube Pods on EC2 Instance.

## Step 1: Create an EC2 Instance with the following Configuration
1. Name: EMS
2. AMI: Ubuntu 20
3. Size: t2.medium
4. Security Group (SG):
  SSH - MyIP
    - 8000 - Anywhere IPV6
    - 8000 - Anywhere IPV4
    - 8080 - Anywhere IPV4

## Step 2: Docker & Kubernetes Setup
```
$ sudo apt update
```
Now install Docker
```
$ sudo apt install docker.io
```
Now we will install the MiniKube with the following commands
```
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
```
Now start the MiniKube using the following command
```
$ minkube start
```
if MiniKube not Stated then put the following command
Add docker as a user by giving the following command
```
$ sudo usermod -aG docker $USER && newgrp docker
$ minkube start --driver=docker
```
Now Install the Kubectl
```
$sudo snap install kubectl --classic
```
Now you can check the namespaces running using the following command
```
$ kubectl get po -A
```
## Step 3: Project Cloning
Now clone the project repo using the following command
```
$ git clone https://github.com/amitgitz/EMS
```
Now `cd` to `EMS` directory and `docker login` using the following command
```
$ docker login
```
Now give the `DockerHub` login ID and Password

## Step 4 : Docker Build & Run the EMS App
```
$ docker build -t amitchoudhary47/ems:latest .
```
```
$ docker run -d -p 8000:8000 amitchoudhary47/ems:latest
```
## Step 5 : Setting up K8's `pod.yaml`, `deployment.yaml` and `service.yaml`
```
$ mkdir k8s
$ cd k8s
```
Now push the Docker image to the DockerHub using the following Command
```
$ docker push amitchoudhary47/ems:latest
```
Now create the `pod.yaml` file in `k8s` directory - `vim pod.yaml`
```
apiVersion: v1

kind: Pod

metadata:

  name: ems

spec:

  containers:

  - name: ems

    image: amitchoudhary47/ems:latest

    ports:

    - containerPort: 8000

```
Now apply `kubectl` command to run `pod.yaml`
```
$ kubectl apply -f pod.yaml
```
Now create `deployment.yaml` file in `k8s` directory  - `vim deployment.yaml`
```
apiVersion: apps/v1

kind: Deployment

metadata:

  name: es-deployment

  labels:

    app: ems

spec:

  replicas: 3

  selector:

    matchLabels:

      app: ems

  template:

    metadata:

      labels:

        app: ems

    spec:

      containers:

      - name: ems

        image: amitchoudhary47/ems:latest

        ports:

        - containerPort: 8000

```
Now apply `kubectl` command to run `deployment.yaml`
```
$ kubectl apply -f deployment.yaml
```
Now create `service.yaml` file in `k8s` directory  - `vim service.yaml`
```
apiVersion: v1
kind: Service
metadata:
  name: ems-service
spec:
  type: NodePort
  selector:
    app.kubernetes.io/name: ems-App
  ports:
      # By default and for convenience, the `targetPort` is set to the same value as the `port` field.
    - port: 80
      targetPort: 8000
      # Optional field
      # By default and for convenience, the Kubernetes control plane will allocate a port from a range (default: 30000-32767)
      nodePort: 30007
```
Now apply `kubectl` command to run `service.yaml`
```
$ kubectl apply -f service.yaml
```

## Step 6 : Host IP allocation
```
$ sudo vim /etc/hosts
```
  - Add the IP Address of EC2 instance in the end of the file as per the following
  ```
  127.0.0.1 localhost
  # The following lines are desirable for IPv6 capable hosts
  ::1 ip6-localhost ip6-loopback
  fe00::0 ip6-localnet
  ff00::0 ip6-mcastprefix
  ff02::1 ip6-allnodes
  ff02::2 ip6-allrouters
  ff02::3 ip6-allhosts
  **192.168.49.2 ems.com**
 ``` 

Congratulations! Your application is deployed using Kubernetes MiniKube(pods).
 - If you have any questions or encounter any problems with this project, feel free to connect with me on https://www.linkedin.com/in/devopsamit/
