cat > README.md << 'EOF'
# 🚀 Docker SOP (Standard Operating Procedure)

This document outlines the standard commands for building Docker images, pushing them to **Docker Hub** and **Azure Container Registry (ACR)**, and managing containers.

---

## 🛠️ Build & Run

### Build an image
docker build -t <image_name>:<tag> .

### Run a container
docker run -d -p <host_port>:<container_port> <image_name>:<tag>

---

## 📦 Docker Hub

### Login
docker login

### Tag image
docker tag <local_image>:<tag> <dockerhub_username>/<repo_name>:<tag>

### Push image
docker push <dockerhub_username>/<repo_name>:<tag>

---

## ☁️ Azure Container Registry (ACR)

### Login
az acr login --name <acr_name>

### Tag image
docker tag <local_image>:<tag> <acr_name>.azurecr.io/<repo_name>:<tag>

### Push image
docker push <acr_name>.azurecr.io/<repo_name>:<tag>

---

## 🔑 Other Important Commands

docker images
docker ps
docker stop <container_id>
docker rm <container_id>
docker rmi <image_id>
docker system prune -a

---

## ✅ Best Practices
- Use consistent tags (`latest`, `dev`, `prod`) across registries.
- Always prune unused images/containers to save space.
- Verify login before pushing to Docker Hub or ACR.
- Keep your Dockerfile optimized for smaller image sizes.

---

## 📌 Next Steps
For deployment, you can pull these images directly into **Kubernetes** or **Azure Web Apps for Containers**.
EOF
