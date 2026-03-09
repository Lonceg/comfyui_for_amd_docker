<div align="center">

# ComfyUI ROCm Docker Image
</div>
This repository provides a Docker image designed to **simplify the process of running ComfyUI with AMD ROCm support** inside a container.  
It bundles all the necessary dependencies and configurations so you can get ComfyUI working on AMD GPUs quickly, without complicated local setups.
Tested on Linux Mint 22.2 Zara, should also work on Ubuntu 24.04 Noble.

---

## ROCm and Docker Installation (Prerequisites)

Before running the Docker image, ensure your system has the AMD ROCm stack and Docker installed.

### Installing ROCm
- Install kernel headers and development packages.
```sudo apt install python3-setuptools python3-wheel```
- Enter the following commands to install the installer script for Ubuntu 24.04/Mint 22.2
```
sudo apt update
wget https://repo.radeon.com/amdgpu-install/7.0/ubuntu/noble/amdgpu-install_7.0.70000-1_all.deb
sudo apt install ./amdgpu-install_7.0.70000-1_all.deb
```
- if host OS use
```
amdgpu-install -y --usecase=graphics,rocm
```
- if Workstation
```
amdgpu-install -y --usecase=workstation,rocm
```
- if WSL
```
amdgpu-install -y --usecase=wsl,rocm --no-dkms
```
- reboot

- change permission groups
```
sudo usermod -a -G render,video $LOGNAME
```  
- reboot

- Verify ROCm installation by running:  
```
dkms status
rocminfo
clinfo
```  

Original instruction at: https://rocm.docs.amd.com/projects/radeon/en/latest/docs/install/native_linux/install-radeon.html

- Install Docker engine:
```
sudo apt install docker.io
```

---
### Pulling the image

```
sudo docker pull lonceg/comfyui_for_amd:rocm7.0_pytorch2.8_py3.12
```
### EDIT
With new versions of ROCm I recommend to use the official ROCm docker image instead
```
sudo docker pull rocm/pytorch
```

This is the link to the docker image on Docker Hub https://hub.docker.com/r/lonceg/comfyui_for_amd.
Here is the original Docker image with PyTorch https://hub.docker.com/r/rocm/pytorch

---

### Clone ComfyUI on hostOS (skip if you use my image)
```
cd ~
git clone https://github.com/Comfy-Org/ComfyUI
```

Comment out those three and install requirements.txt
```
#torch
#torchvision
#torchaudio

pip install -r requirements.txt
```
Or use this:
```
grep -v -E '^(torch|torchvision|torchaudio)' requirements.txt | xargs pip install --no-deps
```

### Clone addons for ComfyUI (skip if you use my image)
```
cd ~
cd ComfyUI/custom_nodes
git clone https://github.com/city96/ComfyUI-GGUF
git clone https://github.com/rgthree/rgthree-comfy.git
git clone https://github.com/kijai/ComfyUI-KJNodes
git clone https://github.com/Kosinkadink/ComfyUI-VideoHelperSuite
git clone https://github.com/MrForExample/ComfyUI-3D-Pack.git

cd ComfyUI-GGUF
pip install -r requirements.txt
cd ..
cd ComfyUI-KJNodes
pip install -r requirements.txt
cd ..
cd ComfyUI-VideoHelperSuite
pip install -r requirements.txt
cd ..
cd ..
```

---

### Running container
Make sure to replace <b><u>username</b></u> and <b><u>image</b></u>

image examples:
  rocm/pytorch:rocm7.2_ubuntu24.04_py3.12_pytorch_release_2.8.0
  
  lonceg/comfyui_for_amd:rocm7.0_pytorch2.8_py3.12

```
sudo docker run -it \
  --name comfy-rocm \
  --cap-add=SYS_PTRACE \
  --security-opt seccomp=unconfined \
  --device=/dev/kfd \
  --device=/dev/dri \
  --group-add video \
  --ipc=host \
  --shm-size 8G \
  -p 8188:8188 \
  -v /home/<username>/ComfyUI:/workspace/ComfyUI \
  -w /workspace/ComfyUI \
  <image>
```

---

### Running container on WSL
Make sure to replace <b><u>username</b></u>
```
sudo docker run -it \
  --name comfy-rocm \
  --cap-add=SYS_PTRACE \
  --security-opt seccomp=unconfined \
  --ipc=host \
  --shm-size 8G \
  -p 8188:8188 \
  --device=/dev/dxg \
  -v /usr/lib/wsl/lib/libdxcore.so:/usr/lib/libdxcore.so \
  -v /opt/rocm/lib/libhsa-runtime64.so.1:/opt/rocm/lib/libhsa-runtime64.so.1 \
  -v /home/<username>/ComfyUI:/workspace/ComfyUI \
  -w /workspace/ComfyUI \
  <image>
```

With ```  -v /home/<username>/ComfyUI:/workspace/ComfyUI \``` line we map models folder inside the container to a volume ```/home/<username>/ComfyUI```. If you delete the container, make a new one, etc. all downloaded models and custom nodes will not disappear. The models should be placed within this volume just as in the original ComfyUI models folder here: https://github.com/comfyanonymous/ComfyUI/tree/master/models. You can also clone this repo and copy the folders as well

### Running and closing existing container
You can restart the container with 
```
sudo docker start comfy-rocm
sudo docker attach comfy-rocm
```
You can close the running comfyui with ctrl+c. To close the container you can type ```exit``` in the terminal.

---

## Acknowledgments

* This project builds on the work of the [ComfyUI](https://github.com/comfyanonymous/ComfyUI) team — many thanks for the amazing UI and framework!
* The [AMD ROCm team](https://github.com/RadeonOpenCompute/ROCm) for providing the ROCm ecosystem and Docker images that make GPU acceleration possible.
* The [official ROCm PyTorch Docker image](https://hub.docker.com/r/rocm/pytorch) which this image is based on.
* ComfyUI additional repos:
  [rgthree-comfy](https://github.com/rgthree/rgthree-comfy),
  [ComfyUI-KJNodes](https://github.com/kijai/ComfyUI-KJNodes),
  [ComfyUI-GGUF](https://github.com/city96/ComfyUI-GGUF),
  [ComfyUI-VideoHelperSuite](https://github.com/Kosinkadink/ComfyUI-VideoHelperSuite)
