# Evaluation Best Practices | OpenAI API

> 来源：https://developers.openai.com/api/docs/guides/evaluation-best-practices/

---

## 核心要点 / Key Takeaways

- 生成式 AI 具有变异性，传统软件测试方法对 AI 架构不够充分。
  Generative AI is variable; traditional software testing methods are insufficient for AI-powered architectures.

- Evals 是测试 AI 系统的方法，despite nondeterminism。
  Evals are the way to test AI systems, despite their nondeterminism.

- 是提高 LLM 应用性能的唯一途径之一（通过 fine-tuning）。
  They are one of the only ways to improve LLM application performance (via fine-tuning).

- Eval 类型：
  Types of evals:
  1. 行业基准（如 MMLU 等）— Industry benchmarks (e.g., MMLU)
  2. 标准数值分数（如 ROUGE、BERTScore）— Standard numeric scores (e.g., ROUGE, BERTScore)
  3. 自定义 LLM 应用性能测试 — Custom LLM application performance tests

- 本指南聚焦第三种：设计自己的 evals。
  This guide focuses on the third type: designing your own evals.

- Anti-patterns（应避免的做法）：
  Anti-patterns to avoid:
  - 过度泛化指标 — Over-generalizing metrics
  - 有偏见的设计 — Biased design
  - 凭感觉评估 — Evaluating by gut feeling
  - 忽视人类反馈 — Ignoring human feedback

- Eval 工作流组件：
  Eval workflow components:
  1. 定义目标 — Define objectives
  2. 收集数据集 — Collect datasets
  3. 定义指标 — Define metrics
  4. 运行比较 — Run comparisons
  5. 持续评估 — Continuous evaluation

- 示例：
  Examples:
  - 摘要成绩单：ROUGE-L >= 0.40, G-Eval coherence >= 80%
    Summarization report card: ROUGE-L >= 0.40, G-Eval coherence >= 80%
  - 文档问答：Context recall >= 0.85, precision > 0.7
    Document Q&A: Context recall >= 0.85, precision > 0.7

- LLMs 更擅长判别任务（成对比较、分类、评分），评估应聚焦于此。
  LLMs excel at discriminative tasks (pairwise comparison, classification, scoring); evaluations should focus on these.

- 创建 eval 数据集时，可用 o3 和 GPT-4.1 生成多样化测试数据，包含典型、边界和对抗案例。
  When creating eval datasets, use o3 and GPT-4.1 to generate diverse test data, including typical, edge, and adversarial cases.
