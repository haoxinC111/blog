# Evaluation Best Practices | OpenAI API

- **Source URL**: https://developers.openai.com/api/docs/guides/evaluation-best-practices/
- **整理日期**: 2026-04-07

Generative AI is variable. Models sometimes produce different output from the same input, which makes traditional software testing methods insufficient for AI architectures. Evaluations (**evals**) are a way to test your AI system despite this variability.

生成式 AI 具有变异性。模型有时会对相同的输入产生不同的输出，这使得传统的软件测试方法不足以应对 AI 架构。评估（**evals**）是在这种变异性下测试 AI 系统的一种方法。

This guide provides high-level guidance on designing evals. To get started with the [Evals API](/api/docs/api-reference/evals), see [evaluating model performance](/api/docs/guides/evals).

本指南提供关于设计评估的高层次指导。要开始使用 [Evals API](/api/docs/api-reference/evals)，请参阅 [评估模型性能](/api/docs/guides/evals)。

Evals are structured tests for measuring a model's performance. They help ensure accuracy, performance, and reliability, despite the nondeterministic nature of AI systems. They're also one of the only ways to *improve* performance of an LLM-based application (through [fine-tuning](/api/docs/guides/model-optimization)).

评估是用于衡量模型性能的结构化测试。它们有助于确保准确性、性能和可靠性，尽管 AI 系统具有非确定性。它们也是提升基于 LLM 的应用程序性能的少数方法之一（通过 [微调](/api/docs/guides/model-optimization)）。

### Types of evals / 评估的类型


When you see the word "evals," it could refer to a few things:

当你看到 "evals" 这个词时，它可能指代以下几种情况：

