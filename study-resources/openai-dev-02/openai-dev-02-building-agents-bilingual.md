# Building Agents Track

> **Source / 原文来源**: https://developers.openai.com/tracks/building-agents/  
> **Compiled for study on / 整理日期**: 2026-04-07

## Introduction

You’ve probably heard of agents, but what does this term actually mean?

你可能听说过智能体（agents），但这个术语到底是什么意思？

Our simple definition is:

我们的简单定义是：

> An AI system that has instructions (what it *should* do), guardrails (what it *should not* do), and access to tools (what it *can* do) to take action on the user’s behalf
>
> 一种 AI 系统，它具备指令（应该做什么）、护栏（不应该做什么）以及工具访问权限（能做什么），从而代表用户采取行动。

Think of it this way: if you’re building a chatbot-like experience, where the AI system is answering questions, you can’t really call it an agent.  
If that system, however, is connected to other systems, and taking action based on the user’s input, that qualifies as an agent.  
Simple agents may use a handful of tools, and complex agentic systems may orchestrate multiple agents to work together.

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

The goal of this track is to provide you with a comprehensive overview, and invite you to dive deeper with the resources linked in each section. Some of these resources are code examples, allowing you to get started building quickly.

本轨道的目标是为你提供全面的概览，并邀请你通过每个章节链接的资源深入探索。其中一些资源是代码示例，可让你快速开始构建。

## Core concepts

The OpenAI platform provides composable primitives to build agents: **models**, **tools**, **state/memory**, and **orchestration**.

OpenAI 平台提供了可组合的原语来构建智能体：**模型**、**工具**、**状态/记忆**和**编排**。

You can build powerful agentic experiences on our stack, with help in choosing the right models, augmenting your agents with tools, using different modalities (voice, vision, etc.), and evaluating and optimizing your application.

你可以在我们的技术栈之上构建强大的智能体体验，包括选择合适的模型、用工具增强智能体、使用不同的模态（语音、视觉等），以及评估和优化你的应用。

### Choosing the right model

Depending on your use case, you might need more or less powerful models. OpenAI offers a wide range of models, from cheap and fast to very powerful models that can handle complex tasks.

根据你的用例，你可能需要更强或更弱的模型。OpenAI 提供了广泛的模型选择，从便宜快速的模型到能够处理复杂任务的非常强大的模型。

#### Reasoning vs non‑reasoning models

In late 2024, with our first reasoning model `o1`, we introduced a new concept: the ability for models to think things through before giving a final answer. That thinking is called a “chain of thought,” and it allows models to provide more accurate and reliable answers, especially when answering difficult questions.  
With reasoning, models have the ability to form hypotheses, then test and refine them before validating the final answer. This process results in higher quality outputs.

2024 年底，随着我们首款推理模型 `o1` 的发布，我们引入了一个新概念：模型在给出最终答案之前先进行思考的能力。这种思考被称为“思维链（chain of thought）”，它使模型能够提供更准确、更可靠的答案，尤其是在回答困难问题时。通过推理，模型能够形成假设，然后进行测试和修正，最后才验证最终答案。这一过程会带来更高质量的输出。

Reasoning models trade latency and cost for reliability and often have adjustable levers (e.g., reasoning effort) that influence how hard the model “thinks.” Use a reasoning model when dealing with complex tasks, like planning, math, code generation, or multi‑tool workflows.

推理模型以延迟和成本换取可靠性，通常具有可调节的杠杆（例如 reasoning effort），影响模型“思考”的深度。在处理复杂任务时（如规划、数学、代码生成或多工具工作流），应使用推理模型。

Non‑reasoning models are faster and usually cheaper, which makes them great for chatlike user experiences (with lots of back-and-forth) and simpler tasks where latency matters.

非推理模型更快，通常也更便宜，非常适合类似聊天的用户体验（有大量来回交互）以及对延迟敏感的简单任务。

#### How to choose

Always start experimenting with a flagship, multi-purpose model—for example `gpt-4.1` or the new `gpt-5` with minimal `reasoning_effort`. If your use case is simple and requires fast responses, start with `gpt-5.4`, then try `gpt-5.4-mini` or `gpt-5.4-nano` if you need lower latency and cost. If your use case however is somewhat complex, you might want to try `gpt-5.4` or a reasoning model like `o4-mini`.

