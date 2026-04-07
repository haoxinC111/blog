# Streaming API Responses | OpenAI API

- **Source URL**: https://developers.openai.com/api/docs/guides/streaming-responses/
- **整理日期**: 2026-04-07

By default, when you make a request to the OpenAI API, we generate the model's entire output before sending it back in a single HTTP response. When generating long outputs, waiting for a response can take time. Streaming responses lets you start printing or processing the beginning of the model's output while it continues generating the full response.

默认情况下，当你向 OpenAI API 发送请求时，我们会在生成模型的完整输出后，将其通过单个 HTTP 响应返回。在生成长输出时，等待响应可能需要一些时间。流式响应让你在模型继续生成完整响应的同时，开始打印或处理输出的开头部分。

This guide focuses on HTTP streaming (`stream=true`) over server-sent events (SSE). For persistent WebSocket transport with incremental inputs via `previous_response_id`, see [the Responses API WebSocket mode](/api/docs/guides/websocket-mode).

本指南聚焦于通过服务器推送事件（SSE）实现的 HTTP 流式传输（`stream=true`）。如需通过 `previous_response_id` 进行增量输入的持久化 WebSocket 传输，请参阅 [Responses API WebSocket 模式](/api/docs/guides/websocket-mode)。

To start streaming responses, set `stream=True` in your request to the Responses endpoint:

要在 Responses 端点开启流式响应，请在请求中设置 `stream=True`：

```python
from openai import OpenAI
client = OpenAI()

stream = client.responses.create(
    model="gpt-5",
    input=[
        {
            "role": "user",
            "content": "Say 'double bubble bath' ten times fast.",
        },
    ],
    stream=True,
)

for event in stream:
    print(event)
```

The Responses API uses semantic events for streaming. Each event is typed with a predefined schema, so you can listen for events you care about.

Responses API 使用语义化事件进行流式传输。每个事件都带有预定义的类型 schema，因此你可以监听你关心的事件。

For a full list of event types, see the [API reference for streaming](/api/docs/api-reference/responses-streaming). Here are a few examples:

有关完整的事件类型列表，请参阅 [流式传输 API 参考](/api/docs/api-reference/responses-streaming)。以下是几个示例：

```
type StreamingEvent =
	| ResponseCreatedEvent
	| ResponseInProgressEvent
	| ResponseFailedEvent
	| ResponseCompletedEvent
	| ResponseOutputItemAdded
	| ResponseOutputItemDone
	| ResponseContentPartAdded
	| ResponseContentPartDone
	| ResponseOutputTextDelta
	| ResponseOutputTextAnnotationAdded
	| ResponseTextDone
	| ResponseRefusalDelta
	| ResponseRefusalDone
	| ResponseFunctionCallArgumentsDelta
	| ResponseFunctionCallArgumentsDone
	| ResponseFileSearchCallInProgress
	| ResponseFileSearchCallSearching
	| ResponseFileSearchCallCompleted
	| ResponseCodeInterpreterInProgress
	| ResponseCodeInterpreterCallCodeDelta
	| ResponseCodeInterpreterCallCodeDone
	| ResponseCodeInterpreterCallInterpreting
	| ResponseCodeInterpreterCallCompleted
	| Error
```

If you're using our SDK, every event is a typed instance. You can also identity individual events using the `type` property of the event.

如果你在使用我们的 SDK，每个事件都是一个带有类型的实例。你也可以通过事件的 `type` 属性来识别单个事件。

Some key lifecycle events are emitted only once, while others are emitted multiple times as the response is generated. Common events to listen for when streaming text are:

一些关键的生命周期事件只发送一次，而另一些会在响应生成过程中多次发送。在流式传输文本时，常见需要监听的事件有：

```
- `response.created`
- `response.output_text.delta`
- `response.completed`
- `error`
```

For a full list of events you can listen for, see the [API reference for streaming](/api/docs/api-reference/responses-streaming).

有关可以监听的完整事件列表，请参阅 [流式传输 API 参考](/api/docs/api-reference/responses-streaming)。

For more advanced use cases, like streaming tool calls, check out the following dedicated guides:

对于更高级的用例，例如流式工具调用，请参阅以下专门指南：

- [Streaming function calls](about:/api/docs/guides/function-calling#streaming)
- [Streaming structured output](about:/api/docs/guides/structured-outputs#streaming)

Note that streaming the model's output in a production application makes it more difficult to moderate the content of the completions, as partial completions may be more difficult to evaluate. This may have implications for approved usage.

请注意，在生产应用中流式传输模型的输出会使审核生成内容变得更加困难，因为部分完成的内容可能更难以评估。这可能会对获准使用产生影响。
