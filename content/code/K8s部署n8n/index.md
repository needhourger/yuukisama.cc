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
- 由于我们仅仅在固定节点上上传了n8n的镜像，因此别忘记添加nodeName或nodeSelector限制服务运行节点

### 完整的Config yaml

```yaml
---
apiVersion: v1
data:
  N8N_ENDPOINT_REST: n8nrest
  N8N_HOST: example.com # 替换对应产线域名
  N8N_PATH: /n8n/
  WEBHOOK_URL: 'https://example.com/n8n/' # 替换对应产线域名
kind: ConfigMap
metadata:
  annotations: {}
  labels: {}
  name: n8n-config
  namespace: default
  resourceVersion: '42737379'

```


### 完整的service yaml范例

```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    k8s.kuboard.cn/displayName: ''
  labels:
    k8s.kuboard.cn/name: n8n
  name: n8n
  namespace: default
  resourceVersion: '42737473'
spec:
  progressDeadlineSeconds: 600
  replicas: 1
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
        kubectl.kubernetes.io/restartedAt: '2025-07-23T16:26:38+08:00'
      creationTimestamp: null
      labels:
        k8s.kuboard.cn/name: n8n
    spec:
      affinity: {}
      containers:
        - envFrom:
            - configMapRef:
                name: n8n-config
          image: 'docker.n8n.io/n8nio/n8n:latest'
          imagePullPolicy: IfNotPresent
          name: n8n
          ports:
            - containerPort: 5678
              name: http
              protocol: TCP
          resources: {}
          terminationMessagePath: /dev/termination-log
          terminationMessagePolicy: File
          volumeMounts:
            - mountPath: /home/node/
              name: volume-n8n-data
      dnsPolicy: ClusterFirst
      nodeSelector:
          kubernetes.io/hostname: your-node-name # 替换成对应的node
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext:
        fsGroup: 1000
        fsGroupChangePolicy: Always
        runAsGroup: 1000
        runAsUser: 1000
        seLinuxOptions: {}
      terminationGracePeriodSeconds: 30
      volumes:
        - name: volume-n8n-data
          persistentVolumeClaim:
            claimName: gl-log-n8n-pvc
status:
  availableReplicas: 1
  conditions:
    - lastTransitionTime: '2025-07-18T07:48:33Z'
      lastUpdateTime: '2025-07-29T03:10:13Z'
      message: ReplicaSet "n8n-6d5df5f769" has successfully progressed.
      reason: NewReplicaSetAvailable
      status: 'True'
      type: Progressing
    - lastTransitionTime: '2025-07-31T02:29:24Z'
      lastUpdateTime: '2025-07-31T02:29:24Z'
      message: Deployment has minimum availability.
      reason: MinimumReplicasAvailable
      status: 'True'
      type: Available
  observedGeneration: 183
  readyReplicas: 1
  replicas: 1
  updatedReplicas: 1

---
apiVersion: v1
kind: Service
metadata:
  annotations: {}
  labels:
    k8s.kuboard.cn/name: n8n
  name: n8n
  namespace: default
  resourceVersion: '40175508'
spec:
  clusterIP: 10.0.0.100
  clusterIPs:
    - 10.0.0.100
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
- 自从n8n 1.102.0版本以后，ingress中对于静态文件反向代理并重写路径和restful api的路径匹配需要分开形成两个ingress 

```yaml
nginx.ingress.kubernetes.io/configuration-snippet: |
  rewrite ^/assets/(.*)$ /n8n/assets/$1 redirect;
  rewrite ^/static/(.*)$ /n8n/static/$1 redirect;
nginx.ingress.kubernetes.io/rewrite-target: /$2
```

### 1.102.0 版本之前

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
  namespace: default
  resourceVersion: "40791462"
spec:
  rules:
    - host: example.com   # 替换成产线环境的域名
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
        - example.com           # 替换成产线环境
      secretName: example.com   # 替换成产线环境
status:
  loadBalancer:
    ingress:
      - hostname: localhost
```

### 1.102.0 版本之后

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
  namespace: default
  resourceVersion: '42736928'
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
  tls:
    - hosts:
        - example.com
      secretName: example.com

---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: n8n-rest-ingress
  namespace: default
  resourceVersion: '42737251'
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
            path: /n8nrest
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

## Third

### Command "start" not found

- 当服务运行内容出现如下报错

```log
Invalid number value for N8N_PORT: tcp://10.0.0.100:5678
No encryption key found - Auto-generating and saving to: /home/node/.n8n/config
Error: Command "start" not found
```

- 则说明所配置的服务存储卷的权限有问题，n8n无法正确写入持久化信息，因此你可能需要在yaml中加入如下字段

```yaml
          securityContext:
            runAsGroup: 1000
            runAsUser: 1000
```

- 如果使用上述安全上下文配置依旧无法解决实际的权限问题，服务还是没有办法启动，那就只能去物理机器上寻找实际映射的存储卷目录修改其存储权限。n8n内使用的Linux权限是```1000:1000```, 因此直接在容器宿主机上使用修改文件权限将整个映射目录权限修改：

