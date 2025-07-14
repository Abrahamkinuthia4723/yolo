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
---

 # Docker Best Practices Followed
 ---
 Multi-stage builds for minimal image size

 Image versioning using tags:

- ***abrahamkinuthia4723/yolo-frontend:1.0.0***
- ***abrahamkinuthia4723/yolo-backend:1.0.0***

 Images pushed to DockerHub

 Image sizes optimized:
 https://github.com/user-attachments/assets/c454ed91-fef7-45fb-813f-439626ccea66

---

# Deployed images on DockerHub
https://github.com/user-attachments/assets/2f6b9168-6408-4103-82db-b28b52686784
https://github.com/user-attachments/assets/91b8d060-b63c-4958-8ebd-85d1b72645c7

---