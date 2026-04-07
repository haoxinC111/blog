# Working with Evals | OpenAI API

> **Source / 原文来源**: https://developers.openai.com/api/docs/guides/evals/  
> **Compiled for study on / 整理日期**: 2026-04-07

Evaluations (often called **evals**) test model outputs to ensure they meet style and content criteria that you specify. Writing evals to understand how your LLM applications are performing against your expectations, especially when upgrading or trying new models, is an essential component to building reliable applications.

评估（通常称为 **evals**）用于测试模型输出，以确保它们符合你指定的风格和内容标准。编写 evals 以了解你的 LLM 应用相对于你的期望表现如何，尤其是在升级或尝试新模型时，这是构建可靠应用的重要组成部分。

In this guide, we will focus on **configuring evals programmatically using the [Evals API](/api/docs/api-reference/evals)**. If you prefer, you can also configure evals [in the OpenAI dashboard](https://platform.openai.com/evaluations).

在本指南中，我们将重点介绍**如何使用 [Evals API](/api/docs/api-reference/evals) 以编程方式配置 evals**。如果你愿意，也可以在 [OpenAI dashboard](https://platform.openai.com/evaluations) 中配置 evals。

If you’re new to evaluations, or want a more iterative environment to experiment in as you build your eval, consider trying [Datasets](/api/docs/guides/evaluation-getting-started) instead.

如果你刚接触评估，或者希望在构建 eval 时有一个更具迭代性的实验环境，可以考虑尝试 [Datasets](/api/docs/guides/evaluation-getting-started)。

Broadly, there are three steps to build and run evals for your LLM application.

总的来说，为你的 LLM 应用构建和运行 evals 有三个步骤。

1. Describe the task to be done as an eval
   将要完成的任务描述为一个 eval
2. Run your eval with test inputs (a prompt and input data)
   使用测试输入（一个 prompt 和输入数据）运行你的 eval
3. Analyze the results, then iterate and improve on your prompt
   分析结果，然后迭代并改进你的 prompt

This process is somewhat similar to behavior-driven development (BDD), where you begin by specifying how the system should behave before implementing and testing the system. Let’s see how we would complete each of the steps above using the [Evals API](/api/docs/api-reference/evals).

这个过程有点类似于行为驱动开发（BDD），即你在实现和测试系统之前先指定系统应该如何表现。让我们看看如何使用 [Evals API](/api/docs/api-reference/evals) 完成上述每个步骤。

Creating an eval begins by describing a task to be done by a model. Let’s say that we would like to use a model to classify the contents of IT support tickets into one of three categories: `Hardware`, `Software`, or `Other`.

创建 eval 始于描述模型要完成的任务。假设我们想使用模型将 IT 支持工单的内容分类为三个类别之一：`Hardware`、`Software` 或 `Other`。

To implement this use case, you can use either the [Chat Completions API](/api/docs/api-reference/chat) or the [Responses API](/api/docs/api-reference/responses). Both examples below combine a [developer message](/api/docs/guides/text) with a user message containing the text of a support ticket.

要实现这个用例，你可以使用 [Chat Completions API](/api/docs/api-reference/chat) 或 [Responses API](/api/docs/api-reference/responses)。下面的两个示例都将一个 [developer message](/api/docs/guides/text) 与包含支持工单文本的 user message 结合在一起。

```python
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
from openai import OpenAI
client = OpenAI()

instructions = """
You are an expert in categorizing IT support tickets. Given the support
ticket below, categorize the request into one of "Hardware", "Software",
or "Other". Respond with only one of those words.
"""

ticket = "My monitor won't turn on - help!"

response = client.responses.create(
    model="gpt-4.1",
    input=[
        {"role": "developer", "content": instructions},
        {"role": "user", "content": ticket},
    ],
)

print(response.output_text)
```

Let’s set up an eval to test this behavior [via API](/api/docs/api-reference/evals). An eval needs two key ingredients:

让我们通过一个 [API](/api/docs/api-reference/evals) 设置一个 eval 来测试这个行为。一个 eval 需要两个关键要素：

* `data_source_config`: A schema for the test data you will use along with the eval.
  `data_source_config`：你将与 eval 一起使用的测试数据的 schema。
* `testing_criteria`: The [graders](/api/docs/guides/graders) that determine if the model output is correct.
  `testing_criteria`：[graders](/api/docs/guides/graders)，用于判断模型输出是否正确。

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
curl https://api.openai.com/v1/evals \
    -H "Authorization: Bearer $OPENAI_API_KEY" \
    -H "Content-Type: application/json" \
    -d '{
        "name": "IT Ticket Categorization",
        "data_source_config": {
            "type": "custom",
            "item_schema": {
                "type": "object",
                "properties": {
                    "ticket_text": { "type": "string" },
                    "correct_label": { "type": "string" }
                },
                "required": ["ticket_text", "correct_label"]
            },
            "include_sample_schema": true
        },
        "testing_criteria": [
            {
                "type": "string_check",
                "name": "Match output to human label",
                "input": "{{ sample.output_text }}",
                "operation": "eq",
                "reference": "{{ item.correct_label }}"
            }
        ]
    }'
```

**Explanation: `data_source_config` parameter**

**说明：`data_source_config` 参数**

Running this eval will require a test data set that represents the type of data you expect your prompt to work with (more on creating the test data set later in this guide). In our `data_source_config` parameter, we specify that each **item** in the data set will conform to a [JSON schema](https://json-schema.org/) with two properties:

运行这个 eval 需要一个测试数据集，它代表你希望你的 prompt 能够处理的数据类型（关于如何创建测试数据集的更多信息见本指南后面）。在我们的 `data_source_config` 参数中，我们指定数据集中的每个 **item** 都将符合具有两个属性的 [JSON schema](https://json-schema.org/)：

* `ticket_text`: a string of text with the contents of a support ticket
  `ticket_text`：一个包含支持工单内容的字符串
* `correct_label`: a “ground truth” output that the model should match, provided by a human
  `correct_label`：模型应该匹配的“真实标签”输出，由人工提供

Since we will be referencing a **sample** in our test criteria (the output generated by a model given our prompt), we also set `include_sample_schema` to `true`.

由于我们将在测试 criteria 中引用一个 **sample**（给定我们的 prompt 后模型生成的输出），我们还将 `include_sample_schema` 设置为 `true`。

```json
{
  "type": "custom",
  "item_schema": {
    "type": "object",
    "properties": {
      "ticket": { "type": "string" },
      "category": { "type": "string" }
    },
    "required": ["ticket", "category"]
  },
  "include_sample_schema": true
}
```

**Explanation: `testing_criteria` parameter**

**说明：`testing_criteria` 参数**

In our `testing_criteria`, we define how we will conclude if the model output satisfies our requirements for each item in the data set. In this case, we just want the model to output one of three category strings based on the input ticket. The string it outputs should exactly match the human-labeled `correct_label` field in our test data. So in this case, we will want to use a `string_check` grader to evaluate the output.

在我们的 `testing_criteria` 中，我们定义如何判断模型输出是否满足数据集中每个 item 的要求。在这个例子中，我们只希望模型根据输入的工单单输出三个类别字符串之一。它输出的字符串应该与我们测试数据中人肉标注的 `correct_label` 字段完全匹配。因此在这种情况下，我们将使用 `string_check` grader 来评估输出。

```json
{
  "type": "string_check",
  "name": "Category string match",
  "input": "{{ sample.output_text }}",
  "operation": "eq",
  "reference": "{{ item.category }}"
}
```

After creating the eval, it will be assigned a UUID that you will need to address it later when kicking off a run.

创建 eval 后，它会分配一个 UUID，你在稍后启动运行时需要用它来引用该 eval。

```json
{
  "object": "eval",
  "id": "eval_67e321d23b54819096e6bfe140161184",
  "data_source_config": {
    "type": "custom",
    "schema": { ... omitted for brevity... }
  },
  "testing_criteria": [
    {
      "name": "Match output to human label",
      "id": "Match output to human label-c4fdf789-2fa5-407f-8a41-a6f4f9afd482",
      "type": "string_check",
      "input": "{{ sample.output_text }}",
      "reference": "{{ item.correct_label }}",
      "operation": "eq"
    }
  ],
  "name": "IT Ticket Categorization",
  "created_at": 1742938578,
  "metadata": {}
}
```

Now that we’ve created an eval that describes the desired behavior of our application, let’s test a prompt with a set of test data.

现在我们已经创建了一个描述应用期望行为的 eval，让我们用一组测试数据来测试一个 prompt。

Now that we have defined how we want our app to behave in an eval, let’s construct a prompt that reliably generates the correct output for a representative sample of test data.

既然我们已经在 eval 中定义了希望应用如何表现，让我们构建一个 prompt，使其能够为有代表性的测试数据样本可靠地生成正确输出。

### Uploading test data

上传测试数据

There are several ways to provide test data for eval runs, but it may be convenient to upload a [JSONL](https://jsonlines.org/) file that contains data in the schema we specified when we created our eval. A sample JSONL file that conforms to the schema we set up is below:

为 eval 运行提供测试数据有多种方式，但上传一个包含符合我们创建 eval 时指定 schema 数据的 [JSONL](https://jsonlines.org/) 文件可能更方便。下面是一个符合我们设置 schema 的 JSONL 文件示例：

```jsonl
1
2
3
{ "item": { "ticket_text": "My monitor won't turn on!", "correct_label": "Hardware" } }
{ "item": { "ticket_text": "I'm in vim and I can't quit!", "correct_label": "Software" } }
{ "item": { "ticket_text": "Best restaurants in Cleveland?", "correct_label": "Other" } }
```

This data set contains both test inputs and ground truth labels to compare model outputs against.

此数据集包含测试输入和用于与模型输出进行比较的真实标签。

Next, let’s upload our test data file to the OpenAI platform so we can reference it later. You can upload files [in the dashboard here](https://platform.openai.com/storage/files), but it’s possible to [upload files via API](/api/docs/api-reference/files/create) as well. The samples below assume you are running the command in a directory where you saved the sample JSON data above to a file called `tickets.jsonl`:

接下来，让我们将测试数据文件上传到 OpenAI 平台，以便稍后引用。你可以在 [这里的仪表板](https://platform.openai.com/storage/files) 中上传文件，但也可以通过 [API 上传文件](/api/docs/api-reference/files/create)。下面的示例假设你在一个目录中运行命令，并将上面的示例 JSON 数据保存到名为 `tickets.jsonl` 的文件中：

```bash
curl https://api.openai.com/v1/files \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -F purpose="evals" \
  -F file="@tickets.jsonl"
```

When you upload the file, make note of the unique `id` property in the response payload (also available in the UI if you uploaded via the browser) - we will need to reference that value later:

上传文件时，请注意响应载荷中唯一的 `id` 属性（如果你通过浏览器上传，也可以在 UI 中看到）——我们稍后需要引用该值：

```json
{
  "object": "file",
  "id": "file-CwHg45Fo7YXwkWRPUkLNHW",
  "purpose": "evals",
  "filename": "tickets.jsonl",
  "bytes": 208,
  "created_at": 1742834798,
  "expires_at": null,
  "status": "processed",
  "status_details": null
}
```

### Creating an eval run

创建 eval run

With our test data in place, let’s evaluate a prompt and see how it performs against our test criteria. Via API, we can do this by [creating an eval run](/api/docs/api-reference/evals/createRun).

有了测试数据，让我们来评估一个 prompt，看看它在我们的测试 criteria 下表现如何。通过 API，我们可以通过 [creating an eval run](/api/docs/api-reference/evals/createRun) 来实现。

Make sure to replace `YOUR_EVAL_ID` and `YOUR_FILE_ID` with the unique IDs of the eval configuration and test data files you created in the steps above.

确保将 `YOUR_EVAL_ID` 和 `YOUR_FILE_ID` 替换为你在上述步骤中创建的 eval 配置和测试数据文件的唯一 ID。

```bash
curl https://api.openai.com/v1/evals/YOUR_EVAL_ID/runs \
    -H "Authorization: Bearer $OPENAI_API_KEY" \
    -H "Content-Type: application/json" \
    -d '{
        "name": "Categorization text run",
        "data_source": {
            "type": "responses",
            "model": "gpt-4.1",
            "input_messages": {
                "type": "template",
                "template": [
                    {"role": "developer", "content": "You are an expert in categorizing IT support tickets. Given the support ticket below, categorize the request into one of Hardware, Software, or Other. Respond with only one of those words."},
                    {"role": "user", "content": "{{ item.ticket_text }}"}
                ]
            },
            "source": { "type": "file_id", "id": "YOUR_FILE_ID" }
        }
    }'
