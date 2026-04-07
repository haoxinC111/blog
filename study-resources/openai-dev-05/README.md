# openai-dev-05: Safety in Building Agents | OpenAI API

- **来源 URL**: https://developers.openai.com/api/docs/guides/agent-builder-safety/
- **整理日期**: 2026-04-07

## 核心要点

- 构建和部署 agents 时的主要安全风险：prompt injections（提示注入）和 private data leakage（私人数据泄露）。
- 通过 user message 传递不可信输入，避免直接注入 developer message，以限制攻击者影响力。
- 使用结构化输出（enums、固定 schema、必填字段）消除自由格式通道，降低注入风险。
- 高风险工作流应在 agent 节点配置更强的模型（更遵循 developer 指令、对越狱/间接注入更鲁棒）。
- 使用 MCP 工具时始终启用 human approval（工具审批），并利用 guardrails 节点对输入进行 PII 脱敏和越狱检测。
- 通过 evals 与 trace grading 持续评估和改进安全性；设计工作流使不可信数据不直接驱动 agent 行为。

## 文档链接

- 主文档: https://developers.openai.com/api/docs/guides/agent-builder-safety/
- 双语全文: [openai-dev-05-safety-in-building-agents-bilingual.md](./openai-dev-05-safety-in-building-agents-bilingual.md)
