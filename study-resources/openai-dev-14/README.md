# openai-dev-14: API Overview | OpenAI API Reference

- **来源 URL**: https://developers.openai.com/api/reference/overview/
- **整理日期**: 2026-04-07

## 核心要点

- OpenAI API 提供 RESTful、streaming 和 realtime API 参考。
- 认证方式：使用 API key 通过 HTTP Bearer 认证。多组织/项目可通过 `OpenAI-Organization` 和 `OpenAI-Project` 头部指定。
- 调试时关注 `openai-processing-ms`、`x-request-id` 和 rate limit 头部，生产中建议记录 request ID，可自定义 `X-Client-Request-Id`。
- 向后兼容性：v1 中避免破坏性变更，但模型 prompting behavior 在不同快照之间可能变化，建议固定模型版本并实施 evals。

## 文档链接

- 主文档: https://developers.openai.com/api/reference/overview/
- 双语全文: [openai-dev-14-api-overview-bilingual.md](./openai-dev-14-api-overview-bilingual.md)
