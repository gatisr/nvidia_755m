# Local server tools

## Docker desktop won`t start temporary fix

```bash
sudo sysctl -w kernel.apparmor_restrict_unprivileged_userns=0
systemctl --user restart docker-desktop
```

## Use correct context

Docker desktop and docker should use the same context. In docker desktop you can change it in settings.

Check available contexts by running `docker context ls`.

In docker you can change it by running:

```bash
docker context use desktop-linux
```

you should restart terminal in order to work properly with new contexts.

## Force close hidden docker apps by port

Find processes which are running on specified ports:

```bash
sudo lsof -i :3000 | grep LISTEN
```

Kill the process by PID (where <pid_1> and <pid_2> are the process IDs found in the previous step)

```bash
sudo kill <pid_1> <pid_2>
```

### fix permissions

```bash
sudo usermod -aG docker $USER

sudo chgrp docker /home/lenovo/.docker/desktop/docker.sock
sudo chmod g+w /home/lenovo/.docker/desktop/docker.sock
```
