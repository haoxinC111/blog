# openai-dev-02: Building Agents

- **来源 URL**: https://developers.openai.com/tracks/building-agents/
- **整理日期**: 2026-04-07

## 核心要点

- OpenAI 将 Agent 定义为具备 instructions（该做什么）、guardrails（不该做什么）和 tools（能做什么）并能代表用户采取行动的 AI 系统。
- 构建 Agent 的四大原语：models、tools、state/memory、orchestration。
- 模型选择：简单/低延迟任务用非推理模型（如 gpt-5.4-mini/nano），复杂任务（规划、数学、代码、多工具工作流）用推理模型（o4-mini/o3 或高 reasoning_effort 的 gpt-5）。
- 核心构建方式：Responses API（更底层、更灵活、默认有状态）与 Agents SDK（更高层抽象，内置 agent loops、guardrails、tracing）。
- 要让 Agent 真正有用，必须为其配备 tools，使其能检索数据、执行任务并连接外部系统。

## 文档链接

- 主文档: https://developers.openai.com/tracks/building-agents/
- 双语全文: [openai-dev-02-building-agents-bilingual.md](./openai-dev-02-building-agents-bilingual.md)
