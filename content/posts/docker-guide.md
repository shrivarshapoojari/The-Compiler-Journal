---
title: "Docker for Absolute Beginners: Daily Commands Explained"
date: 2025-04-14
description: "A comprehensive guide to Docker for beginners with real-world examples and detailed explanations of all essential daily-used commands, including images, containers, volumes, secrets, and Docker Swarm."
tags: ["docker", "containers", "devops","docker commands", "docker swarm"]
---


 

> 🚀 Whether you're developing applications, deploying services, or just getting started with containers — this guide gives you a complete, beginner-friendly reference for Docker. With real-world examples and all the daily-used commands explained clearly, you're about to Docker like a pro 🚀

---

## 🧠 What is Docker?

Docker is a tool that lets you package applications with their environment (libraries, OS, dependencies) into portable containers. It helps you:

- Avoid "It works on my machine" problems
- Standardize environments across dev/staging/prod
- Easily scale and ship apps

---

## 🛠️ Basic Commands

### 🧱 Image Commands

List available images:
```bash
docker images
```
Shows all images (base OS, app, etc.) stored locally on your machine.

Pull an image from Docker Hub:
```bash
docker pull nginx
```
Downloads the nginx image from Docker Hub if not already present.

Build an image from a Dockerfile:
```bash
docker build -t myapp:latest .
```
Builds an image in the current directory using Dockerfile, and tags it as myapp:latest.

Remove an image:
```bash
docker rmi myapp
```
Deletes the specified image from your local system.

---

### 📦 Container Commands

Run a container:
```bash
docker run -d --name mynginx -p 8080:80 nginx
```
Runs the nginx container in detached mode (-d), names it mynginx, and maps port 8080 (host) to 80 (container).

List running containers:
```bash
docker ps
```
Lists all currently running containers.

List all containers (including stopped):
```bash
docker ps -a
```
Lists all containers regardless of their status.

Stop a running container:
```bash
docker stop mynginx
```
Gracefully stops the specified container.

Start a stopped container:
```bash
docker start mynginx
```
Starts a container that was previously stopped.

Restart a container:
```bash
docker restart mynginx
```
Stops and then restarts the container.

Remove a container:
```bash
docker rm mynginx
```
Deletes a stopped container permanently.

Execute a command inside a running container:
```bash
docker exec -it mynginx bash
```
Runs bash inside the container interactively (-it), allowing you to explore the container like a mini VM.

---

### 🔍 Inspecting

View logs from a container:
```bash
docker logs mynginx
```
Displays standard output (like console.log or print) from the container.

Inspect container details:
```bash
docker inspect mynginx
```
Gives JSON with low-level details like IP address, mounts, environment variables, etc.

---

## 📂 Volumes (Persistent Data)

Create a volume:
```bash
docker volume create mydata
```
Creates a named volume that can be shared between containers.

List volumes:
```bash
docker volume ls
```
Shows all volumes on your Docker host.

Mount a volume in a container:
```bash
docker run -v mydata:/data alpine
```
Mounts the mydata volume at /data inside the alpine container.

---

## 🌐 Networks

List Docker networks:
```bash
docker network ls
```
Shows all available networks (bridge, host, none, custom).

Create a new network:
```bash
docker network create mynetwork
```
Creates a custom bridge network named mynetwork.

Connect a container to a network:
```bash
docker network connect mynetwork mycontainer
```
Attaches a running container to the specified network.

---

## 🐝 Docker Compose

docker-compose.yml example:
```yaml
version: '3.8'
services:
  web:
    image: nginx
    ports:
      - "8080:80"
```
Defines a service named web using nginx image, and maps port 8080 to 80.

Start services in background:
```bash
docker-compose up -d
```
Starts all services defined in docker-compose.yml in detached mode.

Stop services:
```bash
docker-compose down
```
Stops and removes all services, networks, and volumes created by docker-compose up.

Rebuild services:
```bash
docker-compose up --build
```
Rebuilds the images before starting the services.

---

## 🐳 Docker Swarm (for Production Clustering)

Initialize Swarm:
```bash
docker swarm init
```
Starts Docker Swarm mode and makes the current node the manager.

Join Swarm from another machine:
```bash
docker swarm join --token <TOKEN> <MANAGER-IP>:2377
```
Allows another node to join the Swarm cluster as a worker or manager.

List all nodes in Swarm:
```bash
docker node ls
```
Only from a manager — shows all machines in the Swarm and their roles.

Deploy a stack using Docker Compose:
```bash
docker stack deploy -c docker-compose.yml mystack
```
Deploys services defined in docker-compose.yml as a stack called mystack.

List deployed stacks:
```bash
docker stack ls
```
Displays all active stacks in the Swarm.

List services in a stack:
```bash
docker stack services mystack
```
Shows all services that belong to the stack mystack.

Remove a stack:
```bash
docker stack rm mystack
```
Deletes the stack and all its services.

📌 Always deploy stacks from the Swarm manager node.

---

## 🔐 Docker Secrets (Store Sensitive Info)

Create secret from a file:
```bash
echo "my_db_password" > db_pass.txt
docker secret create db_password db_pass.txt
```
Creates a secret named db_password using the contents of db_pass.txt.

Create secret using stdin:
```bash
echo "my_secret" | docker secret create my_secret -
```
Creates a secret from piped input, avoiding file creation.

List all secrets:
```bash
docker secret ls
```
Shows all defined secrets in the Swarm.

Use secrets in docker-compose.yml:
```yaml
services:
  app:
    image: myapp
    secrets:
      - db_password

secrets:
  db_password:
    external: true
```
Mounts the secret inside the container at /run/secrets/db_password.

---

## 🧹 Clean Up Unused Resources

Remove stopped containers:
```bash
docker container prune
```

Remove dangling images (untagged):
```bash
docker image prune
```

Remove unused volumes:
```bash
docker volume prune
```

Remove everything unused (⚠️ dangerous):
```bash
docker system prune -a
```

---

## 👨‍🔧 Dockerfile Quick Reference

A sample Dockerfile for Node.js:
```Dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 3000
CMD ["npm", "start"]
```

Build the image:
```bash
docker build -t my-node-app .
```

Run the container:
```bash
docker run -p 3000:3000 my-node-app
```

---

## 🧪 Debugging Tips

Access container’s shell:
```bash
docker exec -it <container_id> /bin/sh
```

Check container environment variables:
```bash
docker exec <container_id> env
```

Monitor real-time resource usage:
```bash
docker stats
```

---

## 📚 Summary

✅ Learn daily-used Docker commands  
✅ Build, run, debug, and deploy containers  
✅ Use volumes, networks, secrets, and stacks  
✅ Practice cleanups and write Dockerfiles  

---

🧠 Further Reading:

- https://docs.docker.com/get-started/
- https://hub.docker.com/
- https://play-with-docker.com/

Happy Dockering! 🐋✨
