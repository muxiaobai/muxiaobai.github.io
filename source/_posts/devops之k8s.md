---
title: devops之k8s
date: 2021-08-19 09:35:35
tags: devops
categories: devops
description: "Kubernetes常用命令"
---

# 限制日志大小
cat > /etc/docker/daemon.json << EOF
{
    "registry-mirrors": [
        "https://1nj0zren.mirror.aliyuncs.com",
        "https://docker.mirrors.ustc.edu.cn",
        "http://f1361db2.m.daocloud.io",
        "https://registry.docker-cn.com"
    ],
        "log-driver": "json-file",
        "log-opts": {
                "max-size": "20m",
                "max-file": "50"
        }
}
EOF

添加工作pod
kubectl apply -f kuboard_2021_01_29_11_27_09.yaml

kubectl get svc --all-namespaces

kubectl get pods --namespace ns | grep app
kubectl get pv -A


几种类型 configmap pod service deployment statefulset

编辑ymal

kubectl -n ns-elasticsearch  edit statefulset elasticsearch-master

kubectl -n ns  edit deployment svc-app-search
kubectl -n ns  edit deployment db-elasticsearch
kubectl -n ns  edit configmap app-search-conf

查看pod
kubectl get pods -A 

删除pod 删除后自动重启
kubectl -n openapp delete pod  pod名称
删除无用的pod
kubectl get pods -A  | egrep 'Error' | awk '{print $2}' | xrgs kubectl  -n openapp delete pod   

查看pod网络
kubectl -n openapp get pods -o wide 
查看pod的ymal
kubectl -n openapp get pod db-elasticsearch-8687d9d95c-gp7tn  -o yaml  

详细信息
kubectl -n openapp describe pod db-elasticsearch-8684c44cb8-lv5wn 

进入执行
kubectl -n openapp  exec -it db-elasticsearch-8684c44cb8-lv5wn -- sh

获取日志
kubectl -n openapp  logs --tail=200  -f svc-app-search-877df6545-j7dzd 


拷贝文件
kubectl cp ns/svc-app-search-fb47d75bb-xz8pd:var/d/application.yml application.yaml
kubectl cp application.yaml ns/svc-app-search-fb47d75bb-xz8pd:var/d/application.yml
