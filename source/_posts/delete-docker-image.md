---
title: 删除私有仓库镜像
date: 2018-07-08 13:06:42
category: docker
tags: [docker, registry]
---

对于私有仓库的镜像，我们无法直接删除。当随着镜像的越来越多，会导致磁盘空间越来越小

阅读registry v2的http API后发现删除镜像需要调用几个API
* 获取image的digest
* 删除镜像的manifests


下面是删除私有registry的镜像脚本clean.sh
Usage：bash clean.sh your-image-name

需要安装jq，jq是终端解析json输出的利器


```bash
#!/usr/bin/env bash

ACCEPT_HEADER="Accept: application/vnd.docker.distribution.manifest.v2+json"
DOCKER_REGISTRY="your-registry:35000/v2"
AUTH="-uusername:password"
REPOSITORY=$1

TAGS=`curl --silent -X GET -k $AUTH $DOCKER_REGISTRY/$REPOSITORY/tags/list | jq -r '."tags"[]'`
echo image $REPOSITORY has tags: $TAGS

for TAG in ${TAGS[@]}
do
	echo  "i am going to delete $REPOSITORY:$TAG"
	digest_value=`curl -X GET -k --head --silent -H "Accept: application/vnd.docker.distribution.manifest.v2+json" $AUTH $DOCKER_REGISTRY/$REPOSITORY/manifests/$TAG 2>&1 | grep Docker-Content-Digest | awk '{print $2}'`

	digest_url="$DOCKER_REGISTRY/$REPOSITORY/manifests/$digest_value"
	echo $digest_url
	URL=${digest_url%$'\r'}
	curl -X DELETE -k -H  "Accept: application/vnd.docker.distribution.manifest.v2+json" $AUTH  $URL
done

#docker exec -it registry bin/registry garbage-collect /etc/docker/registry/config.yml

#REPOSITORY_PATH="/var/lib/registry/docker/registry/v2/repositories"
#docker exec -it registry rm -rf $REPOSITORY_PATH/$REPOSITORY
```

脚本里面注释的几行需要解释一下：
* gc回收，我们删除image的mainfests的时候，仓库并不会删除image，只有当你调用GC回收的时候，才会删除。这个可以用crontab定时执行。
* 删除具体的镜像目录。（当我们用GC回收的时候，其实只是把镜像名字从仓库里面去掉了。但是实际上磁盘还是存在这个镜像的。这个只能是通过rm删除


结尾：
这个脚本只是删除单一镜像。当然可以更进一步的封装，删除所有镜像等等