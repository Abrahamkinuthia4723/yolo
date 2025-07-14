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

- **Small in size** 🚀
- **Fast to build and deploy** ⚡
- **Secure and maintainable** 

---
