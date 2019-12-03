# 问题与解决方案

## 1. [Kubernetes Pod 内容器无法访问外网问题](https://www.xbzdr.com/590.html)

## 2. kubeadm join 超时
kubelet 版本：v1.15.5

查看 docker 的 cgroup-driver 是 cgroupfs 还是 systemd，kubelet 默认为 systemd

修改 /lib/systemd/system/kubelet.service 文件：
```
    ExecStart=/usr/bin/kubelet --address=127.0.0.1 --pod-manifest-path=/etc/kubernetes/manifests --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf --config=/var/lib/kubelet/config.yaml --cgroup-driver=cgroupfs --network-plugin=cni --pod-infra-container-image=truthbean/pause:3.1
```
