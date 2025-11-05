---
title: "Milvus数据迁移"
subtitle: ""
description: ""
date: 2025-04-16T15:51:19+08:00
image: ""
tags: []
categories: []
draft: false
---

## Zero

目前厂子在降本增效，小组里唯一的一台开发用服务器也将要被裁撤，所以需要将其上所有的开发项目迁移到别的服务器上，其中就有 Milvus 2.x 的数据库。因为是使用 docker 容器化部署的所以一开始觉得只需要把外挂文件 down 下来再上传到新的服务器上即可，但是现实是顶着 1MB/s 的速度上传十几 G 压缩包上去之后，重新启用服务挂载目录后并没有任何卵用，数据并没有被正确的识别，翻阅官方有关迁移数据库的文档后才知道需要使用官方提供的工具.

> [milvus 数据库迁移 https://milvus.io/docs/zh/migrate_overview.md](https://milvus.io/docs/zh/migrate_overview.md) > [从 Milvus 2.3.x 迁移数据](https://milvus.io/docs/zh/from-m2x.md)

本文以 milvus 2.x 的版本为例讲述非升级时迁移 milvus 数据库文件的方案

## 使用官方的备份工具 milvus_backup

### 获取备份工具

首先从官方的迁移工具 Github 仓库下载合适的版本： [milvus-backup](https://github.com/zilliztech/milvus-backup/releases)
部分软件仓库可能包含，例如 homebrew

```shell
brew install zilliztech/tap/milvus-backup
```

工具提供两种方式使用，一种是通过 CLI 调用，另一种是通过开启 Web API，以 API 请求的方式使用

- 需要注意的是备份工具兼容的版本

  | Milvus            | Milvus-backup    |
  | ----------------- | ---------------- |
  | v2.3.0 and above  | v0.4.0 and above |
  | v2.2.9 and above  | v0.3.0 and above |
  | v2.2.0 to v2.2.8 | v0.1.0 to v0.2.2 |

### 创建备份配置

在`milvus-backup`工具同级目录创建文件夹`configs`，并在其中创建备份配置文件`backup.yaml`

> [官方配置示例文件 backup.yaml](https://raw.githubusercontent.com/zilliztech/milvus-backup/master/configs/backup.yaml)

```yaml
# configs/backup.yaml
log:
  level: info # 日志级别
  console: true # 是否打印日志到console
  file: # 日志文件路径
    rootPath: ./backup.log
milvus: # 备份及恢复时所用的milvus环境
  address: localhost
  port: 27000
  user: root
  password: milvus
minio:
  backupStorageType: local # 备份文件的类型，这里备份到本地文件
```

**默认 Minio 文件桶的名称随安装 Milvus 的方式而不同。更改 Minio 设置时，请参阅下表。**

| 字段       | Docker Compose | Helm / Milvus 操作符 |
| ---------- | -------------- | -------------------- |
| bucketName | a-bucket       | milvus-bucket        |
| rootPath   | 文件           | 文件                 |

本处因为采用 docker compose 部署，因此直接使用默认值

### 备份数据

使用如下命令创建备份，备份过程中一般情况下是不会影响 milvus 本身运行的，即通常情况下备份和还原期间 milvus 实例不受影响

```shell
./milvus-backup create -n <backup_name>
```

### 恢复数据

在将备份数据拷贝到目标目标服务器后，创建上述备份配置`configs/backup.yaml`和下载对应的备份工具`milvus-backup`，使用如下命令即可还原

```shell
./milvus-backup restore -n my_backup
```

默认会直接恢复创建同名的 collection， 如果希望不覆盖现有恢复环境原有的 collection 可以使用`-s _recover`参数，这样所有恢复创建的 collection 将会加上一个`_recover`的后缀

```shell
./milvus-backup restore -n my_backup -s _recover
```

### Web API 模式

`milvus-backup`工具也提供通过 Web API 的方式创建备份和恢复数据

使用如下命令启动 HTTP 服务模式提供 RESTAPI

```shell
./milvus-backup server -p 8080
```

同时可以通过如下接口访问其 Swagger UI

```plaintext
http://localhost:8080/api/v1/docs/index.html
```

具体的 Web API 操作就不在这里赘述了，需要的话可以参考官方[milvus-backup 项目 README.md](https://github.com/zilliztech/milvus-backup/)

## 使用官方迁移工具 milvus_migrate

### 获取迁移工具

首先从官方迁移工具的 Github 代码仓库下载最新版本的可执行文件压缩包[milvus-migration](http://github.com/zilliztech/milvus-migration/tags)

当然也可以选择下载源码编译可执行文件，使用 Go 语言编译器编译项目得到可执行文件

```
git clone https://github.com/zilliztech/milvus-migration.git

cd milvus-migration

go get & go build
```

### 创建迁移配置文件

在开始迁移之前，需要为迁移工具以及此次迁移操作创建一个对应的迁移配置文件 `migration.yaml`，文件可以被防止在数据源机器的任意目录下。

```
dumper:
  worker:
    workMode: milvus2x
    reader:
      bufferSize: 500

meta:
  mode: config
  version: 2.3.0
  collection: src_table_name

source:
  milvus2x:
    endpoint: {milvus2x_domain}:{milvus2x_port}
    username: xxxx
    password: xxxxx

target:
  milvus2x:
    endpoint: {milvus2x_domain}:{milvus2x_port}
    username: xxxx
    password: xxxxx
```

- dumper

  | 参数                            | 说明                                                    |
  | ------------------------------- | ------------------------------------------------------- |
  | dumper.worker.workMode          | 迁移任务的操作符。从 Milvus 2.x 迁移时设置为 milvus2x。 |
  | dumper.worker.reader.bufferSize | 每批从 Milvus 2.x 读取的缓冲区大小。                    |

- meta

  | 参数            | 说明                                                                                  |
  | --------------- | ------------------------------------------------------------------------------------- |
  | meta.mode       | 指定从何处读取元文件。设置为 config，表示元配置可以从这个 migration.yaml 文件中获取。 |
  | meta.version    | 源 Milvus 版本。设置为 2.3.0 或更高版本。                                             |
  | meta.collection | 源 Collections 名称。                                                                 |

- source

  | 参数                     | 说明                                                                                                                |
  | ------------------------ | ------------------------------------------------------------------------------------------------------------------- |
  | source.milvus2x.endpoint | 源 Milvus 服务器地址。                                                                                              |
  | source.milvus2x.username | Milvus 源服务器的用户名。如果 Milvus 服务器启用了用户身份验证，则需要使用此参数。有关详细信息，请参阅启用身份验证。 |
  | source.milvus2x.password | 源 Milvus 服务器的密码。如果 Milvus 服务器启用了用户身份验证，则需要使用此参数。有关更多信息，请参阅启用身份验证。  |

- target

  | 参数                     | 说明                                                                                                                   |
  | ------------------------ | ---------------------------------------------------------------------------------------------------------------------- |
  | target.milvus2x.endpoint | 目标 Milvus 服务器地址。                                                                                               |
  | target.milvus2x.username | 目标 Milvus 服务器的用户名。如果 Milvus 服务器启用了用户身份验证，则需要使用此参数。有关详细信息，请参阅启用身份验证。 |
  | target.milvus2x.password | 目标 Milvus 服务器的密码。如果 Milvus 服务器启用了用户身份验证，则需要使用此参数。更多信息，请参阅启用身份验证。       |

### 启动迁移任务

1. 使用 CLI 启动任务
   使用如下命令启动迁移任务

```shell
./milvus-migration start --config=/example/path/migration.yaml
```

查看日志的输出，成功迁移可以看到如下内容

```plaintext
[INFO] [migration/milvus2x_starter.go:79] ["=================>JobProcess!"] [Percent=100]
[INFO] [migration/milvus2x_starter.go:27] ["[Starter] migration Milvus2x to Milvus2x finish!!!"] [Cost=94.877717375]
[INFO] [starter/starter.go:109] ["[Starter] Migration Success!"] [Cost=94.878243583]
```

1. 使用 API 请求执行迁移

使用如下命令启动 API 服务器

```shell
./milvus-migration server run -p 8080
```

服务启动后，将`migration.yaml`文件放到项目的`configs/`目录下，然后使用如下请求开始迁移

```shell
curl -XPOST http://localhost:8080/api/v1/start
```

### 其他配置选项

除了基本配置外，还可以根据需求添加其他设置

- 选择性字段迁移: 如果只需要迁移 Collections 中的特定字段而不是所有字段，请在 migration.yaml 文件的 meta 部分指定要迁移的字段。

```plaintext
meta:
  fields:
    - name: id
    - name: title_vector
    - name: reading_time
```

- 自定义 collection： 要自定义目标 Collections 的属性，请在 migration.yaml 文件的 meta 部分添加相关配置。

```plaintext
meta:
  milvus:
    collection: target_collection_name
    shardNum: 2
    closeDynamicField: false
    consistencyLevel: Customized
```

### 后记

这个方法是官方提供的升级迁移数据，附带了数据筛选功能，以及具有的限制是源头服务器和目标服务器以及操作者本身的设备三者之间必须能够互相通讯。如果不满足上述条件或者只是单纯需要备份数据，可以使用 backup 工具

## End

所以最后看来上述两种备份工具的原理基本都属于模拟所有历史数据变更，将历史数据变更重新在新数据库上演绎一遍,属于“重放日志”式的备份恢复？这里仅仅是个人通过对备份文件大致结构以及备份还原过程观察得出的结论，若有纰漏还望指正。
