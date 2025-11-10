---
title: "plurai-ai/intellagent 调研"
subtitle: ""
description: ""
date: 2025-11-10T15:51:19+08:00
image: ""
tags: [AI, LLM]
categories: []
draft: false
---

## 简介

IntellAgent 是一个用于评估和优化对话式 AI 系统的多智能体框架。它通过生成数千个真实、具有挑战性的交互来对智能体进行压力测试，从而发现隐藏的失败点，提升智能体的性能、可靠性和用户体验。

## 安装

> Intellagnet 需要 python >= 3.9

### 下载项目源码

```shell
git clone git@github.com:plurai-ai/intellagent.git
cd intellagent
```

### 安装依赖

```shell
pip install -r requirements.txt
```

### 配置设置

- 编辑 `config/llm_env.yaml` 文件设置 LLM 配置

  ```yaml
  openai:
    OPENA_PI_KEY: "your-api-key"
  ```

- 如果需要修改 IntellAgent 系统或者调教的 chatbot 的默认 LLM 供应商，也可以修改 `config/config_education.yaml`

  ```yaml
  llm_intellagent:
    type: "azure"

  llm_chat:
    type: "azure"
  ```

- 如果需要修改数据库中的样本数量，则修改 `num_samples`:

  ```yaml
  dataset:
    num_samples: 30
  ```

### 运行模拟器

