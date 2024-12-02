# Goals Project

A fullstack MERN project which has Reactjs as frontend, Nodejs as backend, and MongoDB as a Database.

## Project Set Up

Clone this repository from the GitHub URL.

```sh
git clone https://github.com/MdShimulMahmud/goal-projects.git
cd goal-projects
code .
```

### 1. Backend Development

1.1 **Open Backend**
```bash
cd backend
```

1.2 **Install Dependencies**
```bash
npm install
```

1.3 **Run Backend**
```bash
npm start
```

1.4 **Check Application running on:** http://localhost:5000/goals

### 2. Frontend Application

2.1 **Open Frontend**
```bash
cd frontend
```

2.2 **Install Dependencies**
```bash
npm install
```

2.3 **Run Frontend**
```bash
npm start
```

2.4 **Check Application running on:** http://localhost:3000

## Dockerized Application

### Backend Application

#### Dockerfile

```dockerfile
FROM node:16-alpine

WORKDIR /app

COPY package.json .

COPY . .

RUN npm install

EXPOSE 5000

CMD ["npm", "start"]
```

#### Build Docker Image
```bash
docker build -t <image name>:<image tag> <dockerfile path>
```
Best Practice:
```bash
docker build -t your-dockerhub-username/backend:tag -f backend/Dockerfile .
```
For example:
```bash
docker build -t shimulmahmud/backend:v1.0.4 .
```

#### Run Docker Image
```bash
docker run -d -p <host-port>:<container-port> <image name>:<image tag>
```
For example:
```bash
docker run -d -p 5000:5000 shimulmahmud/backend:v1.0.4
```

#### Push Image to Docker Registry
```bash
docker login
docker push your-dockerhub-username/backend:tag
```
For example:
```bash
docker push shimulmahmud/backend:v1.0.4
```

### Frontend Application

#### Multistage Image Build

```dockerfile
FROM node:16-alpine AS build-stage

WORKDIR /app

COPY package.json .

RUN npm install

COPY . .

RUN npm run build

FROM nginx:latest

COPY --from=build-stage /app/build /usr/share/nginx/html
COPY ./conf/nginx.conf /etc/nginx/conf.d/default.conf

CMD ["nginx", "-g", "daemon off;"]
```

Here we used nginx as a web server which provides static web content and listens on port 80 by default.

#### Build Docker Image
```bash
docker build -t <image name>:<image tag> <dockerfile path>
```
Best Practice:
```bash
docker build -t your-dockerhub-username/frontend:tag -f frontend/Dockerfile .
```
For example:
```bash
docker build -t shimulmahmud/frontend:v1.0.4 .
```

#### Run Docker Image
```bash
docker run -d -p <host-port>:<container-port> <image name>:<image tag>
```
For example:
```bash
docker run -d -p 3000:80 shimulmahmud/frontend:v1.0.4
```

#### Push Image to Docker Registry
```bash
docker login
docker push your-dockerhub-username/frontend:tag
```
For example:
```bash
docker push shimulmahmud/frontend:v1.0.4
```

### Reverse Proxy Nginx

To set up Nginx as a reverse proxy for your MERN stack application, you can configure Nginx to route traffic to both your frontend (React) and backend (Node.js/Express) applications.

Here’s how you can configure Nginx to act as a reverse proxy in a Dockerized environment:

You’ll need to create a custom Nginx configuration file (`default.conf`) to set up the reverse proxy.

#### Nginx Configuration

```default.conf```
```bash
upstream frontend {
    server frontend:3000;
}

upstream backend {
    server backend:5000;
}

server {
    listen 80;

    location / {
        proxy_pass http://frontend;
    }

    location /api {
        rewrite /api/(.*) /$1 break;
        proxy_pass http://backend;
    }
}
```

In this configuration:
1. Frontend (React) is served from http://frontend:3000.
2. Backend (Node.js/Express API) is served from http://backend:5000.
3. The location / route handles requests for the frontend.
4. The location /api route handles API requests that should be forwarded to the backend.

#### Dockerize Nginx

To use Nginx as a reverse proxy in a Dockerized environment, you'll need to create a Dockerfile for Nginx and replace its default configuration.

```dockerfile
FROM nginx:alpine

RUN rm /etc/nginx/conf.d/default.conf

COPY default.conf /etc/nginx/conf.d/default.conf
```

