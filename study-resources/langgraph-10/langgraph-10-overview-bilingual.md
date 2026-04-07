# LangGraph Overview

**Source URL:** https://docs.langchain.com/oss/python/langgraph/overview  
**整理日期:** 2026-04-07

---

Trusted by companies shaping the future of agents— including Klarna, Uber, J.P. Morgan, and more— LangGraph is a low-level orchestration framework and runtime for building, managing, and deploying long-running, stateful agents.

受到正在塑造 Agent 未来的公司信任——包括 Klarna、Uber、J.P. Morgan 等——LangGraph 是一个用于构建、管理和部署长期运行、有状态 Agent 的低级别编排框架和运行时。

LangGraph is very low-level, and focused entirely on agent **orchestration**. Before using LangGraph, we recommend you familiarize yourself with some of the components used to build agents, starting with [models](/oss/python/langchain/models) and [tools](/oss/python/langchain/tools).

LangGraph 是非常低级别的框架，完全专注于 Agent **编排**。在使用 LangGraph 之前，我们建议你熟悉用于构建 Agent 的一些组件，从[模型](/oss/python/langchain/models)和[工具](/oss/python/langchain/tools)开始。

We will commonly use [LangChain](/oss/python/langchain/overview) components throughout the documentation to integrate models and tools, but you don't need to use LangChain to use LangGraph. If you are just getting started with agents or want a higher-level abstraction, we recommend you use LangChain's [agents](/oss/python/langchain/agents) that provide prebuilt architectures for common LLM and tool-calling loops.

我们将在整个文档中经常使用 [LangChain](/oss/python/langchain/overview) 组件来集成模型和工具，但你不需要使用 LangChain 也能使用 LangGraph。如果你刚开始接触 Agent，或者想要更高级别的抽象，我们建议你使用 LangChain 的 [Agent](/oss/python/langchain/agents)，它们为常见的 LLM 和工具调用循环提供了预构建的架构。

LangGraph is focused on the underlying capabilities important for agent orchestration: durable execution, streaming, human-in-the-loop, and more.

LangGraph 专注于 Agent 编排所必需的核心底层能力：持久化执行、流式传输、人机协同等。

## Install / 安装


Then, create a simple hello world example:

然后，创建一个简单的 Hello World 示例：

```python
from langgraph.graph import StateGraph, MessagesState, START, END

def mock_llm(state: MessagesState):
    return {"messages": [{"role": "ai", "content": "hello world"}]}

graph = StateGraph(MessagesState)
graph.add_node(mock_llm)
graph.add_edge(START, "mock_llm")
graph.add_edge("mock_llm", END)
graph = graph.compile()

graph.invoke({"messages": [{"role": "user", "content": "hi!"}]})
```

## Core benefits / 核心优势


LangGraph provides low-level supporting infrastructure for *any* long-running, stateful workflow or agent. LangGraph does not abstract prompts or architecture, and provides the following central benefits:

LangGraph 为任何长期运行、有状态的工作流或 Agent 提供低级别的基础设施支持。LangGraph 不抽象提示词或架构，并提供以下核心优势：

* [Durable execution](/oss/python/langgraph/durable-execution): Build agents that persist through failures and can run for extended periods, resuming from where they left off.

  [持久化执行](/oss/python/langgraph/durable-execution)：构建能够在故障中持久化、长时间运行并从中断处恢复的 Agent。

* [Human-in-the-loop](/oss/python/langgraph/interrupts): Incorporate human oversight by inspecting and modifying agent state at any point.

  [人机协同](/oss/python/langgraph/interrupts)：通过在任意时刻检查并修改 Agent 状态来引入人类监督。

* [Comprehensive memory](/oss/python/concepts/memory): Create stateful agents with both short-term working memory for ongoing reasoning and long-term memory across sessions.

  [全面记忆](/oss/python/concepts/memory)：创建有状态的 Agent，既具备用于持续推理的短期工作记忆，也具备跨会话的长期记忆。

* [Debugging with LangSmith](/langsmith/home): Gain deep visibility into complex agent behavior with visualization tools that trace execution paths, capture state transitions, and provide detailed runtime metrics.

  [使用 LangSmith 调试](/langsmith/home)：借助可视化工具追踪执行路径、捕获状态转换并提供详细的运行时指标，从而深入了解复杂的 Agent 行为。

* [Production-ready deployment](/langsmith/deployment): Deploy sophisticated agent systems confidently with scalable infrastructure designed to handle the unique challenges of stateful, long-running workflows.

  [生产级部署](/langsmith/deployment)：利用为处理有状态、长期运行工作流的独特挑战而设计的可扩展基础设施，自信地部署复杂的 Agent 系统。

## LangGraph ecosystem / LangGraph 生态系统


While LangGraph can be used standalone, it also integrates seamlessly with any LangChain product, giving developers a full suite of tools for building agents. To improve your LLM application development, pair LangGraph with:

虽然 LangGraph 可以独立使用，但它也能与任何 LangChain 产品无缝集成，为开发者提供构建 Agent 的整套工具。要提升你的 LLM 应用开发，可以将 LangGraph 与以下工具搭配使用：

## Acknowledgements / 致谢


LangGraph is inspired by [Pregel](https://research.google/pubs/pub37252/) and [Apache Beam](https://beam.apache.org/). The public interface draws inspiration from [NetworkX](https://networkx.org/documentation/latest/). LangGraph is built by LangChain Inc, the creators of LangChain, but can be used without LangChain.

LangGraph 的灵感来自 [Pregel](https://research.google/pubs/pub37252/) 和 [Apache Beam](https://beam.apache.org/)。其公共接口借鉴了 [NetworkX](https://networkx.org/documentation/latest/)。LangGraph 由 LangChain Inc（LangChain 的创建者）构建，但可以在不使用 LangChain 的情况下独立使用。
