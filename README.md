# edu-docker

## Skapa MySQL Docker Container

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

## Skapa MongoDB Docker Container
```bash
docker run -d --name test-mongodb \
    -p 27017:27017 \
    -e MONGO_INITDB_ROOT_USERNAME=root \
    -e MONGO_INITDB_ROOT_PASSWORD=root \
    mongo
    
    docker logs test-mongodb --follow
```

## Skapa NodeJS Dokcer Container

```docker
FROM node:12-alpine

RUN mkdir -p /home/node/app/node_modules && chown -R node:node /home/node/app

WORKDIR /home/node/app

COPY package*.json ./

USER node

RUN npm install

COPY --chown=node:node . .

EXPOSE 3000

CMD [ "node", "app.js" ]
```
