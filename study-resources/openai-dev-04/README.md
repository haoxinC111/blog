# openai-dev-04: Working with Evals | OpenAI API

- **来源 URL**: https://developers.openai.com/api/docs/guides/evals/
- **整理日期**: 2026-04-07

## 核心要点

- Evaluations（evals）用于测试模型输出，以确保符合风格和内容标准，是构建可靠 LLM 应用的关键。
- 本指南聚焦使用 Evals API 以编程方式配置和运行评估（也可在 Dashboard 中配置）。
- 三大步骤：1) 描述任务并创建 eval（定义 `data_source_config` 和 `testing_criteria`）；2) 上传 JSONL 测试数据并创建 eval run；3) 分析结果并迭代改进 prompt。
- `data_source_config` 指定测试数据的 JSON schema；`testing_criteria` 使用 graders（如 `string_check`）判断模型输出是否正确。
- 可通过 webhook 接收 eval run 状态更新，或在 Dashboard 中通过 `report_url` 可视化查看结果。

## 文档链接

- 主文档: https://developers.openai.com/api/docs/guides/evals/
- 双语全文: [openai-dev-04-working-with-evals-bilingual.md](./openai-dev-04-working-with-evals-bilingual.md)
