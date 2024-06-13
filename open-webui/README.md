# Open WebUI

## Set default context for docker

Docker should use default context in order to use nvidia drivers:

```bash
docker context use default
```

## Run Open WebUI

```bash
docker compose -p open-webui up -d
```

## Manage docker image with docker engine

In order to manage image you should use terminal (docker engine instead of desktop).

### Show running containers

```bash
docker ps
```

### Show running open-webui-open-webui-1 logs

```bash
docker logs open-webui-open-webui-1
```

### Start stopped container

```bash
docker start open-webui-open-webui-1
```

### Stop running container

```bash
docker stop open-webui-open-webui-1
```

### Remove container

```bash
docker rm -f open-webui-open-webui-1
```

### Attach container logging

In order to detach the container logging you should press `Ctrl + P` and `Ctrl + Q`.

```bash
  docker attach open-webui-open-webui-1
```

## Turn nvidia drivers on

Check if drivers is running or not with:

```bash
nvidia-smi
```

Turn on these drivers if they appear to be turned off:

```bash
sudo nvidia-smi -pm 1
```
