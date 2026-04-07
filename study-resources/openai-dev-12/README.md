# Prompt Engineering

- **Source URL**: https://developers.openai.com/api/docs/guides/prompt-engineering/
- **整理日期**: 2026-04-07

## 核心要点

- 使用 Responses API 生成文本示例。
- 可用 reasoning models（复杂任务、多步规划，较慢较贵）和 GPT models（快速、成本效益高）。
- 不确定时用 `gpt-4.1`。
- 建议固定生产应用到特定 model snapshot，并建立 evals。
- 使用 `instructions` 参数或 message roles 提供不同权威级别的指令；`developer` messages 优先级高于 `user` messages。
- 可在 Dashboard 中创建可复用的 prompts，支持 `{{variable}}` 变量，通过 `prompt_id` 在 API 中引用。
- 提示工程技术包括：提供上下文和示例（few-shot）、明确指令、分解复杂任务、指定输出格式、指示模型检查答案、使用外部工具、测试不同温度和 prompt 变体。

## 文档链接

- [OpenAI API Prompt Engineering Guide](https://developers.openai.com/api/docs/guides/prompt-engineering/)
