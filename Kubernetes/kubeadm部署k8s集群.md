# kubeadm部署k8s集群

```shell
# 安装docker
sudo yum install docker -y

#修改docker 配置文件 
sudo sed -i '/[# ]*INSECURE_REGISTRY=[a-zA-Z0-9 -'']*/d' /etc/sysconfig/docker
echo "INSECURE_REGISTRY='--insecure-registry docker.dtdream.com -H unix:/var/run/docker.sock'" >> /etc/sysconfig/docker

```