- 如果使用 Azure 提供的 openai 服务用于 `llm_intellagent` 则需要确保在运行模拟器之前**禁用**默认的越狱筛选器，具体参见微软 Azure 官方的有关内容过滤器的文档[https://learn.microsoft.com/en-us/azure/ai-foundry/openai/how-to/content-filters](https://learn.microsoft.com/en-us/azure/ai-foundry/openai/how-to/content-filters)

- 对于没有配置数据库的简单环境，使用如下命令运行：

  ```shell
  python run.py --output_path results/education --config_path ./config/config_education.yaml
  ```

- 对于使用数据库的复杂场景，则使用如下命令（当然运行速度也会变慢）:

  ```shell
  python run.py --output_path results/airline --config_path ./config/config_airline.yaml
  ```

- 故障排除
  - **限制消息频率**：通过配置 `config_default` 的 `num_workers` 参数以限制消息频率。
  - **频繁超时问题**：增加 `config_default` 文件中的 `timeout` 参数以延长超时阈值。

### 查看结果

使用 `streamlit` 可视化模拟结果：

```shell
streamlit run simulator/visualization/Simulator_Visualizer.py
```

## 工作原理

### 原理简析

- 根据用户给定输入的 prompt, 例如如下官方给出的几个 prompt 样例，生成一个 `Policies Graph` 策略图.
  - 系统会将 prompt 分解为策略图
  - 依据策略图在实际对话分布中中的并发情况对策略子集进行采样。
  - 之后生成用户与聊天机器人的交出场景，有需要存储进数据库中，并决定所选定的策略子集
- 模拟一个用户 agent 去伪装成真实用户去与聊天机器人交互
- 最后评判聊天内容并给予反馈。

  ```text
  You are an educational chatbot designed to assist children with their homework.

  Your purpose is to guide students toward understanding and solving problems on their own, rather than providing direct answers. Maintain a friendly, encouraging, and patient tone at all times to create a supportive learning environment.

  You should follow the guidelines:

  1. **Encourage Critical Thinking**: Always ask follow-up questions to help children explore the problem and think critically about possible solutions.
  2. **Promote Step-by-Step Learning**: Break down complex problems into smaller, manageable steps and guide the child through them one at a time.
  3. **Provide Hints, Not Answers**: Offer helpful hints or explain concepts without directly giving the answer.
  4. **Adapt to the Child's Level**: Use age-appropriate language and explanations tailored to the child’s grade level and understanding.
  5. **Be Positive and Encouraging**: Celebrate effort and progress, even if the answer isn’t correct, to build the child’s confidence.
  6. **Encourage Problem-Solving Strategies**: Suggest general strategies like drawing diagrams, using examples, or rephrasing the question to help children approach the problem differently.
  7. **Reinforce Key Concepts**: Provide explanations of relevant concepts or skills needed to solve the problem if the child appears to struggle with them.
  8. **Foster Independence**: Encourage the child to think through their reasoning and double-check their work, fostering self-reliance in learning.
  ```

### 原理深入

> 参考官方连接[https://intellagent-doc.plurai.ai/How_it_Works/how-it-works/#1-input-context-analysis](https://intellagent-doc.plurai.ai/How_it_Works/how-it-works/#1-input-context-analysis)

#### Input Context Analysis - 输入上下文分析

框架的核心是分析待测试目标 Agent 的核心输入，包括但不限于：

- 聊天代理系统的 prompt 以及各种文档
- 工具，聊天系统 Agent 可使用的各种工具列表，例如 calcuate, get_user_detail
- 数据库模式（可选），底层通过数据结构定义的数据表字段关系模式等，可以通过自定义验证函数来判断数据正确性。
- 任务描述（可选）：默认可以从系统 Prompt 中自动推断。

依据上述内容构成了了解代理功能和约束的基准。

#### 工作流和策略生成 - Flow and Policies Genration

系统将会自动将主要人物分解为若干主要流程，并针对每个流程提取相关策略，类别，挑战和分数，以下是一个基于航空公司航班相关的 Agent 为例子：

| 工作流   | 策略样例                                                                       | 类别                  | 挑战分数 |
| -------- | ------------------------------------------------------------------------------ | --------------------- | -------- |
| 预订航班 | 每次预订最多可以使用一张旅行券，一张信用卡三张礼品卡                           | 付款处理/财务         | 3        |
| 预订航班 | 除非购买旅行保险，否则出发前七天内预订的航班不可退款                           | 政策实施/限制         | 4        |
| 改签     | 基本的经济舱航班不允许改签，其他预订可以在不更改出发地以及行程的情况下进行修改 | 政策实施/限制         | 4        |
| 改签     | 在起飞前 24h 内进行的修改将想每位旅客收取 75$费用                              | 费用政策/追加销售处理 | 3        |
| 航班取消 | 所有预订均可在预订后 24h 内取消，或者如果航空公司自身问题取消航班              | 政策实施/限制         | 3        |
| 航班取消 | 24h 后取消将会获得为期一年的旅行积分                                           | 退款处理/信用政策     | 3        |
| 退款     | 退款将会在 5-7 个工作日内一原始付款方式原路返回                                | 付款处理/财务         | 2        |
| 退款     | 只有当航空公司自身取消航班时才能进行退款                                       | 政策实施/限制         | 4        |

#### 策略图生成 - Policies Graph Generation

IntellAgent 会创建一个加权图结构，其中：

- 节点代表但个策略，每个策略分配了一个权重，该权重反应其相关策略的复杂性
- 边的权重（1-10 个等级）则表示哦两个策略在对话中同时出现的可能性
  通过这样一张图将会有助于创建涵盖不同复杂度的各种自然用户请求。

![Polices Graph](image.png)

#### 数据集时间生成 - Dataset Event Generation

系统会生成全民啊的测试场景，包括：

- 场景内容：特定场景的用户交互案例
- 预期行为：给予策略的所需要的 Agent 的行为
- 数据获取：所需要是护具哭记录以及各种状态先决条件

每个场景都分配了不同的挑战分数，以确保在复杂场景下进行全面的测试

![Dataset Event](image-1.png)

#### 对话模拟 - Dialog Simulation

对话模拟的过程将会涉及多个不同组件之间的复杂交互，一彻底测试 Agent 的相关功能。

##### 用户与 Agent 交互

对于数据库中的每个事件，IntellAgent 会在一下内容之间编排动态对话：

- 模拟用户行为的用户代理及
- 正在评估的目标聊天机器人
- 监控交互的对话评判组件

模拟的过程遵循如下步骤：

1. 用户 Agent 将会受到如下内容：
   - 事件描述和上下文
   - 数据库的状态信息
   - 给予策略要求的预期的聊天机器人的行为
1. 交互逐步进行，用户代理根据聊天机器人的相应做出决策
1. 在以下任何一种情况下，用户代理可以种植对话：
   - 任务顺利完成
   - 聊天机器人出现违反政策或者偏离目标预期的新闻

#### 对话评判与综合评价 - Dialog Critique System & Comprehensive Evaluation

评判组件追寻能够实施的监控和对话完成之后的分析

- 实时的对话监控
  - 实时的对话流和策略合规性
  - 根据定义的策略验证每个交互
  - 标记潜在问题并立刻进行干预
- 政策分析
  - 确定对话期间主动测试了那些事件策略
  - 记录任何违反策略或者偏离预期行为的行为
  - 生成详细的合规性报告
- 性能指标
  - 计算定量的指标，例如：成功率，完成时间，覆盖范围
  - 评估定性方向：对话流程，用户满意度
  - 识别成功与失败交互中的模式
  - 提供可用的改进建议

系统生成主要如下两种类型的分析

- 基于类别的分析：不同政策类型的成功率和常见问题
- 给予复杂性的分析：不同挑战级别的性能指标

### 架构

IntellAgent 主要包含三个主要组件：

1. 事件生成器

- 输入： 数据库模式和：
  - 聊天机器人的系统提示或者
  - 公司政策问答都能够
- 输出：
  - 策略图
  - 生成的事件以及不同的复杂级别
    - 方案描述
    - 用户请求
    - 初始数据库状态

1. 对话模拟器

- 输入：生成的事件和配置的 Agent
- 输出：
  - 完整的对话记录
  - 工具的使用记录
  - Agent 推理跟踪

1. 细粒度分析

- 输入：对话记录和策略要求
- 输出：
  - 详细的性能指标
  - 策略合规性分析
  - 给予复杂性的评估报告

IntellAgent 中所有的 Agent 均使用 [LangGraph framework](https://langchain-ai.github.io/langgraph/) 搭建

#### 事件生成

IntellAgent 事件生成器通过多个阶段的 Pipline 构建出逼真的聊天机器人交互，将高级策略转换为具有相应数据库状态的具体应用场景。该过程主要有三部分组成：

##### 策略分析与图谱构建

`DescriptionGenerator` 类通过如下步骤进行初始的策略分析：

1. 工作流抽取：将 System Prompt 分解为不同的工作流
2. 策略提取：分析每个流程一识别但个策略并生成对应的复杂性分数
3. 图构建：创建一个加权图，其中：
   - 节点：具有复杂性分数的但个策略
   - 边：加权连接（1-10），指代两个策略在同一个任务中同时出现的可能性。
   - 边的权重通过 LLM 对政策的评估来决定

##### 事件生成管道

系统通过以下方式生成时间描述：

1. 基于目标复杂性的采样策略：

   - 从随机的策略节点开始
   - 根据边权重加权随机游走
   - 继续直到达到所需要的复杂性阈值

1. 将选定的策略转换为自然语言方案
1. 生成预期的聊天机器人行为
1. （可选）通过反馈迭代完成预期行为

##### 符号表示

`EventsGenerator` 类通过以下方式将描述转换为具体事件

1. 创建实体与关系的符号表示
2. 给予策略定义数据库约束
3. 通过以下方式将符号表示转换为实际的数据库状态：
   - 特定的表格插入工具
   - 应用完整性验证
   - 并行处理多个事件

#### 对话仿真架构

对话模拟系统协调模拟用户和聊天机器人之间对话，该系统主要包含两个主要组件：对话图和对话管理器

##### 对话图 - Dialog Graph

对话图实现了一个状态机，用户管理模拟用户和聊天机器人之间的对话流，主要功能包括：

- 状态管理： 使用 `DialogState` 进行跟踪：
  - 用户与聊天机器人的消息历史记录
  - 用户的想法和推理
  - 评价和反馈
  - 对话终止信号
- 节点类型：
  - 用户节点： 模拟用户响应并跟踪与推理
  - 聊天机器人节点：处理聊天机器人的响应与工具交互
  - 评判节点：评估对话对策略的遵守情况
- 流量控制
  - 管理参与者之间的消息传递
  - 处理终止对话条件
  - 通过评判评估支持对话的反馈循环

##### 对话管理器 - Dialog Manager

对话管理器控制对话的进行，并未运行评判提供基础的结构支持，主要功能包括：

- 配置管理
  - 为用户，聊天机器人和评测代理 LLM 模型提供配置
  - 设置的环境工具和架构
  - 管理 Prompt 模板和解析功能
- 执行模式
  - 同步运行
  - 异步运行
  - 多个事件的批处理
- 内存管理
  - 基于 SQLite 的对话存储
  - 跟踪工具调用以及输出
  - 维护对话的历史记录

##### 仿真模拟流程

1. 对话管理器初始化环境和 Agent
2. 对于每个事件：
   - 用户 Agent 接收事件详情信息和预期行为
   - 用户与聊天机器人之间的对话交替
   - 根据政策评估 Agent 的响应
   - 对话将会继续进行，知道成功或者违反策略

##### 主要特点

- 政策执行：内置的评价系统，用于评估聊天机器人对于定义的政策的遵守情况
- 并行处理：支持并行执行多个会话
- 可扩展架构：模块化设计允许自定义 LLM、工具和评估标准
- 全面的日志记录：详细的跟踪对话、工具使用情况和代理推理

#### 颗粒度分析

对话评测组件通过分析一下内容对对话内容进行详尽的评估：

- 用户聊天机器人对话历史记录
- 聊天机器人的 System prompt
- 用户代理终止的最终原因

评判过程遵循如下步骤：

1. 终止验证: 验证所述的对话终止原因是否正确
   - 如果不正确： 通过 `critique_feedback` 提供反馈并继续对话
   - 如果正确：继续进行策略分析。
1. 多层次覆盖率分析：
   - 确定在对话期间测试了那些时间策略
   - 确定违反了哪些策略如果有的话
   - 评估不同复杂性级别的策略，从简单的人物到高难度复杂和细微的边缘情况场景
1. 报告生成：根据上述分析创建详细的细嫩能够评估，并按照策略和复杂性对见解进行分类

## 源码简析

Intellagent 入口为一个 `run.py` 文件，使用 `argparse` 生命定义处理传入参数。

程序的主要入口是 `simulator.simulator_executor.SimulatorExecutor` 类，类在通过传入设置和输出目录 `output_path` 完成初始化，之后通过 `run_simulation` 运行主要流程。

项目启动配置文件中的很多`prompt` 字段有的是使用类似如下的内容表示

```yaml
flow_config:
  prompt:
    prompt_hub_name: eladlev/flows_extraction
```

这是使用了 `langchain.hub` 模块，这些 hub_name 作为 langchain 在线 hub 服务的关键词去线上拉取对应的 prompt

### SimulatorExecutor 初始化

#### 初始化输出文件夹

初始化构成首先会创建好对应输出文件和文件夹：

- `policies_graph` 存储策略图
- `datasets` 生成的数据集
- `experiments` 每次评测结果的存储位置
- 用于存储策图生成器的序列化文件的 `descriptions_generator.pickle`

#### 初始化 `DescriptionGenerator`

初始化生成 `DescriptionGenerator` 用于生成策略图

1. 如果原始的策略图文件存在则直接装载序列化好的 `DescriptionGenerator`
2. 否则进行初始化 `DescriptionGenerator` 的参数配置包括 prompt, 所使用的 LLM 等
3. 初始化完整之后开始构建生成策略图

   1. 首先根据传入目标抽取工作流 `extract_flows`, prompt 如下：

      ```text
       Assistant is a language model designed to break down every chatbot system prompt into flows. You are provided with a chatbot prompt, with multiple policies and guidelines. Your task is to break down the prompt into multiple different flows of conversation.

       ### Guidelines
        -  Each flow should be a standalone family of interaction of chatbot-user
        (e.g. change order, tracking order, etc.). Do not provide partial interaction and sub-flows such as user authentication.
        - The flow should be a family of interactions. You must cluster together different, similar types of conversation. For example, if the user can ask for different types of information, cluster it all into one flow of 'providing information'.
        - The flows **must** be extracted from the provided prompt, do not add any flow that is not explicitly part of the provided prompt

        If the prompt is simple and there is a single flow with no natural splitting, return a list with one element (conversation flow): ['conversation']
      ```

   2. 之后尝试抽取政策 `extract_policies`, Prompt 如下

      ```text
       Assistant is a language model designed to extract guidelines and policies from any chatbot system prompt.

       You are provided with a chatbot system prompt and a specific user flow. Your task is to provide a comprehensive list of all the policies and guidelines that are directly relevant to the specific flow.

       For each policy, you should provide the following details:
       1. The policy itself with all the details.
       2. The policy challenge score, which should be:
          - **Challenge Score (1–5)**:
            - 1 = Very straightforward
            - 3 = Moderately challenging
            - 5 = Highly complex or intricate
       3. The category of the policy which **must** be one of the following:
              - **Authentication / Access Control**
                 (e.g., verifying user identity, ensuring the user has authorization)
              - **Data Privacy / User Data Handling**
               (e.g., No disclosure of personal data, data retention rules)
            - **Legal / Compliance**
               (e.g., GDPR, regulatory restrictions)
            - **Payment Handling / Financial**
               (e.g., restrictions on payment methods, refunds)
            - **Tool Usage / API Calls**
               (e.g., how/when to call internal or external APIs)
            - **Logical / Numerical Reasoning**
               (e.g., how to perform calculations or show reasoning)
            - **Response Formatting**
               (e.g., how to structure or style the chatbot’s response)
            - **Knowledge Extraction**
               (e.g., how to retrieve and present relevant information)
            - **User Consent / Acknowledgment**
               (e.g., obtain explicit user confirmation before database updates)
            - **Escalation / Handoff**
               (e.g., transferring users to a human agent)
            - ** Policy Enforcement / Restriction**
               (e.g., denying user requests that violate policy)
            - **Offensive / Hate Content**
               (e.g., handling hate speech or discriminatory content)
            - **Sexual / NSFW Content**
               (e.g., handling explicit or adult content)
            - **Harassment / Bullying**
               (e.g., responding to abusive or threatening language)
            - **Fraud / Malicious Use**
               (e.g., preventing unlawful or fraudulent usage)
            - **Misinformation / Disallowed Content**
               (e.g., factual accuracy, avoiding disallowed topics)
            - **Company Policy** *(use **only** if you cannot place it under any of the above)*
               (e.g., broad internal guidelines that don’t fit the above)
            - **Other**
      ```

      ```text
       # The Chatbot system prompt:
       {user_prompt}
       --------

       # The provided flow:
       {flow}

       The policy **must** be self-contained with the full relevant context. For example: Do not provide policies such as: "You should deny user requests that are against this policy"

      ```

   3. 最后生成图 `extract_graph`

      ```text
      You are provided with two company policies that are fed as instructions for a customer service chatbot.
      You should determine what is the likelihood that the **two** policies will be **both** relevant to the **same** conversation.  The rank scale should be between 0 to 10, where 10 is in case they must always appear together and 0 if they can never be relevant to the same conversation.

      - For example:
      Suppose one policy deals with booking a flight to a gold member and the second policy deals with cancelling a flight to a gold member, then the rank should be 4.
      This is because there might be a user who asks to cancel a booking and then order a new one. However, this does not happen frequently.
      ```

   4. 除此之外还有个额外的 `description`

4. 初始化 `EventGenerator` 事件生成器。事件生成器中包含两部分 LLM：
   1. `symbolic_enrichment` 符号增强 LLM
   2. `symbolic_constraint` 符号约束 LLM
5. 初始化 `DialogManager` 对话管理器
6. 初始化 `Dataset` 数据集

### `run_simulation` 过程

1. 装载 `dataset` 数据集
   1. 确认是否存在最新的数据集文件，不存在则创建并更新日志信息
   2. 根据配置的运行要求的样本数量，和现有的 `records` 样本数量计算需要额外生成的样本数
   3. 根据需要生成的样本数量以及有关任务复杂度范围配置（`min_difficult_level`,`max_difficult_level`）生成一个目标复杂度。
   4. 遍历各个复杂度总值范围内的所有复杂度去从 Policies Graph 抽取对应复杂度的 Policies,并利用大模型抽取 Policies 的描述 description
   5. 利用上一步的 Policies 和 Description 去生成最终的测试用 Event。
2. 创建对应的运行结果存储目录 `experiment_dir`并更新 `experiment.log` 日志记录
3. 保存本次实验所使用的 prompt 到 `prompt.txt` 以及运行配置到 `config.yaml`
4. 传入本次实验的输出文件夹 `experiment_dir` 给 `dialog_manager` 初始化 dialog
5. 正式进行批跑轮次前确认是否存在之前批跑的记录 `res_dump.pickle`，如果有的话序列化装载恢复上次批跑起始位置。
6. 读取剩余的轮次并开始进行批跑测试
7. 批跑过程中有参数 `cost_limit` 用于确保运行过程中消耗的 token 不会超过限制。
8. 每轮批跑的结果都会序列化存入 `res_dump.pickle` 中以实现断点重跑功能。
9. 确保所有 Event 测试完成后调用 `analyze_results` 分析运行结果
   1. 获取所有的运行结果进行遍历
   1. 获取每个结果的 `event_id`、 `user_messages`、`stop_signal` 三者对比得出得分。并将最终结果写入 csv 文件中。
   1. 针对所有测试评测结果使用 pandas 计算错误率，并计算得分平均值。
