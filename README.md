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
amdgpu-install -y --usecase=workstation,rocmhttps://hub.docker.com/r/lonceg/comfyui_for_amd
```
- if WSL
```
amdgpu-install -y --usecase=rocm
```
- reboot

- change permission groups
```
sudo usermod -a -G render,video $LOGNAME
```  
- Verify ROCm installation by running:  
```
/opt/rocm/bin/rocminfo
/opt/rocm/opencl/bin/clinfo
```
- Install Docker engine:
```
sudo apt install docker.io
```

---

### Pulling the image
```
sudo docker pull lonceg/comfyui_for_amd:rocm7.0_pytorch2.8_py3.12
```

This is the link to the docker image on Docker Hub https://hub.docker.com/r/lonceg/comfyui_for_amd
Here is the original Docker image with PyTorch https://hub.docker.com/r/rocm/pytorch

---

### Running container
Make sure to replace <b><u>username</b></u>

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
  -v /home/<username>/comfyui_models:/workspace/ComfyUI/models \
  lonceg/comfyui_for_amd:rocm7.0_pytorch2.8_py3.12
```

With ```  -v /home/<username>/comfyui_models:/workspace/ComfyUI/models \``` line we map models folder inside the container to a volume ```/home/<username>/comfyui_models```. If you delete the container, make a new one, etc. these models will not disappear. The models should be placed within this volume just as in the original ComfyUI models folder here: https://github.com/comfyanonymous/ComfyUI/tree/master/models. You can also clone this repo and copy the folders as well

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

---