```shell
sudo chown -R 1000:1000 ./n8n
```

### 并发控制参数

- 在常规模式中n8n并不会限制可以同时运行的生产执行数量，但是这可能会导致过多的并发执行导致事件循环被破坏，进而导致性能下降以及无响应
- 并发控制默认是禁用状态，如果需要进行并发控制可以通过传入如下环境变量：
```export N8N_CONCURRENCY_PRODUCTION_LIMIT=20```
- 并发控制仅适用于从Webhook或者触发器节点启动的执行，不适用于任何其他类型启动的工作流程，例如：手动执行，子工作流执行，错误执行以及从CLI启动的执行。
- 启用并发控制意味着某些操作包括取消，重试工作流的执行操作也会因为被并发控制限制而被从待执行队列删除。
- 如果还需要更高的并发能力可以考虑使用n8n的队列模式，使用redis等作为其消息中间件部署多个n8n节点以实现承载更大的并发。

## Forth N8N Queue模式增加并发

n8n的队列模式，顾名思义，受限于n8n单个节点并发性能限制因而官方提供了 Queue 队列模式以提升并发性能。在Queue模式中节点分为：

- 主节点：负责对外部暴露接口提供Web UI以及API调用
- Worker 节点：负责听从主节点的广播的任务并运行任务

要开启模式首先需要保证同时运行一个 Redis 服务，因为主从节点之间通过 Redis 同步消息，同时目前版本的 n8n 默认情况下使用的是 Sqlite 数据库，但N8N在 Queue 模式不支持使用 Sqlite 数据库运行，官方推荐的支持的实践是同时配置 PostgreSQL 数据库。
本文的例子中将使用默认自带的 Sqlite 进行基于k8s的n8n主从节点搭建。

### 环境变量

环境变量相较于之前的版本将会新增如下配置项：

- EXECUTION_MODE queue 指定当前的运行模式为 Queue队列模式
- N8N_ENCRYPTION_KEY 节点之间共同的密钥凭证
- N8N_RUNNERS_ENABLED 'true' 启用runners
- N8N_SECURE_COOKIE 'false' 禁用cookie
- QUEUE_BULL_REDIS_HOST redis地址
- QUEUE_BULL_PASSWORD redis密码
- QUEUE_BULL_PORT reids的端口
- QUEUE_BULL_USERNAME redis用户名，可缺省（redis低于6.0版本并不支持用户名，6.0以上如果配置了可以使用此环境变量）
- QUEUE_HEALTH_CHECK_ACTIVE 节点健康检查，设置开启后将会提供 /healthz 路径用以提供节点健康检查

Postgre SQL 数据库相关的环境变量

- DB_TYPE设定使用的数据库类型
- DB_POSTGRESDB_DATABASE 数据库名
- DB_POSTGRESDB_HOST 数据库地址
- DB_POSTGRESDB_PORT 数据库端口
- DB_POSTGRESDB_USER 数据库用户名
- DB_POSTGRESDB_PASSWORD 数据库密码

