# Module 10 Assignment — Dockerize and Deploy Express.js App Using Docker Compose with Nginx

## 📌 Assignment Overview

This assignment demonstrates how to containerize a Node.js (Express.js)
application and deploy it alongside Nginx using Docker Compose on an AWS EC2
instance.

**Source Repository:**
[roy35-909/Module-3-deployment](https://github.com/roy35-909/Module-3-deployment)

---

## 🎯 Objectives

- [x] Dockerize the Node.js (Express.js) application
- [x] Push Docker image to DockerHub
- [x] Create a `docker-compose.yml` that runs Nginx and Express.js app together
- [x] Express.js app starts **after** Nginx (`depends_on`)
- [x] Expose the application on **port 8080**
- [x] Verify the running application in the browser

---

## 🐳 DockerHub Image

```
docker pull auladdevops/module10-express-app:latest
```

🔗 **DockerHub Link:**
[https://hub.docker.com/repository/docker/auladdevops/module10-express-app](https://hub.docker.com/repository/docker/auladdevops/module10-express-appp)

## 🛠️ Dockerfile

```dockerfile
# Base image — Node 18 Alpine (lightweight)
FROM node:18-alpine

# Set working directory inside container
WORKDIR /app

# Copy package files first (for layer caching)
COPY package*.json ./

# Install production dependencies
RUN npm install --production

# Copy remaining source code
COPY . .

# Express app runs on port 3
EXPOSE 5000

# Start the application
CMD ["npm", "start"]
```

---

## ⚙️ docker-compose.yml

```yaml
version: '3.8'

services:
  # Service 1: Nginx — runs plain Nginx image
  nginx:
    image: nginx:alpine
    container_name: nginx-proxy
    ports:
      - '8080:80' # Host port 8080 → Nginx port 80
    restart: unless-stopped

  # Service 2: Express.js App — built from Dockerfile
  express-app:
    build:
      context: .
      dockerfile: Dockerfile
    container_name: express-app
    expose:
      - '5000' # Internal network only
    restart: unless-stopped
    depends_on:
      - nginx # Starts after Nginx
```

---

## 🚀 Deployment Steps

### Step 1 — Clone the Repository

### Step 2 — Install Docker & Docker Compose on EC2

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y docker.io docker-compose-plugin
sudo systemctl start docker
sudo systemctl enable docker
sudo usermod -aG docker $USER
newgrp docker
```

### Step 3 — Build Docker Image

```bash
docker build -t YOUR_DOCKERHUB_USERNAME/module10-express-app:latest .
```

### Step 4 — Push Image to DockerHub

```bash
docker login
docker push YOUR_DOCKERHUB_USERNAME/module10-express-app:latest
```

### Step 5 — Run with Docker Compose

```bash
docker compose up -d --build
```

### Step 6 — Verify Running Containers

```bash
docker compose ps
```

Expected output:

```
NAME             IMAGE          STATUS    PORTS
nginx-proxy      nginx:alpine   Up        0.0.0.0:8080->80/tcp
express-app      ...            Up        5000/tcp
```

### Step 7 — Test the Application

```bash
# Test Nginx on port 8080
curl http://localhost:8080

# Test Express app internally
docker exec express-app wget -qO- http://localhost:3000
```

---

## ✅ Verification in Browser

| URL                             | Expected Result       |
| ------------------------------- | --------------------- |
| `http://54.161.65.219:8080`     | Nginx default page    |
| `http://54.161.65.219:8080/api` | Express JSON response |

---

## 🔧 Useful Commands

```bash
# View logs for all services
docker compose logs

# View logs for a specific service
docker compose logs express-app
docker compose logs nginx

# Stop all containers
docker compose down

# Stop and remove images/volumes
docker compose down --rmi all -v

# Enter a running container
docker exec -it express-app sh
docker exec -it nginx-proxy sh
```

---
