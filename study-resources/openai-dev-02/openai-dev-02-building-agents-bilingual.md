# Building Agents

- Source URL: https://developers.openai.com/tracks/building-agents/
- 整理日期: 2026-04-07

## Introduction

You’ve probably heard of agents, but what does this term actually mean?
你可能听说过智能体（agents），但这个术语到底是什么意思？

Our simple definition is:
我们的简单定义是：

> An AI system that has instructions (what it *should* do), guardrails (what it *should not* do), and access to tools (what it *can* do) to take action on the user’s behalf
> 一种 AI 系统，它具备指令（应该做什么）、护栏（不应该做什么）以及工具访问权限（能做什么），从而代表用户采取行动。

Think of it this way: if you’re building a chatbot-like experience, where the AI system is answering questions, you can’t really call it an agent. If that system, however, is connected to other systems, and taking action based on the user’s input, that qualifies as an agent. Simple agents may use a handful of tools, and complex agentic systems may orchestrate multiple agents to work together.
这样来理解：如果你构建的是一个类似聊天机器人的体验，AI 系统只是回答问题，那它不能真正被称为智能体。然而，如果该系统连接了其他系统，并根据用户的输入采取行动，那它就是一个智能体。简单的智能体可能只使用少数工具，而复杂的智能体系统则可能编排多个智能体协同工作。

This learning track introduces you to the core concepts and practical steps required to build AI agents, as well as best practices to keep in mind when building these applications.
本学习轨道将介绍构建 AI 智能体所需的核心概念和实践步骤，以及构建这些应用时需要牢记的最佳实践。

### What we will cover

1. **Core concepts**: how to choose the right models, and how to build the core logic
   **核心概念**：如何选择合适的模型，以及如何构建核心逻辑
2. **Tools**: how to augment your agents with tools to enable them to retrieve data, execute tasks, and connect to external systems
   **工具**：如何通过工具增强智能体，使其能够检索数据、执行任务并连接外部系统
3. **Orchestration**: how to build multi-step flows or networks of agents
   **编排**：如何构建多步骤流程或智能体网络
4. **Example use cases**: practical implementations of different use cases
   **示例用例**：不同用例的实际实现
5. **Best practices**: how to implement guardrails, and next steps to consider
   **最佳实践**：如何实现护栏，以及需要考虑的后续步骤

The goal of this track is to provide you with a comprehensive overview, and invite you to dive deeper with the resources linked in each section.
本轨道的目标是为你提供全面的概览，并邀请你通过每个章节链接的资源深入探索。

## Core concepts

The OpenAI platform provides composable primitives to build agents: **models**, **tools**, **state/memory**, and **orchestration**.
OpenAI 平台提供了可组合的原语来构建智能体：**模型**、**工具**、**状态/记忆**和**编排**。

You can build powerful agentic experiences on our stack, with help in choosing the right models, augmenting your agents with tools, using different modalities (voice, vision, etc.), and evaluating and optimizing your application.
你可以在我们的技术栈之上构建强大的智能体体验，包括选择合适的模型、用工具增强智能体、使用不同的模态（语音、视觉等），以及评估和优化你的应用。

### Choosing the right model

Depending on your use case, you might need more or less powerful models. OpenAI offers a wide range of models, from cheap and fast to very powerful models that can handle complex tasks.
根据你的用例，你可能需要更强或更弱的模型。OpenAI 提供了广泛的模型选择，从便宜快速的模型到能够处理复杂任务的非常强大的模型。

#### Reasoning vs non‑reasoning models

In late 2024, with our first reasoning model `o1`, we introduced a new concept: the ability for models to think things through before giving a final answer. That thinking is called a “chain of thought,” and it allows models to provide more accurate and reliable answers, especially when answering difficult questions. With reasoning, models have the ability to form hypotheses, then test and refine them before validating the final answer. This process results in higher quality outputs.
2024 年底，随着我们首款推理模型 `o1` 的发布，我们引入了一个新概念：模型在给出最终答案之前先进行思考的能力。这种思考被称为“思维链（chain of thought）”，它使模型能够提供更准确、更可靠的答案，尤其是在回答困难问题时。通过推理，模型能够形成假设，然后进行测试和修正，最后才验证最终答案。这一过程会带来更高质量的输出。

Reasoning models trade latency and cost for reliability and often have adjustable levers (e.g., reasoning effort) that influence how hard the model “thinks.” Use a reasoning model when dealing with complex tasks, like planning, math, code生成, or multi‑tool workflows.
推理模型以延迟和成本换取可靠性，通常具有可调节的杠杆（例如 reasoning effort），影响模型“思考”的深度。在处理复杂任务时（如规划、数学、代码生成或多工具工作流），应使用推理模型。

