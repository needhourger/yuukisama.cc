---
title:          "记一次Slurm集群部署"
subtitle:       ""
description:    ""
date:           2023-07-13T00:52:36+08:00
image:          ""
tags:           []
categories:     []
draft: false
---
# 记一次Slurm集群环境部署

工作中遇到了好几次基于Slurm部署HPC计算集群的工作。因此在这里记录一下部署过程。

不过由于集群部署的复杂性，以及这是一篇事后的复盘文章，所以可能有所纰漏，还望指正。

## HPC——Slurm

有关HPC和slurm的相关介绍就不在此赘述了，贴一下相关简介链接，有兴趣的可以去看一下。
1. [HPC](https://zhuanlan.zhihu.com/p/84337096)
2. [Slurm](https://zhuanlan.zhihu.com/p/571973164)

## 环境介绍

1. 本次所有设备均使用ubuntu系统，四台服务器，其中一台作为主节点的同时作为计算节点，另外三台作为从节点（计算节点）
2. 部署slurm软件版本为23.02
3. 集群使用nfs作为统一存储服务
4. 使用Mysql作为slurm服务的存储数据库


## 部署准备

### GPU等相关配套软件调试
由于本集群将多数用于人工智能深度学习相关领域，因而会使用到GPU计算。这里后面会贴一个链接详细讲述GPU配置。

```
TODO
```

1. NVIDIA 驱动安装
2. CUDA Toolkit 安装
3. NVCC 安装
4. MPI并行计算驱动安装

## Slurm部署

### 时间同步
集群部署工作中必定有一个问题是要确保集群内所有设备节点时间一致性，保证服务器时间，时区准确。此处以ubuntu中的操作为例子（后续所有操作例子均为在ubuntu22.04 LTS版本下）

1. 在所有服务器上安装ntpdate服务
  ```
  sudo apt-get install ntpdate
  ```
2. 时间校准 
  ```
  sudo ntpkdate cn.pool.ntp.org
  ```
3. 更新硬件时间
  ```
  sudo hwclock --systohc
  ```
  
### 创建集群所需用户
集群需要用到以下linux账户，因此需要提前创建。

**注意： 不仅仅是用户名和用户组名的一致性，所有集群服务器上的所使用到的用户必须保证用户名，用户ID,组名，组ID一致**

1. munge 集群验证模块
  ```
  sudo groupadd -g 1888 munge
  sudo useradd -r -u 1888 -g 1888 -s /usr/sbin/nologin munge
  ```
2. slurm 部分slurm软件运行所需要的账户
  ```
  sudo groupadd -g 1000 slurm
  sudo useradd -r -u 1000 -g 1000 -s /usr/sbin/nologin slurm
  ```

用户id或者用户名可自定义，确保集群内所有服务器上的一致即可。

### Mysql安装
Slurm集群任务记录，账户记录等可以通过数据库存储记录，方便用户查看管理（当然数据库并不是必须的组件，slurm可以不使用数据库进行部署，即不选择安装slurmdbd）

1. 使用ubuntu软件源中的mysql安装
  ```
  sudo apt instal mysql-server
  ```
2. 启动mysql服务
  ```
  sudo systemctl start mysql #启动mysql.service
  sudo systemctl enable mysql#设置mysql.service开机自启
  ```
3. 配置mysql数据库账户密码
  ```
  ALTER USER 'root'@'localhost' IDENTIFIED BY '123456'  # 配置数据库用户密码
  ```
  简单一点可以直接使用root用户，当然如果出于安全考虑，可以使用专用的mysql用户密码，或者使用docker部署mysql服务，确保环境隔离。

4. 配置mysql数据库
  配置mysql参数确保slurmdbd服务不会出现```error: Database settings not recommended values: innodb_buffer_pool_size innodb_lock_wait_timeout```

  编辑/etc/my.cnf
  ```
  [mysqld]
  innodb_buffer_pool_size=1024M
  innodb_log_file_size=64M
  innodb_lock_wait_timeout=900
  ```  

5. 重启mysql服务
  ```
  sudo systemctl restart mysql
  ```

### Munge 服务安装
munge，slurm的相关组建等软件在ubuntu的软件源中有，但是存在一些版本问题（例如在ubuntu22中slurm的版本就比较落后），所以本文多数使用源码部署的方式。

1. 安装ubuntu编译工具
  ```
  sudo apt instal build-essential -y
  ```

2. 安装munge依赖
  ```
  sudo apt install openssl bzip2 pkgconf libssl-dev -y
  ```

3. 下载munge源码包
  ```
  git clone https://github.com/dun/munge && cd munge
  ```
  
4. 编译安装munge
  这里需要注意的是使用官方渠道下载的源码压缩包，和直接git clone的源码包在编译安装上是有步骤差异的。git clone的版本而需要使用bootstrap生产configure。

  本文使用git clone版本的源码为例子。如果是官方渠道下载的源码压缩包可以跳过接下来的一步。

5. 生成configure
  安装bootstrap所需依赖
  ```
  sudo apt install autoconf automake libtool -y
  ```

  执行bootstrap
  ```
  ./bootstrap
  ```
  如果是直接下载的官方源码包可以跳过本步骤直接到configure配置部分.

6. configure安装参数
  ```
  ./configure \
     --prefix=/usr \
     --sysconfdir=/etc \
     --localstatedir=/var \
     --runstatedir=/run
  ```
  
7. 执行安装
  ```
  make
  make check
  sudo make install
  ```

8. 启动munge服务
  
  ```
  sudo systemctl start munge
  sudo systemctl enable munge
  sudo systemctl status munge
  ```

9.  修改目录权限
修改权限为munge管理。munge:munge (如果是使用软件源安装的munge用户会自动创建,但是提前手动创建可以确保munge用户id以及组id一致性）
```
chown -R munge: /etc/munge/
chmod 400 /etc/munge/munge.key
chown -R munge: /var/lib/munge
chown -R munge: /var/run/munge # 可能不存在
chown -R munge: /var/log/munge
```
10. 生成munge key
  ```
  /usr/sbin/mungekey -f # -f 参数  用于覆盖默认生成的key
  ```

11.  测试是否成功运行
```
$ munge -n | unmunge | grep STATUS
STATUS:          Success (0)
```

12.  **在集群其他机器上部署munge服务**
  
  **无比确保所有集群节点服务器上的linux munge用户存在，且uid,gid一致，munge服务存在，且所有的/etc/munge/munge.key文件为同一个（需要将主节点的munge.key复制到其他从节点上）且文件权限正确**

### 配置hosts文件

1. 在所有slurm集群的服务器的/etc/hosts文件中添加主机名与IP的映射.

  例如在本文的例子中，将用到一个主节点以及三个从节点，分别以master,work1,work2,work3命名，都在192.168.1.1/24网段下，因此hosts文件中的内容大致如下:

  ```
  192.168.1.2 master
  192.168.1.3 work1
  192.168.1.4 work2
  192.168.1.5 work3
  ```

2. 配置hostname
  配置完hosts文件后需要将对应节点是hostname也修改为对应的名字，例如master节点的/etc/hostname文件
  ```
  master
  ```

  work1节点/etc/hostname
  ```
  work1
  ```
 
### Slurm安装

slurm服务分为如下几个组成部分，分别具有不同的作用。

1. slurmd
  基本slurm组件服务，负责slurm计算节点计算的守护节点，理论上所有的计算节点（或者说从节点）都需要运行该服务.
2. slurmctld
  slurm控制服务，是slurm主节点上所必须存在的服务，用户和其他所有从节点通讯并管理从节点。仅仅作为管理节点的主节点需要安装该服务。
3. slurmdbd
  用于和数据库沟通的服务。如果准备部署的slurm集群不考虑记录存储工作记录（即用不到到数据库功能可以缺省）
4. slurmrestd
  非必需服务，用于将slurm操作接口暴露为api供三方软件调用操作集群使用。如果搭建的集群不考虑配套三方GUI服务一类的可缺省

slurm中所罗列的几个服务其实可以都分别安装在不同的设备上，共同组成集群。不过一般情况下我们习惯性的将slurmctld,slurmdbd，slurmrestd，slurmd部署在主节点上（由于存在slurmd所以主节点也作为一个计算节点，从节点）。而其他从节点仅仅部署slurmd。

同时前面说过，ubuntu软件源中虽然存在slurm,但是其软件版本不一定是需要的版本，不一定是最新版，因此使用源码编译的方式安装slurm.

#### 下载slurm源码压缩包
```
wget https://download.schedmd.com/slurm/slurm-23.02.3.tar.bz2

# 解压
tar --bzip -x -f slurm*tar.bz2

# 进入源码目录
cd slurm*
```

#### configure安装配置
```
./configure
      --prefix=/usr
      --sysconfdir=/etc
      --localstatedir=/var
      --runstatedir=/run
      
# 执行make
make

# 执行make check
make check

# 执行安装
sudo make install
```

#### 生成slurm基础配置文件
slurm 官方提供了一个web工具用以生成web配置文件: [https://slurm.schedmd.com/configurator.html](https://slurm.schedmd.com/configurator.html)

根据你的需要修改相关的配置参数，提交后将生成的配置文件内容复制到/etc/slurm/slurm.conf中，**同时该文件应该在每一个集群内的节点中都相同，不论是主节点还是从节点**

接下来会罗列几个重要的需要修改的参数，以及后续会提供所有参数其对应的作用.

修改配置文件目录所有者权限
```
sudo chown -R slurm:slurm /etc/slurm
```
```
# Cluster Name：集群名
 ClusterName=Cluster # 集群名，任意英文和数字名字

# Control Machines：Slurmctld控制进程节点
 SlurmctldHost=master # 启动slurmctld进程的节点名，如这里的master
 BackupController=   # 冗余备份节点，可空着
 SlurmctldParameters=enable_configless # 采用无配置模式

 # Slurm User：Slurm用户
 SlurmUser=slurm # slurmctld启动时采用的用户名

 # Slurm Port Numbers：Slurm服务通信端口
 SlurmctldPort=6817 # Slurmctld服务端口，设为6817，如不设置，默认为6817号端口
 SlurmdPort=6818    # Slurmd服务端口，设为6818，如不设置，默认为6818号端口

 # State Preservation：状态保持
 StateSaveLocation=/var/spool/slurmctld # 存储slurmctld服务状态的目录，如有备份控制节点，则需要所有SlurmctldHost节点都能共享读写该目录
 SlurmdSpoolDir=/var/spool/slurmd # Slurmd服务所需要的目录，为各节点各自私有目录，不得多个slurmd节点共享

 ReturnToService=1 #设定当DOWN（失去响应）状态节点如何恢复服务，默认为0。
     # 0: 节点状态保持DOWN状态，只有当管理员明确使其恢复服务时才恢复
     # 1: 仅当由于无响应而将DOWN节点设置为DOWN状态时，才可以当有效配置注册后使DOWN节点恢复服务。如节点由于任何其它原因（内存不足、意外重启等）被设置为DOWN，其状态将不会自动更改。当节点的内存、GRES、CPU计数等等于或大于slurm.conf中配置的值时，该节点才注册为有效配置。
     # 2: 使用有效配置注册后，DOWN节点将可供使用。该节点可能因任何原因被设置为DOWN状态。当节点的内存、GRES、CPU计数等等于或大于slurm.conf 中配置的值，该节点才注册为有效配置。￼

 # Default MPI Type：默认MPI类型
 MPIDefault=None
     # MPI-PMI2: 对支持PMI2的MPI实现
     # MPI-PMIx: Exascale PMI实现
     # None: 对于大多数其它MPI，建议设置

 # Process Tracking：进程追踪，定义用于确定特定的作业所对应的进程的算法，它使用信号、杀死和记账与作业步相关联的进程
 ProctrackType=proctrack/cgroup
     # Cgroup: 采用Linux cgroup来生成作业容器并追踪进程，需要设定/etc/slurm/cgroup.conf文件
     # Cray XC: 采用Cray XC专有进程追踪
     # LinuxProc: 采用父进程IP记录，进程可以脱离Slurm控制
     # Pgid: 采用Unix进程组ID(Process Group ID)，进程如改变了其进程组ID则可以脱离Slurm控制

 # Scheduling：调度
 # DefMemPerCPU=0 # 默认每颗CPU可用内存，以MB为单位，0为不限制。如果将单个处理器分配给作业（SelectType=select/cons_res 或 SelectType=select/cons_tres），通常会使用DefMemPerCPU
 # MaxMemPerCPU=0 # 最大每颗CPU可用内存，以MB为单位，0为不限制。如果将单个处理器分配给作业（SelectType=select/cons_res 或 SelectType=select/cons_tres），通常会使用MaxMemPerCPU
 # SchedulerTimeSlice=30 # 当GANG调度启用时的时间片长度，以秒为单位
 SchedulerType=sched/backfill # 要使用的调度程序的类型。注意，slurmctld守护程序必须重新启动才能使调度程序类型的更改生效（重新配置正在运行的守护程序对此参数无效）。如果需要，可以使用scontrol命令手动更改作业优先级。可接受的类型为：
     # sched/backfill # 用于回填调度模块以增加默认FIFO调度。如这样做不会延迟任何较高优先级作业的预期启动时间，则回填调度将启动较低优先级作业。回填调度的有效性取决于用户指定的作业时间限制，否则所有作业将具有相同的时间限制，并且回填是不可能的。注意上面SchedulerParameters选项的文档。这是默认配置
     # sched/builtin # 按优先级顺序启动作业的FIFO调度程序。如队列中的任何作业无法调度，则不会调度该队列中优先级较低的作业。对于作业的一个例外是由于队列限制（如时间限制）或关闭/耗尽节点而无法运行。在这种情况下，可以启动较低优先级的作业，而不会影响较高优先级的作业。
     # sched/hold # 如果 /etc/slurm.hold 文件存在，则暂停所有新提交的作业，否则使用内置的FIFO调度程序。

 # Resource Selection：资源选择，定义作业资源（节点）选择算法
 SelectType=select/cons_tres
     # select/cons_tres: 单个的CPU核、内存、GPU及其它可追踪资源作为可消费资源（消费及分配），建议设置
     # select/cons_res: 单个的CPU核和内存作为可消费资源
     # select/cray_aries: 对于Cray系统
     # select/linear: 基于主机的作为可消费资源，不管理单个CPU等的分配

 # SelectTypeParameters：资源选择类型参数，当SelectType=select/linear时仅支持CR_ONE_TASK_PER_CORE和CR_Memory；当SelectType=select/cons_res、SelectType=select/cray_aries和SelectType=select/cons_tres时，默认采用CR_Core_Memory
 SelectTypeParameters=CR_Core_Memory
     # CR_CPU: CPU核数作为可消费资源
     # CR_Socket: 整颗CPU作为可消费资源
     # CR_Core: CPU核作为可消费资源，默认
     # CR_Memory: 内存作为可消费资源，CR_Memory假定MaxShare大于等于1
     # CR_CPU_Memory: CPU和内存作为可消费资源
     # CR_Socket_Memory: 整颗CPU和内存作为可消费资源
     # CR_Core_Memory: CPU和和内存作为可消费资源

 # Task Launch：任务启动
 TaskPlugin=task/cgroup,task/affinity #设定任务启动插件。可被用于提供节点内的资源管理（如绑定任务到特定处理器），TaskPlugin值可为:
     # task/affinity: CPU亲和支持（man srun查看其中--cpu-bind、--mem-bind和-E选项）
     # task/cgroup: 强制采用Linux控制组cgroup分配资源（man group.conf查看帮助）
     # task/none: #无任务启动动作

 # Prolog and Epilog：前处理及后处理
 # Prolog/Epilog: 完整的绝对路径，在用户作业开始前(Prolog)或结束后(Epilog)在其每个运行节点上都采用root用户执行，可用于初始化某些参数、清理作业运行后的可删除文件等
 # Prolog=/opt/bin/prolog.sh # 作业开始运行前需要执行的文件，采用root用户执行
 # Epilog=/opt/bin/epilog.sh # 作业结束运行后需要执行的文件，采用root用户执行

 # SrunProlog/Epilog # 完整的绝对路径，在用户作业步开始前(SrunProlog)或结束后(Epilog)在其每个运行节点上都被srun执行，这些参数可以被srun的--prolog和--epilog选项覆盖
 # SrunProlog=/opt/bin/srunprolog.sh # 在srun作业开始运行前需要执行的文件，采用运行srun命令的用户执行
 # SrunEpilog=/opt/bin/srunepilog.sh # 在srun作业结束运行后需要执行的文件，采用运行srun命令的用户执行

 # TaskProlog/Epilog: 绝对路径，在用户任务开始前(Prolog)和结束后(Epilog)在其每个运行节点上都采用运行作业的用户身份执行
 # TaskProlog=/opt/bin/taskprolog.sh # 作业开始运行前需要执行的文件，采用运行作业的用户执行
 # TaskEpilog=/opt/bin/taskepilog.sh # 作业结束后需要执行的文件，采用运行作业的用户执行行

 # 顺序：
    # 1. pre_launch_priv()：TaskPlugin内部函数
    # 2. pre_launch()：TaskPlugin内部函数
    # 3. TaskProlog：slurm.conf中定义的系统范围每个任务
    # 4. User prolog：作业步指定的，采用srun命令的--task-prolog参数或SLURM_TASK_PROLOG环境变量指定
    # 5. Task：作业步任务中执行
    # 6. User epilog：作业步指定的，采用srun命令的--task-epilog参数或SLURM_TASK_EPILOG环境变量指定
    # 7. TaskEpilog：slurm.conf中定义的系统范围每个任务
    # 8. post_term()：TaskPlugin内部函数

 # Event Logging：事件记录
 # Slurmctld和slurmd守护进程可以配置为采用不同级别的详细度记录，从0（不记录）到7（极度详细）
 SlurmctldDebug=info # 默认为info
 SlurmctldLogFile=/var/log/slurm/slurmctld.log # 如是空白，则记录到syslog
 SlurmdDebug=info # 默认为info
 SlurmdLogFile=/var/log/slurm/slurmd.log # 如为空白，则记录到syslog，如名字中的有字符串"%h"，则"%h"将被替换为节点名

 # Job Completion Logging：作业完成记录
 JobCompType=jobcomp/mysql
 # 指定作业完成是采用的记录机制，默认为None，可为以下值之一:
    # None: 不记录作业完成信息
    # Elasticsearch: 将作业完成信息记录到Elasticsearch服务器
    # FileTxt: 将作业完成信息记录在一个纯文本文件中
    # Lua: 利用名为jobcomp.lua的文件记录作业完成信息
    # Script: 采用任意脚本对原始作业完成信息进行处理后记录
    # MySQL: 将完成状态写入MySQL或MariaDB数据库

 # JobCompLoc= # 设定记录作业完成信息的文本文件位置（若JobCompType=filetxt），或将要运行的脚本（若JobCompType=script），或Elasticsearch服务器的URL（若JobCompType=elasticsearch），或数据库名字（JobCompType为其它值时）

 # 设定数据库在哪里运行，且如何连接
 JobCompHost=localhost # 存储作业完成信息的数据库主机名
 # JobCompPort=3306 # 存储作业完成信息的数据库服务器监听端口
 JobCompUser=root # 用于与存储作业完成信息数据库进行对话的用户名
 JobCompPass=123456 # 用于与存储作业完成信息数据库进行对话的用户密码

 # Job Accounting Gather：作业记账收集
 JobAcctGatherType=jobacct_gather/linux # Slurm记录每个作业消耗的资源，JobAcctGatherType值可为以下之一：
    # jobacct_gather/none: 不对作业记账
    # jobacct_gather/cgroup: 收集Linux cgroup信息
    # jobacct_gather/linux: 收集Linux进程表信息，建议
 JobAcctGatherFrequency=30 # 设定轮寻间隔，以秒为单位。若为-，则禁止周期性抽样

 # Job Accounting Storage：作业记账存储
 AccountingStorageType=accounting_storage/slurmdbd # 与作业记账收集一起，Slurm可以采用不同风格存储可以以许多不同的方式存储会计信息，可为以下值之一：
     # accounting_storage/none: 不记录记账信息
     # accounting_storage/slurmdbd: 将作业记账信息写入Slurm DBD数据库
 # AccountingStorageLoc: 设定文件位置或数据库名，为完整绝对路径或为数据库的数据库名，当采用slurmdb时默认为slurm_acct_db

 # 设定记账数据库信息，及如何连接
 AccountingStorageHost=localhost # 记账数据库主机名
 AccountingStoragePort=3306 # 记账数据库服务监听端口
 AccountingStorageUser=root # 记账数据库用户名
 AccountingStoragePass=123456 # 记账数据库用户密码。对于SlurmDBD，提供企业范围的身份验证，如采用于Munge守护进程，则这是应该用munge套接字socket名（/var/run/munge/global.socket.2）代替。默认不设置
 # AccountingStoreFlags= # 以逗号（,）分割的列表。选项是：
     # job_comment：在数据库中存储作业说明域
     # job_script：在数据库中存储脚本
     # job_env：存储批处理作业的环境变量
 AccountingStorageTRES=gres/gpu # 设置GPU时需要
 GresTypes=gpu # 设置GPU时需要

 # Process ID Logging：进程ID记录，定义记录守护进程的进程ID的位置
 SlurmctldPidFile=/var/run/slurmctld.pid # 存储slurmctld进程号PID的文件
 SlurmdPidFile=/var/run/slurmd.pid # 存储slurmd进程号PID的文件

 # Timers：定时器
 SlurmctldTimeout=120 # 设定备份控制器在主控制器等待多少秒后成为激活的控制器
 SlurmdTimeout=300 # Slurm控制器等待slurmd未响应请求多少秒后将该节点状态设置为DOWN
 InactiveLimit=0 # 潜伏期控制器等待srun命令响应多少秒后，将在考虑作业或作业步骤不活动并终止它之前。0表示无限长等待
 MinJobAge=300 # Slurm控制器在等待作业结束多少秒后清理其记录
 KillWait=30 # 在作业到达其时间限制前等待多少秒后在发送SIGKILLL信号之前发送TERM信号以优雅地终止
 WaitTime=0 # 在一个作业步的第一个任务结束后等待多少秒后结束所有其它任务，0表示无限长等待

 # Compute Machines：计算节点
 NodeName=work[1-3] CPUs=48 RealMemory=192000 Sockets=2 CoresPerSocket=24 ThreadsPerCore=1 Gres=gpu:3090:4 State=UNKNOWN
 NodeName=master Gres=gpu:v100:4 CPUs=40 RealMemory=385560 Sockets=2 CoresPerSocket=20 ThreadsPerCore=1 State=UNKNOWN #GPU节点例子，主要为Gres=gpu:v100:2
     # NodeName=node[1-10] # 计算节点名，node[1-10]表示为从node1、node2连续编号到node10，其余类似
     # NodeAddr=192.168.1.[1-10] # 计算节点IP
     # CPUs=48 # 节点内CPU核数，如开着超线程，则按照2倍核数计算，其值为：Sockets*CoresPerSocket*ThreadsPerCore
     # RealMemory=192000 # 节点内作业可用内存数(MB)，一般不大于free -m的输出，当启用select/cons_res插件限制内存时使用
     # Sockets=2 # 节点内CPU颗数
     # CoresPerSocket=24 # 每颗CPU核数
     # ThreadsPerCore=1 # 每核逻辑线程数，如开了超线程，则为2
     # State=UNKNOWN # 状态，是否启用，State可以为以下之一：
         # CLOUD   # 在云上存在
         # DOWN    # 节点失效，不能分配给在作业
         # DRAIN   # 节点不能分配给作业
         # FAIL    # 节点即将失效，不能接受分配新作业
         # FAILING # 节点即将失效，但上面有作业未完成，不能接收新作业
         # FUTURE  # 节点为了将来使用，当Slurm守护进程启动时设置为不存在，可以之后采用scontrol命令简单地改变其状态，而不是需要重启slurmctld守护进程。当这些节点有效后，修改slurm.conf中它们的State。在它们被设置为有效前，采用Slurm看不到它们，也尝试与其联系。
               # 动态未来节点(Dynamic Future Nodes)：
                  # slurmd启动时如有-F[<feature>]参数，将关联到一个与slurmd -C命令显示配置(sockets、cores、threads)相同的配置的FUTURE节点。节点的NodeAddr和NodeHostname从slurmd守护进程自动获取，并且当被设置为FUTURE状态后自动清除。动态未来节点在重启时保持non-FUTURE状态。利用scontrol可以将其设置为FUTURE状态。
                  # 若NodeName与slurmd的HostName映射未通过DNS更新，动态未来节点不知道在之间如何进行通信，其原因在于NodeAddr和NodeHostName未在slurm.conf被定义，而且扇出通信(fanout communication)需要通过将TreeWidth设置为一个较高的数字（如65533）来使其无效。若做了DNS映射，则可以使用cloud_dns SlurmctldParameter。
          # UNKNOWN # 节点状态未被定义，但将在节点上启动slurmd进程后设置为BUSY或IDLE，该为默认值。

 PartitionName=batch Nodes=node[1-3] Default=YES MaxTime=INFINITE State=UP
 PartitionName=master nodes=master Default=NO MaxTime=INFINTE State=UP
     # PartitionName=batch # 队列分区名
     # Nodes=node[1-10] # 节点名
     # Default=Yes # 作为默认队列，运行作业不知明队列名时采用的队列
     # MaxTime=INFINITE # 作业最大运行时间，以分钟为单位，INFINITE表示为无限制
     # State=UP # 状态，是否启用
     # Gres=gpu:v100:2 # 设置节点有两块v100 GPU卡，需要在GPU节点 /etc/slum/gres.conf 文件中有类似下面配置：
         #AutoDetect=nvml
         Name=gpu Type=v100 File=/dev/nvidia[0-1] #设置资源的名称Name是gpu，类型Type为v100，名称与类型可以任意取，但需要与其它方面配置对应，File=/dev/nvidia[0-1]指明了使用的GPU设备。
         #Name=mps Count=100
```

#### 配置GPU资源文件/etc/slurm/gres.conf

gres.conf也尽量也应当所有的集群节点也都配置相关参数（~~存疑~~）

```
# AutoDetect=nvml
Name=gpu Type=3090 File=/dev/nvidia[0-4]
#设置资源的名称Name是gpu，类型Type为3090，名称与类型可以任意取，但需要与其它方面配置对应，File=/dev/nvidia[0-1]指明了使用的GPU设备。
```

#### 配置slurmdbd

```
# Authentication info   一些munge的认证信息
AuthType=auth/munge
AuthInfo=/var/run/munge/munge.socket.2
# DebugLevel=info

# slurmDBD info        slurmdbd相关的配置信息
DbdHost=slurmmaster
DbdPort=6819
SlurmUser=root
DebugLevel=verbose
LogFile=/var/log/slurm/slurmdbd.log

# Database info        连接mysql的相关信息
StorageType=accounting_storage/mysql
StorageHost=slurmmaster
StoragePort=3306
StoragePass=123456
StorageUser=root
StorageLoc=slurm_acct_db
```


#### 启动slurm服务master节点
```
sudo systemctl start slurmd
sudo systemctl start slurmctld
sudo systemctl start slurmdbd

sudo systemctl status slurmd # 确保是运行状态
sudo systemctl status slurmctld # 确保是运行状态
sudo systemctl status slurmdbd

sudo systemctl enable slurmctld # 开机自启
sudo systemctl enable slurmd # 开机自启
sudo systemctl enable slurmdbd # 开机自启
```

#### 启动slurm从节点
```
sudo systemctl start slurmd
sudo systemctl enable slurmd
```

### sharding 配置
sharding是slurm23.04最新推出的基于nvidia 企业级GPU（例如A100）等，方便用户在一个GPU上同时平行计算多个任务（而非按照一个任务GPU独占的方式，最大化利用GPU资源）的功能。

**该功能仅支持在企业级GPU以及slurm23.04版本之后**

### 配置/etc/gres.conf
```
# Example 1 of gres.conf
# Configure four GPUs (with Sharding)
# AutoDetect=nvml
Name=gpu Type=A100 File=/dev/nvidia[0-3]
# Set gres/shard Count value to 8 on each of the 4 available GPUs
Name=shard Count=32
```

上述配合文件也可以写成如下格式,二者含义等价
```
Name=gpu Type=A100 File=/dev/nvidia0
Name=gpu Type=A100 File=/dev/nvidia1
Name=gpu Type=A100 File=/dev/nvidia2
Name=gpu Type=A100 File=/dev/nvidia3
Name=shard Count=8    File=/dev/nvidia0
Name=shard Count=8    File=/dev/nvidia1
Name=shard Count=8    File=/dev/nvidia2
Name=shard Count=8    File=/dev/nvidia3
```

#### 配置sharding
修改slurm.conf配置文件中需要分享GPU资源的节点参数
```
AccountingStorageTRES=gres/gpu,gres/shard
GresTypes=gpu,shard
NodeName=master Gres=gpu:4,shard:32
```
如上配置，将使得master节点上的四张gpu每个gpu都可以同时跑最多8个任务，也就是合计32个任务。


#### 将配置文件分发到所有主从节点
**注意：一定保证所有主从节点的slurm.conf配置文件一致性，以及gres.conf的一致性**
#### 重启所有主从节点的slurm相关服务
```
sudo systemctl restart slurmd
sudo systemctl restart slurmctld
```