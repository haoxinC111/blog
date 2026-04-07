# API Overview | OpenAI API Reference

> **Source / 原文来源**: https://developers.openai.com/api/reference/overview/  
> **Compiled for study on / 整理日期**: 2026-04-07

Reference docs for API Overview.

API Overview 参考文档。

This API reference describes the RESTful, streaming, and realtime APIs you can use to interact with the OpenAI platform. REST APIs are usable via HTTP in any environment that supports HTTP requests. Language-specific SDKs are listed [on the libraries page](/docs/libraries).

本 API 参考描述了可用于与 OpenAI 平台交互的 RESTful、streaming 和 realtime API。REST API 可以在任何支持 HTTP 请求的环境中通过 HTTP 使用。特定语言的 SDK 列在 [libraries 页面](/docs/libraries) 上。

The OpenAI API uses API keys for authentication. Create, manage, and learn more about API keys in your [organization settings](/settings/organization/api-keys).

OpenAI API 使用 API 密钥进行身份验证。在你的 [organization settings](/settings/organization/api-keys) 中创建、管理和了解有关 API 密钥的更多信息。

**Remember that your API key is a secret!** Do not share it with others or expose it in any client-side code (browsers, apps). API keys should be securely loaded from an environment variable or key management service on the server.

**请记住，你的 API 密钥是秘密！** 不要与他人分享，也不要在任何客户端代码（浏览器、应用）中暴露它。API 密钥应安全地从服务器上的环境变量或密钥管理服务加载。

