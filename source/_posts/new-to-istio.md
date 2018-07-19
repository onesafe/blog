---
title: kubernetes安装istio
date: 2018-07-10 11:21:59
category: ServiceMesh
tags: [istio]
---

### 下载Istio

下载最新版的istio
```bash
curl -L https://git.io/getLatestIstio | sh -
```

```bash
cd istio-0.8.0
export PATH=$PWD/bin:$PATH
```
安装文件在install目录下，`istioctl`执行文件在bin目录下，一些应用在samples目录下


### 安装Istio

```bash
kubectl apply -f install/kubernetes/istio-demo.yaml
```
由于我用的是docker-for-mac，我的kubernetes是没有Loadbalancer的，所以我把istio-demo.yaml文件里的LoadBalancer改成了NodePort


### 验证Istio

查看service和pod是否都正常运行
![](https://wx4.sinaimg.cn/mw690/006yibQ1ly1ft4p018cjgj31860h6455.jpg)

![](https://wx1.sinaimg.cn/mw690/006yibQ1ly1ft4p00sx4qj31ga0ycdp7.jpg)


### 部署应用

部署应用有两种方式：
* 安装了Istio-sidecar-injecor
```bash
kubectl label namespace <namespace> istio-injection=enabled
kubectl create -n <namespace> -f <your-app-spec>.yaml
```
* 没有安装Istio-sidecar-injecor
```bash
kubectl create -f <(istioctl kube-inject -f <your-app-spec>.yaml)
```

在这里我们部署samples里的bookinfo应用
```bash
kubectl label namespace bran istio-injection=enabled
```
![](https://wx3.sinaimg.cn/mw690/006yibQ1ly1ft4p00d4g3j30lc0ay0u6.jpg)

```bash
kubectl apply -f samples/bookinfo/kube/bookinfo.yaml -n bran
```

创建应用的ingress gateway
```bash
istioctl create -f samples/bookinfo/routing/bookinfo-gateway.yaml
```
其中在bookinfo-gateway.yaml的meta里面加了`namesapce: bran`

创建完成了可以看到
![](https://wx2.sinaimg.cn/mw690/006yibQ1ly1ft4p008ge7j312407a40u.jpg)


### 访问应用

通过nodeport:ip/productpage访问bookinfo应用
![](https://wx4.sinaimg.cn/mw690/006yibQ1ly1ft4p001910j32820zudo7.jpg)

多点几次，发现有不同的页面出现。应用用到了负载均衡
