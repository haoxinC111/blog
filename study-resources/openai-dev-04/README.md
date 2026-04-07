# Working with Evals

- **Source URL**: https://developers.openai.com/api/docs/guides/evals/
- **整理日期**: 2026-04-07

## 核心要点

- Evaluations（evals）测试模型输出以确保符合风格和内容标准。
- 本指南关注用 Evals API 配置评估。
- 步骤：1) 描述任务作为评估；2) 用测试输入运行评估；3) 分析结果并迭代改进。
- 示例场景：将 IT 支持工单分类为 Hardware/Software/Other。
- 创建 eval 需要 `data_source_config`（测试数据 schema）和 `testing_criteria`（评分器决定输出是否正确）。
- 上传 JSONL 测试数据后，通过 API 创建 eval run，然后在 Dashboard 中查看结果。

## 文档链接

- [OpenAI API Evals Guide](https://developers.openai.com/api/docs/guides/evals/)
