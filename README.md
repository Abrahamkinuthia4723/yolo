# E-commerce Microservice Platform

A containerized e-commerce web application built with Docker and Docker Compose, featuring product management capabilities and MongoDB persistence.

## 🚀 Features

- **Product Management**: Add, view, and manage retail products through an intuitive dashboard
- **Microservice Architecture**: Containerized components for scalability and maintainability
- **Data Persistence**: MongoDB integration with persistent storage

## 🛠 Technology Stack

- **Backend**: Node.js/Express.js
- **Database**: MongoDB
- **Frontend**: HTML/CSS/JavaScript
- **Containerization**: Docker & Docker Compose
- **Orchestration**: Custom bridge networks

## 📋 Prerequisites

Before running this application, ensure you have the following installed:

- [Docker](https://docs.docker.com/get-docker/) (version 20.10+)
- [Docker Compose](https://docs.docker.com/compose/install/) (version 1.29+)
- [Git](https://git-scm.com/downloads)

## 🔧 Installation & Setup

### 1. Clone the Repository
```bash
git clone git@github.com:Sophie-Muchiri12/yolo.git
cd yolo
```



### 3. Build and Run with Docker Compose
```bash
# Build and start all services
docker-compose up --build


```

### 4. Access the Application
- **E-commerce Platform**: http://localhost:3000
- **Backend product API**: http://localhost:5000/api/products



## 🐳 Docker Architecture

### Base Images

#### Backend (Node.js API)

Base Image: node:18-alpine
Reasoning:

Alpine Linux provides minimal footprint (~5MB base) reducing overall image size
Node.js 18 is the current LTS version ensuring stability and long-term support
Alpine's package manager (apk) allows easy installation of additional tools like curl
Security benefits due to minimal attack surface



#### Frontend (React Application)

Base Image: node:18-alpine (multi-stage build)
Reasoning:

Multi-stage build separates build environment from runtime environment
First stage compiles React application with full development dependencies
Second stage creates minimal production image serving only built static files
Reduces final image size by ~70% compared to single-stage build
Uses serve package for efficient static file serving


#### Docker Images with Semantic Versioning

   Both Backend and Client images are versioned and are less than 400MB of size
   
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

## 🚀 Usage

### Adding Products
1. Navigate to the admin dashboard
2. Fill out the product form with:
   - Product name
   - Description
   - Price
  
3. Click "Add Product" to save

- NB:  Add product and reload page (this is caused by improper update of state management however not the goalfor the challenge which is strictly practicing on Docker)


## 📊 API Endpoints

### Product Management
```
GET    /api/products          # Get all products
POST   /api/products          # Create new product

```


### Docker Compose 
Create `docker-compose.yml` for local customizations:
```yaml
version: '3.8'
services:
  web-app:
    environment:
      - DEBUG=true
    volumes:
      - ./src:/app/src:ro
```


### Commits (Minimum 10):

 Added Docker compose
 Set up a docker image for backend with semantic versioning
 Added multistage build and image pushed to dockerhub
 Client Docker file created
 Initial Project setup & Cleanup
 Final README.md & explanation.md


## Dockerhub Repo 
https://hub.docker.com/repositories/sophiesky12

backend image : backend-final v1.0.0
client image : clients-final v1.0.0
