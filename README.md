# Yolo E-commerce App 

A containerized full-stack e-commerce platform with MongoDB, Express (Node.js), and React, built using Docker and Docker Compose.


How to access this project in your local machine.

### 1. Fork this respiratory.

### 2. Open your local terminal.

### 3. Git clone the respiratory SSH url into your local terminal.

### 4. cd into yolo folder.

### 5. Finally, enter into your vs code by writing code . in your terminal and start writing your code

# Base Image Choices 

This section outlines the carefully chosen base images used in the Dockerized e-commerce platform. Each image was selected for its balance of **performance, security, and minimal size**, to ensure efficient container builds.

---

## 1. Backend – Node.js Server

| Stage        | Base Image           | Reason                                                                 |
|--------------|----------------------|------------------------------------------------------------------------|
| **Build**    | `node:18-alpine`     | ✔ Lightweight Node.js base image<br>✔ Fast npm installs<br>✔ Secure    |
| **Runtime**  | `alpine:3.18`        | ✔ Only includes what’s needed to run the app<br>✔ Reduces image size drastically (≈5MB) |

---

## 2. Frontend – React Application

| Stage        | Base Image           | Reason                                                                 |
|--------------|----------------------|------------------------------------------------------------------------|
| **Build**    | `node:18-alpine`     | ✔ Compiles React app using `npm run build`<br>✔ Alpine reduces build container bloat |
| **Runtime**  | `nginx:alpine`       | ✔ Efficiently serves static React files<br>✔ Small footprint for fast startup |

---

## 3. Database – MongoDB

| Container    | Base Image   | Reason                                                                 |
|--------------|--------------|------------------------------------------------------------------------|
| **MongoDB**  | `mongo:6`    | ✔ Official and stable image<br>✔ Supports volume persistence<br>✔ Great for local development |

---

##  Summary

By using **multi-stage builds** and minimal base images (`alpine`, `nginx:alpine`, etc.), the final Docker images remain:

- **Small in size** 
- **Fast to build and deploy** 
- **Secure and maintainable** 

---

# Dockerfile Directives

This section explains the Dockerfile setup for both the **backend** and **frontend**, using **multi-stage builds** to ensure small, secure, and production-ready container images.

---

## Backend – Node.js Server (`./backend/Dockerfile`)

```dockerfile
FROM node:18-alpine AS build

WORKDIR /usr/src/app

COPY package*.json ./
RUN npm install

COPY . .

FROM alpine:3.18

WORKDIR /app

RUN apk add --no-cache nodejs

COPY --from=build /usr/src/app /app

ENV NODE_ENV=production
ENV PORT=5000

EXPOSE 5000

CMD ["node", "server.js"]

```
---

| **Directive**                   | **Description**                                                                                      |
| ------------------------------- | ---------------------------------------------------------------------------------------------------- |
| `FROM node:18-alpine AS build`  | Uses a lightweight Node.js base image for the build stage. Improves security and reduces build time. |
| `WORKDIR /usr/src/app`          | Sets the working directory inside the container for build phase.                                     |
| `COPY package*.json ./`         | Copies package definitions for dependency installation.                                              |
| `RUN npm install`               | Installs all Node.js dependencies for backend.                                                       |
| `COPY . .`                      | Copies the entire application codebase into the image.                                               |
| `FROM alpine:3.18`              | Uses a clean, minimal Alpine Linux image (\~5MB) for runtime.                                        |
| `RUN apk add --no-cache nodejs` | Installs Node.js for running the server in the runtime container.                                    |
| `COPY --from=build`             | Transfers the compiled app from the build stage to the runtime stage.                                |
| `EXPOSE 5000`                   | Declares port 5000 to allow external access to the backend API.                                      |
| `CMD ["node", "server.js"]`     | Defines the command to start the Node.js server when the container runs.                             |

---

## Frontend – React App (`./client/Dockerfile`)

```dockerfile
FROM node:18-alpine AS build

WORKDIR /app

ENV NODE_OPTIONS=--openssl-legacy-provider

COPY package*.json ./
RUN npm install

COPY . .
RUN npm run build

FROM nginx:alpine
COPY --from=build /app/build /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```
---

| **Directive**                                        | **Description**                                                                   |
| ---------------------------------------------------- | --------------------------------------------------------------------------------- |
| `FROM node:18-alpine AS build`                       | Uses a lightweight Node.js image for building the React app.                      |
| `WORKDIR /app`                                       | Sets the working directory inside the container.                                  |
| `ENV NODE_OPTIONS=--openssl-legacy-provider`         | Fixes a common OpenSSL issue with Node 17+ when using Webpack.                    |
| `COPY package*.json ./`                              | Copies the dependency files (package.json & lock file) for faster caching.        |
| `RUN npm install`                                    | Installs frontend dependencies.                                                   |
| `COPY . .`                                           | Copies the entire source code into the container.                                 |
| `RUN npm run build`                                  | Compiles the React app into production-ready static files in the `build/` folder. |
| `FROM nginx:alpine`                                  | Starts a new container stage using a lightweight NGINX server.                    |
| `COPY --from=build /app/build /usr/share/nginx/html` | Moves the built React files to the default NGINX directory.                       |
| `EXPOSE 80`                                          | Exposes port 80 so Docker knows where NGINX listens.                              |
| `CMD ["nginx", "-g", "daemon off;"]`                 | Keeps NGINX running in the foreground (Docker best practice).                     |

