---
title:          "初识Agentic RAG"
subtitle:       ""
description:    ""
date:           2024-09-03T10:45:00+08:00
image:          ""
tags:           ["AI"]
categories:     []
draft: false
---

## What is Agentic RAG

> Agentic RAG(Agentic Retrieval-Augmented Generation)被誉为一种新的革命性的处理信息方式，其核心在与给RAG框架注入智能以及自主性，使其能够做出自己的决定并采取行动以实现特定目标的Agent.

这样说或许非常的模糊，但是其实可以将Agentic RAG所做的事情分解开来来解释ta究竟与传统RAG的差别。

* Context is King——上下文为王

  对于传统RAG实现而言最大的限制是无法真正理解和考虑更广泛的对话上下文。而Agentic RAG被重新设计天生具有上下文感知能力。他们可以理解对话的细微差别，考虑对话历史以针对性的改变他们自身的行为——这意味着对话将会变得更加流利以及更加具有相关性。

* 智能检索策略

  过去的传统RAG智能遵循既定的规则去检索特定的知识库，而Agentic RAG所实现的智慧检索策略将会动态根据用户的查询对话内容，目前可用的检索工具（亦或者是数据源）以及上下文线索去决定最优的检索方式。

* 多智能体编排

  对于复杂的用户查询输入往往得到答案需要跨越多个文档或者是数据源。而在Agentic RAG的世界中，其拥有多个智能体（Agent），每一个都是在其负责的特定领域的专家，Agentic RAG会协作并综合他们给出的内容并给出最全面的回答。

* 主体推理

  Agentic RAG同时具备推理能力使得其回答不同于简单的检索和生成。其拥有的智能体们会对召回的数据进行评估，更正，以及质量检测，确保其给出的回答是准确可信的。

* 检测后生成

  Agentic RAG的智能体们会在生成后执行一个检测逻辑，从而确保生成内容的真实性，或者从多个生成结果中选择最佳的答案给用户。

* 适应性学习

  Agentic RAG的架构被融入了学习机制，允许其拥有的智能体们自适应地随着时间去提升他们的能力。

## Agentic RGA架构

在了解完什么手机Agentic RAG后我们开始介绍其具体采用的架构。Agentic RAG是一个只能协调者，ta接受用户的输入查询，并自主决定何时的处理方式。同时配备了一系列成套的工具，每个工具都负责特定的领域和文档数据源。这些工具作为专业化的智能体或者是作为函数而被Agentic RAG检索，调用并最终生成信息。

例如我们现在有“工具A”是用来专门负责处理财务报表，“工具B”用来处理消费者数据。Agentic RAG就可以基于用户给出的查询动态的选择和组合工具使得他可以综合多个数据源给出明确的回答。同时这些工具，数据源头也可以是多种多样的，他们可能是结构化的也可能是非结构化的，可能是数据库也可能是基于文本文档知识库，亦或者是多媒体性质的知识库。

所以当用户给出一个复杂的问题跨越多个领域或者是数据来源时，Agentic RAG会协调整个过程，决定什么工具去使用，去召回相关的信息文档，并生成一个最终生成一个量身定制的恢复。在这个过程中，Agent利用只能推理，上下文感知，以及检测后生成去确保输出结果。

当然，Agentic RAG也有更简单的表述，在实际应用上，Agentic RAG的实现可能涉及到许多额外的组成，列入语言模型，知识库，以及其他支持系统，这具体取决于实际的应用常见以及需求。

## Agentic RAG 碎碎念

之前的那些基本上是一些现有文档的翻译陈述来解释Agentic RAG的概念架构，而从个人理解而言，Agentic RAG就是对“传统RAG的一种套娃实现”。当然或许不应该这么草率的说是套娃，但是套娃似乎可以算得上是一种实现形式。

在传统RAG中，AI大模型根据用户的输入，依照给定的输入去知识库或者是别的什么数据源中召回资料，并结合资料给出回答。而对于Agentic RAG而言，ta将这个“从给定知识库召回资料”的过程变成了从特定Agent召回输出，并综合这些输出给出一个更加综合而全面的答案。也就是把传统RAG中的知识库换成一个独立的AI Agent,这个Agent可能有自己专门负责的相关知识库或者其他工具。非常类似于“再套一层”，把这种知识库，数据来源，工具AI化，Agent化，即“Agentic”。

