# openai-dev-03: Prompting | OpenAI API

- **来源 URL**: https://developers.openai.com/api/docs/guides/prompting/
- **整理日期**: 2026-04-07

## 核心要点

- OpenAI 提供长生命周期的 prompt 对象，支持版本控制和模板变量 `{{variable}}`，可在项目内的 API、SDK 和 Dashboard 之间共享。
- 可在 Playground 中创建、保存、版本化并共享 prompts；可通过 prompt ID 在 Responses API 中调用。
- 关键提示策略：system message 放整体语气/角色，user message 放任务细节和 few-shot 示例；示例建议整理为 YAML 风格或项目符号块。
- 每次发布时重新运行关联的 eval，确保 prompt 改动未引入回归。
- 相关能力：Prompt caching（降延迟/成本）、Prompt engineering（提示工程技巧）。

## 文档链接

- 主文档: https://developers.openai.com/api/docs/guides/prompting/
- 双语全文: [openai-dev-03-prompting-bilingual.md](./openai-dev-03-prompting-bilingual.md)
