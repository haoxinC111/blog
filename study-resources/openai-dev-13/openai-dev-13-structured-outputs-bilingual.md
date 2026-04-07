# Structured Model Outputs | OpenAI API

- **Source URL**: https://developers.openai.com/api/docs/guides/structured-outputs/
- **整理日期**: 2026-04-07
---

Structured Outputs ensure that model responses are generated according to a provided JSON Schema.

Structured Outputs 确保模型生成的响应符合提供的 JSON Schema。

Benefits include reliable type safety, explicit refusals, and simpler prompting.

优点包括：可靠的类型安全、明确的拒绝表达、更简化的提示。

Pydantic (Python) and Zod (JS) are supported for defining schemas.

支持使用 Pydantic（Python）和 Zod（JS）定义 schema。

There are two forms: function calling and `text.format` (the `json_schema` response format).

有两种形式：function calling 和 `text.format`（即 `json_schema` response format）。

This goes beyond JSON mode: it guarantees not only valid JSON, but also schema adherence.

这比 JSON mode 更进一步：不仅保证有效的 JSON，还保证符合 schema。

Chain of thought structured step-by-step outputs are supported.

支持 chain of thought 的结构化步骤输出。

The steps are: 1) define the schema, and 2) provide the schema in the API call using `text: { format: { type: "json_schema", strict: true, schema: ... } }`.

步骤为：1) 定义 schema；2) 在 API 调用中提供 schema：`text: { format: { type: "json_schema", strict: true, schema: ... } }`。

JSON Schema best practices: use clear key names, create clear titles and descriptions, and use evals to determine the best structure.

JSON Schema 建议：键名清晰、创建清晰的标题和描述、用 evals 确定最佳结构。
