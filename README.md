# Kubernetes Nginx Deployment using Kind

This project demonstrates how to deploy a custom Nginx website on a
Kubernetes cluster created with **Kind (Kubernetes in Docker)**.

------------------------------------------------------------------------

## Prerequisites

-   Ubuntu
-   Docker
-   kubectl
-   Kind

## Step 1: Install Docker

``` bash
sudo apt update
sudo apt install docker.io -y
```

## Step 2: Install kubectl

``` bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
```

## Step 3: Install Kind

``` bash
[ $(uname -m) = x86_64 ] && curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.32.0/kind-linux-amd64
chmod +x kind
sudo mv kind /usr/local/bin/
```

## Step 4: Configure Docker Permission

``` bash
sudo usermod -aG docker $USER
newgrp docker
```

## Step 5: Create `config.yml`

``` yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4

nodes:
- role: control-plane
  extraPortMappings:
  - containerPort: 30080
    hostPort: 8080

- role: worker
```

Create cluster:

``` bash
kind create cluster --name kind-demo --config config.yml
```

## Step 6: Create Namespace

``` bash
kubectl create namespace test-ns
```

## Step 7: Create `deployment.yml`

``` yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: nginx-deployment
  namespace: test-ns

spec:
  replicas: 1

  selector:
    matchLabels:
      app: nginx

  template:
    metadata:
      labels:
        app: nginx

    spec:
      containers:
      - name: nginx
        image: vishalpawar2672/nginxweb:latest
        imagePullPolicy: Always
        ports:
        - containerPort: 80
```

Apply:

``` bash
kubectl apply -f deployment.yml
```

## Step 8: Create `service.yml`

``` yaml
apiVersion: v1
kind: Service

metadata:
  name: nginx-service
  namespace: test-ns

spec:
  type: NodePort

  selector:
    app: nginx

  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
    nodePort: 30080
```

Apply:

``` bash
kubectl apply -f service.yml
```

## Step 9: Verify

``` bash
kubectl get deployment -n test-ns
kubectl get pods -n test-ns
kubectl get svc -n test-ns
```

## Step 10: Access Application

Open:

``` text
http://localhost:8080
```

## Architecture

``` text
Browser (localhost:8080)
        │
        ▼
Host Port 8080
        │
        ▼
Kind Port Mapping
        │
        ▼
NodePort 30080
        │
        ▼
Service
        │
        ▼
Deployment
        │
        ▼
Nginx Pod
        │
        ▼
Container Port 80
```

## Docker Image

`vishalpawar2672/nginxweb:latest`

## Author

**Vishal Pawar**
