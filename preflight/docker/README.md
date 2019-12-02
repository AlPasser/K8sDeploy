# Docker 安装

```
    curl -fsSL http://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo apt-key add -
    sudo add-apt-repository "deb [arch=amd64] http://mirrors.aliyun.com/docker-ce/linux/ubuntu $(lsb_release -cs) stable"
    sudo apt update -y
    # apt-cache madison docker-ce
    # apt-cache madison docker-ce-cli
    os_codename=$(lsb_release -cs)
    sudo apt-get -y install docker-ce=5:18.09.7~3-0~ubuntu-${os_codename} docker-ce-cli=5:18.09.7~3-0~ubuntu-${os_codename}
    sudo apt-mark hold docker-ce docker-ce-cli
    sudo groupadd docker
    sudo usermod -aG docker $USER
```