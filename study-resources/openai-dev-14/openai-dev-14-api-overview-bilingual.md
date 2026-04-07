# API Overview | OpenAI API Reference

- Source URL: https://developers.openai.com/api/reference/overview/
- 整理日期: 2026-04-07

## API 概览

提供 RESTful、streaming 和 realtime API 参考。
Provides RESTful, streaming, and realtime API references.

## 认证

使用 API key 通过 HTTP Bearer 认证。
Authenticate using an API key via HTTP Bearer authentication.

## 多组织/项目

可通过 `OpenAI-Organization` 和 `OpenAI-Project` 头部指定。
Specify via the `OpenAI-Organization` and `OpenAI-Project` headers.

## 调试头部

调试请求时关注以下头部：`openai-processing-ms`、`x-request-id`、rate limit 头部。
When debugging requests, pay attention to these headers: `openai-processing-ms`, `x-request-id`, and rate limit headers.

建议在生产中记录 request ID。
It is recommended to log the request ID in production.

可自定义 `X-Client-Request-Id`。
You can customize `X-Client-Request-Id`.

## 向后兼容性

向后兼容性承诺：避免在 v1 中做破坏性变更，但 model prompting behavior 在 snapshot 之间可能变化，建议固定模型版本并实施 evals。
Backward compatibility commitment: avoid breaking changes in v1, but model prompting behavior may change between snapshots. It is recommended to pin model versions and implement evals.