始终先从旗舰级多用途模型开始实验——例如 `gpt-4.1` 或使用最小 `reasoning_effort` 的新 `gpt-5`。如果你的用例简单且需要快速响应，从 `gpt-5.4` 开始，如果需要更低的延迟和成本，再尝试 `gpt-5.4-mini` 或 `gpt-5.4-nano`。然而，如果你的用例比较复杂，你可能想尝试 `gpt-5.4` 或像 `o4-mini` 这样的推理模型。

As you try different options, be mindful of prompting strategies: you don’t prompt a reasoning model the same way you prompt a GPT model. So don’t just swap out the model name when you’re experimenting, try different prompts and see what works best—you can learn more in the evaluation section below.

在尝试不同选项时，要注意提示策略：你不能用提示 GPT 模型的方式来提示推理模型。因此，在实验时不要只更换模型名称，要尝试不同的提示，看看哪种效果最好——你可以在下面的评估部分了解更多。

If you need more horsepower, you can use more powerful models, like `o3` or `gpt-5` with high `reasoning_effort`. However, if your application presents a conversational interface, we recommend having a faster model to chat back and forth with the user, and then delegating to a more powerful model to perform specific tasks.

如果你需要更强的算力，可以使用更强大的模型，如 `o3` 或高 `reasoning_effort` 的 `gpt-5`。然而，如果你的应用具有对话界面，我们建议使用更快的模型与用户进行来回对话，然后将特定任务委托给更强大的模型。

You can refer to our models page for more information on the different models available on the OpenAI platform, including information on performance, latency, capabilities, and pricing.

你可以参考我们的模型页面，了解 OpenAI 平台上不同模型的更多信息，包括性能、延迟、能力和定价。

Reasoning models and general-purpose models respond best to different kinds of prompts, and flagship models like `gpt-5` and `gpt-4.1` follow instructions differently. Check out our prompting guide below for how to get the best out of `gpt-5`.

推理模型和通用模型对不同类型的提示响应最佳，而像 `gpt-5` 和 `gpt-4.1` 这样的旗舰模型遵循指令的方式也有所不同。请查看下面的提示指南，了解如何充分发挥 `gpt-5` 的潜力。

### Building the core logic

To get started building an agent, you have several options to choose from: We have multiple core APIs you can use to talk to our models, but our flagship API that was specifically designed for building powerful agents is the **Responses API**.

要开始构建智能体，你有几种选择：我们有多个核心 API 可以用来与模型交互，但我们专门为构建强大智能体设计的旗舰 API 是 **Responses API**。

When you’re building with the Responses API, you’re responsible for defining the core logic, and orchestrating the different parts of your application. If you want a higher level of abstraction, you can also use the **Agents SDK**, our framework to build and orchestrate agents.

使用 Responses API 构建时，你需要负责定义核心逻辑并编排应用的不同部分。如果你希望更高的抽象层次，也可以使用 **Agents SDK**，这是我们用于构建和编排智能体的框架。

Which option you choose depends on personal preference: if you want to get started quickly or build networks of agents that work together, we recommend using the **Agents SDK**. If you want to have more control over the different parts of your application, and really understand what’s going on under the hood, you can use the **Responses API**. The Agents SDK is based on the Responses API, but you can also use it with other APIs and even external model providers if you choose. Think of it as another layer on top of the core APIs that makes it easier to build agentic applications. It abstracts away the complexity, but the trade-off is that it might be harder to have fine-grained control over the core logic. The Responses API is more flexible, but building with it requires more work to get started.

选择哪种方案取决于个人偏好：如果你想快速上手或构建协同工作的智能体网络，我们推荐使用 **Agents SDK**。如果你想对应用的不同部分有更多控制权，并真正理解底层发生了什么，可以使用 **Responses API**。Agents SDK 基于 Responses API，但如果你愿意，也可以将它与其他 API 甚至外部模型提供商一起使用。可以把它看作核心 API 之上的另一层，让构建智能体应用变得更加容易。它抽象了复杂性，但代价是你可能更难对核心逻辑进行细粒度控制。Responses API 更灵活，但用它来构建需要更多的前期工作。

#### Building with the Responses API

The Responses API is our flagship core API to interact with our models. It was designed to work well with our latest models’ capabilities, notably reasoning models, and comes with a set of built-in tools to augment your agents. It’s a flexible foundation for building agentic applications.

Responses API 是我们与模型交互的旗舰核心 API。它的设计充分发挥了我们最新模型的能力，特别是推理模型，并配备了一套内置工具来增强你的智能体。它是构建智能体应用的灵活基础。

