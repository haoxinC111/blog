# Safety in Building Agents | OpenAI API

- Source URL: https://developers.openai.com/api/docs/guides/agent-builder-safety/
- 整理日期: 2026-04-07

## 风险

### Prompt injections（提示注入）

- 当不信任的文本进入 AI 系统时，恶意内容试图覆盖 AI 指令。可能导致通过下游工具调用泄露私人数据、执行不对齐操作。
- When untrusted text enters the AI system, malicious content attempts to override AI instructions. This can lead to leaking private data via downstream tool calls or executing misaligned actions.

### Private data leakage（私人数据泄露）

- 模型可能意外共享私人数据，例如向 MCP 发送比用户预期更多的数据。
- Models may inadvertently share private data, such as sending more data to an MCP server than the user expected.

## 缓解措施

1. 通过 user message 传递不可信输入，不要在 developer message 中直接注入。
   Pass untrusted input via the user message; do not inject it directly into the developer message.
2. 定义结构化输出（enums、固定 schema、必填字段）消除自由格式通道。
   Define structured outputs (enums, fixed schema, required fields) to eliminate free-form channels.
3. 用良好文档和清晰示例强化提示，预测无意场景。
   Strengthen prompts with good documentation and clear examples, anticipating unintended scenarios.
4. 为高风险工作流在 agent 节点配置更强的模型（更遵循开发者指令、对越狱和间接注入更鲁棒）。
   For high-risk workflows, use stronger models at agent nodes that better follow developer instructions and are more robust to jailbreaks and indirect injections.
5. 使用 MCP 工具时始终启用工具审批（human approval node）。
   Always enable tool approval (human approval node) when using MCP tools.
6. 使用 guardrails 节点清理输入（PII 脱敏、越狱检测）。
   Use guardrails nodes to sanitize inputs (PII redaction, jailbreak detection).
7. 使用 evals 和 trace grading 评估和改进性能。
   Use evals and trace grading to assess and improve performance.
8. 设计工作流使不可信数据永远不会直接驱动 agent 行为，提取特定结构化字段限制注入风险。
   Design workflows so untrusted data never directly drives agent behavior; extract specific structured fields to limit injection risk.
