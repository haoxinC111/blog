# Prompt Engineering | OpenAI API

- **Source URL**: https://developers.openai.com/api/docs/guides/prompt-engineering/
- **整理日期**: 2026-04-07
---

This guide covers generating text examples using the Responses API.

本指南介绍如何使用 Responses API 生成文本示例。

You can use reasoning models for complex tasks and multi-step planning, which are slower and more expensive, or GPT models for fast and cost-effective responses.

你可以使用 reasoning models 处理复杂任务和多步规划（较慢且更贵），或使用 GPT models 获得快速且高性价比的响应。

When in doubt, use `gpt-4.1`.

如果不确定选择哪个模型，建议使用 `gpt-4.1`。

It is recommended to pin production applications to a specific model snapshot and establish evals.

建议将生产应用固定到特定的模型 snapshot，并建立 evals。

You can provide instructions of varying authority levels using the `instructions` parameter or message roles.

你可以使用 `instructions` 参数或 message roles 提供不同权威级别的指令。

`developer` messages take priority over `user` messages.

`developer` messages 的优先级高于 `user` messages。

You can create reusable prompts in the dashboard, which support `{{variable}}` placeholders and can be referenced in the API via `prompt_id`.

你可以在 Dashboard 中创建可复用的 prompts，支持 `{{variable}}` 变量，并通过 `prompt_id` 在 API 中引用。

Key prompt engineering techniques include: providing context and examples (few-shot), giving explicit instructions, breaking down complex tasks, specifying output formats, instructing the model to check its answers, using external tools, and testing different temperatures and prompt variations.

关键的提示工程技术包括：提供上下文和示例（few-shot）、给出明确指令、分解复杂任务、指定输出格式、指示模型检查答案、使用外部工具、测试不同温度和 prompt 变体。