```

When we create the run, we set up a prompt using either a [Chat Completions](/api/docs/guides/text?api-mode=chat) messages array or a [Responses](/api/docs/api-reference/responses) input.

当我们创建运行时，我们使用 [Chat Completions](/api/docs/guides/text?api-mode=chat) 消息数组或 [Responses](/api/docs/api-reference/responses) 输入来设置一个 prompt。

This prompt is used to generate a model response for every line of test data in your data set. We can use the double curly brace syntax to template in the dynamic variable `item.ticket_text`, which is drawn from the current test data item.

这个 prompt 用于为数据集中每一行测试数据生成模型响应。我们可以使用双花括号语法来模板化动态变量 `item.ticket_text`，它来自当前测试数据 item。

If the eval run is successfully created, you’ll receive an API response that looks like this:

如果 eval run 创建成功，你会收到如下所示的 API 响应：

```json
{
    "object": "eval.run",
    "id": "evalrun_67e44c73eb6481909f79a457749222c7",
    "eval_id": "eval_67e44c5becec81909704be0318146157",
    "report_url": "https://platform.openai.com/evaluation/evals/abc123",
    "status": "queued",
    "model": "gpt-4.1",
    "name": "Categorization text run",
    "created_at": 1743015028,
    "result_counts": { ... },
    "per_model_usage": null,
    "per_testing_criteria_results": null,
    "data_source": {
        "type": "responses",
        "source": {
            "type": "file_id",
            "id": "file-J7MoX9ToHXp2TutMEeYnwj"
        },
        "input_messages": {
            "type": "template",
            "template": [
                {
                    "type": "message",
                    "role": "developer",
                    "content": {
                        "type": "input_text",
                        "text": "You are an expert in...."
                    }
                },
                {
                    "type": "message",
                    "role": "user",
                    "content": {
                        "type": "input_text",
                        "text": "{{item.ticket_text}}"
                    }
                }
            ]
        },
        "model": "gpt-4.1",
        "sampling_params": null
    },
    "error": null,
    "metadata": {}
}
```

Your eval run has now been queued, and it will execute asynchronously as it processes every row in your data set, generating responses for testing with the prompt and model we specified.

你的 eval run 现在已经进入队列，它将异步执行，处理数据集中的每一行，并使用我们指定的 prompt 和模型生成响应进行测试。

To receive updates when a run succeeds, fails, or is canceled, create a webhook endpoint and subscribe to the `eval.run.succeeded`, `eval.run.failed`, and `eval.run.canceled` events. See the [webhooks guide](/api/docs/guides/webhooks) for more details.

要在运行成功、失败或被取消时接收更新，请创建一个 webhook 端点并订阅 `eval.run.succeeded`、`eval.run.failed` 和 `eval.run.canceled` 事件。更多详情请参见 [webhooks guide](/api/docs/guides/webhooks)。

Depending on the size of your dataset, the eval run may take some time to complete. You can view current status in the dashboard, but you can also [fetch the current status of an eval run via API](/api/docs/api-reference/evals/getRun):

根据数据集的大小，eval run 可能需要一些时间才能完成。你可以在仪表板中查看当前状态，但也可以通过 API [获取 eval run 的当前状态](/api/docs/api-reference/evals/getRun)：

```bash
curl https://api.openai.com/v1/evals/YOUR_EVAL_ID/runs/YOUR_RUN_ID \
    -H "Authorization: Bearer $OPENAI_API_KEY" \
    -H "Content-Type: application/json"
