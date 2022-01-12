# Drone CI Fast Setup
## Version
server=1.9.1
runner-docker:1.5.2

## Prerequisites
### Github Oauth Setup
https://docs.drone.io/server/provider/github/

## Setup
## Way1: Via docker run
```
docker pull drone/drone:1.9.1
docker pull drone/drone-runner-docker:1.5.2
```
### Run drone server
```
docker run \
  --volume=/var/lib/drone:/data \
  --env=DRONE_GITHUB_CLIENT_ID={{DRONE_GITHUB_CLIENT_ID}} \
  --env=DRONE_GITHUB_CLIENT_SECRET={{DRONE_GITHUB_CLIENT_SECRET}} \
  --env=DRONE_RPC_SECRET={{DRONE_RPC_SECRET}} \
  --env=DRONE_SERVER_HOST={{DRONE_SERVER_HOST}} \
  --env=DRONE_SERVER_PROTO={{DRONE_SERVER_PROTO}} \
  --publish=80:80 \
  --publish=443:443 \
  --restart=always \
  --detach=true \
  --name=drone \
  drone/drone:1.9.1
```

### Run runner
```
docker run -d \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -e DRONE_RPC_PROTO=https \
  -e DRONE_RPC_HOST=drone.company.com \
  -e DRONE_RPC_SECRET=super-duper-secret \
  -e DRONE_RUNNER_CAPACITY=2 \
  -e DRONE_RUNNER_NAME=${HOSTNAME} \
  -p 3000:3000 \
  --restart always \
  --name runner \
  drone/drone-runner-docker:1.5.2
```
## Way2: Via docker-compose and raw env
1. `docker-compose.yml`
```
version: "3"
services:
  server:
    image: drone/drone:1.9.1
    ports:
      - 80:80
      - 443:443
    environment:
      DRONE_GITHUB_CLIENT_ID: 12312312312
      DRONE_GITHUB_CLIENT_SECRET: asdfjasdkfjas;dkfj
      DRONE_RPC_SECRET: hohohoitssecret
      DRONE_SERVER_HOST: my_raw_host_url
      DRONE_SERVER_PROTO: http
    restart: always
    volumes:
      - /private/var/lib/drone:/data
  docker_runner:
    image: drone/drone-runner-docker:1.5.2
    environment:
      DRONE_RPC_SECRET: hohohoitssecret
      DRONE_RPC_HOST: my_raw_host_url
      DRONE_RPC_PROTO: http
      DRONE_RUNNER_CAPACITY: 2
      DRONE_RUNNER_NAME: ${HOSTNAME}
    ports:
      - "3000:3000"
    restart: always
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
```
2. `docker-compose up`
## Way3: Via docker-compose and template env
1. have `.env` in local
2. `docker-compose.yml`
```
version: "3"
services:
  server:
    image: drone/drone:1.9.1
    ports:
      - 80:80
      - 443:443
    environment:
      DRONE_GITHUB_CLIENT_ID: ${DRONE_GITHUB_CLIENT_ID}
      DRONE_GITHUB_CLIENT_SECRET: ${DRONE_GITHUB_CLIENT_SECRET}
      DRONE_RPC_SECRET: ${DRONE_RPC_SECRET}
      DRONE_SERVER_HOST: ${DRONE_SERVER_HOST}
      DRONE_SERVER_PROTO: ${DRONE_SERVER_PROTO}
    restart: always
    volumes:
      - /private/var/lib/drone:/data
  docker_runner:
    image: drone/drone-runner-docker:1.5.2
    environment:
      DRONE_RPC_SECRET: ${DRONE_RPC_SECRET}
      DRONE_RPC_HOST: ${DRONE_RPC_HOST}
      DRONE_RPC_PROTO: ${DRONE_RPC_PROTO}
      DRONE_RUNNER_CAPACITY: ${DRONE_RUNNER_CAPACITY}
      DRONE_RUNNER_NAME: ${HOSTNAME}
    ports:
      - "3000:3000"
    restart: always
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
```
2. `export $(cat .env | sed 's/#.*//g' | xargs) && docker-compose up`

Then you can navigate to my_raw_host_url and setup repos
### References
[EnvVarsInYml](https://stackoverflow.com/questions/29377853/how-to-use-environment-variables-in-docker-compose)

[Load Dotenv When Running bash](https://gist.github.com/mihow/9c7f559807069a03e302605691f85572)
