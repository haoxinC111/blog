# Streaming API Responses | OpenAI API

- **Source URL**: https://developers.openai.com/api/docs/guides/streaming-responses/
- **整理日期**: 2026-04-07
---

By default, the API returns the complete output after generation is finished.

默认情况下，API 在生成完整输出后才返回结果。

Streaming responses allow you to start printing or processing the beginning of the output while the model is still generating the rest.

流式响应允许在模型继续生成其余内容时，就开始打印或处理输出的开头部分。

This guide focuses on HTTP streaming (SSE). For WebSocket, see the Responses API WebSocket mode.

本指南聚焦 HTTP streaming（SSE）。关于 WebSocket，请参阅 Responses API WebSocket mode。

Set `stream=True` to start a streaming response.

设置 `stream=True` 以开始流式响应。

The Responses API uses semantic events, where each event has a predefined schema.

Responses API 使用语义化事件（semantic events），每个事件都有预定义的 schema。

Important event types include: `response.created`, `response.output_text.delta`, `response.completed`, and `error`.

重要事件类型包括：`response.created`、`response.output_text.delta`、`response.completed`、`error`。

In the SDK, each event is a typed instance.

在 SDK 中，每个事件都是一个带类型的实例。

Advanced use cases include streaming function calls and streaming structured outputs.

高级用例包括：流式 function calls、流式 structured outputs。

Note that streaming output makes content moderation more difficult in production applications.

注意：在生产应用中，流式输出会使内容审核更加困难。
