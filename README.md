# Client Frontend – Dockerized React App

This repository contains the **frontend client application**, built with React, and containerized using **Docker** with a multi-stage build for optimal image size.

## 🚀 Features

- React frontend setup
- Multi-stage Docker build for production optimization
- Clean `.gitignore` for client-specific files
- Docker image push to Docker Hub

## 🗂️ Project Structure

```
root/
├── client/
│   ├── Dockerfile
│   ├── .gitignore
│   ├── package.json
│   └── src/
└── README.md
```

## 🐳 Docker Setup

### 🔧 Build Docker Image



```bash
sudo docker build -t sophiesky12/client-app:latest -f client/Dockerfile client/
```

### 🛠 Dockerfile (client/Dockerfile)

This project uses a multi-stage Dockerfile reducing the image size to below 400MB

![image](https://github.com/user-attachments/assets/418fdacd-cf38-4b37-9af7-4c30defaa121)


### 📤 Push to Docker Hub (Semantic versioning)

After building the image, push it to Docker Hub:

```bash
sudo docker push sophiesky12/client-app:v1.0.0
```

Client image at:
👉 [https://hub.docker.com/repository/docker/sophiesky12/client-app](https://hub.docker.com/repository/docker/sophiesky12/client-app)

![image](https://github.com/user-attachments/assets/df432870-8a30-42e7-86d8-44c5b698e0af)


## 🧑‍💻 Author

**Sophie**
- Docker Hub: [sophiesky12](https://hub.docker.com/u/sophiesky12)

