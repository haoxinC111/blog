# Safety in Building Agents | OpenAI API

> **Source / 原文来源**: https://developers.openai.com/api/docs/guides/agent-builder-safety/  
> **Compiled for study on / 整理日期**: 2026-04-07

As you build and deploy agents with [Agent Builder](/api/docs/guides/agent-builder), it’s important to understand the risks. Learn about risk types and how to mitigate them when building multi-agent workflows.

当你使用 [Agent Builder](/api/docs/guides/agent-builder) 构建和部署智能体时，了解风险非常重要。了解风险类型以及在构建多智能体工作流时如何缓解它们。

Certain agent workflow patterns are more vulnerable to risk. In chat workflows, two important considerations are protecting user input and being careful about MCP tool calling.

某些智能体工作流模式更容易受到风险影响。在聊天工作流中，两个重要的考虑因素是保护用户输入以及谨慎使用 MCP 工具调用。

### Prompt injections

**Prompt injections** are a common and dangerous type of attack. A prompt injection happens when untrusted text or data enters an AI system, and malicious contents in that text or data attempt to override instructions to the AI. The end goals of prompt injections vary but can include exfiltrating private data via downstream tool calls, taking misaligned actions, or otherwise changing model behavior in an unintended way. For example, a prompt might trick a data lookup agent into sending raw customer records instead of the intended summary. See an example in context in the [Codex internet access docs](https://developers.openai.com/codex/cloud/internet-access/).

**提示注入（Prompt injections）**是一种常见且危险的攻击类型。当不受信任的文本或数据进入 AI 系统时，就会发生提示注入，这些文本或数据中的恶意内容试图覆盖给 AI 的指令。提示注入的最终目标各不相同，但可能包括通过下游工具调用窃取私人数据、采取不一致的操作，或以其他方式以非预期方式改变模型行为。例如，一个提示可能会欺骗数据查询智能体，使其发送原始客户记录而不是预期的摘要。在 [Codex internet access docs](https://developers.openai.com/codex/cloud/internet-access/) 中可以看到上下文中的示例。

### Private data leakage

**Private data leakage**, when an agent accidentally shares private data, is also a risk to guard against. It’s possible for a model to leak private data in a way that’s not intended, without an attacker behind it. For example, a model may send more data to an MCP than the user expected or intended. While guardrails provide better control to limit the information included in context, you don’t have full control over what the model chooses to share with connected MCPs.

**私人数据泄露（Private data leakage）**，即智能体意外共享私人数据，也是一种需要防范的风险。模型有可能在没有攻击者的情况下，以非预期的方式泄露私人数据。例如，模型可能会向 MCP 发送比用户预期或意图更多的数据。虽然护栏提供了更好的控制来限制上下文中包含的信息，但你无法完全控制模型选择与连接的 MCP 共享什么内容。

Use the following guidance to reduce the attack surface and mitigate these risks. However, *even with these mitigations*, agents won’t be perfect and can still make mistakes or be tricked; as a result, it’s important to understand these risks and use caution in what access you give agents and how you apply agents.

使用以下指导来减少攻击面并缓解这些风险。然而，*即使采取了这些缓解措施*，智能体也不会完美，仍然可能犯错或被欺骗；因此，了解这些风险并在赋予智能体何种权限以及如何应用智能体时保持谨慎非常重要。

Because developer messages take precedence over user and assistant messages, injecting untrusted input directly into developer messages gives attackers the highest degree of control. Pass untrusted inputs through user messages to limit their influence. This is especially important for workflows where user inputs are passed to sensitive tools or privileged contexts.

由于 developer message 优先于 user 和 assistant message，将不受信任的输入直接注入 developer message 会给攻击者最高程度的控制权。通过 user message 传递不受信任的输入以限制其影响。这对于用户输入被传递给敏感工具或特权上下文的工作流尤其重要。

Prompt injections often rely on the model freely generating unexpected text or commands that propagate downstream. By defining structured outputs between nodes (e.g., enums, fixed schemas, required field names), you eliminate freeform channels that attackers can exploit to smuggle instructions or data.

提示注入通常依赖于模型自由生成非预期的文本或命令并向下游传播。通过在节点之间定义结构化输出（例如 enums、固定 schema、必填字段名），你可以消除攻击者可利用来走私指令或数据的自由格式通道。

Agent workflows may do something you don’t want due to hallucination, misunderstanding, ambiguous user input, etc. For example, an agent may offer a refund it’s not supposed to or delete information it shouldn’t. The best way to mitigate this risk is to strengthen your prompts with good documentation of your desired policies and clear examples. Anticipate unintended scenarios and provide examples so the agent knows what to do in these cases.

由于幻觉、误解、模糊的用户输入等原因，智能体工作流可能会做出一些你不希望发生的事情。例如，智能体可能会提供不允许的退款，或者删除不应删除的信息。缓解这种风险的最佳方法是用对你的期望策略的良好文档和清晰示例来强化你的 prompts。预测非预期场景并提供示例，以便智能体知道在这些情况下该怎么做。

These models are more disciplined about following developer instructions and exhibit stronger robustness against jailbreaks and indirect prompt injections. Configure these models at the agent node level for a more resilient default posture, especially for higher-risk workflows.

这些模型在遵循 developer 指令方面更加自律，并且对越狱和间接提示注入表现出更强的鲁棒性。在智能体节点级别配置这些模型，以获得更具弹性的默认姿态，特别是对于高风险工作流。

When using MCP tools, always enable tool approvals so end users can review and confirm every operation, including reads and writes. In Agent Builder, use the [human approval](about:/api/docs/guides/node-reference#human-approval) node.

使用 MCP 工具时，始终启用工具审批，以便最终用户可以审查和确认每个操作，包括读取和写入。在 Agent Builder 中，使用 [human approval](about:/api/docs/guides/node-reference#human-approval) 节点。

Sanitize incoming inputs using built-in [guardrails](about:/api/docs/guides/node-reference#guardrails) to redact personally identifiable information (PII) and detect jailbreak attempts. While the guardrails nodes in Agent Builder alone are not foolproof, they’re an effective first wave of protection.

使用内置的 [guardrails](about:/api/docs/guides/node-reference#guardrails) 对传入输入进行消毒，以编辑个人身份信息（PII）并检测越狱尝试。虽然仅靠 Agent Builder 中的护栏节点并非万无一失，但它们是有效的第一道防线。

If you understand what models are doing, you can better catch and prevent mistakes. Use [evals](/api/docs/guides/evaluation-getting-started) to evaluate and improve performance. Trace grading provides scores and annotations to specific parts of an agent’s trace—such as decisions, tool calls, or reasoning steps—to assess where the agent performed well or made mistakes.

如果你了解模型在做什么，你就能更好地发现和预防错误。使用 [evals](/api/docs/guides/evaluation-getting-started) 来评估和改进性能。Trace grading 为智能体 trace 的特定部分（如决策、工具调用或推理步骤）提供分数和注释，以评估智能体在哪些方面表现出色或在哪些地方犯了错误。

By combining these techniques and hardening critical steps, you can significantly reduce risks of prompt injection, malicious tool use, or unexpected agent behavior.

通过结合这些技术并加固关键步骤，你可以显著降低提示注入、恶意工具使用或意外智能体行为的风险。

Design workflows so untrusted data never directly drives agent behavior. Extract only specific structured fields (e.g., enums or validated JSON) from external inputs to limit injection risk from flowing between nodes. Use guardrails, tool confirmations, and variables passed via user messages to validate inputs.

设计工作流时，确保不受信任的数据永远不会直接驱动智能体行为。仅从外部输入中提取特定的结构化字段（例如 enums 或经过验证的 JSON），以限制注入风险在节点之间流动。使用护栏、工具确认以及通过 user message 传递的变量来验证输入。

Risk rises when agents process arbitrary text that influences tool calls. Structured outputs and isolation greatly reduce, *but don’t fully remove*, this risk.

当智能体处理影响工具调用的任意文本时，风险会上升。结构化输出和隔离大大降低了这种风险，但*并没有完全消除*它。
