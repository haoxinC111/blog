# Working with Evals | OpenAI API

- **Source URL**: https://developers.openai.com/api/docs/guides/evals/
- **整理日期**: 2026-04-07
---

Evaluations (evals) test your model's outputs to ensure they meet your style and content standards.

Evaluations（evals）用于测试模型的输出，以确保其符合你的风格和内容标准。

This guide focuses on configuring evaluations using the Evals API.

本指南重点介绍如何使用 Evals API 配置评估。

The process involves three main steps: 1) describing your task as an evaluation, 2) running the evaluation with test inputs, and 3) analyzing the results and iterating to improve.

该流程包含三个主要步骤：1) 将任务描述为评估，2) 使用测试输入运行评估，3) 分析结果并迭代改进。

---

## Example: Categorizing IT Support Tickets

 Below is an example of generating a classification for an IT support ticket using the `Responses API`.

 以下示例展示了如何使用 `Responses API` 为 IT 支持工单生成分类。

 ```python
 from openai import OpenAI

 client = OpenAI()
 instructions = "You are an expert in categorizing IT support tickets..."
 response = client.responses.create(
     model="gpt-4.1",
     input=[
         {"role": "developer", "content": instructions},
         {"role": "user", "content": ticket},
     ],
 )
 ```

## Creating an Eval

 To create an eval, you will need:

- `data_source_config`: the schema for your testing data
- `testing_criteria`: graders that determine whether the model output is correct

 创建 eval 需要：

- `data_source_config`：测试数据的 schema
- `testing_criteria`：评分器（graders），用于决定模型输出是否正确

## Example JSONL Testing Data

 ```jsonl
 { "item": { "ticket_text": "My monitor won't turn on!", "correct_label": "Hardware" } }
 ```

 After uploading the file, you can create an eval run via the API, and then view the results in the dashboard.

 上传文件后，可通过 API 创建 eval run，然后在 Dashboard 中查看结果。