---

# Docker-compose networking.

To enable efficient communication between containers, I implemented a custom bridge network in the docker-compose.yml file. This approach ensures that all services can resolve each other by container name and interact seamlessly while remaining isolated from the host environment unless explicitly exposed.

---

### Network Configuration

```yaml
networks:
  yolo-network:
    driver: bridge
```
---

| Key            | Description                                                              |
| -------------- | ------------------------------------------------------------------------ |
| `yolo-network` | Custom user-defined bridge network for internal container communication. |
| `bridge`       | The default Docker driver, ideal for linking standalone containers.      |

---

### Network Usage in Services

All services (MongoDB, Backend, Frontend) are connected to the same bridge network to enable internal communication:

```yaml
services:
  mongo:
    networks:
      - yolo-network

  backend:
    networks:
      - yolo-network

  frontend:
    networks:
      - yolo-network
```
---

This allows:

- **The backend to connect to MongoDB via the hostname mongo.**
- **The frontend to communicate with the backend using backend as the hostname.**

| Service  | Container Port | Host Port | Purpose                               |
| -------- | -------------- | --------- | ------------------------------------- |
| MongoDB  | `27017`        | `27017`   | Database access (for tools/CLI).      |
| Backend  | `5000`         | `5000`    | API server (Node.js).                 |
| Frontend | `80`           | `3000`    | NGINX serving React app at port 3000. |

---

# Volume Definition & Usage

```yaml
volumes:
  mongo-data:
```

**mongo-data:/data/db**: Ensures MongoDB stores data persistently (e.g. added products).

---

# Git Workflow
---
Below is a summary of the Git workflow followed during this project:
#### Deleted initial Dockerfiles and docker-compose
#### Add Dockerfile for backend service using node:18-alpine
#### Add Dockerfile for frontend service with multi-stage build
#### Create docker-compose.yaml with MongoDB, backend, and frontend services
#### Push backend and frontend images to DockerHub with version tags
#### Update README.md file
#### Explain reasoning for base image choices
#### Document Dockerfile directives in README.md
#### Explain Docker Compose networking strategy
#### Explain Docker volume definition and usage
#### Deleted existing Vagrantfile, playbook.yml etc.  
#### Initialize Vagrantfile  
#### Write basic Ansible inventory and playbook.yml  
#### Add variable file with container and network config  
#### Add common role to install Docker and set up network  
#### Add mongo role to run MongoDB container with volume  
#### Add backend role to run Node.js API container  
#### Add frontend role to run React container  
#### Updated the Vagrantfile  
#### Fixed errors  
#### Updated readme
#### Created manifest folder and added yolo-frontend Deployment
#### Add frontend service for browser access
#### Create backend deployment manifest
#### Add backend service for internal API communication
#### Add MongoDB StatefulSet configuration
#### configure MongoDB service for database connectivity
#### update readme
---

 # Docker Best Practices Followed
 ---
 Multi-stage builds for minimal image size

 Image versioning using tags:

- ***abrahamkinuthia4723/yolo-frontend:1.0.0***
- ***abrahamkinuthia4723/yolo-backend:1.0.0***

 Images pushed to DockerHub

 Image sizes optimized:
<div align="center">
  <img src="https://github.com/user-attachments/assets/c454ed91-fef7-45fb-813f-439626ccea66" alt="Deployment on Vagrant" style="width:800px; max-width:100%; height:auto;" />
</div>

---

# Deployed Images on DockerHub

<div align="center">
  <img src="https://github.com/user-attachments/assets/2f6b9168-6408-4103-82db-b28b52686784" alt="Backend Docker Image" style="width:700px; max-width:100%; height:auto;" />
  <br/><br/>
  <img src="https://github.com/user-attachments/assets/91b8d060-b63c-4958-8ebd-85d1b72645c7" alt="Frontend Docker Image" style="width:700px; max-width:100%; height:auto;" />
</div>

---

# How the Ansible automation works in this project and the reasoning behind each part

This project automates the deployment of a containerized e-commerce web application using:

- **Vagrant** to create a virtual machine (VM)
- **Ansible** to configure the VM and run Docker containers
- **Docker** to run the application services:
  - MongoDB (Database)
  - Backend (Node.js API)
  - Frontend (React.js App)

All services are deployed inside the VM with a single command: `vagrant up`.

---

## Deployment Workflow

The automation flow happens in four main roles:

1. **Common** – Installs tools like Docker and Git, and creates a Docker network.
2. **MongoDB** – Starts the MongoDB container and creates a data volume.
3. **Backend** – Runs the backend service in a container.
4. **Frontend** – Launches the frontend React web application.

Each step builds upon the previous one, ensuring all dependencies are in place.

---