It’s also stateful by default, meaning you don’t have to manage the conversation history on your side. You can if your application requires it, but you can also rely on us to carry over the conversation history from one request to the next. This makes it easier to build conversations that handle conversation threads without having to store the full conversation state client-side. It’s especially helpful when you’re using tools that return large payloads as managing that context on your side can impact performance.

它默认是有状态的，这意味着你无需在自己的一侧管理对话历史。如果你的应用需要，你也可以自行管理，但也可以依赖我们来将对话历史从一个请求传递到下一个请求。这使得构建处理对话线程的会话变得更加容易，而无需在客户端存储完整的对话状态。当你使用返回大负载的工具时，这尤其有帮助，因为在你这一侧管理这些上下文可能会影响性能。

You can get started building with the Responses API by cloning our starter app and customizing it to your needs.

你可以通过克隆我们的入门应用并根据你的需求进行定制，开始使用 Responses API 构建。

#### Building with the Agents SDK

The Agents SDK is a lightweight framework that makes it easy to build single agents or orchestrate networks of agents.

Agents SDK 是一个轻量级框架，可以轻松构建单个智能体或编排智能体网络。

It takes care of the complexity of handling agent loops, has built-in support for guardrails (making sure your agents don’t do anything unsafe or wrong), and introduces the concept of tracing that allows to monitor your workflows. It works really well with our suite of optimization tools such as our evaluation tool, or our distillation and fine-tuning products. If you want to learn more about how to optimize your applications, you can check out our [optimization track](/tracks/model-optimization).

它负责处理智能体循环的复杂性，内置支持护栏（确保你的智能体不会做不安全或错误的事情），并引入了追踪（tracing）概念，允许你监控工作流。它与我们的优化工具套件（如评估工具，或蒸馏和微调产品）配合得非常好。如果你想了解更多关于如何优化应用的信息，可以查看我们的 [optimization track](/tracks/model-optimization)。

