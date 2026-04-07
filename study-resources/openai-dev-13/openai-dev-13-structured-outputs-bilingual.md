# Structured Model Outputs | OpenAI API

> **Source / 原文来源**: https://developers.openai.com/api/docs/guides/structured-outputs/  
> **Compiled for study on / 整理日期**: 2026-04-07

JSON is one of the most widely used formats in the world for applications to exchange data.

JSON 是世界上应用之间交换数据最常用的格式之一。

Structured Outputs is a feature that ensures the model will always generate responses that adhere to your supplied [JSON Schema](https://json-schema.org/overview/what-is-jsonschema), so you don't need to worry about the model omitting a required key, or hallucinating an invalid enum value.

Structured Outputs 是一项功能，确保模型始终生成符合你提供的 [JSON Schema](https://json-schema.org/overview/what-is-jsonschema) 的响应，这样你无需担心模型遗漏必需的键，或幻觉出无效的枚举值。

Some benefits of Structured Outputs include:

Structured Outputs 的一些好处包括：

1. **Reliable type-safety:** No need to validate or retry incorrectly formatted responses
2. **可靠的类型安全：** 无需验证或重试格式不正确的响应
3. **Explicit refusals:** Safety-based model refusals are now programmatically detectable
4. **明确的拒绝：** 基于安全性的模型拒绝现在可以通过编程检测到
5. **Simpler prompting:** No need for strongly worded prompts to achieve consistent formatting
6. **更简单的提示：** 无需措辞强烈的提示即可实现一致的格式

In addition to supporting JSON Schema in the REST API, the OpenAI SDKs for [Python](https://github.com/openai/openai-python/blob/main/helpers.md#structured-outputs-parsing-helpers) and [JavaScript](https://github.com/openai/openai-node/blob/master/helpers.md#structured-outputs-parsing-helpers) also make it easy to define object schemas using [Pydantic](https://docs.pydantic.dev/latest/) and [Zod](https://zod.dev/) respectively. Below, you can see how to extract information from unstructured text that conforms to a schema defined in code.

除了在 REST API 中支持 JSON Schema 之外，OpenAI 的 [Python](https://github.com/openai/openai-python/blob/main/helpers.md#structured-outputs-parsing-helpers) 和 [JavaScript](https://github.com/openai/openai-node/blob/master/helpers.md#structured-outputs-parsing-helpers) SDK 还分别支持使用 [Pydantic](https://docs.pydantic.dev/latest/) 和 [Zod](https://zod.dev/) 轻松定义对象 schema。下面，你可以看到如何从非结构化文本中提取符合代码中定义的 schema 的信息。

```
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
from openai import OpenAI
from pydantic import BaseModel

client = OpenAI()

class CalendarEvent(BaseModel):
    name: str
    date: str
    participants: list[str]

response = client.responses.parse(
    model="gpt-4o-2024-08-06",
    input=[
        {"role": "system", "content": "Extract the event information."},
        {
            "role": "user",
            "content": "Alice and Bob are going to a science fair on Friday.",
        },
    ],
    text_format=CalendarEvent,
)

event = response.output_parsed
```

### Supported models / 支持的模型


Structured Outputs is available in our [latest large language models](/api/docs/models), starting with GPT-4o. Older models like `gpt-4-turbo` and earlier may use [JSON mode](#json-mode) instead.

Structured Outputs 在我们的 [latest large language models](/api/docs/models) 中可用，从 GPT-4o 开始。像 `gpt-4-turbo` 及更早的模型可以使用 [JSON mode](#json-mode) 代替。

## When to use Structured Outputs via function calling vs via text.format / 何时通过 function calling 使用 Structured Outputs，何时通过 text.format


Structured Outputs is available in two forms in the OpenAI API:

Structured Outputs 在 OpenAI API 中有两种形式：

1. When using [function calling](/api/docs/guides/function-calling)
2. 使用 [function calling](/api/docs/guides/function-calling) 时
3. When using a `json_schema` response format
4. 使用 `json_schema` response format 时

Function calling is useful when you are building an application that bridges the models and functionality of your application.

当你正在构建一个连接模型和应用程序功能的应用时，function calling 非常有用。

For example, you can give the model access to functions that query a database in order to build an AI assistant that can help users with their orders, or functions that can interact with the UI.

例如，你可以让模型访问查询数据库的函数，以构建一个可以帮助用户处理订单的 AI 助手，或者可以访问与 UI 交互的函数。

Conversely, Structured Outputs via `response_format` are more suitable when you want to indicate a structured schema for use when the model responds to the user, rather than when the model calls a tool.

相反，当你想指定一个结构化 schema 供模型响应用户使用，而不是模型调用工具时，通过 `response_format` 的 Structured Outputs 更为合适。

For example, if you are building a math tutoring application, you might want the assistant to respond to your user using a specific JSON Schema so that you can generate a UI that displays different parts of the model's output in distinct ways.

例如，如果你正在构建一个数学辅导应用，你可能希望助手使用特定的 JSON Schema 响应你的用户，这样你就可以生成一个 UI，以不同的方式显示模型输出的不同部分。

Put simply:

简单来说：

* If you are connecting the model to tools, functions, data, etc. in your system, then you should use function calling
* 如果你将模型连接到你系统中的工具、函数、数据等，那么你应该使用 function calling
* If you want to structure the model's output when it responds to the user, then you should use a structured `text.format`
* 如果你想在模型响应用户时结构化其输出，那么你应该使用结构化的 `text.format`

The remainder of this guide will focus on non-function calling use cases in the Responses API. To learn more about how to use Structured Outputs with function calling, check out the [Function Calling](/api/docs/guides/function-calling#function-calling-with-structured-outputs) guide.

本指南的其余部分将重点介绍 Responses API 中非 function calling 的用例。要了解如何将 Structured Outputs 与 function calling 结合使用，请查看 [Function Calling](/api/docs/guides/function-calling#function-calling-with-structured-outputs) 指南。

### Structured Outputs vs JSON mode / Structured Outputs 与 JSON mode 对比


Structured Outputs is the evolution of [JSON mode](#json-mode). While both ensure valid JSON is produced, only Structured Outputs ensure schema adherence. Both Structured Outputs and JSON mode are supported in the Responses API, Chat Completions API, Assistants API, Fine-tuning API and Batch API.

Structured Outputs 是 [JSON mode](#json-mode) 的演进。虽然两者都确保生成有效的 JSON，但只有 Structured Outputs 确保符合 schema。Responses API、Chat Completions API、Assistants API、Fine-tuning API 和 Batch API 都支持 Structured Outputs 和 JSON mode。

We recommend always using Structured Outputs instead of JSON mode when possible.

我们建议只要可能，始终使用 Structured Outputs 而不是 JSON mode。

However, Structured Outputs with `response_format: {type: "json_schema", ...}` is only supported with the `gpt-4o-mini`, `gpt-4o-mini-2024-07-18`, and `gpt-4o-2024-08-06` model snapshots and later.

但是，带有 `response_format: {type: "json_schema", ...}` 的 Structured Outputs 仅支持 `gpt-4o-mini`、`gpt-4o-mini-2024-07-18` 和 `gpt-4o-2024-08-06` 及之后的模型快照。

|  | Structured Outputs | JSON Mode |
| --- | --- | --- |
| **Outputs valid JSON** | Yes | Yes |
| **Adheres to schema** | Yes (see [supported schemas](#supported-schemas)) | No |
| **Compatible models** | `gpt-4o-mini`, `gpt-4o-2024-08-06`, and later | `gpt-3.5-turbo`, `gpt-4-*` and `gpt-4o-*` models |
| **Enabling** | `text: { format: { type: "json_schema", "strict": true, "schema": ... } }` | `text: { format: { type: "json_object" } }` |

|  | Structured Outputs | JSON Mode |
| --- | --- | --- |
| **输出有效 JSON** | 是 | 是 |
| **符合 schema** | 是（见 [supported schemas](#supported-schemas)） | 否 |
| **兼容模型** | `gpt-4o-mini`、`gpt-4o-2024-08-06` 及之后 | `gpt-3.5-turbo`、`gpt-4-*` 和 `gpt-4o-*` 模型 |
| **启用方式** | `text: { format: { type: "json_schema", "strict": true, "schema": ... } }` | `text: { format: { type: "json_object" } }` |

Chain of thought

思维链

### Chain of thought / 思维链


You can ask the model to output an answer in a structured, step-by-step way, to guide the user through the solution.

你可以要求模型以结构化、逐步的方式输出答案，以引导用户完成解决方案。

```
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
from openai import OpenAI
from pydantic import BaseModel

client = OpenAI()

class Step(BaseModel):
    explanation: str
    output: str

class MathReasoning(BaseModel):
    steps: list[Step]
    final_answer: str

response = client.responses.parse(
    model="gpt-4o-2024-08-06",
    input=[
        {
            "role": "system",
            "content": "You are a helpful math tutor. Guide the user through the solution step by step.",
        },
        {"role": "user", "content": "how can I solve 8x + 7 = -23"},
    ],
    text_format=MathReasoning,
)

math_reasoning = response.output_parsed
```

#### Example response / 示例响应


```
{
  "steps": [
    {
      "explanation": "Start with the equation 8x + 7 = -23.",
      "output": "8x + 7 = -23"
    },
    {
      "explanation": "Subtract 7 from both sides to isolate the term with the variable.",
      "output": "8x = -23 - 7"
    },
    {
      "explanation": "Simplify the right side of the equation.",
      "output": "8x = -30"
    },
    {
      "explanation": "Divide both sides by 8 to solve for x.",
      "output": "x = -30 / 8"
    },
    {
      "explanation": "Simplify the fraction.",
      "output": "x = -15 / 4"
    }
  ],
  "final_answer": "x = -15 / 4"
}
```

## How to use Structured Outputs with text.format / 如何通过 text.format 使用 Structured Outputs


Step 1: Define your schema

步骤 1：定义你的 schema

First you must design the JSON Schema that the model should be constrained to follow. See the [examples](about:/api/docs/guides/structured-outputs#examples) at the top of this guide for reference.

首先，你必须设计模型应受约束遵循的 JSON Schema。请参阅本指南顶部的 [示例](about:/api/docs/guides/structured-outputs#examples)。

While Structured Outputs supports much of JSON Schema, some features are unavailable either for performance or technical reasons. See [here](about:/api/docs/guides/structured-outputs#supported-schemas) for more details.

虽然 Structured Outputs 支持大部分 JSON Schema，但出于性能或技术原因，某些功能不可用。更多详情请参见 [此处](about:/api/docs/guides/structured-outputs#supported-schemas)。

#### Tips for your JSON Schema / JSON Schema 技巧


To maximize the quality of model generations, we recommend the following:

为了最大化模型生成内容的质量，我们建议如下：

* Name keys clearly and intuitively
* 键名清晰直观
* Create clear titles and descriptions for important keys in your structure
* 为结构中重要的键创建清晰的标题和描述
* Create and use evals to determine the structure that works best for your use case
* 创建并使用 evals 来确定最适合你用例的结构

Step 2: Supply your schema in the API call

步骤 2：在 API 调用中提供你的 schema

To use Structured Outputs, simply specify

要使用 Structured Outputs，只需指定

```
text: { format: { type: "json_schema", "strict": true, "schema": … } }
```

For example:

例如：

```
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
response = client.responses.create(
    model="gpt-4o-2024-08-06",
    input=[
        {"role": "system", "content": "You are a helpful math tutor. Guide the user through the solution step by step."},
        {"role": "user", "content": "how can I solve 8x + 7 = -23"}
    ],
    text={
        "format": {
            "type": "json_schema",
            "name": "math_response",
            "schema": {
                "type": "object",
                "properties": {
                    "steps": {
                        "type": "array",
                        "items": {
                            "type": "object",
                            "properties": {
                                "explanation": {"type": "string"},
                                "output": {"type": "string"}
                            },
                            "required": ["explanation", "output"],
                            "additionalProperties": False
                        }
                    },
                    "final_answer": {"type": "string"}
                },
                "required": ["steps", "final_answer"],
                "additionalProperties": False
            },
            "strict": True
        }
    }
)

print(response.output_text)
```

**Note:** the first request you make with any schema will have additional latency as our API processes the schema, but subsequent requests with the same schema will not have additional latency.

**注意：** 你使用任何 schema 发出的第一个请求会有额外的延迟，因为 API 需要处理该 schema，但后续使用相同 schema 的请求不会有额外延迟。

Step 3: Handle edge cases

步骤 3：处理边界情况

In some cases, the model might not generate a valid response that matches the provided JSON schema.

在某些情况下，模型可能不会生成与提供的 JSON schema 匹配的有效响应。

This can happen in the case of a refusal, if the model refuses to answer for safety reasons, or if for example you reach a max tokens limit and the response is incomplete.

如果模型出于安全原因拒绝回答，或者例如你达到了最大 tokens 限制且响应不完整，这种情况就可能发生。

```
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40
41
42
try:
    response = client.responses.create(
        model="gpt-4o-2024-08-06",
        input=[
            {
                "role": "system",
                "content": "You are a helpful math tutor. Guide the user through the solution step by step.",
            },
            {"role": "user", "content": "how can I solve 8x + 7 = -23"},
        ],
        text={
            "format": {
                "type": "json_schema",
                "name": "math_response",
                "strict": True,
                "schema": {
                    "type": "object",
                    "properties": {
                        "steps": {
                            "type": "array",
                            "items": {
                                "type": "object",
                                "properties": {
                                    "explanation": {"type": "string"},
                                    "output": {"type": "string"},
                                },
                                "required": ["explanation", "output"],
                                "additionalProperties": False,
                            },
                        },
                        "final_answer": {"type": "string"},
                    },
                    "required": ["steps", "final_answer"],
                    "additionalProperties": False,
                },
                "strict": True,
            },
        },
    )
except Exception as e:
    # handle errors like finish_reason, refusal, content_filter, etc.
    pass
```

### Refusals with Structured Outputs / Structured Outputs 中的拒绝


When using Structured Outputs with user-generated input, OpenAI models may occasionally refuse to fulfill the request for safety reasons. Since a refusal does not necessarily follow the schema you have supplied in `response_format`, the API response will include a new field called `refusal` to indicate that the model refused to fulfill the request.

当将 Structured Outputs 与用户生成的输入一起使用时，OpenAI 模型有时会出于安全原因拒绝 fulfill 请求。由于拒绝不一定遵循你在 `response_format` 中提供的 schema，因此 API 响应将包含一个名为 `refusal` 的新字段，以指示模型拒绝了该请求。

When the `refusal` property appears in your output object, you might present the refusal in your UI, or include conditional logic in code that consumes the response to handle the case of a refused request.

当 `refusal` 属性出现在你的输出对象中时，你可以在你的 UI 中展示拒绝，或者在消费响应的代码中包含条件逻辑来处理请求被拒绝的情况。

```
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
class Step(BaseModel):
    explanation: str
    output: str

class MathReasoning(BaseModel):
    steps: list[Step]
    final_answer: str

completion = client.chat.completions.parse(
    model="gpt-4o-2024-08-06",
    messages=[
        {"role": "system", "content": "You are a helpful math tutor. Guide the user through the solution step by step."},
        {"role": "user", "content": "how can I solve 8x + 7 = -23"},
    ],
    response_format=MathReasoning,
)

math_reasoning = completion.choices[0].message

# If the model refuses to respond, you will get a refusal message

if math_reasoning.refusal:
    print(math_reasoning.refusal)
else:
    print(math_reasoning.parsed)
```

The API response from a refusal will look something like this:

拒绝的 API 响应看起来像这样：

```
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
{
  "id": "resp_1234567890",
  "object": "response",
  "created_at": 1721596428,
  "status": "completed",
  "completed_at": 1721596429,
  "error": null,
  "incomplete_details": null,
  "input": [],
  "instructions": null,
  "max_output_tokens": null,
  "model": "gpt-4o-2024-08-06",
  "output": [{
    "id": "msg_1234567890",
    "type": "message",
    "role": "assistant",
    "content": [
      {
        "type": "refusal",
        "refusal": "I'm sorry, I cannot assist with that request."
      }
    ]
  }],
  "usage": {
    "input_tokens": 81,
    "output_tokens": 11,
    "total_tokens": 92,
    "output_tokens_details": {
      "reasoning_tokens": 0,
    }
  },
}
```

### Tips and best practices / 技巧与最佳实践


#### Handling user-generated input / 处理用户生成的输入


If your application is using user-generated input, make sure your prompt includes instructions on how to handle situations where the input cannot result in a valid response.

如果你的应用使用用户生成的输入，请确保你的提示包含如何处理输入无法产生有效响应的情况的指令。

The model will always try to adhere to the provided schema, which can result in hallucinations if the input is completely unrelated to the schema.

模型始终会尝试遵循提供的 schema，如果输入与 schema 完全无关，这可能会导致幻觉。

You could include language in your prompt to specify that you want to return empty parameters, or a specific sentence, if the model detects that the input is incompatible with the task.

你可以在提示中加入说明，指定如果模型检测到输入与任务不兼容，你希望返回空参数或特定句子。

#### Handling mistakes / 处理错误


Structured Outputs can still contain mistakes. If you see mistakes, try adjusting your instructions, providing examples in the system instructions, or splitting tasks into simpler subtasks. Refer to the [prompt engineering guide](/api/docs/guides/prompt-engineering) for more guidance on how to tweak your inputs.

Structured Outputs 仍然可能包含错误。如果你发现错误，请尝试调整指令、在系统指令中提供示例，或将任务拆分为更简单的子任务。请参阅 [prompt engineering guide](/api/docs/guides/prompt-engineering) 了解更多关于调整输入的指导。

#### Avoid JSON schema divergence / 避免 JSON schema 分歧


To prevent your JSON Schema and corresponding types in your programming language from diverging, we strongly recommend using the native Pydantic/zod sdk support.

为了防止你的 JSON Schema 和编程语言中相应类型出现分歧，我们强烈建议使用原生的 Pydantic/zod SDK 支持。

If you prefer to specify the JSON schema directly, you could add CI rules that flag when either the JSON schema or underlying data objects are edited, or add a CI step that auto-generates the JSON Schema from type definitions (or vice-versa).

如果你更喜欢直接指定 JSON schema，你可以添加 CI 规则，在 JSON schema 或底层数据对象被编辑时发出标记，或者添加一个 CI 步骤，从类型定义自动生成 JSON Schema（反之亦然）。

You can use streaming to process model responses or function call arguments as they are being generated, and parse them as structured data.

你可以使用流式处理来在模型响应或函数调用参数生成时对其进行处理，并将其解析为结构化数据。

That way, you don't have to wait for the entire response to complete before handling it.
This is particularly useful if you would like to display JSON fields one by one, or handle function call arguments as soon as they are available.

这样，你无需等待整个响应完成后再处理它。
如果你希望逐个显示 JSON 字段，或在函数调用参数可用时立即处理它们，这尤其有用。

We recommend relying on the SDKs to handle streaming with Structured Outputs.

我们建议依赖 SDK 来处理 Structured Outputs 的流式传输。

```
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
from typing import List

from openai import OpenAI
from pydantic import BaseModel

class EntitiesModel(BaseModel):
    attributes: List[str]
    colors: List[str]
    animals: List[str]

client = OpenAI()

with client.responses.stream(
    model="gpt-4.1",
    input=[
        {"role": "system", "content": "Extract entities from the input text"},
        {
            "role": "user",
            "content": "The quick brown fox jumps over the lazy dog with piercing blue eyes",
        },
    ],
    text_format=EntitiesModel,
) as stream:
    for event in stream:
        if event.type == "response.refusal.delta":
            print(event.delta, end="")
        elif event.type == "response.output_text.delta":
            print(event.delta, end="")
        elif event.type == "response.error":
            print(event.error, end="")
        elif event.type == "response.completed":
            print("Completed") # print(event.response.output)

    final_response = stream.get_final_response()
    print(final_response)
```

Structured Outputs supports a subset of the [JSON Schema](https://json-schema.org/docs) language.

Structured Outputs 支持 [JSON Schema](https://json-schema.org/docs) 语言的一个子集。

#### Supported types / 支持的类型


The following types are supported for Structured Outputs:

Structured Outputs 支持以下类型：

* String
* 字符串
* Number
* 数字
* Boolean
* 布尔值
* Integer
* 整数
* Object
* 对象
* Array
* 数组
* Enum
* 枚举
* anyOf
* anyOf

#### Supported properties / 支持的属性


In addition to specifying the type of a property, you can specify a selection of additional constraints:

除了指定属性的类型外，你还可以指定一些额外的约束：

**Supported `string` properties:**

**支持的 `string` 属性：**

* `pattern` — A regular expression that the string must match.
* `pattern` — 字符串必须匹配的正则表达式。
* `format` — Predefined formats for strings. Currently supported:
  + `date-time`
  + `time`
  + `date`
  + `duration`
  + `email`
  + `hostname`
  + `ipv4`
  + `ipv6`
  + `uuid`
* `format` — 字符串的预定义格式。当前支持：
  + `date-time`
  + `time`
  + `date`
  + `duration`
  + `email`
  + `hostname`
  + `ipv4`
  + `ipv6`
  + `uuid`

**Supported `number` properties:**

**支持的 `number` 属性：**

* `multipleOf` — The number must be a multiple of this value.
* `multipleOf` — 该数字必须是此值的倍数。
* `maximum` — The number must be less than or equal to this value.
* `maximum` — 该数字必须小于或等于此值。
* `exclusiveMaximum` — The number must be less than this value.
* `exclusiveMaximum` — 该数字必须小于此值。
* `minimum` — The number must be greater than or equal to this value.
* `minimum` — 该数字必须大于或等于此值。
* `exclusiveMinimum` — The number must be greater than this value.
* `exclusiveMinimum` — 该数字必须大于此值。

**Supported `array` properties:**

**支持的 `array` 属性：**

* `minItems` — The array must have at least this many items.
* `minItems` — 数组必须至少包含这么多项。
* `maxItems` — The array must have at most this many items.
* `maxItems` — 数组必须最多包含这么多项。

Here are some examples on how you can use these type restrictions:

以下是一些关于如何使用这些类型限制的示例：

String Restrictions

字符串限制

```
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
{
    "name": "user_data",
    "strict": true,
    "schema": {
        "type": "object",
        "properties": {
            "name": {
                "type": "string",
                "description": "The name of the user"
            },
            "username": {
                "type": "string",
                "description": "The username of the user. Must start with @",
                "pattern": "^@[a-zA-Z0-9_]+$"
            },
            "email": {
                "type": "string",
                "description": "The email of the user",
                "format": "email"
            }
        },
        "additionalProperties": false,
        "required": [
            "name", "username", "email"
        ]
    }
}
```

Note these constraints are [not yet supported for fine-tuned models](#some-type-specific-keywords-are-not-yet-supported).

请注意，这些约束 [尚未支持微调模型](#some-type-specific-keywords-are-not-yet-supported)。

#### Root objects must not be `anyOf` and must be an object / 根对象不能是 `anyOf`，必须是对象


Note that the root level object of a schema must be an object, and not use `anyOf`. A pattern that appears in Zod (as one example) is using a discriminated union, which produces an `anyOf` at the top level. So code such as the following won't work:

请注意，schema 的根级对象必须是对象，不能使用 `anyOf`。Zod 中出现的一种模式是使用 discriminated union，这会在顶层产生一个 `anyOf`。因此，像下面这样的代码将无法工作：

```
1
2
3
4
5
6
7
8
9
10
11
12
13
import { z } from 'zod';
import { zodResponseFormat } from 'openai/helpers/zod';

const BaseResponseSchema = z.object({/* ... */});
const UnsuccessfulResponseSchema = z.object({/* ... */});

const finalSchema = z.discriminatedUnion('status', [
    BaseResponseSchema,
    UnsuccessfulResponseSchema,
]);

// Invalid JSON Schema for Structured Outputs
const json = zodResponseFormat(finalSchema, 'final_schema');
```

#### All fields must be `required` / 所有字段必须是 `required`


To use Structured Outputs, all fields or function parameters must be specified as `required`.

要使用 Structured Outputs，所有字段或函数参数都必须指定为 `required`。

```
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
{
    "name": "get_weather",
    "description": "Fetches the weather in the given location",
    "strict": true,
    "parameters": {
        "type": "object",
        "properties": {
            "location": {
                "type": "string",
                "description": "The location to get the weather for"
            },
            "unit": {
                "type": "string",
                "description": "The unit to return the temperature in",
                "enum": ["F", "C"]
            }
        },
        "additionalProperties": false,
        "required": ["location", "unit"]
    }
}
```

Although all fields must be required (and the model will return a value for each parameter), it is possible to emulate an optional parameter by using a union type with `null`.

虽然所有字段都必须是必需的（模型将为每个参数返回值），但可以通过使用与 `null` 的联合类型来模拟可选参数。

```
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
{
    "name": "get_weather",
    "description": "Fetches the weather in the given location",
    "strict": true,
    "parameters": {
        "type": "object",
        "properties": {
            "location": {
                "type": "string",
                "description": "The location to get the weather for"
            },
            "unit": {
                "type": ["string", "null"],
                "description": "The unit to return the temperature in",
                "enum": ["F", "C"]
            }
        },
        "additionalProperties": false,
        "required": [
            "location", "unit"
        ]
    }
}
```

#### Objects have limitations on nesting depth and size / 对象在嵌套深度和大小方面有限制


A schema may have up to 5000 object properties total, with up to 10 levels of nesting.

一个 schema 最多可以有 5000 个对象属性，最多 10 层嵌套。

#### Limitations on total string size / 总字符串大小限制


In a schema, total string length of all property names, definition names, enum values, and const values cannot exceed 120,000 characters.

在 schema 中，所有属性名称、定义名称、枚举值和常量值的总字符串长度不能超过 120,000 个字符。

#### Limitations on enum size / 枚举大小限制


A schema may have up to 1000 enum values across all enum properties.

一个 schema 在所有枚举属性中最多可以有 1000 个枚举值。

For a single enum property with string values, the total string length of all enum values cannot exceed 15,000 characters when there are more than 250 enum values.

对于具有字符串值的单个枚举属性，当枚举值超过 250 个时，所有枚举值的总字符串长度不能超过 15,000 个字符。

#### `additionalProperties: false` must always be set in objects / 对象中必须始终设置 `additionalProperties: false`


`additionalProperties` controls whether it is allowable for an object to contain additional keys / values that were not defined in the JSON Schema.

`additionalProperties` 控制对象是否允许包含 JSON Schema 中未定义的额外键/值。

Structured Outputs only supports generating specified keys / values, so we require developers to set `additionalProperties: false` to opt into Structured Outputs.

Structured Outputs 仅支持生成指定的键/值，因此我们要求开发者设置 `additionalProperties: false` 以选择使用 Structured Outputs。

```
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
{
    "name": "get_weather",
    "description": "Fetches the weather in the given location",
    "strict": true,
    "schema": {
        "type": "object",
        "properties": {
            "location": {
                "type": "string",
                "description": "The location to get the weather for"
            },
            "unit": {
                "type": "string",
                "description": "The unit to return the temperature in",
                "enum": ["F", "C"]
            }
        },
        "additionalProperties": false,
        "required": [
            "location", "unit"
        ]
    }
}
```

#### Key ordering / 键排序


When using Structured Outputs, outputs will be produced in the same order as the ordering of keys in the schema.

使用 Structured Outputs 时，输出将按照 schema 中键的顺序生成。

#### Some type-specific keywords are not yet supported / 某些特定类型的关键字尚不支持


* **Composition:** `allOf`, `not`, `dependentRequired`, `dependentSchemas`, `if`, `then`, `else`
* **组合：** `allOf`、`not`、`dependentRequired`、`dependentSchemas`、`if`、`then`、`else`

For fine-tuned models, we additionally do not support the following:

对于微调模型，我们 additionally 不支持以下功能：

* **For strings:** `minLength`, `maxLength`, `pattern`, `format`
* **对于字符串：** `minLength`、`maxLength`、`pattern`、`format`
* **For numbers:** `minimum`, `maximum`, `multipleOf`
* **对于数字：** `minimum`、`maximum`、`multipleOf`
* **For objects:** `patternProperties`
* **对于对象：** `patternProperties`
* **For arrays:** `minItems`, `maxItems`
* **对于数组：** `minItems`、`maxItems`

If you turn on Structured Outputs by supplying `strict: true` and call the API with an unsupported JSON Schema, you will receive an error.

如果你通过提供 `strict: true` 启用 Structured Outputs，并使用不支持的 JSON Schema 调用 API，你将收到错误。

#### For `anyOf`, the nested schemas must each be a valid JSON Schema per this subset / 对于 `anyOf`，嵌套的 schema 每个都必须是此子集的有效 JSON Schema


Here's an example supported anyOf schema:

以下是一个支持的 anyOf schema 示例：

```
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40
41
42
43
44
45
46
47
48
49
50
51
52
53
54
55
56
{
    "type": "object",
    "properties": {
        "item": {
            "anyOf": [
                {
                    "type": "object",
                    "description": "The user object to insert into the database",
                    "properties": {
                        "name": {
                            "type": "string",
                            "description": "The name of the user"
                        },
                        "age": {
                            "type": "number",
                            "description": "The age of the user"
                        }
                    },
                    "additionalProperties": false,
                    "required": [
                        "name",
                        "age"
                    ]
                },
                {
                    "type": "object",
                    "description": "The address object to insert into the database",
                    "properties": {
                        "number": {
                            "type": "string",
                            "description": "The number of the address. Eg. for 123 main st, this would be 123"
                        },
                        "street": {
                            "type": "string",
                            "description": "The street name. Eg. for 123 main st, this would be main st"
                        },
                        "city": {
                            "type": "string",
                            "description": "The city of the address"
                        }
                    },
                    "additionalProperties": false,
                    "required": [
                        "number",
                        "street",
                        "city"
                    ]
                }
            ]
        }
    },
    "additionalProperties": false,
    "required": [
        "item"
    ]
}
```

#### Definitions are supported / 支持定义


You can use definitions to define subschemas which are referenced throughout your schema. The following is a simple example.

你可以使用 definitions 来定义在整个 schema 中引用的子 schema。以下是一个简单的示例。

```
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
{
    "type": "object",
    "properties": {
        "steps": {
            "type": "array",
            "items": {
                "$ref": "#/$defs/step"
            }
        },
        "final_answer": {
            "type": "string"
        }
    },
    "$defs": {
        "step": {
            "type": "object",
            "properties": {
                "explanation": {
                    "type": "string"
                },
                "output": {
                    "type": "string"
                }
            },
            "required": [
                "explanation",
                "output"
            ],
            "additionalProperties": false
        }
    },
    "required": [
        "steps",
        "final_answer"
    ],
    "additionalProperties": false
}
```

#### Recursive schemas are supported / 支持递归 schema


Sample recursive schema using `#` to indicate root recursion.

使用 `#` 表示根递归的递归 schema 示例。

```
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40
41
42
43
44
45
46
47
{
    "name": "ui",
    "description": "Dynamically generated UI",
    "strict": true,
    "schema": {
        "type": "object",
        "properties": {
            "type": {
                "type": "string",
                "description": "The type of the UI component",
                "enum": ["div", "button", "header", "section", "field", "form"]
            },
            "label": {
                "type": "string",
                "description": "The label of the UI component, used for buttons or form fields"
            },
            "children": {
                "type": "array",
                "description": "Nested UI components",
                "items": {
                    "$ref": "#"
                }
            },
            "attributes": {
                "type": "array",
                "description": "Arbitrary attributes for the UI component, suitable for any element",
                "items": {
                    "type": "object",
                    "properties": {
                        "name": {
                            "type": "string",
                            "description": "The name of the attribute, for example onClick or className"
                        },
                        "value": {
                            "type": "string",
                            "description": "The value of the attribute"
                        }
                    },
                    "additionalProperties": false,
                    "required": ["name", "value"]
                }
            }
        },
        "required": ["type", "label", "children", "attributes"],
        "additionalProperties": false
    }
}
```

Sample recursive schema using explicit recursion:

使用显式递归的递归 schema 示例：

```
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
{
    "type": "object",
    "properties": {
        "linked_list": {
            "$ref": "#/$defs/linked_list_node"
        }
    },
    "$defs": {
        "linked_list_node": {
            "type": "object",
            "properties": {
                "value": {
                    "type": "number"
                },
                "next": {
                    "anyOf": [
                        {
                            "$ref": "#/$defs/linked_list_node"
                        },
                        {
                            "type": "null"
                        }
                    ]
                }
            },
            "additionalProperties": false,
            "required": [
                "next",
                "value"
            ]
        }
    },
    "additionalProperties": false,
    "required": [
        "linked_list"
    ]
}
```

JSON mode is a more basic version of the Structured Outputs feature. While JSON mode ensures that model output is valid JSON, Structured Outputs reliably matches the model's output to the schema you specify. We recommend you use Structured Outputs if it is supported for your use case.

JSON mode 是 Structured Outputs 功能的更基础版本。虽然 JSON mode 确保模型输出是有效的 JSON，但 Structured Outputs 能可靠地将模型输出与你指定的 schema 匹配。如果它支持你的用例，我们建议你使用 Structured Outputs。

When JSON mode is turned on, the model's output is ensured to be valid JSON, except for in some edge cases that you should detect and handle appropriately.

当 JSON mode 开启时，模型输出被确保为有效的 JSON，除了一些你应该适当检测和处理的边界情况。

To turn on JSON mode with the Responses API you can set the `text.format` to `{ "type": "json_object" }`. If you are using function calling, JSON mode is always turned on.

要在 Responses API 中开启 JSON mode，你可以将 `text.format` 设置为 `{ "type": "json_object" }`。如果你使用 function calling，JSON mode 始终处于开启状态。

Important notes:

重要提示：

* When using JSON mode, you must always instruct the model to produce JSON via some message in the conversation, for example via your system message. If you don't include an explicit instruction to generate JSON, the model may generate an unending stream of whitespace and the request may run continually until it reaches the token limit. To help ensure you don't forget, the API will throw an error if the string "JSON" does not appear somewhere in the context.
* 使用 JSON mode 时，你必须始终通过在对话中的某条消息（例如系统消息）指示模型生成 JSON。如果你未包含生成 JSON 的明确指令，模型可能会生成一连串不间断的空白字符，并且请求可能会持续运行直到达到 token 限制。为了帮助你确保不会忘记，如果上下文中某处未出现字符串 "JSON"，API 将抛出错误。
* JSON mode will not guarantee the output matches any specific schema, only that it is valid and parses without errors. You should use Structured Outputs to ensure it matches your schema, or if that is not possible, you should use a validation library and potentially retries to ensure that the output matches your desired schema.
* JSON mode 不保证输出符合任何特定的 schema，只保证它是有效的且可以无错误解析。你应该使用 Structured Outputs 来确保它符合你的 schema，如果不可能，你应该使用验证库并可能需要重试，以确保输出符合你期望的 schema。
* Your application must detect and handle the edge cases that can result in the model output not being a complete JSON object (see below)
* 你的应用必须检测和处理可能导致模型输出不是完整 JSON 对象的边界情况（见下文）

Handling edge cases

处理边界情况

```
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40
41
42
we_did_not_specify_stop_tokens = True

try:
    response = client.responses.create(
        model="gpt-3.5-turbo-0125",
        input=[
            {"role": "system", "content": "You are a helpful assistant designed to output JSON."},
            {"role": "user", "content": "Who won the world series in 2020? Please respond in the format {winner: ...}"}
        ],
        text={"format": {"type": "json_object"}}
    )

    # Check if the conversation was too long for the context window, resulting in incomplete JSON
    if response.status == "incomplete" and response.incomplete_details.reason == "max_output_tokens":
        # your code should handle this error case
        pass

    # Check if the OpenAI safety system refused the request and generated a refusal instead
    if response.output[0].content[0].type == "refusal":
        # your code should handle this error case
        # In this case, the .content field will contain the explanation (if any) that the model generated for why it is refusing
        print(response.output[0].content[0]["refusal"])

    # Check if the model's output included restricted content, so the generation of JSON was halted and may be partial
    if response.status == "incomplete" and response.incomplete_details.reason == "content_filter":
        # your code should handle this error case
        pass

    if response.status == "completed":
        # In this case the model has either successfully finished generating the JSON object according to your schema, or the model generated one of the tokens you provided as a "stop token"

        if we_did_not_specify_stop_tokens:
            # If you didn't specify any stop tokens, then the generation is complete and the content key will contain the serialized JSON object
            # This will parse successfully and should now contain  "{"winner": "Los Angeles Dodgers"}"
            print(response.output_text)
        else:
            # Check if the response.output_text ends with one of your stop tokens and handle appropriately
            pass
except Exception as e:
    # Your code should handle errors here, for example a network error calling the API
    print(e)
```

To learn more about Structured Outputs, we recommend browsing the following resources:

要了解有关 Structured Outputs 的更多信息，我们建议浏览以下资源：

* Check out our [introductory cookbook](/cookbook/examples/structured_outputs_intro) on Structured Outputs
* 查看我们关于 Structured Outputs 的 [入门 cookbook](/cookbook/examples/structured_outputs_intro)
* Learn [how to build multi-agent systems](/cookbook/examples/structured_outputs_multi_agent) with Structured Outputs
* 学习如何使用 Structured Outputs [构建多 agent 系统](/cookbook/examples/structured_outputs_multi_agent)