To understand how a reverse proxy works, you can [Visit Here](https://www.upguard.com/blog/reverse-proxy-vs-load-balancer)

![Reverse Proxy](https://github.com/MdShimulMahmud/goal-projects/blob/master/reverse_proxy.png)

## Docker Compose

To set up a Docker Compose configuration for your MERN stack application (MongoDB, Express, React, Node.js) along with Nginx as a reverse proxy, here’s a structured example.

### Project Structure

```bash
/goal-projects
  ├── backend/
  │   ├── Dockerfile
  │   └── ...
  ├── frontend/
  │   ├── Dockerfile
  │   └── ...
  ├── nginx/
  │   ├── Dockerfile
  │   └── nginx.conf
  ├── docker-compose.yml
  └── ...
```

### Docker Compose Configuration (`docker-compose.yml`)

```yaml
version: "3.8"

services:
  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile

  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile
    depends_on:
      - backend

  nginx:
    build:
      context: ./nginx
      dockerfile: Dockerfile
    ports:
      - "80:80"
    depends_on:
      - backend
      - frontend
```
Run docker-compose : ```docker-compose up --build```
1. Docker Compose manages multiple containers for the MERN stack, including frontend, backend, and nginx.
2. The Nginx container acts as a reverse proxy to route traffic to the frontend (/) and backend (/api).
3. Dockerfiles are defined for the frontend, backend, and Nginx.

## Kubernetes Manifests

### Overview

- **Deployment**: Describes the pods that should run for your frontend, backend, and MongoDB services.
- **Service**: Exposes each component (frontend, backend, MongoDB) to the network within the Kubernetes cluster.
- **Ingress**: Manages external access to your services, routing traffic to the frontend and backend.
- **Secrets**: Stores sensitive information such as MongoDB credentials.
- **ConfigMap**: Stores configuration data that can be accessed by the application pods.
- **PersistentVolumeClaim (PVC)**: Allows MongoDB to store data persistently.

### Example Manifests

#### Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend-deployment
  labels:
    app: frontend
    type: frontend
spec:
  replicas: 1
  selector:
    matchLabels:
      app: frontend
      type: frontend
  template:
    metadata:
      labels:
        app: frontend
        type: frontend
    spec:
      containers:
      - name: frontend-container
        image: shimulmahmud/frontend:v1.0.4
        ports:
        - containerPort: 3000

--- 

apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend-deployment
  labels:
    app: backend
    type: backend
spec:
  replicas: 1
  selector:
    matchLabels:
      app: backend
      type: backend
  template:
    metadata:
      labels:
        app: backend
        type: backend
    spec:
      containers:
      - name: backend-conatiner
        image: shimulmahmud/backend:v1.0.4
        ports:
        - containerPort: 5000
        volumeMounts:
          - mountPath: /app/logs/
            name: access-log-volume
            # subPath: access.log
        env:
          - name: DATABASE_PASSWORD
            valueFrom:
              secretKeyRef:
                name: database-secret
                key: password
          - name: DATABASE_USERNAME
            valueFrom:
              secretKeyRef:
                name: database-secret
                key: username
          - name: DATABASE_URL
            valueFrom:
              configMapKeyRef:
                name: configmap
                key: database
      volumes:
        - name: access-log-volume
          persistentVolumeClaim:
            claimName: test-claim
```

#### Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: client-service
spec:
  selector:
    app: frontend
    type: frontend
  ports:
    - protocol: TCP
      port: 3000
      targetPort: 3000
  type: ClusterIP 

---

apiVersion: v1
kind: Service
metadata:
  name: backend-service
spec:
  selector:
    app: backend
    type: backend
  ports:
    - protocol: TCP
      port: 5000
      targetPort: 5000
  type: ClusterIP 
```

#### Ingress

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: minimal-ingress
  annotations:
    kubernetes.io/ingress.class: 'nginx'
    nginx.ingress.kubernetes.io/use-regex: 'true'
    # nginx.ingress.kubernetes.io/rewrite-target: /$1
spec:
  ingressClassName: nginx
  rules:
  - http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: client-service
            port:
              number: 3000
      - path: /goals
        pathType: Prefix
        backend:
          service:
            name: backend-service
            port:
              number: 5000
```

#### Secrets

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: database-secret
type: Opaque
data:
  password: <base64 your mongodb password>
  username: <base64 your mongodb username>
```

#### ConfigMap

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: configmap
data:
  database: "cluster1.uktxj.mongodb.net/?retryWrites=true&w=majority&appName=Cluster1"
```

#### PersistentVolumeClaim (PVC)

To set up nfs-server for persistent volume you can [read this](https://github.com/nasirnjs/kubernetes/blob/main/k8s-cluster-setup/dynamic-nfs-provisioning_k8s.md) .

```yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: test-claim
spec:
  storageClassName: nfs-client
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 100Mi
```

To apply and create corresponding services, run the command in the terminal ```cd k8s``` and 
```kubectl apply -f .```

## CI/CD Pipeline Build with GitHub Actions

[ **Workflow Overview** ](https://github.com/MdShimulMahmud/goal-projects/blob/master/.github/workflows/ci.yaml)

### Triggers

- **Push**: Triggers on push events to the `master` branch, excluding changes to `k8s/deployment.yaml`.
- **Pull Request**: Triggers on pull requests to the `master` branch, excluding changes to `k8s/deployment.yaml`.

### Jobs

#### 1. Test

- **Runs on**: `ubuntu-latest`
- **Steps**:
    - Checkout code
    - Run Approval Action
    - Store the approval output as an environment variable
    - Get the output

#### 2. Backend

- **Runs on**: `ubuntu-latest`
- **Needs**: `test` job
- **Condition**: Runs if the `test` job's approval output is `true`
- **Steps**:
    - Checkout code
    - Set up Node.js
    - Install dependencies
    - Debug Run Number
    - Check current version

#### 3. Frontend

- **Runs on**: `ubuntu-latest`
- **Needs**: `backend` job
- **Steps**:
    - Checkout code
    - Set up Node.js
    - Install dependencies
    - Debug Run Number
    - Check current version

#### 4. Version Check

- **Runs on**: `ubuntu-latest`
- **Needs**: `frontend` and `backend` jobs
- **Steps**:
    - Checkout code
    - Extract current version
    - Read previous version from `deployment.yaml`
    - Check if current version differs from previous

#### 5. Docker

- **Runs on**: `ubuntu-latest`
- **Needs**: `frontend`, `backend`, and `version_check` jobs
- **Condition**: Runs if the `version_check` job's version_diff output is `true`
- **Steps**:
    - Checkout code
    - Login to Docker Hub
    - Set up Docker Buildx
    - Build and push frontend image
    - Build and push backend image

#### 6. Tags

- **Runs on**: `ubuntu-latest`
- **Needs**: `docker` and `version_check` jobs
- **Condition**: Runs if the `version_check` job's version_diff output is `true`
- **Steps**:
    - Checkout code
    - Set the new image version
    - Update Kubernetes `deployment.yaml` image tags
    - Commit and push changes

## CI/CD Pipeline Build with Jenkins

This [Jenkins pipeline ](https://github.com/MdShimulMahmud/goal-projects/blob/master/Jenkinsfile) is designed to automate the build and deployment process for the Goals project. The pipeline consists of three main stages: Backend, Frontend, and Docker. It also includes environment variables for GitHub and Docker credentials.

Your jenkins server running on http://localhost:8080 or ```your_server_ip_address:8080```

## Environment Variables

- `GH_TOKEN`: GitHub token for accessing private repositories.
- `DOCKER_USERNAME`: Docker Hub username for logging in and pushing images.
- `DOCKER_PASSWORD`: Docker Hub access token for authentication.

## Stages

### 1. Backend

- **Agent**: Any available agent.
- **Steps**:
    - Change directory to `backend`.
    - Install npm dependencies.
    - Print the current build number.
    - Read and print the current version from the `VERSION` file.

### 2. Frontend

- **Agent**: Any available agent.
- **Steps**:
    - Change directory to `frontend`.
    - Install npm dependencies.
    - Build the frontend application.
    - Print the current build number.
    - Read and print the current version from the `VERSION` file.

### 3. Docker

- **Agent**: Any available agent.
- **Steps**:
    - Read the current version from the `VERSION` file.
    - Log in to Docker Hub using the provided credentials.
    - Build and push Docker images for both frontend and backend using the current version as the tag.
    - Handle any exceptions that occur during the Docker build or push process.

### 4. Update Manifest Pipeline Trigger

This project contains a Jenkins pipeline stage to trigger the `update-k8s-manifests` job by passing the current version as an `IMAGE_TAG` parameter.

The `Trigger Update Manifest Pipeline` stage performs the following tasks:
1. Reads the current version from a `VERSION` file.
2. Triggers the `update-k8s-manifests` pipeline with the current version as a parameter.

#### Code Snippet

```groovy
stage('Trigger Update Manifest Pipeline') {
    agent any
    steps {
        script {
            def current_version = sh(script: "cat VERSION", returnStdout: true).trim()
            
            build job: 'update-k8s-manifests', parameters: [
                string(name: 'IMAGE_TAG', value: "${current_version}")
            ]
        }
    }
}
```
In Infra Repository, you will find all the manifests used in this project. [Repo url](https://github.com/MdShimulMahmud/goal-projects-infra.git)


## Post Actions

- **Always**: Print a message indicating that the pipeline has completed.
- **Failure**: Print a message indicating that the pipeline has failed and suggest checking the logs for errors.
- **Success**: Print a message indicating that the pipeline has succeeded.

## Example Usage

To use this Jenkinsfile, place it in the root directory of your project and configure your Jenkins job to use it. Ensure that the necessary credentials (`github-token`, `docker-username`, and `docker-password`) are configured in Jenkins.

Happy Coding!!!
