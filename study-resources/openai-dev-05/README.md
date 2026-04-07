# openai-dev-05: Safety in Building Agents | OpenAI API

- **来源 URL**: https://developers.openai.com/api/docs/guides/agent-builder-safety/
- **整理日期**: 2026-04-07

## 核心要点

- 构建 agents 时的主要安全风险：prompt injections 和 private data leakage。
- 通过 user message 传递不可信输入、定义结构化输出、配置 human approval node、使用 guardrails 节点（PII 脱敏/越狱检测）等手段降低风险。
- 高风险工作流应使用更强的模型，并通过 evals 与 trace grading 持续评估和改进安全性。

## 文档链接

- 主文档: https://developers.openai.com/api/docs/guides/agent-builder-safety/
- 双语全文: [openai-dev-05-safety-in-building-agents-bilingual.md](./openai-dev-05-safety-in-building-agents-bilingual.md)