The Agents SDK repositories contain examples in JavaScript and Python to get started quickly, and you can learn more about it in the [Orchestration section](#orchestration) below.

Agents SDK 仓库包含 JavaScript 和 Python 示例，可帮助你快速上手，你可以在下面的 [Orchestration section](#orchestration) 中了解更多。

### Augmenting your agents with tools

Agents become useful when they can take action. And for them to be able to do that, you need to equip your agents with *tools*.

当智能体能够采取行动时，它们才会变得有用。为了让它们能够做到这一点，你需要用*工具*来装备你的智能体。

Tools are functions that your agent can call to perform specific tasks. They can be used to retrieve data, execute tasks, or even interact with external systems. You can define any tools you want and tell the models how to use them using function calling, or you can rely on our offering of built-in tools - you can find out more about the tools available to you in the next section.

工具是你的智能体可以调用以执行特定任务的函数。它们可用于检索数据、执行任务，甚至与外部系统交互。你可以定义任何想要的工具，并通过函数调用告诉模型如何使用它们，也可以依赖我们提供的内置工具——你可以在下一节了解更多可用工具的信息。

Rule of thumb: If the capability already exists as a built‑in tool, start there. Move to function calling with your own functions when you need custom logic.

经验法则：如果某项能力已经以内置工具的形式存在，就从那里开始。当你需要自定义逻辑时，再转向使用你自己的函数进行函数调用。

## Tools

Explore how you can give your agents access to tools to enable actions like retrieving data, executing tasks, and connecting to external systems.

探索如何让你的智能体访问工具，以实现检索数据、执行任务和连接外部系统等活动。

There are two types of tools:

工具有两种类型：

* Custom tools that you define yourself, that the agent can call via function calling
  你自己定义的自定义工具，智能体可以通过函数调用来调用
* Built-in tools provided by OpenAI, that you can use out-of-the-box
  OpenAI 提供的内置工具，开箱即用

### Function calling vs built‑in tools

Function calling happens in multiple steps:

函数调用分多个步骤进行：

* First, you define what functions you want the model to use and which parameters are expected
  首先，你定义希望模型使用哪些函数以及预期参数
* Once the model is aware of the functions it can call, it can decide based on the conversation to call them with the corresponding parameters
  一旦模型知道它可以调用哪些函数，它就可以根据对话决定用相应的参数调用它们
* When that happens, you need to execute the execution of the function on your side
  当这种情况发生时，你需要在你这一侧执行该函数
* You can then tell the model what the result of the function execution is by adding it to the conversation context
  然后，你可以通过将函数执行结果添加到对话上下文来告诉模型结果是什么
* The model can then use this result to generate the next response
  然后，模型可以使用这个结果来生成下一个回复

![Function calling diagram](https://cdn.openai.com/devhub/tracks/fc-diagram.png)

With built-in tools, you don’t need to handle the execution of the function on your side (except for the computer use tool, more on that below).  
When the model decides to use a built-in tool, it’s automatically executed, and the result is added to the conversation context without you having to do anything.  
In one conversation turn, you get output that already takes into account the tool result, since it’s executed on our infrastructure.

使用内置工具时，你无需在你这一侧处理函数的执行（computer use 工具除外，详见下文）。当模型决定使用内置工具时，它会自动执行，结果会自动添加到对话上下文中，你无需做任何操作。在一个对话轮次中，你得到的输出已经考虑了工具结果，因为它是在我们的基础设施上执行的。

### Built‑in tools

Built-in tools are an easy way to add capabilities to your agents, without having to build anything on your side. You can give the model access to external or internal data, the ability to generate code or images, or even the ability to use computer interfaces, with very low effort. There are a range of built-in tools you can choose from, each serving a specific purpose:

内置工具是一种简单的方式，可以为你的智能体添加能力，而无需在你这一侧构建任何东西。你可以让模型访问外部或内部数据、生成代码或图像的能力，甚至使用计算机界面的能力，而且所需的工作量非常小。你可以选择一系列内置工具，每种都有特定的用途：

* **Web search**: Search the web for up-to-date information
  **网络搜索**：在网络上搜索最新信息
* **File search**: Search across your internal knowledge base
  **文件搜索**：在你的内部知识库中搜索
* **Code interpreter**: Let the model run python code
  **代码解释器**：让模型运行 Python 代码
* **Computer use**: Let the model use computer interfaces
  **计算机使用**：让模型使用计算机界面
* **Image generation**: Generate images with our latest image generation model
  **图像生成**：用我们最新的图像生成模型生成图像
* **MCP**: Use any hosted [MCP](https://modelcontextprotocol.io/) server
  **MCP**：使用任何托管的 [MCP](https://modelcontextprotocol.io/) 服务器

Read more about each tool and how you can use them below or check out our build hour showing web search, file search, code interpreter and the MCP tool in action.

在下面阅读有关每种工具及其使用方法的更多信息，或查看我们的 build hour，了解网络搜索、文件搜索、代码解释器和 MCP 工具的实际应用。

#### Web search

LLMs know a lot about the world, but they have a cutoff date in their training data, which means they don’t know about anything that happened after that date. For example, `gpt-5` has a cutoff date of late September 2024. If you want your agents to know about recent events, you need to give them access to the web.

大语言模型对世界了解很多，但它们的训练数据有一个截止日期，这意味着它们不知道该日期之后发生的任何事情。例如，`gpt-5` 的截止日期是 2024 年 9 月下旬。如果你想让你的智能体了解最近的事件，你需要让它们能够访问网络。

With the **web search** tool, you can do this in one line of code. Simply add web search as a tool your agent can use, and that’s it.

使用 **web search** 工具，你只需一行代码即可实现。只需将网络搜索添加为智能体可以使用的工具，就行了。

#### File search

With **file search**, you can give your agent access to internal knowledge that it would not find on the web. If you have a lot of proprietary data, feeding everything into the agent’s instructions might result in poor performance. The more text you have in the input request, the slower (and more expensive) the request, and the agent could also get confused by all this information.

使用 **file search**，你可以让你的智能体访问它在网络上找不到的内部知识。如果你拥有大量专有数据，将所有内容都输入到智能体的指令中可能会导致性能不佳。输入请求中的文本越多，请求就越慢（也越贵），智能体也可能被所有这些信息搞混。

Instead, you want to retrieve just the information you need when you need it, and feed it to the agent so that it can generate relevant responses. This process is called RAG (Retrieval-Augmented Generation), and it’s a very common technique used when building AI applications. However, there are many steps involved in building a robust RAG pipeline, and many parameters you need to think about:

相反，你希望在需要时仅检索所需的信息，并将其提供给智能体，以便它能够生成相关的响应。这个过程被称为 RAG（检索增强生成），它是构建 AI 应用时非常常用的技术。然而，构建一个稳健的 RAG 流水线涉及很多步骤，你需要考虑很多参数：

1. First, you need to prepare the data to create your knowledge base. This means **pre-processing** the files that contain your knowledge, and often you’ll need to split them into smaller chunks.
   If you have very large PDF files for example, you want to chunk them into smaller pieces so that each chunk covers a specific topic.
   首先，你需要准备数据来创建知识库。这意味着对包含知识的文件进行**预处理**，而且通常需要将它们拆分成更小的块。例如，如果你有非常大的 PDF 文件，你会想将它们分成更小的片段，使每个片段涵盖一个特定主题。
2. Then, you need to **embed** the chunks and store them in a vector database. This conversion into a numerical representation is how we can later on use algorithms to find the chunks most similar to a given text.
   There are many vector databases to choose from, some are managed, some are self-hosted, but either way you would need to store the chunks somewhere.
   然后，你需要将这些块**嵌入（embed）**并存储在向量数据库中。这种转换为数值表示的方式，使我们日后可以使用算法找到与给定文本最相似的块。向量数据库有很多选择，有些是托管的，有些是自托管的，但无论如何你都需要将这些块存储在某个地方。
3. Then, when you get an input request, you need to find the right chunks to give to the model to produce the best answer. This is the **retrieval** step.
   Once again, it is not that straightforward: you might need to process the input to使搜索更相关，then you might need to “re-rank” the results you get from the vector database to make sure you pick the best.
   接下来，当你收到输入请求时，你需要找到合适的块提供给模型以生成最佳答案。这就是**检索**步骤。同样，这并不那么简单：你可能需要处理输入以使搜索更相关，然后你可能需要对从向量数据库中得到的结果进行“重新排序（re-rank）”，以确保你选出最好的结果。
4. Finally, once you have the most relevant chunks, you can include them in the context you send to the model to generate the final answer.
   最后，一旦你有了最相关的块，你可以将它们包含在你发送给模型的上下文中，以生成最终答案。

As you may have noticed, there is complexity involved with building a custom RAG pipeline, and it requires a lot of work to get right. The **file search** tool allows you to bypass that complexity and get started quickly. All you have to do is add your files to one of our managed vector stores, and we take care of the rest for you: we pre-process the files, embed them, and store them for later use.

正如你可能注意到的，构建自定义 RAG 流水线涉及复杂性，需要做很多工作才能做好。**file search** 工具让你可以绕过这种复杂性并快速起步。你所要做的就是将你的文件添加到我们的托管向量存储之一，其余的由我们来处理：我们预处理文件、嵌入它们，并存储起来供以后使用。

Then, you can add the file search tool to your application, specify which vector store to use, and that’s it: the model will automatically decide when to use it and how to produce a final response.

然后，你可以将文件搜索工具添加到你的应用中，指定使用哪个向量存储，这就完成了：模型会自动决定何时使用它以及如何生成最终响应。

#### Code interpreter

The **code interpreter** tool allows the model to come up with python code to solve a problem or answer a question, and execute it in a dedicated environment.

**代码解释器**工具允许模型想出 Python 代码来解决问题或回答问题，并在专用环境中执行它。

LLMs are great with words, but sometimes the best way to get to a result is through code, especially when there are numbers involved. That’s when **code interpreter** comes in: it combines the power of LLMs for answer generation with the deterministic nature of code execution.

大语言模型擅长处理文字，但有时获得结果的最佳方式是通过代码，尤其是涉及数字的时候。**代码解释器**就是这样一种工具：它将 LLM 生成答案的能力与代码执行的确定性结合起来。

It accepts file inputs, so for example you could provide the model with a spreadsheet export that it can manipulate and analyze through code.

它接受文件输入，因此例如你可以向模型提供一个电子表格导出文件，它可以通过代码对其进行操作和分析。

It can also generate files, for example charts or csv files that would be the output of the code execution.

它还可以生成文件，例如图表或 CSV 文件，这些将是代码执行的输出。

This can be a powerful tool for agents that need to manipulate data or perform complex analysis.

对于需要操作数据或执行复杂分析的智能体来说，这可以是一个强大的工具。

#### Computer use

The **computer use** tool allows the model to perform actions on computer interfaces like a human would. For example, it can navigate to a website, click on buttons or fill in forms.

**计算机使用（computer use）**工具允许模型像人类一样在计算机界面上执行操作。例如，它可以导航到网站、点击按钮或填写表单。

This tool works a little differently: unlike with the other built-in tools, the tool result can’t be automatically appended to the conversation history, because we need to wait for the action to be executed to see what the next step should be.

这个工具的工作方式略有不同：与其他内置工具不同，工具结果不能自动追加到对话历史中，因为我们需要等待动作执行完毕，才能知道下一步应该是什么。

So similarly to function calling, this tool call comes with parameters that define suggested actions: “click on this position”, “scroll by that amount”, etc. It is then up to you to execute the action on your environment, either a virtual computer or a browser, and then send an update in the form of a screenshot. The model can then assess what it should do next based on the visual interface, and may decide to perform another computer use call with the next action.

因此，与函数调用类似，这个工具调用带有定义建议操作的参数：“点击这个位置”、“滚动这么多距离”等等。然后由你在你的环境（无论是虚拟计算机还是浏览器）中执行该操作，然后以截图的形式发送更新。模型随后可以根据视觉界面评估下一步该做什么，并可能决定用下一个动作再次执行 computer use 调用。

![Computer use diagram](https://cdn.openai.com/devhub/tracks/cua-diagram.png)

This can be useful if your agent needs to use services that don’t necessarily have an API available, or to automate processes that would normally be done by humans.

如果你的智能体需要使用不一定有 API 的服务，或者自动化通常由人类完成的过程，这会非常有用。

#### Image generation

The **image generation** tool allows the model to generate images based on a text prompt.

**图像生成**工具允许模型根据文本提示生成图像。

It is based on our latest image generation model, **GPT-Image**, which is a state-of-the-art model for image generation with world knowledge.

它基于我们最新的图像生成模型 **GPT-Image**，这是一个具有世界知识的最先进的图像生成模型。

This is a powerful tool for agents that need to generate images within a conversation, for example to create a visual summary of the conversation, edit user-provided images, or generate and iterate on images with a lot of context.

对于需要在对话中生成图像的智能体来说，这是一个强大的工具，例如创建对话的视觉摘要、编辑用户提供的图像，或在大量上下文的基础上生成和迭代图像。

## Orchestration

**Orchestration** is the concept of handling multiple steps, tool use, handoffs between different agents, guardrails, and context. Put simply, it’s how you manage the conversation flow.

**编排（Orchestration）**是处理多个步骤、工具使用、不同智能体之间的交接、护栏和上下文的概念。简而言之，它就是管理对话流的方式。

For example, in reaction to a user input, you might need to perform multiple steps to generate a final answer, each step feeding into the next. You might also have a lot of complexity in your use case requiring a separation of concerns, and to do that you need to define multiple agents that work in concert.

例如，针对用户输入，你可能需要执行多个步骤才能生成最终答案，每一步都输入到下一步。你的用例可能也有很多复杂性，需要关注点分离，为此你需要定义多个协同工作的智能体。

If you’re building with the **Responses API**, you can manage this entirely on your side, maintaining state and context across steps, switching between models and instructions appropriately, etc. However, for orchestration we recommend relying on the **Agents SDK**, which provides a set of primitives to help you easily define networks of agents, inject guardrails, define context, and more.

如果你使用 **Responses API** 构建，你可以完全在你这一侧管理这些，跨步骤维护状态和上下文，适当地在模型和指令之间切换，等等。然而，对于编排，我们建议依赖 **Agents SDK**，它提供了一组原语，帮助你轻松定义智能体网络、注入护栏、定义上下文等等。

### Foundations of the Agents SDK

The Agents SDK uses a few core primitives:

Agents SDK 使用几个核心原语：

| Primitive | What it is |
| --- | --- |
| Agent | model + instructions + tools |
| Handoff | other agent the current agent can hand off to |
| Guardrail | policy to filter out unwanted inputs |
| Session | automatically maintains conversation history across agent runs |

| 原语 | 它是什么 |
| --- | --- |
| Agent（智能体） | 模型 + 指令 + 工具 |
| Handoff（交接） | 当前智能体可以移交到的其他智能体 |
| Guardrail（护栏） | 过滤掉不需要的输入的策略 |
| Session（会话） | 在智能体运行之间自动维护对话历史 |

Each of these primitives is an abstraction allowing you to build faster, as the complexity that comes with handling these aspects is handled for you.

这些原语中的每一个都是一种抽象，让你能够更快地构建，因为处理这些方面的复杂性已经由 SDK 代劳。

For example, the Agents SDK automatically handles:

例如，Agents SDK 自动处理：

* **Agent loop**: calling tools and executing function calls over multiple turns if needed
  **智能体循环**：如果需要，在多个回合中调用工具并执行函数调用
* **Handoffs**: switching instructions, models and available tools based on conversation state
  **交接**：根据对话状态切换指令、模型和可用工具
* **Guardrails**: running inputs through filters to stop the generation if required
  **护栏**：将输入通过过滤器运行，以便在需要时停止生成

In addition to these features, the Agents SDK has built-in support for tracing, which allows you to monitor and debug your agents workflows. Without any additional code, you can understand what happened: which tools were called, which agents were used, which guardrails were triggered, etc. This allows you to iterate on your agents quickly and efficiently.

除了这些功能之外，Agents SDK 还内置支持追踪（tracing），这让你能够监控和调试智能体工作流。无需任何额外代码，你就可以了解发生了什么：调用了哪些工具、使用了哪些智能体、触发了哪些护栏，等等。这让你能够快速高效地迭代你的智能体。

![agentic workflows](https://cdn.openai.com/devhub/tracks/diagram-19.png)

To try practical examples with the Agents SDK, check out our examples in the repositories below.

要尝试 Agents SDK 的实用示例，请查看下方仓库中的示例。

### Multi-agent collaboration

In some cases, your application might benefit from having not just one, but multiple agents working together.

在某些情况下，你的应用可能会受益于让多个智能体协同工作，而不仅仅是一个。

This shouldn’t be your go-to solution, but something you might consider if you have separate tasks that do not overlap and if for one or more of those tasks you have:

这不应该成为你的首选解决方案，但如果你有一些互不重叠的独立任务，并且对于其中一项或多项任务，你具备以下条件，那么可以考虑：

* Very complex or long instructions
  非常复杂或冗长的指令
* A lot of tools (or similar tools across tasks)
  大量工具（或跨任务的类似工具）

For example, if you have for each task several tools to retrieve, update or create data, but these actions work differently depending on the task, you don’t want to group all of these tools and give them to one agent. The agent could get confused and use the tool meant for task A when the user needs the tool for task B.

例如，如果你针对每项任务都有多个用于检索、更新或创建数据的工具，但这些操作根据任务的不同而工作方式不同，你就不希望把所有这些工具归为一组并交给一个智能体。该智能体可能会感到困惑，在用户需要任务 B 的工具时使用了任务 A 的工具。

Instead, you might want to have a separate agent for each task, and a “routing” agent that is the main interface for the user. Once the routing agent has determined which task to perform, it can hand off to the appropriate agent than can use the right tool for the task.

相反，你可能想为每个任务配备一个独立的智能体，以及一个作为用户主界面的“路由”智能体。一旦路由智能体确定了要执行哪个任务，它就可以移交给能够使用该任务正确工具的相应智能体。

Similarly, if you have a task that has very complex instructions, or that needs to use a model with high reasoning power, you might want to have a separate agent for that task that is only called when needed, and use a faster, cheaper model for the main agent.

同样，如果你有一个指令非常复杂的任务，或者需要使用高推理能力的模型，你可能想为该任务配备一个仅在需要时才调用的独立智能体，而为主要智能体使用一个更快、更便宜的模型。

### Multi‑agent collaboration

Why multiple agents instead of one mega‑prompt?

为什么要用多个智能体而不是一个巨型提示？

* **Separation of concerns**: Research vs. drafting vs. QA
  **关注点分离**：研究 vs. 起草 vs. 质量保证
* **Parallelism**: Faster end‑to‑end execution of tasks
  **并行性**：任务端到端执行更快
* **Focused evals**: Score agents differently, depending on their scoped goals
  **专注于评估**：根据各自的目标范围对不同智能体进行不同评分

Use **agent‑as‑tool** (expose one agent as a callable tool for another) and share memory keyed by `conversation_id`.

使用 **agent-as-tool**（将一个智能体作为另一个智能体可调用的工具暴露出来），并通过 `conversation_id` 共享记忆。

## Example use cases

There are many different use cases for agents, some that require a conversational interface, some where the agents are meant to be deeply integrated in an application.

智能体有许多不同的用例，有些需要对话界面，有些则需要将智能体深度集成到应用中。

For example, some agents use structured data as an input, others a simple query to trigger a series of actions before generating a final output.

例如，有些智能体使用结构化数据作为输入，有些则使用简单查询来触发一系列操作，然后生成最终输出。

Depending on the use case, you might want to optimize for different things—for example:

根据用例的不同，你可能想要针对不同的目标进行优化——例如：

* **Speed**: if the user is interacting back and forth with the agent
  **速度**：如果用户与智能体进行反复交互
* **Reliability**: if the agent is meant to tackle complex tasks and come out with a final, optimized output
  **可靠性**：如果智能体旨在处理复杂任务并得出最终优化输出
* **Cost**: if the agent is meant to be used frequently and at scale
  **成本**：如果智能体打算频繁且大规模地使用

We have compiled a few example applications below you can use as starting points, each covering different interaction patterns:

下面我们整理了一些示例应用，你可以将它们作为起点，每个都涵盖不同的交互模式：

* **Support agent**: a simple support agent built on top of the Responses API, with a “human in the loop” angle—the agent is meant to be used by a human that can accept or reject the agent’s suggestions
  **支持智能体**：一个基于 Responses API 构建的简单支持智能体，带有“人在回路（human in the loop）”的角度——该智能体供人类使用，人类可以接受或拒绝智能体的建议
* **Customer service agent**: a network of multiple agents working together to handle a customer request, built with the Agents SDK
  **客户服务智能体**：一个由多个智能体协同工作来处理客户请求的网络，使用 Agents SDK 构建
* **Frontend testing agent**: a computer using agent that requires a single user input to test a frontend application
  **前端测试智能体**：一个使用计算机的智能体，只需一个用户输入即可测试前端应用

## Best practices

When you build agents, keep in mind that they might be unpredictable—that’s the nature of LLMs.

当你构建智能体时，请记住它们可能是不可预测的——这就是大语言模型的本质。

There are a few things you can do to make your agents more reliable, but it depends on what you are building and for whom.

你可以做一些事情来提高智能体的可靠性，但这取决于你在构建什么以及为谁构建。

### User inputs

If your agent accepts user inputs, you might want to include guardrails to make sure it can’t be jailbreaked or you don’t incur costs processing irrelevant inputs. Depending on the tools you use, the level of risk you are willing to take, and the scale of your application, you can implement more or less robust guardrails. It can be as simple as something to include in your prompt (for example “don’t answer any question unrelated to X, Y or Z”) or as complex as a full-fledged multi-step guardrail system.

如果你的智能体接受用户输入，你可能想加入护栏，以确保它不会被越狱（jailbreak），或者你不会在处理无关输入上产生成本。根据你使用的工具、你愿意承担的风险水平以及你的应用规模，你可以实施更强或更弱的护栏。它可以简单到在提示中加入一句话（例如“不要回答与 X、Y 或 Z 无关的任何问题”），也可以复杂到一个成熟的多步骤护栏系统。

### Model outputs

A good practice is to use **structured outputs** whenever you want to use the model’s output as part of your application instead of simply displaying it to the user. Structured outputs are a way to constrain the model to a strict json schema, so you always know what the output shape will be.

一个好的做法是，每当你想将模型的输出作为应用的一部分使用，而不是简单地展示给用户时，就使用**结构化输出**。结构化输出是一种将模型约束到严格 JSON schema 的方式，因此你始终知道输出形状会是什么样的。

If your agent is user-facing, once again depending on the level of risk you’re comfortable with, you might want to implement output guardrails to make sure the output doesn’t break any rules (for example, if you’re a car公司，you don’t want the model to tell customers they can buy your car for $1 and it’s contractually binding).

如果你的智能体是面向用户的，同样取决于你愿意承担的风险水平，你可能想实施输出护栏，以确保输出不会违反任何规则（例如，如果你是一家汽车公司，你不希望模型告诉客户他们可以花 1 美元买你的车并且具有合同约束力）。

### Optimizing for production

If you plan to ship your agent to production, there are additional things to consider—you might want to optimize costs and latency, or monitor your agent to make sure it performs well. To learn about these topics, you can check out our [AI application development track](/tracks/ai-application-development).

如果你计划将智能体投入生产，还有一些额外的事情需要考虑——你可能想优化成本和延迟，或者监控你的智能体以确保它表现良好。要了解这些主题，可以查看我们的 [AI application development track](/tracks/ai-application-development)。

## Conclusion and next steps

In this track you:

在本轨道中，你：

* Learned about the core concepts behind agents and how to build them
  了解了智能体背后的核心概念以及如何构建它们
* Gained practical experience with the Responses API and the Agents SDK
  获得了使用 Responses API 和 Agents SDK 的实践经验
* Discovered our built-in tools offering
  发现了我们提供的内置工具
* Learned about agent orchestration and multi-agent networks
  学习了智能体编排和多智能体网络
* Explored example use cases
  探索了示例用例
* Learned about best practices to manage user inputs and model outputs
  学习了管理用户输入和模型输出的最佳实践

These are the foundations to build your own agentic applications.

这些是你构建自己智能体应用的基础。

As a next step, you can learn how to deploy them in production with our [AI application development track](/tracks/ai-application-development).

作为下一步，你可以通过我们的 [AI application development track](/tracks/ai-application-development) 学习如何将它们部署到生产中。
