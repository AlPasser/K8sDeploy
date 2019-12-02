# K8s 安装

```
    sudo sh -c 'cat << EOF > /etc/apt/sources.list.d/kubernetes.list
deb https://mirrors.aliyun.com/kubernetes/apt/ kubernetes-xenial main
EOF'

    curl -s https://repo.huaweicloud.com/kubernetes/apt/doc/apt-key.gpg | sudo apt-key add - 
    sudo apt update -y
    # apt-cache madison kubelet
    # apt-cache madison kubeadm
    # apt-cache madison kubectl
    sudo apt install -y kubeadm=1.15.5-00 kubelet=1.15.5-00 kubectl=1.15.5-00
    sudo apt-mark hold kubelet kubeadm kubectl
    sudo systemctl enable kubelet
    sudo systemctl start kubelet
```