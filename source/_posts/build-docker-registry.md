---
title: 用registry搭建docker私有仓库
date: 2018-07-08 12:11:34
category: docker
tags: [docker, registry]
---


### 准备目录
首先在根目录下创建一个文件夹registry（取名随意），在registry目录下面创建三个文件夹
data，auth，certs

### 设置registry的用户密码
```bash
docker run --rm --entrypoint htpasswd registry:2 -Bbn  {{username}} {{password}} > auth/htpasswd
```

### 生成证书文件
```bash
docker run --rm -e COMMON_NAME=pek2-office-9th-10-117-169-121.eng.vmware.com -e KEY_NAME=domain -v /root/registry/certs:/certs centurylink/openssl
```
COMMON_NAME 是你的FQDN
上面的命令执行完成，你会在/root/registry/certs目录下面生成三个文件：domain.crt, domain.csr, domain.key
如果你遇到permission error， 有可能是selinux的问题，可以执行`setenforce 0`


### 运行registry
```bash
docker run -d --name registry -p 5000:5000 -v /root/registry/data:/var/lib/registry -v /root/registry/auth:/auth -v /root/registry/certs:/certs -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/domain.crt -e REGISTRY_HTTP_TLS_KEY=/certs/domain.key -e "REGISTRY_AUTH=htpasswd" -e "REGISTRY_AUTH_HTPASSWD_REALM=Registry Realm" -e REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd registry:2
```


### 拷贝证书文件

```bash
mkdir -p /etc/docker/certs.d/pek2-office-9th-10-117-169-121.eng.vmware.com:5000
cp /root/registry/certs/domain.crt /etc/docker/certs.d/pek2-office-9th-10-117-169-121.eng.vmware.com:5000
```

### push镜像到私有Registry

```bash
docker login -u {{username}} -p {{password}} pek2-office-9th-10-117-169-121.eng.vmware.com:5000
docker tag ubuntu pek2-office-9th-10-117-169-121.eng.vmware.com:5000/ubuntu:firstimage
docker push pek2-office-9th-10-117-169-121.eng.vmware.com:5000/ubuntu:firstimage
```

到此，我们搭建好了我们的私有仓库
