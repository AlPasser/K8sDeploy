# K8s 部署

## 初始化
```
    sudo kubeadm reset
    rm $HOME/.kube/ -rf
```

## 主节点
```
    sudo swapoff -a
    sudo sed -i 's/.*swap.*/#&/' /etc/fstab
    sudo kubeadm init --image-repository truthbean --kubernetes-version=v1.15.5 --pod-network-cidr=10.244.0.1/16 > init.log
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

