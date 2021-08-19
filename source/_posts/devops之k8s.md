---
title: devops之k8s
date: 2021-08-19 09:35:35
tags: devops
categories: devops
description: "Kubernetes常用命令"
---

kubectl apply -f kuboard_2021_01_29_11_27_09.yaml

kubectl get svc --all-namespaces

kubectl get pods --namespace ns | grep app
kubectl get pv -A

进入执行
kubectl -n ns  exec -it svc-app-877df6545-j7dzd sh
获取日志
kubectl -n ns  logs  -f svc-app-877df6545-j7dzd

kubectl -n ns  logs --tail=200  -f svc-app-877df6545-j7dzd

拷贝文件
kubectl cp ns/svc-app-search-fb47d75bb-xz8pd:var/d/application.yml application.yaml
kubectl cp application.yaml ns/svc-app-search-fb47d75bb-xz8pd:var/d/application.yml

kubectl patch pv pvc-8477bd20-ffbe-4fa2-85b9-f864d7b3690c  -p '{"metadata":{"finalizers":null}}' --type=merge
kubectl patch pvc elasticsearch-master-db-elasticsearch-0 -p '{"metadata":{"finalizers":null}}' --type=merge
