# edu-docker

[Docker Desktop](https://www.docker.com/products/docker-desktop/)
: Here you download Docker Desktop for installation  

[Docker Hub](https://hub.docker.com)
: Here you find all virtual machines, enter node in the search and you find all images created by node.js - It is beneficial to register and login with Docker Desktop

[Alpine](https://alpinelinux.org/)  
: This is the Linux distro we use  

<hr>

## Docker Network

> We create a virtual network where we can run business-frontend, admin-frontend, business-backend, admin-backend and auth-backend

```bash
docker network create --subnet=172.20.0.0/24 fwk-net
docker network ls
docker inspect fwk-net
```

## Auth Server

```bash
cd ~
cd ws
cd auth-server
```


### .dockerignore <heredoc

```bash
cat > .dockerignore << 'EOF'
node_modules
build
dist
.git
.env
EOF
```
### Dockerfile <heredoc

```bash
cat > Dockerfile << 'EOF'
# Use an official Node runtime as a parent image
FROM node:18-alpine

# Install bash
RUN apk update && apk add --no-cache bash

# Set the working directory in the container
WORKDIR /usr/src/app

# Copy package.json and package-lock.json into the working directory
COPY package*.json ./

# Install any needed packages specified in package.json
RUN npm install

# Bundle app source inside Docker image
COPY . .

# Make port 3000 available to the world outside this container
EXPOSE 3000

# Define environment variable
ENV PORT=3000

# Run the app when the container launches
CMD ["node", "src/service.js"]
EOF
```

### Build and run your image

```bash
docker build -t auth-server .
docker image ls
docker run --name fwk-auth --network fwk-net --ip 172.20.0.2 -p 3000:3000 -d auth-server
docker ps
```

### Rebuild your image and run it

> When we change our application, we need to stop the container, remove the image, rebuild the image and then run it again

```bash
docker stop fwk-auth
docker rm fwk-auth
docker image rm auth-server
docker build -t auth-server .
docker run --name fwk-auth --network fwk-net --ip 172.20.0.2 -p 3000:3000 -d auth-server
```

### Visit your running docker image (your virtual machine)

```bash
docker exec -it fwk-auth /bin/bash # ctrl-d - exit back to real machine
```
<hr>

## Docker for frontend

> This is for a React app built with Vite.


```bash
cd ~
cd ws
cd frontend # Change to the name you  use for the business server
```

### .dockerignore <heredoc

```bash
cat > .dockerignore << 'EOF'
node_modules
build
dist
.git
.env
EOF
```
### Dockerfile <heredoc

```bash
cat > Dockerfile << 'EOF'
FROM node:18-alpine

COPY package.json .

WORKDIR /app

COPY . .

RUN npm install

ENV VITE_AUTH_URL=http://172.20.0.2:3000

ENV VITE_API_URL=http://172.20.0.4:3000

RUN npm run build

EXPOSE 5000

CMD ["npm", "run", "preview"]
EOF
```

### vite.config.js <heredoc

```bash
cat > vite.config.js << 'EOF'
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";

export default defineConfig({
  base: "/",
  plugins: [react()],
  preview: {
    port: 5000,
    strictPort: true,
  },
  server: {
    port: 5000,
    strictPort: true,
    host: true,
    origin: "http://0.0.0.0:5000",
  },
});
EOF
```

### Build & Run the Image connect port 3001 to it

```bash
docker build -t frontend .
docker run --name fwk-front --network fwk-net --ip 172.20.0.3 -p 3001:5000 -d frontend
```

### Rebuild your image and run it

> When we change our application, we need to stop the container, remove the image, rebuild the image and then run it again

```bash
docker stop fwk-front
docker rm fwk-front
docker image rm fwk-front
docker build -t fwk-front .
docker run --name fwk-front --network fwk-net --ip 172.20.0.3 -p 3001:5000 -d frontend
```

<hr>

## Inspect Your Network

```bash
docker inspect fwk-net
```

## Common Docker Commands

`docker build [OPTIONS] PATH | URL | -`
: Builds an image from a Dockerfile.

`docker run [OPTIONS] IMAGE [COMMAND] [ARG...]`
: Runs a command in a new container from a specified image.

`docker pull IMAGE`
: Downloads an image from a Docker registry.

`docker push IMAGE`
: Uploads an image to a Docker registry.

`docker ps [OPTIONS]`
: Lists all running containers.

`docker ps -a`
: Lists all containers, both running and stopped.

`docker images`
: Lists all available images on the local system.

`docker rmi IMAGE`
: Removes one or more images from the local system.

`docker stop CONTAINER`
: Stops a running container.

`docker start CONTAINER`
: Starts a stopped container.

`docker restart CONTAINER`
: Restarts a running or stopped container.

`docker exec [OPTIONS] CONTAINER COMMAND [ARG...]`
: Runs a command in a running container.

`docker rm CONTAINER`
: Removes one or more containers.

`docker logs [OPTIONS] CONTAINER`
: Fetches the logs of a container.

`docker inspect CONTAINER | IMAGE`
: Returns detailed information about a container or image.

`docker-compose up [OPTIONS]`
: Builds, (re)creates, and starts containers as defined in a `docker-compose.yml` file.

`docker-compose down`
: Stops and removes containers, networks, and volumes defined in a `docker-compose.yml` file.

`docker network ls`
: Lists all networks.

`docker network create NETWORK_NAME`
: Creates a new network.

`docker volume ls`
: Lists all volumes.

`docker volume create VOLUME_NAME`
: Creates a new volume.

<hr>

## Bonus

### Skapa MySQL Docker Container

```bash
docker run \
    --name test-mysql \
    -e MYSQL_ROOT_PASSWORD=root \
    -e MYSQL_USER=test \
    -e MYSQL_PASSWORD=test \
    -e MYSQL_DATABASE=test \
    -p 3306:3306 \
    --tmpfs /var/lib/mysql  \
    -d mysql/mysql-server:latest
```

### Skapa MongoDB Docker Container
```bash
docker run -d --name test-mongodb \
    -p 27017:27017 \
    -e MONGO_INITDB_ROOT_USERNAME=root \
    -e MONGO_INITDB_ROOT_PASSWORD=root \
    mongo
    
    docker logs test-mongodb --follow
```
