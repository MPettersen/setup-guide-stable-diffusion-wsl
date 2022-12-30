# Set up local Stable Diffusion locally in WSL on Windows

## Overview

## 1. Install latest NVIDIA drivers (Windows)

Install latest NVIDIA drivers from [here](https://www.nvidia.com/download/index.aspx).
Run installer and follow instructions.

## 2. Setup WSL environment

### 1.1 Install WSL from powershell
```powershell
wsl --install
```

### 1.2 Install linux distribution
From Microsoft Store download and install Ubuntu, [here](https://www.microsoft.com/store/productId/9PDXGNCFSCZV).
After the installation, open the Ubuntu app (console) from the Windows start menu.
On first startup you will be prompted to set username and password.
Save your login credentials somewhere, if you forget you have to start over again.

Close the console window.

### 1.3 Make sure it runs with WSL version 2 (In powershell)
Check version
```powershell
wsl -l -v
```

If version 1, change to wsl version 2:
```powershell
wsl --set-version Ubuntu 2
```

## 2. Setup Ubuntu environment

Open the Ubuntu console again.

### 2.1 Update packages (In Ubuntu)
```
sudo apt-get update
```

### 2.2 Install git (In Ubuntu)
```powershell
sudo apt-get install git-all
```

### 2.3 Install miniconda with python 3.8 (In Ubuntu)
1. Download installer
    ```powershell
    curl -sL "https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh" > "Miniconda3.sh"
    ```
1. Run installer
    ```powershell
    bash Miniconda3.sh
    ```
    - Answer `y` or `yes` whenever promted
    - Review license
    - Confirm folder location
1. Remove installer
    ```powershell
    rm -r Miniconda3.sh
    ```
1. Restart console
    - You know that conda is active and running if you see `(base)` at the beginning of the of line in the console when you write commands.

### 2.4 Update conda (In Ubuntu)
```powershell
conda update conda
conda install wget
```

If you get som sort of http error, then exit Ubuntu go into powershell and execute the following command.
```powershell
wsl --shutdown
```
Then open Ubuntu again and run the conda update commands.


## 3. Check systemd (Ubuntu)

Check which init system is being used
```powershell
ps -p 1 -o comm=
```

If it says `init` you're kinda screwed.
Use the table from this [site](https://linuxhandbook.com/system-has-not-been-booted-with-systemd/) to translate between systemd commands to init commands.

Try running these commands to install systemd

Check for systemd
```powershell
sudo dpkg -l | grep systemd
```

Install systemd
```powershell
sudo apt-get install systemd
```

Reinstall systemd
```powershell
sudo apt-get install --reinstall systemd
```

### 3.1 Enable Systemd in Ubuntu
https://askubuntu.com/questions/1379425/system-has-not-been-booted-with-systemd-as-init-system-pid-1-cant-operate

```powershell
sudo -e /etc/wsl.conf
```

Copy paste this into the .conf file:
```
[boot]
systemd=true
```
Press `CTRL+X` and confirm with `y` then `ENTER` to save and exit.

Exit Ubuntu and shutdown wsl from the powershell terminal:
```powershell
wsl --shutdown
```

Restart the Ubuntu app, just open the app from the search bar.

Verify that Systemd is working:
```powershell
sudo systemctl status
```

If that doesn't work install this bullshit:
https://www.catalog.update.microsoft.com/Search.aspx?q=KB5020030

Then rerun the previous steps until the command `ps -p 1 -o comm=` returns `systemd` and not `init`.
Good luck, you'll need it.

## 3. Setup CUDA
https://learn.microsoft.com/en-us/windows/ai/directml/gpu-cuda-in-wsl

### 3.1 Install CUDA toolkit on Windows
Download installer from [here](https://developer.nvidia.com/cuda-downloads?target_os=Windows&target_arch=x86_64&target_version=11&target_type=exe_network).

### 3.2 Setup CUDA for WSL (In Ubuntu)
https://docs.nvidia.com/cuda/wsl-user-guide/index.html

```powershell
sudo apt-key del 7fa2af80
```

```powershell
wget https://developer.download.nvidia.com/compute/cuda/repos/wsl-ubuntu/x86_64/cuda-wsl-ubuntu.pin
sudo mv cuda-wsl-ubuntu.pin /etc/apt/preferences.d/cuda-repository-pin-600
wget https://developer.download.nvidia.com/compute/cuda/12.0.0/local_installers/cuda-repo-wsl-ubuntu-12-0-local_12.0.0-1_amd64.deb
sudo dpkg -i cuda-repo-wsl-ubuntu-12-0-local_12.0.0-1_amd64.deb
sudo cp /var/cuda-repo-wsl-ubuntu-12-0-local/cuda-*-keyring.gpg /usr/share/keyrings/
sudo apt-get update
sudo apt-get -y install cuda
```
### 3.2 Install NVIDIA container toolkit (In Ubuntu)
https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html#docker

```powershell
curl https://get.docker.com | sh && sudo systemctl --now enable docker
```

Setting up NVIDIA Container Toolkit  
Setup the package repository and the GPG key:
```powershell
distribution=$(. /etc/os-release;echo $ID$VERSION_ID) \
    && curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg \
    && curl -s -L https://nvidia.github.io/libnvidia-container/$distribution/libnvidia-container.list | \
    sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
    sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list
```

Install the nvidia-docker2 package (and dependencies) after updating the package listing:
```powershell
sudo apt-get update
sudo apt-get install -y nvidia-docker2
```

Restart the Docker daemon to complete the installation after setting the default runtime:
```powershell
sudo systemctl restart docker
```

At this point, a working setup can be tested by running a base CUDA container:
```powershell
sudo docker run --rm --gpus all nvidia/cuda:11.6.2-base-ubuntu20.04 nvidia-smi
```

Expected output
```
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 525.65       Driver Version: 527.56       CUDA Version: 12.0     |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|                               |                      |               MIG M. |
|===============================+======================+======================|
|   0  NVIDIA GeForce ...  On   | 00000000:F3:00.0 Off |                  N/A |
| N/A   34C    P8     4W /  33W |      0MiB /  4096MiB |      0%      Default |
|                               |                      |                  N/A |
+-------------------------------+----------------------+----------------------+

+-----------------------------------------------------------------------------+
| Processes:                                                                  |
|  GPU   GI   CI        PID   Type   Process name                  GPU Memory |
|        ID   ID                                                   Usage      |
|=============================================================================|
|    0   N/A  N/A        33      G   /Xwayland                       N/A      |
+-----------------------------------------------------------------------------+
```

## 4. Stable Diffusion setup
https://www.howtogeek.com/830179/how-to-run-stable-diffusion-on-your-pc-to-generate-ai-images/
https://github.com/Stability-AI/stablediffusion

### 4.1 Clone Stable Diffusion repo

```powershell
mkdir stable-diffusion
cd stable-diffusion
git clone https://github.com/Stability-AI/stablediffusion.git
cd stablediffusion
```

### 4.3 Add checkpoint to model

#### 4.3.1 Download checkpoint
Download the checkpoint from [here](https://huggingface.co/stabilityai/stable-diffusion-2/blob/main/768-v-ema.ckpt).
You must be logged in to download the file.

If you have windows 11 you can easily copy the from C-drive into the Ubuntu mount storage.

If you are using an older OS, then you are on your own.

#### 4.3.2 Copy ckpt into repo folder

Create folder
```powershell
mkdir models && cd models
mkdir ldm && cd ldm
mkdir stable-diffusion-v2 && cd stable-diffusion-v2
```
Copy the ckpt into the created folder.
For ease of use you can rename ckpt file to `model.ckpt`.

### 4.4 Setup conda environment

```powershell
cd ~/stable-diffusion/stablediffusion
conda env create -f environment.yaml
conda activate ldm
```

### 4.5 Run test script

```powershell
python scripts/txt2img.py --prompt "a close-up portrait of a cat by pablo picasso, vivid, abstract art, colorful, vibrant" --plms --n_iter 5 --n_samples 1 --ckpt models/ldm/stable-diffusion-v2/model.ckpt

python scripts/txt2img.py --prompt "a professional photograph of an astronaut riding a horse" --ckpt models/ldm/stable-diffusion-v2/model.ckpt --config configs/stable-diffusion/v2-inference-v.yaml --H 768 --W 768  
```
