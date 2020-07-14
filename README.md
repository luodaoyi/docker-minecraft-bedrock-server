[![Docker Pulls](https://img.shields.io/docker/pulls/luodaoyi/minecraft-bedrock-server.svg)](https://hub.docker.com/r/luodaoyi/minecraft-bedrock-server/)
[![GitHub Issues](https://img.shields.io/github/issues-raw/luodaoyi/docker-minecraft-bedrock-server.svg)](https://github.com/luodaoyi/docker-minecraft-bedrock-server/issues)
[![Build](https://github.com/luodaoyi/docker-minecraft-bedrock-server/workflows/Build/badge.svg)](https://github.com/luodaoyi/docker-minecraft-bedrock-server/actions?query=workflow%3ABuild)


## 为了方便中国朋友使用我改了下dockerfile 执行的时候不需要再下载，方便开服 请一定要阅读下面说明
>注意！ 请一定不要指定环境变量 VERSION=xxx 缺点是不能灵活切换版本，但是切换版本要下载 国内又下载不了 所以你直接指定docker images的tag就行了

>PS： 如果懒得折腾可以来我的服务器  mc.ptrace.cn:19132
>交流qq群： 1140238048

### 直接使用docker-compose 部署

```yaml
version: '3.8'
services:
  bds:
    image: luodaoyi/minecraft-bedrock-server
    environment:
      EULA: "TRUE"
      GAMEMODE: survival
      DIFFICULTY: easy
      SERVER_NAME: 'Evolution World'
      SERVER_PORT: 19132
      MAX_PLAYERS: 100
      MAX_THREADS: 8
      ALLOW_CHEATS: "FALSE"
      LEVEL_NAME: '20200714'
    ports:
      - 19132:19132/udp
    volumes:
      - bds:/data
    restart: always
    stdin_open: true
    tty: true

volumes:
  bds:
```

## Quickstart

The following starts a Bedrock Dedicated Server running a default version and
exposing the default UDP port: 

```bash
docker run -d -it -e EULA=TRUE -p 19132:19132/udp itzg/minecraft-bedrock-server
```

## Looking for a Java Edition Server

For Minecraft Java Edition you'll need to use this image instead:

[itzg/minecraft-server](https://hub.docker.com/r/itzg/minecraft-server)

## Environment Variables

### Container Specific

- `EULA` (no default) : must be set to `TRUE` to 
  accept the [Minecraft End User License Agreement](https://minecraft.net/terms)
- `VERSION` (`LATEST`) : can be set to a specific server version or the following special values can be used:
  - `LATEST` : determines the latest version and can be used to auto-upgrade on container start
  - `PREVIOUS` : uses the previously maintained major version. Useful when the mobile app is gradually being upgraded across devices
  - `1.11` : the latest version of 1.11
  - `1.12` : the latest version of 1.12
  - `1.13` : the latest version of 1.13
  - `1.14` : the latest version of 1.14
  - `1.16` : the latest version of 1.16
  - otherwise any specific server version can be provided to allow for temporary bug avoidance, etc
- `UID` (default derived from `/data` owner) : can be set to a specific user ID to run the
  bedrock server process
- `GID` (default derived from `/data` owner) : can be set to a specific group ID to run the
  bedrock server process

### Server Properties

The following environment variables will set the equivalent property in `server.properties`, where each [is described here](https://minecraft.gamepedia.com/Server.properties#Bedrock_Edition_3).

- `SERVER_NAME`
- `SERVER_PORT`
- `GAMEMODE`
- `DIFFICULTY`
- `LEVEL_TYPE`
- `ALLOW_CHEATS`
- `MAX_PLAYERS`
- `ONLINE_MODE`
- `WHITE_LIST`
- `VIEW_DISTANCE`
- `TICK_DISTANCE`
- `PLAYER_IDLE_TIMEOUT`
- `MAX_THREADS`
- `LEVEL_NAME`
- `LEVEL_SEED`
- `DEFAULT_PLAYER_PERMISSION_LEVEL`
- `TEXTUREPACK_REQUIRED`

For example, to configure a flat, creative server instead of the default use:

```bash
docker run -d -it --name bds-flat-creative \
  -e EULA=TRUE -e LEVEL_TYPE=flat -e GAMEMODE=creative \
  -p 19132:19132/udp itzg/minecraft-bedrock-server
```

## Exposed Ports

- **UDP** 19132 : the Bedrock server port. 
  **NOTE** that you must append `/udp` when exposing the port, such as `-p 19132:19132/udp`
  
## Volumes

- `/data` : the location where the downloaded server is expanded and ran. Also contains the
  configuration properties file `server.properties`

You can create a `named volume` and use it as:

```shell
docker volume create mc-volume
docker run -d -it --name mc-server -e EULA=TRUE -p 19132:19132/udp -v mc-volume:/data itzg/minecraft-bedrock-server
```

## Connecting

When running the container on your LAN, you can find and connect to the dedicated server
in the "LAN Games" part of the "Friends" tab, such as:

![](docs/example-client.jpg)

## More information

For more information about managing Bedrock Dedicated Servers in general, [check out this Reddit post](https://old.reddit.com/user/ProfessorValko/comments/9f438p/bedrock_dedicated_server_tutorial/).

## Executing server commands

Assuming you started container with stdin and tty enabled (such as using `-it`), you can attach to the container's console by its name or ID using:

```shell script
docker attach CONTAINER_NAME_OR_ID
``` 

While attached, you can execute any server-side commands, such as op'ing your player to be admin:

```
op YOUR_XBOX_USERNAME
```

When finished, detach from the server console using Ctrl-p, Ctrl-q

## Deploying with Docker Compose

The [examples](examples) directory contains [an example Docker compose file](examples/docker-compose.yml) that declares:
- a service running the bedrock server container and exposing UDP port 19132
- a volume to be attached to the service

The service configuration includes some examples of configuring the server properties via environment variables:
```yaml
environment:
  EULA: "TRUE"
  GAMEMODE: survival
  DIFFICULTY: normal
```

From with in the `examples` directory, you can deploy the composition by using:

```bash
docker-compose up -d
```

You can follow the logs using:
```bash
docker-compose logs -f bds
```

## Deploying with Kubernetes

The [examples](examples) directory contains [an example Kubernetes manifest file](examples/kubernetes.yml) that declares:
- a peristent volume claim (using default storage class)
- a pod deployment that uses the declared PVC
- a service of type LoadBalancer

The pod deployment includes some examples of configuring the server properties via environment variables:
```yaml
env:
- name: EULA
  value: "TRUE"
- name: GAMEMODE
  value: survival
- name: DIFFICULTY
  value: normal
```

The file is deploy-able as-is on most clusters, but has been confirmed on [Docker for Desktop](https://docs.docker.com/docker-for-windows/kubernetes/) and [Google Kubernetes Engine](https://cloud.google.com/kubernetes-engine/docs/):

```bash
kubectl apply -f examples/kubernetes.yml
```

You can follow the logs of the deployment using:

```bash
kubectl logs -f deployment/bds
```
