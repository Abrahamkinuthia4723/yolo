# Yolo E-commerce App 🛒

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