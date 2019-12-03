# K8s 高可用性

## 安装 keepalived 和 haproxy
```
    sudo apt install -y keepalived haproxy
```

## 配置并启动 keepalived
### 修改 /etc/keepalived/keepalived.conf 文件，仅 priority 和 interface 不同
```
global_defs {
    notification_email {
        770340084@qq.com
    }
    notification_email_from ubuntu@oceanai.com
    smtp_server 127.0.0.1
    smtp_connect_timeout 30
    router_id LVS_1
}

vrrp_instance VI_1 {
    state MASTER
    interface eth0
    virtual_router_id 88
    advert_int 1
    priority 100
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.1.88/24
    }
}
```
### 启动
```
    sudo systemctl enable keepalived
    sudo systemctl start keepalived
```

## 配置并启动 haproxy
### 修改 /etc/haproxy/haproxy.cfg 文件
```
global
        chroot /var/lib/haproxy
        daemon
        group haproxy
        user haproxy
        log 127.0.0.1:514 local0 warning
        pidfile /var/lib/haproxy.pid
        maxconn 20000
        spread-checks 3
        nbproc 8

defaults
        log global
        mode tcp
        retries 3
        option redispatch

listen https-apiserver
        bind 192.168.1.88:8443
        mode tcp
        balance roundrobin
        timeout server 900s
        timeout connect 15s
        timeout client 1m
        server apiserver01 192.168.1.3:6443 check port 6443 inter 5000 fall 5
        server apiserver02 192.168.1.12:6443 check port 6443 inter 5000 fall 5
        server apiserver03 192.168.1.7:6443 check port 6443 inter 5000 fall 5
```
### 启动
```
    sudo systemctl enable haproxy
    sudo systemctl start haproxy
```

## 部署 K8s
### Use IPVS Mode，参考[这里](https://github.com/kubernetes/kubernetes/blob/master/pkg/proxy/ipvs/README.md)
```
    sudo apt install ipset
```
### 导出默认配置
```
    kubeadm config print init-defaults > kubeadm-init.yaml
```
### 修改默认配置
```
apiVersion: kubeadm.k8s.io/v1beta2
bootstrapTokens:
- groups:
  - system:bootstrappers:kubeadm:default-node-token
  token: abcdef.0123456789abcdef
  ttl: 24h0m0s
  usages:
  - signing
  - authentication
kind: InitConfiguration
localAPIEndpoint:
  advertiseAddress: 192.168.1.3
  bindPort: 6443
nodeRegistration:
  criSocket: /var/run/dockershim.sock
  name: cloud03
  taints:
  - effect: NoSchedule
    key: node-role.kubernetes.io/master
---
apiServer:
  timeoutForControlPlane: 4m0s
apiVersion: kubeadm.k8s.io/v1beta2
certificatesDir: /etc/kubernetes/pki
clusterName: kubernetes
controlPlaneEndpoint: "192.168.1.88:8443"
controllerManager: {}
dns:
  type: CoreDNS
etcd:
  local:
    dataDir: /var/lib/etcd
imageRepository: truthbean
kind: ClusterConfiguration
kubernetesVersion: v1.15.5
networking:
  dnsDomain: cluster.local
  serviceSubnet: 10.244.0.1/16
  podSubnet: ""
scheduler: {}
---
apiVersion: kubeproxy.config.k8s.io/v1alpha1
kind: KubeProxyConfiguration
mode: "ipvs"
```
### 使用修改后的配置
```
    sudo swapoff -a
    sudo sed -i 's/.*swap.*/#&/' /etc/fstab
    sudo kubeadm init --config kubeadm-init.yaml
    mkdir -p $HOME/.kube
    sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
    sudo chown -R $(id -u):$(id -g) $HOME/.kube/config
```
### 将证书文件拷贝至其它 Master 节点
#### 1. 创建文件夹
```
    sudo mkdir -p /etc/kubernetes/pki/etcd
```
#### 2. 要拷贝的文件
```
    /etc/kubernetes/pki/ca.*
    /etc/kubernetes/pki/sa.*
    /etc/kubernetes/pki/front-proxy-ca.*
    /etc/kubernetes/pki/etcd/ca.*
    /etc/kubernetes/admin.conf
```
### 加入其它 Master 节点
```
    sudo kubeadm join 192.168.1.88:8443 --token abcdef.0123456789abcdef --discovery-token-ca-cert-hash sha256:7b1787f08945957d995bef12e2ac234322dfc1a8eee0bdbe060bbf81b04bbc7c --experimental-control-plane
    mkdir -p $HOME/.kube
    sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
    sudo chown -R $(id -u):$(id -g) $HOME/.kube/config
```
### 加入其它非 Master 节点
```
    sudo kubeadm join 192.168.1.88:8443 --token abcdef.0123456789abcdef --discovery-token-ca-cert-hash sha256:7b1787f08945957d995bef12e2ac234322dfc1a8eee0bdbe060bbf81b04bbc7c
```