```

You’ll need the UUID of both your eval and eval run to fetch its status. When you do, you’ll see eval run data that looks like this:

你需要 eval 和 eval run 的 UUID 才能获取其状态。当你这样做时，你会看到如下 eval run 数据：

```json
{
    "object": "eval.run",
    "id": "evalrun_67e44c73eb6481909f79a457749222c7",
    "eval_id": "eval_67e44c5becec81909704be0318146157",
    "report_url": "https://platform.openai.com/evaluation/evals/xxx",
    "status": "completed",
    "model": "gpt-4.1",
    "name": "Categorization text run",
    "created_at": 1743015028,
    "result_counts": {
        "total": 3,
        "errored": 0,
        "failed": 0,
        "passed": 3
    },
    "per_model_usage": [
        {
            "model_name": "gpt-4o-2024-08-06",
            "invocation_count": 3,
            "prompt_tokens": 166,
            "completion_tokens": 6,
            "total_tokens": 172,
            "cached_tokens": 0
        }
    ],
    "per_testing_criteria_results": [
        {
            "testing_criteria": "Match output to human label-40d67441-5000-4754-ab8c-181c125803ce",
            "passed": 3,
            "failed": 0
        }
    ],
    "data_source": {
        "type": "responses",
        "source": {
            "type": "file_id",
            "id": "file-J7MoX9ToHXp2TutMEeYnwj"
        },
        "input_messages": {
            "type": "template",
            "template": [
                {
                    "type": "message",
                    "role": "developer",
                    "content": {
                        "type": "input_text",
                        "text": "You are an expert in categorizing IT support tickets. Given the support ticket below, categorize the request into one of Hardware, Software, or Other. Respond with only one of those words."
                    }
                },
                {
                    "type": "message",
                    "role": "user",
                    "content": {
                        "type": "input_text",
                        "text": "{{item.ticket_text}}"
                    }
                }
            ]
        },
        "model": "gpt-4.1",
        "sampling_params": null
    },
    "error": null,
    "metadata": {}
}
```

The API response contains granular information about test criteria results, API usage for generating model responses, and a `report_url` property that takes you to a page in the dashboard where you can explore the results visually.

API 响应包含有关测试 criteria 结果、生成模型响应的 API 使用情况的详细信息，以及一个 `report_url` 属性，它会带你进入仪表板中一个可以直观地探索结果的页面。

In our simple test, the model reliably generated the content we wanted for a small test case sample. In reality, you will often have to run your eval with more criteria, different prompts, and different data sets. But the process above gives you all the tools you need to build robust evals for your LLM apps!

在我们这个简单的测试中，模型为一个小的测试用例样本可靠地生成了我们想要的内容。实际上，你通常需要使用更多的 criteria、不同的 prompts 和不同的数据集来运行 evals。但上述过程为你提供了构建稳健 LLM 应用 evals 所需的全部工具！

Now you know how to create and run evals via API, and using the dashboard! Here are a few other resources that may be useful to you as you continue to improve your model results.

现在你已经知道如何通过 API 以及使用仪表板来创建和运行 evals 了！以下是一些其他资源，它们可能在你继续改进模型结果时对你有所帮助。

- [Cookbook: Detecting prompt regressions](https://cookbook.openai.com/examples/evaluation/use-cases/regression) — Keep tabs on the performance of your prompts as you iterate on them.
  在迭代提示时跟踪提示的性能。
- [Cookbook: Bulk model and prompt experimentation](https://cookbook.openai.com/examples/evaluation/use-cases/bulk-experimentation) — Compare the results of many different prompts and models at once.
  一次性比较许多不同提示和模型的结果。
- [Cookbook: Monitoring stored completions](https://cookbook.openai.com/examples/evaluation/use-cases/completion-monitoring) — Examine stored completions to test for prompt regressions.
  检查存储的 completions 以测试提示回归。
- [Fine-tuning](/api/docs/guides/fine-tuning) — Improve a model’s ability to generate responses tailored to your use case.
  提高模型生成针对你用例定制的响应的能力。
- [Model distillation](/api/docs/guides/distillation) — Learn how to distill large model results to smaller, cheaper, and faster models.
  学习如何将大模型的结果蒸馏到更小、更便宜、更快的模型。
