# openai-01: A Practical Guide to Building Agents

| 字段 | 内容 |
|:---|:---|
| **来源** | [OpenAI Business Guides and Resources](https://cdn.openai.com/business-guides-and-resources/a-practical-guide-to-building-agents.pdf) |
| **60天计划引用** | [OpenAI][1] |
| **文档类型** | 中英对照 |
| **整理日期** | 2026-04-07 |

## 核心要点

- 智能体定义：能够独立代表用户完成任务的系统，用LLM控制工作流执行并动态选择工具
- 适合构建智能体的三类场景：复杂决策、难以维护的规则、严重依赖非结构化数据
- 设计基础：模型（Model）、工具（Tools）、指令（Instructions）三要素
- 编排模式：单智能体优先；多智能体分管理器模式（manager）和去中心化移交模式（hand-off）
- 护栏体系：分层防御，包括相关性/安全分类器、PII过滤、内容审核、工具安全措施、基于规则的保护
- 高风险操作必须设置人工干预（Human-in-the-loop）

## 文档

- [`openai-01-practical-guide-to-building-agents-bilingual.md`](./openai-01-practical-guide-to-building-agents-bilingual.md) — 完整中英对照正文