## Role Breakdown

### 1. Common Role

**Purpose**: Prepares the environment by installing essential tools.

**Key Tasks**:
- Updates system package cache.
- Installs necessary packages:
  - `git`
  - `docker.io`
- Installs Docker Compose manually (latest release).
- Starts and enables the Docker service.
- Creates a custom Docker bridge network (`yolo-network`).
- Clones the YOLO E-Commerce application repository from GitHub into a specified directory.

---

### 2. MongoDB Role

**Purpose**: Sets up the MongoDB database.

**Key Tasks**:
- Pull the MongoDB image: `mongo:6`
- Create a container named `mongo-db`
- Expose port `27017` for internal connections
- Mount a volume: `mongo-data:/data/db` (for data persistence)
- Connect to the `yolo-network`

---

### 3. Backend Role

**Purpose**: Launches the backend API service.

**Key Tasks**:
- Pull image: `abrahamkinuthia4723/yolo-backend:1.0.0`
- Create a container named `backend-container`
- Expose port `5000`
- Connect to the `yolo-network`
- Ensure it depends on the MongoDB container

---

### 4. Frontend Role

**Purpose**: Runs the user-facing React application.

**Key Tasks**:
- Pull image: `abrahamkinuthia4723/yolo-frontend:1.0.0`
- Create a container named `frontend-container`
- Map internal port `80` to host port `3000`
- Connect to the `yolo-network`
- Ensure it depends on the backend

---

## Ansible Features Used

### Roles

Each role (common, mongo, backend, frontend) is isolated in its own directory under `roles/` for better structure and reuse.

### Variables

Defined in `vars/main.yml`, variables make it easy to change image names, ports, and other settings from one file.

**Example**:
```yaml
backend_image: abrahamkinuthia4723/yolo-backend:1.0.0
```

---

## Kubernetes Objects Used

- **StatefulSet** for MongoDB: Provides stable network ID and persistent volumes, ensuring data persists across pod restarts.
- **Deployments** for backend and frontend: Enables easy scaling, rolling updates, and self-healing of pods.
- **Services** to expose pods internally and externally:
  - `ClusterIP` services for internal communication between backend and MongoDB.
  - `LoadBalancer` service to expose the frontend to the internet.
- **PersistentVolumeClaims** to store MongoDB data securely and persistently.

---

## Prerequisites

- A Kubernetes cluster (preferably GKE) with `kubectl` configured and connected.
- Docker images for backend and frontend pushed to Docker Hub:
  - `abrahamkinuthia4723/yolo-backend:1.0.0`
  - `abrahamkinuthia4723/yolo-frontend:1.0.0`

---

## Kubernetes Manifests Setup

## Step 1: Create Kubernetes Manifest Files for Deployments and StatefulSet

- Start by creating the necessary YAML files that will define the deployments for your application components:

``` bash
mkdir -p manifests
cd manifests
touch frontend-deploy.yaml
touch backend-deploy.yaml
touch mongo-statefulset.yaml
```

- **frontend-deploy.yaml**: Defines the deployment configuration for the frontend application.
- **backend-deploy.yaml**: Contains the deployment setup for the backend service.
- **mongo-statefulset.yaml**: Sets up MongoDB using a StatefulSet to ensure stable network identity and persistent storage.

## Step 2: Create Service Definitions to Expose Components

- Define Kubernetes Services to handle internal communication and external access for your application components.

```bash
touch manifests/backend-service.yaml
touch manifests/frontend-service.yaml
touch manifests/mongo-service.yaml
```

### Internal Services:

- **backend-service.yaml**: Exposes the backend internally to other services within the cluster.

- **mongo-service.yaml**: Provides stable access to MongoDB 

### Public Access:

- **frontend-service.yaml**: Exposes the frontend to the internet using a LoadBalancer on port 80, allowing users to access the app through a standard web port.


## Deployment Instructions

1. Apply the MongoDB service and StatefulSet:

    ```bash
    kubectl apply -f manifests/mongo-svc.yml
    kubectl apply -f manifests/mongo-statefulset.yml
    ```

2. Deploy the backend service and deployment:

    ```bash
    kubectl apply -f manifests/backend-svc.yml
    kubectl apply -f manifests/backend-deploy.yml
    ```

3. Deploy the frontend service and deployment:

    ```bash
    kubectl apply -f manifests/frontend-svc.yml
    kubectl apply -f manifests/frontend-deploy.yml
    ```
4. Or simply run:

    ```bash
     kubectl apply -f manifests/
     ```
    which deploys or updates all Kubernetes resources defined in the YAML files inside the manifests/ folder to your cluster.

5. Check the status of the pods:

    ```bash
    kubectl get pods
    ```
---

## Running pods
  
  <img src="https://github.com/user-attachments/assets/b1d0c5af-017b-4d3b-a198-b0525d55c101" alt="running pods" style="width:800px; max-width:100%; height:auto;" />

---

## Persistence

MongoDB uses a StatefulSet backed by PersistentVolumeClaims. This ensures data is not lost if the MongoDB pod restarts or is rescheduled.

---
