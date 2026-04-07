# Streaming API Responses | OpenAI API

- **Source URL**: https://developers.openai.com/api/docs/guides/streaming-responses/
- **整理日期**: 2026-04-07

## 核心要点

- 默认情况下 API 在生成完整输出后才返回；流式响应允许在模型继续生成时开始打印或处理输出开头。
- 本指南聚焦 HTTP streaming（SSE）；WebSocket 见 Responses API WebSocket mode。
- 设置 `stream=True` 开始流式响应。
- Responses API 使用语义化事件（semantic events），每个事件都有预定义 schema。
- 重要事件类型：`response.created`、`response.output_text.delta`、`response.completed`、`error`。
- SDK 中每个事件都是带类型的实例。
- 高级用例：流式 function calls、流式 structured outputs。
- 注意：生产应用中流式输出使内容审核更困难。

## 文档链接

- [openai-dev-16-streaming-responses-bilingual.md](./openai-dev-16-streaming-responses-bilingual.md)
