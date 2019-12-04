# Nvidia-Docker 安装

## Docker 版本 >= 19.03，参考[这里](https://github.com/NVIDIA/nvidia-docker)
### 1. 安装
```
    distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
    curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo apt-key add -
    curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | sudo tee /etc/apt/sources.list.d/nvidia-docker.list
    sudo apt-get update
    sudo apt-get install -y nvidia-container-toolkit
    sudo systemctl restart docker
```
### 2. 使用
```
    docker run --gpus all nvidia/cuda:9.0-base nvidia-smi
```

## Docker 版本 < 19.03，参考[这里](https://blog.csdn.net/qq_19307465/article/details/84453901)
### 1. 安装
```
    distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
    curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo apt-key add -
    curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | sudo tee /etc/apt/sources.list.d/nvidia-docker.list
    sudo apt-get update
    sudo apt-get install -y nvidia-docker2=2.0.3+docker18.09.7-3 nvidia-container-runtime=2.0.0+docker18.09.7-3 nvidia-container-runtime-hook=1.4.0-1
    sudo systemctl restart docker
```
### 2. 使用
```
    nvidia-docker run nvidia/cuda:9.0-base nvidia-smi
```
修改 /etc/docker/daemon.json 文件：
```
    "default-runtime": "nvidia"
```
重启 docker 并运行
```
    sudo systemctl restart docker
    docker run nvidia/cuda:9.0-base nvidia-smi
```