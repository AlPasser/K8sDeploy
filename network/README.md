# 常见问题

## 1. flannel timeout

错误日志：
```
Failed to create SubnetManager: error retrieving pod spec for 'kube-system/kube-flannel-ds-l8g7l': Get "https://10.96.0.1:443/api/v1/namespaces/kube-system/pods/kube-flannel-ds-l8g7l": dial tcp 10.96.0.1:443: i/o timeout
```

[问题解决参考链接](https://blog.fanfengqiang.com/2019/07/27/kubernetes%E7%8E%AF%E5%A2%83%E4%B8%ADflannel%E7%BD%91%E7%BB%9C%E6%8F%92%E4%BB%B6%E7%9A%84DNS%E4%B8%8Ehosts%E6%96%87%E4%BB%B6%E7%9A%84%E4%BC%98%E5%85%88%E7%BA%A7%E9%97%AE%E9%A2%98/)

解决方案：
```
主节点上执行命令：

    kubectl edit ds -n kube-system kube-flannel-ds
    

添加环境变量（确保“dnsPolicy: ClusterFirst”）：

    - name: KUBERNETES_SERVICE_HOST
      value: 211.67.19.251
    - name: KUBERNETES_SERVICE_PORT
      value: "46443"
```

## 2. 不同 Node 上的 Pod 间无法通信

在每个节点上执行（[参考链接](https://my.oschina.net/u/4275236/blog/3354231)）：
```
  sudo bash -c "iptables -P OUTPUT ACCEPT && iptables -P FORWARD ACCEPT && iptables -F && iptables -L -n"
```

## 3. Failed to create pod sandbox
错误日志：
```
  Failed to create pod sandbox: rpc error: code = Unknown desc = failed to set up sandbox container "572d5fd6d13d2dd6ac68c056ae69b2b3a78bcf7c36fb8e4435a7518150ebef02" network for pod "coldmountain-f465d6868-8bpt4": networkPlugin cni failed to set up pod "coldmountain-f465d6868-8bpt4_coldworld" network: failed to set bridge addr: "cni0" already has an IP address different from 10.244.2.1/24
```
在 Pod 部署节点上执行：
```
  ifconfig cni0 down 
  ip link delete cni0
```

## 4. 网络插件残留问题

```
  sudo rm /etc/cni/net.d/相关文件
```

## 5. 网关代理导致不同 Node 上的 Pod 间无法通信

[参考链接](https://cloud.tencent.com/developer/article/1819134)

问题定位命令：
```
  sudo tcpdump -i eno1 port 8472  # UDP 协议
  kubectl describe node cloud04  # cloud04 为被代理的节点，实际场景中为主节点
```

解决方案：
```
主节点上执行：
  kubectl annotate node cloud04 flannel.alpha.coreos.com/public-ip-overwrite=211.67.19.251 --overwrite
  kubectl -n kube-system get pods --field-selector spec.nodeName=cloud04 | grep flannel
  kubectl -n kube-system get pod kube-flannel-ds-cpwc8 -o yaml | kubectl replace --force -f -
网关节点上执行（211.67.19.251 为网关 IP）：
  iptables -t nat -A PREROUTING -d 211.67.19.251 -p udp --dport 8472 -j DNAT --to-destination 192.168.1.4:8472
  iptables -t nat -A POSTROUTING -d 192.168.1.4 -p udp --dport 8472 -j SNAT --to 192.168.1.1
```