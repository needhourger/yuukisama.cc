---
title: "K8s部署n8n"
subtitle: ""
description: ""
date: 2025-07-22T10:18:47+08:00
image: ""
tags: [ai, k8s, n8n]
categories: []
draft: false
---

## Zero

[n8n](https://n8n.io/) 是一个专门为技术团队打造的灵活的 AI 自动化工作流。也是目前最火的几个 AI 工作流工具之一。与其相同的竞品还有 [Dify](https://dify.ai/) 由中国团队主导开发的工作流产品。

不过有意思的是 n8n 这个主要贡献人由外国人组成的项目使用的反而是 Vue 框架外加全套 typescript，而由国人主导开发的 Dify 使用的反而是 Next.js 框架和 python。当然二者均为开源软件，均提供免费的社区版本用于自我部署使用。同时也提供商业版本提供更多的技术支持。本文主要以 n8n 社区版自我部署为主。

由于 n8n 官方文档仅提供了有关使用 docker 部署的实践文档，因此基于 k8s 部署仍有一些需要踩坑的地方。本文主要讨论基于 k8s v1.20.14 以及 Kuboard v3.5.2.7 为平台搭建 n8n 工作流平台的过程。

## First - 基础服务部署

首先由于国内网络的特殊性，目标部署的服务器位于宁夏无法直接拉取 n8n 的官方镜像，因此我们需要在本地机器上将 n8n 的镜像拉取到本地机器，然后上传至 k8s 集群内目标部署的节点服务器上。

1. 使用如下命令拉取镜像以及导出生成压缩文件

```shell
# 拉取镜像
docker pull docker.n8n.io/n8nio/n8n
# 导出镜像并使用 gzip 压缩
docker save docker.n8n.io/n8nio/n8n:latest | gzip > n8n.tar.gz
```

1. 在上传完成之后使用如下命令将其导入目标服务器 docker

```shell
docker load < n8n.tar.gz
```

1. 接下来编写部署我们的 k8s service yaml

- n8n 服务运行需要一个固定的存储卷存储数据, 因此我们需要为这个服务添加存储卷，映射路径是`/path:/home/node/.n8n`
- 除此之外，因为在这个 k8s 集群上还部署有很多其他的服务，并且服务之间是通过路由路径区分的，而不是通过子域名的方式，而 n8n 官方文档中默认推荐是按照子域名的方式部署。因此我们需要给容器服务添加一个环境变量`N8N_PATH=/n8n/`，以使得我们可以通过类似 example.com/n8n 这样的路径去访问 n8n 服务。
- 容器服务运行在 `5678` 端口，因此需要开放此端口。

```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    k8s.kuboard.cn/displayName: ""
  labels:
    k8s.kuboard.cn/name: n8n
  name: n8n
  namespace: dev
  resourceVersion: "40818369"
spec:
  progressDeadlineSeconds: 600
  replicas: 0
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      k8s.kuboard.cn/name: n8n
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      annotations:
        kubectl.kubernetes.io/restartedAt: "2025-07-18T17:35:21+08:00"
      creationTimestamp: null
      labels:
        k8s.kuboard.cn/name: n8n
    spec:
      containers:
        - env:
            - name: N8N_PATH
              value: /n8n/
          image: "docker.n8n.io/n8nio/n8n:latest"
          imagePullPolicy: IfNotPresent
          name: n8n
          ports:
            - containerPort: 5678
              name: http
              protocol: TCP
          resources: {}
          terminationMessagePath: /dev/termination-log
          terminationMessagePolicy: File
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
      volumes:
        - name: volume-n8n-data
          persistentVolumeClaim:
            claimName: gl-log-ai-copilot-pvc
status:
  conditions:
    - lastTransitionTime: "2025-07-21T05:45:32Z"
      lastUpdateTime: "2025-07-21T05:45:32Z"
      message: Deployment has minimum availability.
      reason: MinimumReplicasAvailable
      status: "True"
      type: Available
    - lastTransitionTime: "2025-07-18T07:48:33Z"
      lastUpdateTime: "2025-07-21T06:20:45Z"
      message: ReplicaSet "n8n-6bc8bfcccb" has successfully progressed.
      reason: NewReplicaSetAvailable
      status: "True"
      type: Progressing
  observedGeneration: 50

---
apiVersion: v1
kind: Service
metadata:
  annotations: {}
  labels:
    k8s.kuboard.cn/name: n8n
  name: n8n
  namespace: dev
  resourceVersion: "40175508"
spec:
  clusterIP: 10.33.52.139
  clusterIPs:
    - 10.33.52.139
  ports:
    - name: skn4eq
      port: 5678
      protocol: TCP
      targetPort: 5678
  selector:
    k8s.kuboard.cn/name: n8n
  sessionAffinity: None
  type: ClusterIP
status:
  loadBalancer: {}
```

## Second - 应用路由

n8n 默认的部署方式是推荐基于子域名，因此需要按照路径路由来实现服务部署的话需要配置路径重写。以下是基于 k8s ingress 的路径重写服务的 yaml。

- 标准的端口服务映射，我们将 `example.com/n8n`端口映射到 n8n service 的 5678 端口上。
- 同是重头戏的 path rewrite, 需要重写 除了默认的 n8n 重写到 / 路径，还需要重写如下两条：
  - /assets/ -> /n8n/assets/
  - /static/ -> /n8n/static/

```yaml
nginx.ingress.kubernetes.io/configuration-snippet: |
  rewrite ^/assets/(.*)$ /n8n/assets/$1 redirect;
  rewrite ^/static/(.*)$ /n8n/static/$1 redirect;
nginx.ingress.kubernetes.io/rewrite-target: /$2
```

- 完整的 ingress 服务 yaml

```yaml
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    kubernetes.io/ingress.class: nginx
    nginx.ingress.kubernetes.io/configuration-snippet: |
      rewrite ^/assets/(.*)$ /n8n/assets/$1 redirect;
      rewrite ^/static/(.*)$ /n8n/static/$1 redirect;
    nginx.ingress.kubernetes.io/rewrite-target: /$2
  name: n8n-ingress
  namespace: dev
  resourceVersion: "40791462"
spec:
  rules:
    - host: example.com
      http:
        paths:
          - backend:
              service:
                name: n8n
                port:
                  number: 5678
            path: /n8n(/|$)(.*)
            pathType: Prefix
          - backend:
              service:
                name: n8n
                port:
                  number: 5678
            path: /rest
            pathType: Prefix
  tls:
    - hosts:
        - example.com
      secretName: example.com
status:
  loadBalancer:
    ingress:
      - hostname: localhost
```

## End

到目前总结就完成了 n8n 的基础部署，就可以通过 example.com/n8n 这个路径去访问部署于 k8s 集群中的 n8n 服务了。当然可能目前本文中还有些遗漏逻辑，没有测试部署完成后这个 n8n 的所有功能包括 webhook ，以及各种 api 等。 可能后续有问题还需要补充。