- Industry benchmarks for comparing models in isolation, like [MMLU](https://github.com/openai/evals/blob/main/examples/mmlu.ipynb) and those listed on [HuggingFace's leaderboard](https://huggingface.co/collections/open-llm-leaderboard/the-big-benchmarks-collection-64faca6335a7fc7d4ffe974a)
- Standard numerical scores—like [ROUGE](https://aclanthology.org/W04-1013/), [BERTScore](https://arxiv.org/abs/1904.09675)—that you can use as you design evals for your use case
- Specific tests you implement to measure your LLM application's performance

- 用于单独比较模型的行业基准，例如 [MMLU](https://github.com/openai/evals/blob/main/examples/mmlu.ipynb) 以及 [HuggingFace排行榜](https://huggingface.co/collections/open-llm-leaderboard/the-big-benchmarks-collection-64faca6335a7fc7d4ffe974a) 上的那些
- 标准数值分数——例如 [ROUGE](https://aclanthology.org/W04-1013/)、[BERTScore](https://arxiv.org/abs/1904.09675)——你可以在为特定用例设计评估时使用
- 你为衡量 LLM 应用程序性能而实现的具体测试

This guide is about the third type: designing your own evals.

本指南关注的是第三种：设计你自己的评估。

### How to read evals / 如何理解评估


You'll often see numerical eval scores between 0 and 1. There's more to evals than just scores. Combine metrics with human judgment to ensure you're answering the right questions.

你经常会看到 0 到 1 之间的数值评估分数。评估不仅仅是分数。将指标与人类判断结合起来，以确保你在回答正确的问题。

**Evals tips**

**评估技巧**

- Adopt eval-driven development: Evaluate early and often. Write scoped tests at every stage.
- Design task-specific evals: Make tests reflect model capability in real-world distributions.
- Log everything: Log as you develop so you can mine your logs for good eval cases.
- Automate when possible: Structure evaluations to allow for automated scoring.
- It's a journey, not a destination: Evaluation is a continuous process.
- Maintain agreement: Use human feedback to calibrate automated scoring.

- 采用评估驱动开发：尽早并频繁地评估。在每个阶段编写有范围的测试。
- 设计任务特定的评估：让测试反映模型在真实世界分布中的能力。
- 记录一切：在开发过程中记录日志，以便从中挖掘出好的评估案例。
- 尽可能自动化：构建评估结构以支持自动评分。
- 这是一段旅程，而非终点：评估是一个持续的过程。
- 保持一致性：使用人类反馈来校准自动评分。

**Anti-patterns**

**反模式**

- Overly generic metrics: Relying solely on academic metrics like perplexity or BLEU score.
- Biased design: Creating eval datasets that don't faithfully reproduce production traffic patterns.
- Vibe-based evals: Using "it seems like it's working" as an evaluation strategy, or waiting until you ship before implementing any evals.
- Ignoring human feedback: Not calibrating your automated metrics against human evals.

- 过于通用的指标：仅依赖困惑度或 BLEU 分数等学术指标。
- 有偏见的设计：创建不能忠实再现生产流量模式的评估数据集。
- 基于感觉的评估：使用"看起来好像有用"作为评估策略，或者等到发布后才实施任何评估。
- 忽视人类反馈：不根据人类评估来校准你的自动指标。

There are a few important components of an eval workflow:

评估工作流有几个重要组成部分：

1. **Define eval objective**. What's the success criteria for the eval?
2. **Collect dataset**. Which data will help you evaluate against your objective? Consider synthetic eval data, domain-specific eval data, purchased eval data, human-curated eval data, production data, and historical data.
3. **Define eval metrics**. How will you check that the success criteria are met?
4. **Run and compare evals**. Iterate and improve model performance for your task or system.
5. **Continuously evaluate**. Set up continuous evaluation (CE) to run evals on every change, monitor your app to identify new cases of nondeterminism, and grow the eval set over time.

1. **定义评估目标**。评估的成功标准是什么？
2. **收集数据集**。哪些数据有助于你针对目标进行评估？考虑合成评估数据、领域特定评估数据、购买的评估数据、人工策划的评估数据、生产数据和历史数据。
3. **定义评估指标**。你将如何检查是否达到了成功标准？
4. **运行并比较评估**。迭代并改进模型在您的任务或系统上的性能。
5. **持续评估**。建立持续评估（CE），在每次变更时运行评估，监控应用程序以识别新的非确定性案例，并随着时间的推移扩展评估集。

Let's run through a few examples.

让我们来看几个例子。

### Example: Summarizing transcripts / 示例：转录摘要


To test your LLM-based application's ability to summarize transcripts, your eval design might be:

要测试你的基于 LLM 的应用程序总结转录内容的能力，你的评估设计可能是：

1. **Define eval objective**  
   The model should be able to compete with reference summaries for relevance and accuracy.
2. **Collect dataset**  
   Use a mix of production data (collected from user feedback on generated summaries) and datasets created by domain experts (writers) to determine a "good" summary.
3. **Define eval metrics**  
   On a held-out set of 1000 reference transcripts -> summaries, the implementation should achieve a ROUGE-L score of at least 0.40 and coherence score of at least 80% using G-Eval.
4. **Run and compare evals**  
   Use the [Evals API](/api/docs/guides/evals) to create and run evals in the OpenAI dashboard.
5. **Continuously evaluate**  
   Set up continuous evaluation (CE) to run evals on every change, monitor your app to identify new cases of nondeterminism, and grow the eval set over time.

1. **定义评估目标**
   模型应该能够在相关性和准确性方面与参考摘要竞争。
2. **收集数据集**
   混合使用生产数据（收集自用户对生成摘要的反馈）和由领域专家（撰稿人）创建的数据集，以确定什么是"好的"摘要。
3. **定义评估指标**
   在 1000 个参考转录 -> 摘要的留出集上，实现应至少达到 0.40 的 ROUGE-L 分数，并使用 G-Eval 达到至少 80% 的连贯性分数。
4. **运行并比较评估**
   使用 [Evals API](/api/docs/guides/evals) 在 OpenAI 仪表板中创建并运行评估。
5. **持续评估**
   建立持续评估（CE），在每次变更时运行评估，监控应用程序以识别新的非确定性案例，并随着时间的推移扩展评估集。

LLMs are better at discriminating between options. Therefore, evaluations should focus on tasks like pairwise comparisons, classification, or scoring against specific criteria instead of open-ended generation. Aligning evaluation methods with LLMs' strengths in comparison leads to more reliable assessments of LLM outputs or model comparisons.

大语言模型更擅长在不同选项之间进行区分。因此，评估应侧重于成对比较、分类或针对特定标准打分等任务，而不是开放式的生成。将评估方法与 LLM 的比较优势相结合，可以对 LLM 输出或模型比较做出更可靠的评估。

### Example: Q&A over docs / 示例：文档问答


To test your LLM-based application's ability to do Q&A over docs, your eval design might be:

要测试你的基于 LLM 的应用程序进行文档问答的能力，你的评估设计可能是：

1. **Define eval objective**  
   The model should be able to provide precise answers, recall context as needed to reason through user prompts, and provide an answer that satisfies the user's need.
2. **Collect dataset**  
   Use a mix of production data (collected from users' satisfaction with answers provided to their questions), hard-coded correct answers to questions created by domain experts, and historical data from logs.
3. **Define eval metrics**  
   Context recall of at least 0.85, context precision of over 0.7, and 70+% positively rated answers.
4. **Run and compare evals**  
   Use the [Evals API](/api/docs/guides/evals) to create and run evals in the OpenAI dashboard.
5. **Continuously evaluate**  
   Set up continuous evaluation (CE) to run evals on every change, monitor your app to identify new cases of nondeterminism, and grow the eval set over time.

1. **定义评估目标**
   模型应能够提供精确答案，在需要时回忆上下文以推理用户提示，并提供满足用户需求的答案。
2. **收集数据集**
   混合使用生产数据（收集自用户对所提供答案的满意度）、由领域专家创建的问题的硬编码正确答案以及日志中的历史数据。
3. **定义评估指标**
   上下文召回率至少为 0.85，上下文精确度超过 0.7，以及 70% 以上的正面评价答案。
4. **运行并比较评估**
   使用 [Evals API](/api/docs/guides/evals) 在 OpenAI 仪表板中创建并运行评估。
5. **持续评估**
   建立持续评估（CE），在每次变更时运行评估，监控应用程序以识别新的非确定性案例，并随着时间的推移扩展评估集。

When creating an eval dataset, o3 and GPT-4.1 are useful for collecting eval examples and edge cases. Consider using o3 to help you generate a diverse set of test data across various scenarios. Ensure your test data includes typical cases, edge cases, and adversarial cases. Use human expert labellers.

在创建评估数据集时，o3 和 GPT-4.1 有助于收集评估示例和边界案例。考虑使用 o3 来帮助你跨各种场景生成多样化的测试数据。确保你的测试数据包含典型案例、边界案例和对抗性案例。使用人类专家标注员。

Complexity increases as you move from simple to more complex architectures. Here are four common architecture patterns:

随着从简单架构过渡到更复杂的架构，复杂性也会增加。以下是四种常见的架构模式：

- [Single-turn model interactions](#single-turn-model-interactions)
- [Workflows](#workflow-architectures)
- [Single-agent](#single-agent-architectures)
- [Multi-agent](#multi-agent-architectures)

- [单轮模型交互](#single-turn-model-interactions)
- [工作流](#workflow-architectures)
- [单智能体](#single-agent-architectures)
- [多智能体](#multi-agent-architectures)

Read about each architecture below to identify where nondeterminism enters your system. That's where you'll want to implement evals.

阅读以下每种架构的描述，以确定非确定性进入系统的位置。这些就是你想要实施评估的地方。

### Single-turn model interactions / 单轮模型交互


In this kind of architecture, the user provides input to the model, and the model processes these inputs (along with any developer prompts provided) to generate a corresponding output.

在这种架构中，用户向模型提供输入，模型处理这些输入（以及任何提供的开发者提示）以生成相应的输出。

#### Example / 示例


As an example, consider an online retail scenario. Your system prompt instructs the model to **categorize the customer's question** into one of the following:

例如，考虑一个在线零售场景。你的系统提示指示模型将**客户的问题**分类为以下之一：

- `order_status`
- `return_policy`
- `technical_issue`
- `cancel_order`
- `other`

To ensure a consistent, efficient user experience, the model should **only return the label that matches user intent**. Let's say the customer asks, "What's the status of my order?"

为了确保一致且高效的用户体验，模型应**只返回与用户意图匹配的标签**。假设客户问："我的订单状态是什么？"

| Nondeterminism introduced | Corresponding area to evaluate | Example eval questions |
| --- | --- | --- |
| Inputs provided by the developer and user | **Instruction following**: Does the model accurately understand and act according to the provided instructions?  **Instruction following**: Does the model prioritize the system prompt over a conflicting user prompt? | Does the model stay focused on the triage task or get swayed by the user's question? |
| Outputs generated by the model | **Functional correctness**: Are the model's outputs accurate, relevant, and thorough enough to fulfill the intended task or objective? | Does the model's determination of intent correctly match the expected intent? |

| 引入的非确定性 | 对应的评估领域 | 示例评估问题 |
| --- | --- | --- |
| 开发者和用户提供的输入 | **遵循指令**：模型是否准确理解并按照提供的指令行事？**遵循指令**：模型是否将系统提示置于冲突的用户提示之上？ | 模型是否保持专注于分类任务，还是会被用户的问题所左右？ |
| 模型生成的输出 | **功能正确性**：模型的输出是否准确、相关且足够全面，以实现预期的任务或目标？ | 模型对意图的判断是否与预期意图正确匹配？ |

### Workflow architectures / 工作流架构


As you look to solve more complex problems, you'll likely transition from a single-turn model interaction to a multistep workflow that chains together several model calls. Workflows don't introduce any new elements of nondeterminism, but they involve multiple underlying model interactions, which you can evaluate in isolation.

当你寻求解决更复杂的问题时，你可能会从单轮模型交互过渡到一个将多个模型调用链接在一起的多步骤工作流。工作流不会引入任何新的非确定性元素，但它们涉及多个底层模型交互，你可以单独对这些交互进行评估。

#### Example / 示例


Take the same example as before, where the customer asks about their order status. A workflow architecture triages the customer request and routes it through a step-by-step process:

采用与之前相同的示例，客户询问他们的订单状态。工作流架构对客户请求进行分类，并将其路由到一个分步骤的过程：

1. Extracting an Order ID
2. Looking up the order details
3. Providing the order details to a model for a final response

1. 提取订单 ID
2. 查询订单详情
3. 将订单详情提供给模型以生成最终回复

Each step in this workflow has its own system prompt that the model must follow, putting all fetched data into a friendly output.

工作流中的每一步都有自己的系统提示，模型必须遵循这些提示，将所有获取的数据放入一个友好的输出中。

| Nondeterminism introduced | Corresponding area to evaluate | Example eval questions |
| --- | --- | --- |
| Inputs provided by the developer and user | **Instruction following**: Does the model accurately understand and act according to the provided instructions?  **Instruction following**: Does the model prioritize the system prompt over a conflicting user prompt? | Does the model stay focused on the triage task or get swayed by the user's question?  Does the model follow instructions to attempt to extract an Order ID?  Does the final response include the order status, estimated arrival date, and tracking number? |
| Outputs generated by the model | **Functional correctness**: Are the model's outputs are accurate, relevant, and thorough enough to fulfill the intended task or objective? | Does the model's determination of intent correctly match the expected intent?  Does the final response have the correct order status, estimated arrival date, and tracking number? |

| 引入的非确定性 | 对应的评估领域 | 示例评估问题 |
| --- | --- | --- |
| 开发者和用户提供的输入 | **遵循指令**：模型是否准确理解并按照提供的指令行事？**遵循指令**：模型是否将系统提示置于冲突的用户提示之上？ | 模型是否保持专注于分类任务，还是会被用户的问题所左右？模型是否按照指示尝试提取订单 ID？最终回复是否包含订单状态、预计到达日期和追踪号码？ |
| 模型生成的输出 | **功能正确性**：模型的输出是否准确、相关且足够全面，以实现预期的任务或目标？ | 模型对意图的判断是否与预期意图正确匹配？最终回复中的订单状态、预计到达日期和追踪号码是否正确？ |

### Single-agent architectures / 单智能体架构


Unlike workflows, agents solve unstructured problems that require flexible decision making. An agent has instructions and a set of tools and dynamically selects which tool to use. This introduces a new opportunity for nondeterminism.

与工作流不同，智能体解决需要灵活决策的非结构化问题。智能体拥有指令和一组工具，并动态选择使用哪个工具。这引入了一个新的非确定性机会。

Tools are developer defined chunks of code that the model can execute. This can range from small helper functions to API calls for existing services. For example, `check_order_status(order_id)` could be a tool, where it takes the argument `order_id` and calls an API to check the order status.

工具是开发者定义的代码块，模型可以执行这些代码。这可以从小型辅助函数到现有服务的 API 调用。例如，`check_order_status(order_id)` 可以是一个工具，它接受参数 `order_id` 并调用 API 来检查订单状态。

#### Example

#### Example / 示例

Let's adapt our customer service example to use a single agent. The agent has access to three distinct tools:

让我们调整客户服务示例以使用单个智能体。该智能体可以访问三种不同的工具：

- Order lookup tool
- Password reset tool
- Product FAQ tool

- 订单查询工具
- 密码重置工具
- 产品常见问题解答工具

When the customer asks about their order status, the agent dynamically decides to either invoke a tool or respond to the customer. For example, if the customer asks, "What is my order status?" the agent can now follow up by requesting the order ID from the customer. This helps create a more natural user experience.

当客户询问他们的订单状态时，智能体会动态决定是调用工具还是回复客户。例如，如果客户问"我的订单状态是什么？"，智能体现在可以通过向客户请求订单 ID 来进行跟进。这有助于创造更自然的用户体验。

| Nondeterminism | Corresponding area to evaluate | Example eval questions |
| --- | --- | --- |
| Inputs provided by the developer and user | **Instruction following**: Does the model accurately understand and act according to the provided instructions?  **Instruction following**: Does the model prioritize the system prompt over a conflicting user prompt? | Does the model stay focused on the triage task or get swayed by the user's question?  Does the model follow instructions to attempt to extract an Order ID? |
| Outputs generated by the model | **Functional correctness**: Are the model's outputs are accurate, relevant, and thorough enough to fulfill the intended task or objective? | Does the model's determination of intent correctly match the expected intent? |
| Tools chosen by the model | **Tool selection**: Evaluations that test whether the agent is able to select the correct tool to use.  **Data precision**: Evaluations that verify the agent calls the tool with the correct arguments. Typically these arguments are extracted from the conversation history, so the goal is to validate this extraction was correct. | When the user asks about their order status, does the model correctly recommend invoking the order lookup tool?  Does the model correctly extract the user-provided order ID to the lookup tool? |

| 非确定性 | 对应的评估领域 | 示例评估问题 |
| --- | --- | --- |
| 开发者和用户提供的输入 | **遵循指令**：模型是否准确理解并按照提供的指令行事？**遵循指令**：模型是否将系统提示置于冲突的用户提示之上？ | 模型是否保持专注于分类任务，还是会被用户的问题所左右？模型是否按照指示尝试提取订单 ID？ |
| 模型生成的输出 | **功能正确性**：模型的输出是否准确、相关且足够全面，以实现预期的任务或目标？ | 模型对意图的判断是否与预期意图正确匹配？ |
| 模型选择的工具 | **工具选择**：测试智能体是否能够选择正确工具的评估。**数据精确性**：验证智能体是否使用正确参数调用工具的评估。通常这些参数是从对话历史中提取的，因此目标是验证这次提取是否正确。 | 当用户询问他们的订单状态时，模型是否正确建议调用订单查询工具？模型是否正确提取用户提供的订单 ID 以传给查询工具？ |

### Multi-agent architectures / 多智能体架构


As you add tools and tasks to your single-agent architecture, the model may struggle to follow instructions or select the correct tool to call. Multi-agent architectures help by creating several distinct agents who specialize in different areas. This triaging and handoff among multiple agents introduces a new opportunity for nondeterminism.

当你在单智能体架构中添加更多工具和任务时，模型可能会在遵循指令或选择正确工具调用方面遇到困难。多智能体架构通过创建多个专注于不同领域的不同智能体来提供帮助。多个智能体之间的这种分类和交接引入了一个新的非确定性机会。

The decision to use a multi-agent architecture should be driven by your evals. Starting with a multi-agent architecture adds unnecessary complexity that can slow down your time to production.

是否使用多智能体架构应由你的评估结果来驱动。一开始就用多智能体架构会增加不必要的复杂性，从而减慢你的上线时间。

#### Example / 示例


Splitting the single-agent example into a multi-agent architecture, we'll have four distinct agents:

将单智能体示例拆分为多智能体架构，我们将有四个不同的智能体：

1. Triage agent
2. Order agent
3. Account management agent
4. Sales agent

1. 分类智能体
2. 订单智能体
3. 账户管理智能体
4. 销售智能体

When the customer asks about their order status, the triage agent may hand off the conversation to the order agent to look up the order. If the customer changes the topic to ask about a product, the order agent should hand the request back to the triage agent, who then hands off to the sales agent to fetch product information.

当客户询问他们的订单状态时，分类智能体可能会将对话交接给订单智能体以查询订单。如果客户转换话题询问某个产品，订单智能体应将请求交回给分类智能体，然后分类智能体再交接给销售智能体以获取产品信息。

| Nondeterminism | Corresponding area to evaluate | Example eval questions |
| --- | --- | --- |
| Inputs provided by the developer and user | **Instruction following**: Does the model accurately understand and act according to the provided instructions? **Instruction following**: Does the model prioritize the system prompt over a conflicting user prompt? | Does the model stay focused on the triage task or get swayed by the user's question? Assuming the `lookup_order` call returned, does the order agent return a tracking number and delivery date (doesn't have to be the correct one)? |
| Outputs generated by the model | **Functional correctness**: Are the model's outputs are accurate, relevant, and thorough enough to fulfill the intended task or objective? | Does the model's determination of intent correctly match the expected intent? Assuming the `lookup_order` call returned, does the order agent provide the correct tracking number and delivery date in its response?  Does the order agent follow system instructions to ask the customer their reason for requesting a return before processing the return? |
| Tools chosen by the model | **Tool selection**: Evaluations that test whether the agent is able to select the correct tool to use. **Data precision**: Evaluations that verify the agent calls the tool with the correct arguments. Typically these arguments are extracted from the conversation history, so the goal is to validate this extraction was correct. | Does the order agent correctly call the lookup order tool? Does the order agent correctly call the `refund_order` tool?  Does the order agent call the lookup order tool with the correct order ID?  Does the account agent correctly call the `reset_password` tool with the correct account ID? |
| Agent handoff | **Agent handoff accuracy**: Evaluations that test whether each agent can appropriately recognize the decision boundary for triaging to another agent | When a user asks about order status, does the triage agent correctly pass to the order agent? When the user changes the subject to talk about the latest product, does the order agent hand back control to the triage agent? |

| 非确定性 | 对应的评估领域 | 示例评估问题 |
| --- | --- | --- |
| 开发者和用户提供的输入 | **遵循指令**：模型是否准确理解并按照提供的指令行事？**遵循指令**：模型是否将系统提示置于冲突的用户提示之上？ | 模型是否保持专注于分类任务，还是会被用户的问题所左右？假设 `lookup_order` 调用已返回，订单智能体是否返回了追踪号码和 delivery date（不一定要正确）？ |
| 模型生成的输出 | **功能正确性**：模型的输出是否准确、相关且足够全面，以实现预期的任务或目标？ | 模型对意图的判断是否与预期意图正确匹配？假设 `lookup_order` 调用已返回，订单智能体是否在回复中提供了正确的追踪号码和 delivery date？订单智能体是否遵循系统提示，在处理退货之前询问客户退货原因？ |
| 模型选择的工具 | **工具选择**：测试智能体是否能够选择正确工具的评估。**数据精确性**：验证智能体是否使用正确参数调用工具的评估。通常这些参数是从对话历史中提取的，因此目标是验证这次提取是否正确。 | 订单智能体是否正确调用了订单查询工具？订单智能体是否正确调用了 `refund_order` 工具？订单智能体是否使用了正确的订单 ID 调用订单查询工具？账户智能体是否使用了正确的账户 ID 调用 `reset_password` 工具？ |
| 智能体交接 | **智能体交接准确性**：测试每个智能体是否能够适当地识别分类到另一个智能体的决策边界 | 当用户询问订单状态时，分类智能体是否正确传递给订单智能体？当用户转换话题谈论最新产品时，订单智能体是否将控制权交回给分类智能体？ |

As you design your own evals, there are several specific evaluator types to choose from. Another way to think about this is what role you want the evaluator to play.

在设计你自己的评估时，有几种具体的评估器类型可供选择。另一种思考方式是：你希望评估器扮演什么角色。

### Metric-based evals / 基于指标的评估


Quantitative evals provide a numerical score you can use to filter and rank results. They provide useful benchmarks for automated regression testing.

定量评估提供一个数值分数，你可以用来筛选和排序结果。它们为自动化回归测试提供了有用的基准。

- **Examples**: Exact match, string match, ROUGE/BLEU scoring, function call accuracy, executable evals (executed to assess functionality or behavior—e.g., text2sql)
- **Challenges**: May not be tailored to specific use cases, may miss nuance

- **示例**：精确匹配、字符串匹配、ROUGE/BLEU 评分、函数调用准确性、可执行评估（执行以评估功能或行为——例如 text2sql）
- **挑战**：可能不适合特定用例，可能忽略细微差别

### Human evals / 人类评估


Human judgment evals provide the highest quality but are slow and expensive.

人类判断评估提供最高质量，但速度慢且成本高。

- **Examples**: Skim over system outputs to get a sense of whether they look better or worse; create a randomized, blinded test in which employees, contractors, or outsourced labeling agencies judge the quality of system outputs (e.g., ranking a small set of possible outputs, or giving each a grade of 1-5)
- **Challenges**: Disagreement among human experts, expensive, slow
- **Recommendations**:
  + Conduct multiple rounds of detailed human review to refine the scorecard
  + Implement a "show rather than tell" policy by providing examples of different score levels (e.g., 1, 3, and 8 out of 10)
  + Include a pass/fail threshold in addition to the numerical score
  + A simple way to aggregate multiple reviewers is to take consensus votes

- **示例**：浏览系统输出以了解它们看起来更好还是更差；创建一个随机盲测，让员工、承包商或外包标注机构评判系统输出的质量（例如对一小部分可能的输出进行排序，或给每个打分 1-5）
- **挑战**：人类专家之间存在分歧，昂贵，速度慢
- **建议**：
  + 进行多轮详细的人工审查以完善评分卡
  + 通过提供不同分数水平的示例来实施"展示而非讲述"策略（例如 1 分、3 分和 8 分（满分 10 分））
  + 除了数值分数外，还包括通过/未通过的阈值
  + 汇总多个评审员意见的一种简单方法是采用共识投票

### LLM-as-a-judge and model graders / LLM 作为裁判和模型评分器


Using models to judge output is cheaper to run and more scalable than human evaluation. Strong LLM judges like GPT-4.1 can match both controlled and crowdsourced human preferences, achieving over 80% agreement (the same level of agreement between humans).

使用模型来判断输出比人类评估更便宜且更具可扩展性。像 GPT-4.1 这样强大的 LLM 裁判可以与受控和众包的人类偏好相匹配，达到超过 80% 的一致性（与人类之间的一致性水平相同）。

- **Examples**:
  + Pairwise comparison: Present the judge model with two responses and ask it to determine which one is better based on specific criteria
  + Single answer grading: The judge model evaluates a single response in isolation, assigning a score or rating based on predefined quality metrics
  + Reference-guided grading: Provide the judge model with a reference or "gold standard" answer, which it uses as a benchmark to evaluate the given response
- **Challenges**: Position bias (response order), verbosity bias (preferring longer responses)
- **Recommendations**:
  + Use pairwise comparison or pass/fail for more reliability
  + Use the most capable model to grade if you can (e.g., o3)—o-series models excel at auto-grading from rubics or from a collection of reference expert answers
  + Control for response lengths as LLMs bias towards longer responses in general
  + Add reasoning and chain-of-thought as reasoning before scoring improves eval performance
  + Once the LLM judge reaches a point where it's faster, cheaper, and consistently agrees with human annotations, scale up
  + Structure questions to allow for automated grading while maintaining the integrity of the task—a common approach is to reformat questions into multiple choice formats
  + Ensure eval rubrics are clear and detailed

- **示例**：
  + 成对比较：向裁判模型展示两个回复，并要求它根据特定标准判断哪个更好
  + 单答案评分：裁判模型单独评估一个回复，根据预定义的质量指标分配分数或等级
  + 参考引导评分：向裁判模型提供一个参考或"黄金标准"答案，作为评估给定回复的基准
- **挑战**：位置偏见（回复顺序）、冗长偏见（偏好更长的回复）
- **建议**：
  + 使用成对比较或通过/未通过来提高可靠性
  + 如果可能，使用最有能力的模型进行评分（例如 o3）——o 系列模型擅长根据评分标准或一组参考专家答案进行自动评分
  + 控制回复长度，因为 LLM 通常偏爱更长的回复
  + 在评分前添加推理和思维链，因为先推理可以提高评估性能
  + 一旦 LLM 裁判达到更快、更便宜且始终与人类注释一致的程度，就扩大规模
  + 构建问题结构以允许自动评分，同时保持任务的完整性——一种常见方法是将问题重新格式化为多项选择题
  + 确保评估评分标准清晰详细

No strategy is perfect. The quality of LLM-as-Judge varies depending on problem context while using expert human annotators to provide ground-truth labels is expensive and time-consuming.

没有策略是完美的。LLM 作为裁判的质量取决于问题上下文，而使用专家人类标注员提供真实标签则既昂贵又耗时。

While your evaluations should cover primary, happy-path scenarios for each architecture, real-world AI systems frequently encounter edge cases that challenge system performance. Evaluating these edge cases is important for ensuring reliability and a good user experience.

虽然你的评估应涵盖每种架构的主要正常路径场景，但现实世界的 AI 系统经常会遇到挑战系统性能的边界案例。评估这些边界案例对于确保可靠性和良好的用户体验非常重要。

We see these edge cases fall into a few buckets:

我们认为这些边界案例可以分为几类：

### Input variability / 输入变异性


Because users provide input to the model, our system must be flexible to handle the different ways our users may interact, like:

因为用户向模型提供输入，我们的系统必须足够灵活以处理用户可能采用的不同交互方式，例如：

- Non-English or multilingual inputs
- Formats other than input text (e.g., XML, JSON, Markdown, CSV)
- Input modalities (e.g., images)

- 非英语或多语言输入
- 除输入文本之外的格式（例如 XML、JSON、Markdown、CSV）
- 输入模态（例如图像）

Your evals for instruction following and functional correctness need to accommodate inputs that users might try.

你对遵循指令和功能正确性的评估需要适应用户可能尝试的输入。

### Contextual complexity / 上下文复杂性


Many LLM-based applications fail due to poor understanding of the context of the request. This context could be from the user or noise in the past conversation history.

许多基于 LLM 的应用程序失败的原因是对请求上下文的理解不佳。这个上下文可能来自用户，也可能来自过去对话历史中的噪音。

Examples include:

示例包括：

- Multiple questions or intents in a single request
- Typos and misspellings
- Short requests with minimal context (e.g., if a user just says: "returns")
- Long context or long-running conversations
- Tool calls that return data with ambiguous property names (e.g., `"on: 123"`, where "on" is the order number)
- Multiple tool calls, sometimes leading to incorrect arguments
- Multiple agent handoffs, sometimes leading to circular handoffs

- 单个请求中包含多个问题或意图
- 拼写错误和打字错误
- 上下文极短的请求（例如，如果用户只说："returns"）
- 长上下文或长时运行对话
- 返回带有歧义属性名的数据的工具调用（例如 `"on: 123"`，其中 "on" 是订单号）
- 多次工具调用，有时导致错误参数
- 多次智能体交接，有时导致循环交接

### Personalization and customization / 个性化与定制化


While AI improves UX by adapting to user-specific requests, this flexibility introduces many edge cases. Clearly define evals for use cases you want to specifically support and block:

虽然 AI 通过适应用户特定请求来改善用户体验，但这种灵活性也引入了许多边界案例。为你希望专门支持或拦截的用例明确定义评估：

- Jailbreak attempts to get the model to do something different
- Formatting requests (e.g., format as JSON, or use bullet points)
- Cases where user prompts conflict with your system prompts

- 越狱尝试，试图让模型做不同的事情
- 格式化请求（例如格式化为 JSON，或使用项目符号）
- 用户提示与你的系统提示冲突的情况

When your evals reach a level of maturity that consistently measures performance, shift to using your evals data to improve your application's performance.

当你的评估达到能够持续衡量性能的成熟度时，转而使用你的评估数据来提升应用程序的性能。

Learn more about [reinforcement fine-tuning](/api/docs/guides/reinforcement-fine-tuning) to create a data flywheel.

了解更多关于[强化微调](/api/docs/guides/reinforcement-fine-tuning)的信息，以创建数据飞轮。

For more inspiration, visit the [OpenAI Cookbook](/cookbook), which contains example code and links to third-party resources, or learn more about our tools for evals:

如需更多灵感，请访问 [OpenAI Cookbook](/cookbook)，其中包含示例代码和第三方资源链接，或了解更多关于我们的评估工具：

- [Evaluating model performance](/api/docs/guides/evals)
- [How to evaluate a summarization task](/cookbook/examples/evaluation/how_to_eval_abstractive_summarization)
- [Fine-tuning](/api/docs/guides/model-optimization)
- [Graders](/api/docs/guides/graders)
- [Evals API reference](/api/docs/api-reference/evals)
