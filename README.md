# NVIDIA 755M SLI on Ubuntu 22.04

CUDA Version: 11.4

## Update Ubuntu

```bash
sudo apt update
sudo apt upgrade -y
```

## Remove existing NVIDIA driver

except the package nvidia-common all other packages should be purged.

```bash
dpkg -l | grep -i nvidia

sudo apt-get remove --purge '^nvidia-.*'
sudo apt-get install ubuntu-desktop
sudo rm /etc/X11/xorg.conf
echo 'nouveau' | sudo tee -a /etc/modules
```

## Install NVIDIA drivers

Latest drivers that support NVIDIA 755M are 418.113. You can install them by running:

```bash
sudo apt install build-essential libglvnd-dev pkg-config
```

Run installer (you can download it from <https://www.nvidia.com/Download/index.aspx>):

```bash
  chmod +x NVIDIA-x86_64-418.113.run
  sudo telinit 3
  sudo ./NVIDIA-x86_64-418.113.run
```

```bash
  sudo reboot
```

```bash
sudo apt-get install nvidia-driver-470
```

```bash
  sudo reboot
```

### More commands that might be useful

nvidia.NVreg_PreserveVideoMemoryAllocations=1

If you dont have this file, just create.

```bash
sudo vim /etc/udev/rules.d/10-remove-nvidia-audio.rules
```

then add this line

```
ACTION=="add", KERNEL=="0000:01:00.1", SUBSYSTEM=="pci", RUN+="/bin/sh -c 'echo 1 > /sys/bus/pci/devices/0000:01:00.1/remove'"
```

type `:wq` then press enter to save and exit.

### Install drivers for Docker support

NVIDIA drivers sadly do not support docker desktop. So you need to use docker runtime and manage containers via terminal.

Check if docker runtime context is set to default:

```bash
docker context ls
```

Set default context as current:

```bash
docker context use default
```

check if nvidia drivers are enabled:

```bash
udo docker run --rm --runtime=nvidia --gpus all ubuntu nvidia-smi
```

if you see similar error like this:

```
docker: Error response from daemon: could not select device driver "" with capabilities: [[gpu]].
```

then you need to install nvidia-container-toolkit by configure prod repo:

```bash
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg \
  && curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | \
    sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
    sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list
```

update package repository:

```bash
sudo apt-get update
```

install nvidia-container-toolkit:

```bash
sudo apt-get install -y nvidia-container-toolkit
```

Configure docker daemon to use nvidia runtime:

```bash
sudo nvidia-ctk runtime configure --runtime=docker
```

Restart docker daemon:

```bash
sudo systemctl restart docker
```

To configure the container runtime for Docker running in Rootless mode, follow these steps:

1. Configure the container runtime by using the nvidia-ctk command:

```bash
nvidia-ctk runtime configure --runtime=docker --config=$HOME/.config/docker/daemon.json
```

2. Restart the Rootless Docker daemon:

```bash
systemctl --user restart docker
```

3. Configure /etc/nvidia-container-runtime/config.toml by using the sudo nvidia-ctk command:

```bash
sudo nvidia-ctk config --set nvidia-container-cli.no-cgroups --in-place
```

### Install nvidia cuda toolkit

```bash
sudo apt install nvidia-cuda-toolkit
sudo reboot
```

sudo nvidia-smi -i 0 -mig 0
sudo nvidia-smi -i 0 --query-gpu=pci.bus_id,mig.mode.current --format=csv
sudo reboot

sudo nvidia-xconfig --enable-all-gpus

download cuda installer
<https://developer.nvidia.com/cuda-downloads?target_os=Linux&target_arch=x86_64&Distribution=Ubuntu&target_version=22.04>

```bash
wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2204/x86_64/cuda-ubuntu2204.pin
sudo mv cuda-ubuntu2204.pin /etc/apt/preferences.d/cuda-repository-pin-600
wget https://developer.download.nvidia.com/compute/cuda/12.5.0/local_installers/cuda-repo-ubuntu2204-12-5-local_12.5.0-555.42.02-1_amd64.deb
sudo dpkg -i cuda-repo-ubuntu2204-12-5-local_12.5.0-555.42.02-1_amd64.deb
sudo cp /var/cuda-repo-ubuntu2204-12-5-local/cuda-*-keyring.gpg /usr/share/keyrings/
sudo apt-get update
sudo apt-get -y install cuda-toolkit-12-5
```

NVIDIA Driver Instructions (choose one option)
To install the legacy kernel module flavor:

```bash
sudo apt-get install -y cuda-drivers
```

To install the open kernel module flavor:

```bash
sudo apt-get install -y nvidia-driver-555-open
sudo apt-get install -y cuda-drivers-555
```

To switch between NVIDIA Driver kernel module flavors see here.
nvcc --version

info:
<https://www.linkedin.com/pulse/how-installuninstallmanage-nvidia-driver-cuda-ubuntu-2004-sutradhar-x4xwf/>
