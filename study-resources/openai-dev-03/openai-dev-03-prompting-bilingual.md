# Prompting | OpenAI API

- Source URL: https://developers.openai.com/api/docs/guides/prompting/
- 整理日期: 2026-04-07

## Prompt 对象与版本控制

使用长生命周期的 prompt 对象，支持版本控制和模板变量。   
Use long-lived prompt objects that support versioning and template variables.

可以在 Playground 中创建、保存、版本化和共享提示。   
You can create, save, version, and share prompts in the Playground.

使用 `{{variable}}` 语法注入动态值。   
Use the `{{variable}}` syntax to inject dynamic values.

## 关键建议

- 将整体语气或角色指导放在 system message 中。   
  Place overall tone or role instructions in the system message.
- 将任务特定细节和示例放在 user message 中。   
  Put task-specific details and examples in the user message.
- 将 few-shot 示例组合成简洁的 YAML 风格或项目符号块。   
  Group few-shot examples into concise YAML-style or bullet-point blocks.
- 每次发布时重新运行关联的评估。   
  Re-run associated evaluations with each release.
