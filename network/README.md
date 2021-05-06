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

