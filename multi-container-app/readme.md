## Multi-Container App

This is a realistic-looking app which allows user to create, read and delete goals. It consists of three images

- mongo
- nodejs API
- react front-end

Node code is running on server. React code is running on browser! (outside of container) Cannot use container name in react code. So still need to expose node port so browser can reach it.

Features:

- mongo data must survive container shutdown
- source code changes are instantly taking effect without rebuilding image/restarting container
- restrict access to mongo

```bash
cd ./multi-container-app

docker network create goals-network

docker run --name mongodb --rm -d --network goals-network mongo

cd ./backend
docker build -t goals-backend .

# node needs network to reach mongo
# also needs to expose port to browser app
docker run \
  --name goals-backend \
  --rm -d \
  --network goals-network \
  -p 8080:8080 \
  goals-backend

# define my own env var, use in app.js
docker run \
  --name goals-backend \
  --rm -d \
  --network goals-network \
  -e MONGODB_USERNAME=shaw \
  -e MONGODB_PASSWORD=secret \
  -p 8080:8080 \
  goals-backend

# the react app must be run in interactive mode, or it will immediately stop
cd ./frontend
docker build -t goals-react .

# don't need network, but still need to expose port
docker run --name goals-frontend --rm -d -p 3000:3000 -it goals-react
```

### How to Persist Data across Container Removal

1. using named volume: `-v data:/data/db`
2. using bind mount: `-v $LOCAL_PATH:/data/db`

```bash
# using named volume
docker run --name mongodb --rm -d --network goals-network -v data:/data/db mongo
```

### Set Mongodb Credential

- Set db username and password when starting container
- nodeJS must use username and password to connect to mongo
- when setting initial user, must create a brand new named volume, or connection will fail
  - must completely remove volume before setting user

```bash
docker run \
  --name mongodb \
  --rm -d --network goals-network \
  -v db:/data/db \
  -e MONGO_INITDB_ROOT_USERNAME=shaw \
  -e MONGO_INITDB_ROOT_PASSWORD=secret \
  mongo
```

### Persist Node App Log

- using named volume `-v logs:/app/logs`
- using bind mount to map host machine code to container
- logs folder is more specific (`/app/logs`) so it won't be overwritten by bind mount
- use anonymous volume to avoid overwriting node_module

```bash
docker run \
  --name goals-backend \
  -v /Users/shaw.lu/Documents/proj/docker-basics/multi-container-app/backend:/app \
  -v logs:/app/logs \
  -v /app/node_modules \
  --rm -d \
  --network goals-network \
  -p 8080:8080 \
  goals-backend
```

### Front-end Live Update

```bash
docker run \
  -v /Users/shaw.lu/Documents/proj/docker-basics/multi-container-app/frontend/src:/app/src \
  --name goals-frontend \
  --rm -d \
  -p 3000:3000 \
  -it \
  goals-react
```

---

## Docker Compose

Write project setup in [docker-compose.yaml](docker-compose.yaml). See official [doc](https://docs.docker.com/compose/compose-file/).

- by default, containers are detatched
- by default services will be removed when shutdown
- by default, all containers started up by the same compose file are placed in the same network
  - can add `networks` section too to specify name
  - can use container name specified in services to reference container in code
- need to declare named volume in `volumes` (top-level section)
  - don't need to list anonymous volume and bind mount
- can use reletive path for bind mount
- use `depends_on` section to specify order of starting up containers

```bash
docker-compose --version
docker-compose up
docker-compose up -d # detatched mode
docker-compose up --build # rebuild no matter what

# build images without starting containers
docker-compose build

# delete all containers and network
docker-compose down

# delete volumes too
docker-compose down -v
```
