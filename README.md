# NVIDIA 755M SLI on Ubuntu 22.04 with docker support

## Requirements

This guide is for NVIDIA 755M SLI with latest available drivers and software at the time of writing.

1. Ubuntu 22.04 (UEFI boot mode)
2. Kernel 5.2
3. GCC 9.X
4. G++ 9.X
5. CUDA 11.4
6. NVIDIA Driver 470
7. Docker
8. NVIDIA Container Toolkit

## Downgrade kernel to 5.2

1. Check current kernel version:

    ```bash
    uname -r
    ```

2. Install mainline kernels to manage kernel versions:

    ```bash
    sudo apt install mainline
    ```

3. Open mainline kernel manager in UI, mark to show old kernels in settings, then install kernel 5.2.

4. Reboot and select kernel 5.2 from grub menu (Advanced options for Ubuntu). _Note: If you don't see kernel 5.2 in the list, you might try to reboot and select it again._

__Note:__ You can now remove previous kernels to free up space and avoid confusion.

## Install GCC and G++

1. Install build-essential package:

    ```bash
    sudo apt install build-essential
    ```

2. Check installed versions:

    ```bash
    sudo update-alternatives --config gcc
    ```

3. Install GCC 9 and G++ 9:

    ```bash
    sudo apt install g++-9
    sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-9 9
    ```

4. Make sure you have the correct version:

    ```bash
    sudo update-alternatives --config gcc
    ```

## Install NVIDIA Driver 470

1. Check if you have NVIDIA already installed and remove it:

    ```bash
      dpkg -l | grep -i nvidia

      sudo apt-get remove --purge '^nvidia-.*'
      sudo apt-get --purge remove "*cublas*" "cuda*" "nsight*" 
      sudo rm -rf /usr/local/cuda*
    ```

2. Reboot `sudo reboot`

3. Install NVIDIA driver 470:

    ```bash
    sudo apt update
    sudo apt upgrade -y
    sudo apt-get install nvidia-driver-470 -y
    ```

4. Reboot `sudo reboot`

__Note:__ If command `nvidia-smi` doesn't work, you might need to make theses changes:

1. Create a file `/etc/udev/rules.d/10-remove-nvidia-audio.rules`:

    ```bash
    sudo vim /etc/udev/rules.d/10-remove-nvidia-audio.rules
    ```

2. Add this line:

    ```
    # This file prevents the nvidia audio drivers from being loaded
    SUBSYSTEM=="sound", ATTRS{id="nvidia"}, RUN+="/bin/sh -c 'echo 1 > /sys$env{DEVPATH}/remove'"
    ```

3. Reboot `sudo reboot`

## Install CUDA 11.4

1. Go to cuda toolkit archive (<https://developer.nvidia.com/cuda-toolkit-archive>), select [version 11.4]([https://developer.nvidia.com/cuda-11-4-0-download-archive) and download the runfile for Ubuntu 20.04 and run it like instructed in the official documentation e.g.:

    ```bash
    wget https://developer.download.nvidia.com/compute/cuda/11.4.0/local_installers/cuda_11.4.0_470.42.01_linux.run
    sudo sh cuda_11.4.0_470.42.01_linux.run
    ```

2. Add the following lines to your `.bashrc`:

    ```bash
    export PATH=/usr/local/cuda-11.4/bin${PATH:+:${PATH}}
    export LD_LIBRARY_PATH=/usr/local/cuda-11.4/lib64${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}
    ```

    You can do this by running:

    ```bash
    vim ~/.bashrc
    ```

    and adding the lines at the end of the file. Exit by pressing `Esc` and typing `:wq` then press `Enter`.

3. Reboot `sudo reboot`

4. Test the installation:

    ```bash
    nvcc --version
    nvidia-smi
    ```

## Install NVIDIA Container Toolkit for Docker

1. install nvidia-container-toolkit:

    ```bash
    sudo apt-get install -y nvidia-container-toolkit
    ```

2. Configure docker daemon to use nvidia runtime:

    ```bash
    sudo nvidia-ctk runtime configure --runtime=docker
    ```

3. Restart docker daemon:

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
    sudo systemctl restart docker
    ```

3. Configure /etc/nvidia-container-runtime/config.toml by using the sudo nvidia-ctk command:

    ```bash
    sudo nvidia-ctk config --set nvidia-container-cli.no-cgroups --in-place
    ```

sudo nvidia-ctk config --set nvidia-container-cli.no-cgroups --in-place