更多参数详情 [https://docs.n8n.io/hosting/configuration/environment-variables/database/#postgresql](https://docs.n8n.io/hosting/configuration/environment-variables/database/#postgresql)

```yaml
---
apiVersion: v1
data:
  EXECUTION_MODE: queue
  N8N_ENCRYPTION_KEY: testEncryptKey
  N8N_ENDPOINT_REST: n8nrest
  N8N_HOST: menusifu-crm.wiz.ai
  N8N_PATH: /n8n/
  N8N_RUNNERS_ENABLED: 'true'
  N8N_SECURE_COOKIE: 'false'
  QUEUE_BULL_REDIS_HOST: 172.31.41.152
  QUEUE_BULL_REDIS_PASSWORD: wizDEVredis2021 # 有需要可以通过 secret 传入密码
  QUEUE_BULL_REDIS_PORT: '6379'
  QUEUE_HEALTH_CHECK_ACTIVE: 'true'
  WEBHOOK_URL: 'https://menusifu-crm.wiz.ai/n8n/'
  DB_TYPE: postgresdb
  DB_POSTGRESDB_HOST: postgres
  DB_POSTGRESDB_PORT: 5432
  DB_POSTGRESDB_DATABASE: ${POSTGRES_DB}
  DB_POSTGRESDB_USER: ${POSTGRES_NON_ROOT_USER}
  DB_POSTGRESDB_PASSWORD: ${POSTGRES_NON_ROOT_PASSWORD}
kind: ConfigMap
metadata:
  name: n8n-config
  namespace: dev
  resourceVersion: '43776571'
```

### 主节点服务

- 主节点和之前没有太大区别，直接使用更新后的 configMap 引入新的环境变量即可让主节点工作在 Queue 模式

```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    k8s.kuboard.cn/displayName: ''
  labels:
    k8s.kuboard.cn/name: n8n
  name: n8n
  namespace: dev
  resourceVersion: '43774451'
spec:
  progressDeadlineSeconds: 600
  replicas: 1
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
        kubectl.kubernetes.io/restartedAt: '2025-07-23T16:26:38+08:00'
      creationTimestamp: null
      labels:
        k8s.kuboard.cn/name: n8n
    spec:
      affinity: {}
      containers:
        - envFrom:
            - configMapRef:
                name: n8n-config
          image: 'docker.n8n.io/n8nio/n8n:latest'
          imagePullPolicy: IfNotPresent
          name: n8n
          ports:
            - containerPort: 5678
              name: http
              protocol: TCP
          resources: {}
          terminationMessagePath: /dev/termination-log
          terminationMessagePolicy: File
          volumeMounts:
            - mountPath: /home/node/.n8n
              name: volume-n8n-data
      dnsPolicy: ClusterFirst
      nodeName: test-02-aws-m01
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext:
        fsGroup: 1000
        fsGroupChangePolicy: Always
        runAsGroup: 1000
        runAsUser: 1000
        seLinuxOptions: {}
      terminationGracePeriodSeconds: 30
      volumes:
        - name: volume-n8n-data
          persistentVolumeClaim:
            claimName: gl-log-n8n-pvc
status:
  availableReplicas: 1
  conditions:
    - lastTransitionTime: '2025-07-18T07:48:33Z'
      lastUpdateTime: '2025-08-05T05:37:45Z'
      message: ReplicaSet "n8n-549dd6f64c" has successfully progressed.
      reason: NewReplicaSetAvailable
      status: 'True'
      type: Progressing
    - lastTransitionTime: '2025-08-05T06:33:22Z'
      lastUpdateTime: '2025-08-05T06:33:22Z'
      message: Deployment has minimum availability.
      reason: MinimumReplicasAvailable
      status: 'True'
      type: Available
  observedGeneration: 214
  readyReplicas: 1
  replicas: 1
  updatedReplicas: 1

---
apiVersion: v1
kind: Service
metadata:
  annotations: {}
  labels:
    k8s.kuboard.cn/name: n8n
  name: n8n
  namespace: dev
  resourceVersion: '40175508'
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

### 从（Worker）节点

- 主从节点使用同一个 configMap 的提供的环境变量连接到同一个 Redis 中以实现同步
- 从节点和主节点唯一的不同就是修改了启动命令，从节点通过 n8n woker --concurrency=30命令启动让其工作在 Queue 模式下作为 worker 节点， concurrency 参数则用以限制节点最大并行作业数（默认是10）
- 这样在主节点唯一的情况下可以通过动态扩容从节点以提升或降低服务承载能力。

```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations: {}
  labels:
    k8s.kuboard.cn/name: n8n-workers
  name: n8n-workers
  namespace: dev
  resourceVersion: '43776739'
spec:
  progressDeadlineSeconds: 600
  replicas: 2
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      k8s.kuboard.cn/name: n8n-workers
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      creationTimestamp: null
      labels:
        k8s.kuboard.cn/name: n8n-workers
    spec:
      containers:
        - args:
            - '--concurrency=30'
          command:
            - n8n
            - worker
          envFrom:
            - configMapRef:
                name: n8n-config
          image: 'docker.n8n.io/n8nio/n8n:latest'
          imagePullPolicy: IfNotPresent
          name: n8n-workers
          resources: {}
          terminationMessagePath: /dev/termination-log
          terminationMessagePolicy: File
          volumeMounts:
            - mountPath: /home/node/.n8n
              name: volume-3zjiw
      dnsPolicy: ClusterFirst
      nodeName: test-02-aws-m01
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext:
        fsGroup: 1000
        runAsGroup: 1000
        runAsUser: 1000
        seLinuxOptions: {}
      terminationGracePeriodSeconds: 30
      volumes:
        - name: volume-3zjiw
          persistentVolumeClaim:
            claimName: gl-log-n8n-pvc
status:
  availableReplicas: 2
  conditions:
    - lastTransitionTime: '2025-08-05T03:40:24Z'
      lastUpdateTime: '2025-08-05T06:44:18Z'
      message: ReplicaSet "n8n-workers-7bc7b64688" has successfully progressed.
      reason: NewReplicaSetAvailable
      status: 'True'
      type: Progressing
    - lastTransitionTime: '2025-08-05T06:47:30Z'
      lastUpdateTime: '2025-08-05T06:47:30Z'
      message: Deployment has minimum availability.
      reason: MinimumReplicasAvailable
      status: 'True'
      type: Available
  observedGeneration: 29
  readyReplicas: 2
  replicas: 2
  updatedReplicas: 2
```

## End

到目前总结就完成了 n8n 的基础部署，就可以通过 example.com/n8n 这个路径去访问部署于 k8s 集群中的 n8n 服务了。当然可能目前本文中还有些遗漏逻辑，没有测试部署完成后这个 n8n 的所有功能包括 webhook ，以及各种 api 等。 可能后续有问题还需要补充。