API keys should be provided via [HTTP Bearer authentication](https://swagger.io/docs/specification/v3_0/authentication/bearer-authentication/).

API 密钥应通过 [HTTP Bearer authentication](https://swagger.io/docs/specification/v3_0/authentication/bearer-authentication/) 提供。

```
Authorization: Bearer OPENAI_API_KEY
```

If you belong to multiple organizations or access projects through a legacy user API key, pass a header to specify which organization and project to use for an API request:

如果你属于多个组织，或通过旧版用户 API 密钥访问项目，请传递一个标头以指定哪个组织和项目用于 API 请求：

```
curl https://api.openai.com/v1/models \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -H "OpenAI-Organization: $ORGANIZATION_ID" \
  -H "OpenAI-Project: $PROJECT_ID"
```

Usage from these API requests counts as usage for the specified organization and project. Organization IDs can be found on your [organization settings](/settings/organization/general) page.
Project IDs can be found on your [general settings](/settings) page by selecting the specific project.

这些 API 请求的用量计入指定组织和项目的用量。组织 ID 可以在你的 [organization settings](/settings/organization/general) 页面上找到。
项目 ID 可以通过选择特定项目在你的 [general settings](/settings) 页面上找到。

## Debugging requests / 调试请求


In addition to [error codes](/docs/guides/error-codes) returned from API responses, you can inspect HTTP response headers containing the unique ID of a particular API request or information about rate limiting applied to your requests. Below is an incomplete list of HTTP headers returned with API responses:

除了 API 响应返回的 [error codes](/docs/guides/error-codes) 之外，你还可以检查 HTTP 响应标头，其中包含特定 API 请求的唯一 ID 或有关应用于你请求的速率限制的信息。以下是 API 响应返回的 HTTP 标头的不完整列表：

**API meta information**

**API 元信息**

* `openai-organization`: The [organization](/docs/guides/production-best-practices#setting-up-your-organization) associated with the request
* `openai-organization`: 与请求关联的 [organization](/docs/guides/production-best-practices#setting-up-your-organization)
* `openai-processing-ms`: Time taken processing your API request
* `openai-processing-ms`: 处理你的 API 请求所花费的时间
* `openai-version`: REST API version used for this request (currently `2020-10-01`)
* `openai-version`: 用于此请求的 REST API 版本（当前为 `2020-10-01`）
* `x-request-id`: Unique identifier for this API request (used in troubleshooting)
* `x-request-id`: 此 API 请求的唯一标识符（用于故障排除）

**[Rate limiting information](/docs/guides/rate-limits)**

**[速率限制信息](/docs/guides/rate-limits)**

* `x-ratelimit-limit-requests`
* `x-ratelimit-limit-tokens`
* `x-ratelimit-remaining-requests`
* `x-ratelimit-remaining-tokens`
* `x-ratelimit-reset-requests`
* `x-ratelimit-reset-tokens`

**OpenAI recommends logging request IDs in production deployments** for more efficient troubleshooting with our [support team](https://help.openai.com/en/), should the need arise. Our [official SDKs](/docs/libraries) provide a property on top-level response objects containing the value of the `x-request-id` header.

**OpenAI 建议在生产部署中记录 request ID**，以便在需要时与我们的 [support team](https://help.openai.com/en/) 更高效地进行故障排除。我们的 [official SDKs](/docs/libraries) 在顶级响应对象上提供了一个属性，其中包含 `x-request-id` 标头的值。

### Supplying your own request ID with `X-Client-Request-Id` / 使用 `X-Client-Request-Id` 提供你自己的请求 ID


In addition to the server-generated `x-request-id`, you can supply your own unique identifier for each request via the `X-Client-Request-Id` request header. This header is not added automatically; you must explicitly set it on the request.

除了服务器生成的 `x-request-id` 之外，你还可以通过 `X-Client-Request-Id` 请求标头为每个请求提供你自己的唯一标识符。此标头不会自动添加；你必须在请求上显式设置它。

When you include `X-Client-Request-Id`:

当你包含 `X-Client-Request-Id` 时：

* You control the ID format (for example, a UUID or your internal trace ID), but it must contain only ASCII characters and be no more than 512 characters long; otherwise, the request will fail with a 400 error. We strongly recommend making this value unique per request.
* 你可以控制 ID 格式（例如，UUID 或你的内部 trace ID），但它必须仅包含 ASCII 字符且长度不超过 512 个字符；否则，请求将以 400 错误失败。我们强烈建议使每个请求的该值唯一。
* OpenAI will log this value in our internal logs for supported endpoints, including chat/completions, embeddings, responses, and more.
* OpenAI 将在支持端点（包括 chat/completions、embeddings、responses 等）的内部日志中记录此值。
* In cases like timeouts or network issues when you can't get the `X-Request-Id` response header, you can share the `X-Client-Request-Id` value with our support team, and we can look up whether we received the request and何时收到的。
* 在超时或网络问题等情况下，当你无法获取 `X-Request-Id` 响应标头时，你可以与我们的支持团队共享 `X-Client-Request-Id` 值，我们可以查找是否收到该请求以及何时收到。

**Example:**

**示例：**

```
curl https://api.openai.com/v1/chat/completions \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -H "X-Client-Request-Id: 123e4567-e89b-12d3-a456-426614174000"
```

## Backwards compatibility / 向后兼容性


OpenAI is committed to providing stability to API users by avoiding breaking changes in major API versions whenever reasonably possible. This includes:

OpenAI 致力于通过在合理可能的情况下避免主要 API 版本中的破坏性变更来为用户提供稳定性。这包括：

* The REST API (currently `v1`)
* REST API（当前为 `v1`）
* Our first-party [SDKs](/docs/libraries) (released SDKs adhere to [semantic versioning](https://semver.org/))
* 我们的第一方 [SDKs](/docs/libraries)（发布的 SDK 遵循 [semantic versioning](https://semver.org/)）
* [Model](/docs/models) families (like `gpt-4o` or `o4-mini`)
* [模型](/docs/models) 系列（如 `gpt-4o` 或 `o4-mini`）

**Model prompting behavior between snapshots is subject to change**.
Model outputs are by their nature variable, so expect changes in prompting and model behavior between snapshots. For example, if you moved from `gpt-4o-2024-05-13` to `gpt-4o-2024-08-06`, the same `system` or `user` messages could function differently between versions. The best way to ensure consistent prompting behavior and model output is to use pinned model versions, and to implement [evals](/docs/guides/evals) for your applications.

**快照之间的模型提示行为可能会发生变化**。
模型输出本质上是可变的，因此预计快照之间的提示和模型行为会发生变化。例如，如果你从 `gpt-4o-2024-05-13` 迁移到 `gpt-4o-2024-08-06`，相同的 `system` 或 `user` 消息在不同版本之间可能会有不同的表现。确保一致的提示行为和模型输出的最佳方式是使用固定的模型版本，并为你的应用实施 [evals](/docs/guides/evals)。

**Backwards-compatible API changes**:

**向后兼容的 API 变更**：

* Adding new resources (URLs) to the REST API and SDKs
* 向 REST API 和 SDK 添加新资源（URL）
* Adding new optional API parameters
* 添加新的可选 API 参数
* Adding new properties to JSON response objects or event data
* 向 JSON 响应对象或事件数据添加新属性
* Changing the order of properties in a JSON response object
* 更改 JSON 响应对象中属性的顺序
* Changing the length or format of opaque strings, like resource identifiers and UUIDs
* 更改不透明字符串的长度或格式，如资源标识符和 UUID
* Adding new event types (in either streaming or the Realtime API)
* 添加新的事件类型（在 streaming 或 Realtime API 中）

See the [changelog](/docs/changelog) for a list of backwards-compatible changes and rare breaking changes.

请参阅 [changelog](/docs/changelog) 了解向后兼容的变更和罕见的破坏性变更列表。
