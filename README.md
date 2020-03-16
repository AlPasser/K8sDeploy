# K8s 部署

## 安装 kubeadm
```
    # 安装系统工具
    sudo apt-get update && sudo apt-get install -y apt-transport-https
    # 安装 GPG 证书
    curl https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg | sudo apt-key add -
    # 写入软件源；注意：我们用系统代号为 bionic，但目前阿里云不支持，所以沿用 16.04 的 xenial
    cat << EOF >/etc/apt/sources.list.d/kubernetes.list
    > deb https://mirrors.aliyun.com/kubernetes/apt/ kubernetes-xenial main
    > EOF
    # 安装
    sudo apt-get update  
    sudo apt-get install -y kubelet kubeadm kubectl
    # 设置 kubelet 自启动，并启动 kubelet
    sudo systemctl enable kubelet && sudo systemctl start kubelet
```

## 初始化
```
    sudo kubeadm reset
    rm $HOME/.kube/ -rf
```

## 主节点
```
    sudo swapoff -a
    sudo sed -i 's/.*swap.*/#&/' /etc/fstab
    sudo kubeadm init --image-repository registry.aliyuncs.com/google_containers --kubernetes-version=v1.15.5 --pod-network-cidr=10.244.0.1/16 > init.log
    mkdir -p $HOME/.kube
    sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
    sudo chown -R $(id -u):$(id -g) $HOME/.kube/config
```

## 网络
### flannel
```
    kubectl -n kube-system apply -f ./network/kube-flannel.yml
```
### canal
```
    kubectl -n kube-system apply -f ./network/canal.yaml
```

## 其它节点
```
    sudo swapoff -a
    sudo sed -i 's/.*swap.*/#&/' /etc/fstab
```
### 1. 已知
```
    sudo kubeadm join 192.168.1.3:6443 --token ifn8u4.4wno652ls3cclhii --discovery-token-ca-cert-hash sha256:08b2358b349e96871a4372e6d14d714dadc34f372b8a9ca79dd328689233ad71
```
### 2. 未知
#### 创建 token
```
    sudo kubeadm token create
```
#### 查看 token
```
    sudo kubeadm token list
```
#### 获取 CA 公钥的哈希值
```
    openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | openssl dgst -sha256 -hex | sed 's/^.* //'
```