当然这个“套娃”的过程并不是简答的套娃，最外面的最大的“俄罗斯套娃”还需要负责做出各种优化性的决策，根据用户输入决定这次问题的解决套谁，会用到哪几个Agent,并综合最后Agent们的输出。

综上所述，Agentic RAG是又一次针对传统RAG的一次改进，对传统RAG以及各类知识库应用的一次组装应用，从而再度提升AI智能的能力。

## Agentic RAG实现——LangGraph

> Building language agents as graphs

LangGraph是langchain推出的用于构建多状态多参与者的大模型应用程序。从“Building language agents as graphs”这句话可以看出其作用，将语言大模型的Agents构建成一个graphs. 有点类似计算机理论中的图论中的知识点：

将所有的工具，特定领域模型的Agent定义为这样一张图（工作流，Graph）中的节点，而节点之间的边则有决策函数决定工作流的下一步调用。

LangGraph目前有的Features：

1. Cycles and Branching: 实现Agents之间的循环以及条件调用
2. Persistence：自动保存每一步每一个Agent调用的结果，随时暂停和恢复工作流执行，以支持错误恢复，人机交互流程，以及回溯功能。
3. Human-in-the-loop： 终端工作流的执行以及批准，编辑，计划下一个操作。
4. Streaming Support：针对所有的Node节点支持流式输出
5. Integration with Langchain: 无缝集成Langchain以及LangSmith

### Install

```
pip install langchain-anthropic
```

### Example

以下是LangGraph中构建一个能够调用搜索引擎检索的Agentic RAG例子
```
from typing import Annotated, Literal, TypedDict

from langchain_core.messages import HumanMessage
from langchain_anthropic import ChatAnthropic
from langchain_core.tools import tool
from langgraph.checkpoint.memory import MemorySaver
from langgraph.graph import END, START, StateGraph, MessagesState
from langgraph.prebuilt import ToolNode


# 工具函数 实现网络搜索功能
@tool
def search(query: str):
    # 当然这个例子中并没有完整的实现，只是例子
    if "sf" in query.lower() or "san francisco" in query.lower():
        return "It's 60 degrees and foggy."
    return "It's 90 degrees and sunny."


tools = [search]

tool_node = ToolNode(tools)

# 创建所使用的模型并绑定上面的工具函数Search
model = ChatAnthropic(model="claude-3-5-sonnet-20240620", temperature=0).bind_tools(tools)

# 决策函数 决定工作流是否继续以及下一步的工作流内容
def should_continue(state: MessagesState) -> Literal["tools", END]:
    messages = state['messages']
    last_message = messages[-1]
    # 如果LLM决定调用tools工具，那么返回决策结果tools
    if last_message.tool_calls:
        return "tools"
    # 否则的话直接终止工作流
    return END


# 模型调用函数 在这个例子中即是入口也是工具函数调用后总结结果时调用的模型
def call_model(state: MessagesState):
    messages = state['messages']
    response = model.invoke(messages)
    # We return a list, because this will get added to the existing list
    return {"messages": [response]}


# 定义一个Graph 即一个完整的工作流
workflow = StateGraph(MessagesState)

# 给这个工作流添加上面定义的两个节点 在这个例子中工作会在两个节点直接循环切换
workflow.add_node("agent", call_model)
workflow.add_node("tools", tool_node)

# 设定工作流的实际入口
workflow.add_edge(START, "agent")

# 添加决策边界
workflow.add_conditional_edges(
    # 首先我们定义了入口节点，使用上面定义的agent
    "agent",
    # 接下使用上面定义的决策函数来决定接下来要调用的节点
    should_continue,
)

# 接下来在tools和agent之间在添加一条边 这意味着当tools中的工具调用后，紧接着agent会再度被调用
workflow.add_edge("tools", 'agent')

#  初始化用于存储工作流运行状态的Memory
checkpointer = MemorySaver()

# 到目前为我们完成了对Graph 工作流的定义 接下来是实例化
# 注意 通常我们不会在实例化构建工作流的时候传入Memory
app = workflow.compile(checkpointer=checkpointer)

# 使用我们实例化的工作流
final_state = app.invoke(
    {"messages": [HumanMessage(content="what is the weather in sf")]},
    config={"configurable": {"thread_id": 42}}
)
final_state["messages"][-1].content
```



## Reference

1. [https://medium.com/@bijit211987/agentic-rag-81ed8527212b](https://medium.com/@bijit211987/agentic-rag-81ed8527212b)
2. [https://langchain-ai.github.io/langgraph/](https://langchain-ai.github.io/langgraph/)