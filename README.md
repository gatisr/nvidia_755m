# NVIDIA 755M SLI on Ubuntu 22.04

CUDA Version: 11.4

SOLUTION
Solved it thanks to reading the link provided by u/david279
As they've figured out there exists no kernel modules of 5.17.5 for 470
to keep using 470 one needs to revert back to old kernel

Check the current kernel by uname -r

Then switch to 5.16.19 with: sudo kernelstub -v -k /boot/vmlinuz-5.16.19-76051619-generic -i /boot/initrd.img-5.16.19-76051619-generic

The weird problem I had is that even though I was on 5.16.19 and confirmed it with uname -r, IF the 5.17.5 kernel wasn't removed during the sudo apt install nvidia-driver-470 process the kernel would revert (I don't know a better term for it) back to 5.17.5.

So if you want to use 470:

Change kernel to an old one

Remove new kernel 5.17.5

Install nvidia driver and be happy

I sincerely hope that I haven't messed up with the system while downgrading and uninstalling kernels. If anyone else have the same problem this would be it.

## Update Ubuntu

```bash
sudo apt update
sudo apt upgrade -y
```

## Downgrade kernel to 5.8

```bash
wget http://archive.ubuntu.com/ubuntu/pool/main/o/openssl/libssl1.1_1.1.0g-2ubuntu4_amd64.deb
sudo dpkg -i libssl1.1_1.1.0g-2ubuntu4_amd64.deb
```

download patched driver for 5.8 kernel

```bash
sudo apt-get update
sudo apt-get install git-lfs
git lfs install

git clone --depth 1 --filter=blob:none --sparse https://github.com/MeowIce/nvidia-legacy.git
cd nvidia-legacy
git sparse-checkout init --cone
git sparse-checkout set 5.8

git lfs pull --include "5.8/NVIDIA-Linux-x86_64-418.113-kernel-5.8.run"
ls 5.8/NVIDIA-Linux-x86_64-418.113-kernel-5.8.run

```

## Remove existing NVIDIA driver

except the package nvidia-common all other packages should be purged.

```bash
dpkg -l | grep -i nvidia

sudo apt-get remove --purge '^nvidia-.*'
sudo apt-get --purge remove "*cublas*" "cuda*" "nsight*" 
sudo rm -rf /usr/local/cuda*

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
sudo docker run --rm --runtime=nvidia --gpus all ubuntu nvidia-smi
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

sudo nvidia-smi -i 0 -mig 0
sudo nvidia-smi -i 0 --query-gpu=pci.bus_id,mig.mode.current --format=csv
sudo reboot

sudo nvidia-xconfig --enable-all-gpus
```

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

sudo prime-select nvidia
