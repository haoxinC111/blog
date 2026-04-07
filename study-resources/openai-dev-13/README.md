# openai-dev-13: Structured Model Outputs | OpenAI API

- **来源 URL**: https://developers.openai.com/api/docs/guides/structured-outputs/
- **整理日期**: 2026-04-07

## 核心要点

- Structured Outputs 确保模型生成符合提供的 JSON Schema 的响应。
- 优点：可靠类型安全、明确拒绝、更简化的提示。
- 支持 Pydantic（Python）和 Zod（JS）定义 schema。
- 两种形式：function calling 和 `text.format`（`json_schema` response format）。
- 比 JSON mode 更进一步：不仅保证有效 JSON，还保证 schema 遵循。
- 支持 chain of thought 的结构化步骤输出。
- 步骤为：1) 定义 schema；2) 在 API 调用中提供 schema（`text: { format: { type: "json_schema", strict: true, schema: ... } }`）。
- JSON Schema 建议：键名清晰、创建清晰的标题和描述、用 evals 确定最佳结构。

## 文档链接

- 主文档: https://developers.openai.com/api/docs/guides/structured-outputs/
- 忠实双语全文: [openai-dev-13-structured-outputs-bilingual.md](./openai-dev-13-structured-outputs-bilingual.md)