Non‑reasoning models are faster and usually cheaper, which makes them great for chatlike user experiences (with lots of back-and-forth) and simpler tasks where latency matters.
非推理模型更快，通常也更便宜，非常适合类似聊天的用户体验（有大量来回交互）以及对延迟敏感的简单任务。

#### How to choose

Always start experimenting with a flagship, multi-purpose model—for example `gpt-4.1` or the new `gpt-5` with minimal `reasoning_effort`. If your use case is simple and requires fast responses, start with `gpt-5.4`, then try `gpt-5.4-mini` or `gpt-5.4-nano` if you need lower latency and cost. If your use case however is somewhat complex, you might want to try `gpt-5.4` or a reasoning model like `o4-mini`.
始终先从旗舰级多用途模型开始实验——例如 `gpt-4.1` 或使用最小 `reasoning_effort` 的新 `gpt-5`。如果你的用例简单且需要快速响应，从 `gpt-5.4` 开始，如果需要更低的延迟和成本，再尝试 `gpt-5.4-mini` 或 `gpt-5.4-nano`。然而，如果你的用例比较复杂，你可能想尝试 `gpt-5.4` 或像 `o4-mini` 这样的推理模型。

As you try different options, be mindful of prompting strategies: you don’t prompt a reasoning model the same way you prompt a GPT model.
在尝试不同选项时，要注意提示策略：你不能用提示 GPT 模型的方式来提示推理模型。

If you need more horsepower, you can use more powerful models, like `o3` or `gpt-5` with high `reasoning_effort`. However, if your application presents a conversational interface, we recommend having a faster model to chat back and forth with the user, and then delegating to a more powerful model to perform specific tasks.
如果你需要更强的算力，可以使用更强大的模型，如 `o3` 或高 `reasoning_effort` 的 `gpt-5`。然而，如果你的应用具有对话界面，我们建议使用更快的模型与用户进行来回对话，然后将特定任务委托给更强大的模型。

### Building the core logic

To get started building an agent, you have several options to choose from: We have multiple core APIs you can use to talk to our models, but our flagship API that was specifically designed for building powerful agents is the **Responses API**.
要开始构建智能体，你有几种选择：我们有多个核心 API 可以用来与模型交互，但我们专门为构建强大智能体设计的旗舰 API 是 **Responses API**。

When you’re building with the Responses API, you’re responsible for defining the core logic, and orchestrating the different parts of your application. If you want a higher level of abstraction, you can also use the **Agents SDK**, our framework to build and orchestrate agents.
使用 Responses API 构建时，你需要负责定义核心逻辑并编排应用的不同部分。如果你希望更高的抽象层次，也可以使用 **Agents SDK**，这是我们用于构建和编排智能体的框架。

If you want to get started quickly or build networks of agents that work together, we recommend using the **Agents SDK**. If you want to have more control over the different parts of your application, and really understand what’s going on under the hood, you can use the **Responses API**.
如果你想快速上手或构建协同工作的智能体网络，我们推荐使用 **Agents SDK**。如果你想对应用的不同部分有更多控制权，并真正理解底层发生了什么，可以使用 **Responses API**。

#### Building with the Responses API

The Responses API is our flagship core API to interact with our models. It was designed to work well with our latest models’ capabilities, notably reasoning models, and comes with a set of built-in tools to augment your agents. It’s a flexible foundation for building agentic applications.
Responses API 是我们与模型交互的旗舰核心 API。它的设计充分发挥了我们最新模型的能力，特别是推理模型，并配备了一套内置工具来增强你的智能体。它是构建智能体应用的灵活基础。

It’s also stateful by default, meaning you don’t have to manage the conversation history on your side.
它默认是有状态的，这意味着你无需在自己的一侧管理对话历史。

#### Building with the Agents SDK

The Agents SDK is a lightweight framework that makes it easy to build single agents or orchestrate networks of agents. It takes care of the complexity of handling agent loops, has built-in support for guardrails, and introduces the concept of tracing that allows to monitor your workflows.
Agents SDK 是一个轻量级框架，可以轻松构建单个智能体或编排智能体网络。它负责处理智能体循环的复杂性，内置支持护栏，并引入了追踪（tracing）概念，允许你监控工作流。

### Augmenting your agents with tools

Agents become useful when they can take action. And for them to be able to do that, you need to equip your agents with *tools*.
当智能体能够采取行动时，它们才会变得有用。为了让它们能够做到这一点，你需要用*工具*来装备你的智能体。
