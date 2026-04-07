# Introducing OpenClaw

**Source URL:** https://openclaw.ai/blog/introducing-openclaw  
**整理日期:** 2026-04-07

---

Two months ago, I hacked together a weekend project. What started as "WhatsApp Relay" now has over 100,000 GitHub stars and drew 2 million visitors in a single week.

两个月前，我在一个周末随手搞了一个小项目。最初名为 "WhatsApp Relay" 的项目，如今在 GitHub 上已获得超过 100,000 颗星，并在单周内吸引了 200 万访客。

Today, I'm excited to announce our new name: **OpenClaw**.

今天，我很高兴地宣布我们的新名字：**OpenClaw**。

## The Naming Journey

## 命名之旅

We've been through some names.

我们经历了好几轮取名。

**Clawd** was born in November 2025—a playful pun on "Claude" with a claw. It felt perfect until Anthropic's legal team politely asked us to reconsider. Fair enough.

**Clawd** 诞生于 2025 年 11 月——一个将 "Claude" 与 "claw（爪子）" 结合的双关玩笑。在我们看来它近乎完美，直到 Anthropic 的法务团队礼貌地要求我们重新考虑。这也很合理。

**Moltbot** came next, chosen in a chaotic 5am Discord brainstorm with the community. Molting represents growth - lobsters shed their shells to become something bigger. It was meaningful, but [it never quite rolled off the tongue](https://x.com/NetworkChuck/status/2016254397496414317).

接下来是 **Moltbot**，这个名字来自社区一次凌晨 5 点在 Discord 上的混乱头脑风暴。蜕壳（Molting）象征着成长——龙虾蜕去外壳以变得更强大。它很有意义，但[总是有些拗口](https://x.com/NetworkChuck/status/2016254397496414317)。

**OpenClaw** is where we land. And this time, we did our homework: trademark searches came back clear, domains have been purchased, migration code has been written. The name captures what this project has become:

**OpenClaw** 是我们最终的落点。这一次，我们做足了功课：商标检索结果清晰，域名已购买，迁移代码已编写完毕。这个名字精准地捕捉了项目当下的本质：

* **Open**: Open source, open to everyone, community-driven
* **Claw**: Our lobster heritage, a nod to where we came from

* **Open**：开源、对所有人开放、社区驱动
* **Claw**：我们的龙虾血统，向我们的起源致敬

## What OpenClaw Is

## 什么是 OpenClaw

OpenClaw is an open agent platform that runs on your machine and works from the chat apps you already use. WhatsApp, Telegram, Discord, Slack, Teams—wherever you are, your AI assistant follows.

OpenClaw 是一个开放的 Agent 平台，运行在你的机器上，并直接在你已经在使用的聊天应用中工作。WhatsApp、Telegram、Discord、Slack、Teams——无论你在哪里，你的 AI 助手都随行。

**Your assistant. Your machine. Your rules.**

**你的助手。你的机器。你的规则。**

Unlike SaaS assistants where your data lives on someone else's servers, OpenClaw runs where you choose—laptop, homelab, or VPS. Your infrastructure. Your keys. Your data.

与数据存放在他人服务器上的 SaaS 助手不同，OpenClaw 运行在你选择的地方——笔记本电脑、家庭实验室或 VPS。你的基础设施。你的密钥。你的数据。

## What's New in This Release

## 本次发布的新内容

Along with the rebrand, we're shipping:

除了品牌重塑，我们还将发布：

* **New Channels**: Twitch and Google Chat plugins
* **Models**: Support for KIMI K2.5 & Xiaomi MiMo-V2-Flash
* **Web Chat**: Send images just like you can in messaging apps
* **Security**: 34 security-related commits to harden the codebase

* **新 Channels**：Twitch 和 Google Chat 插件
* **新模型**：支持 KIMI K2.5 和 Xiaomi MiMo-V2-Flash
* **Web 聊天**：像在消息应用中一样发送图片
* **安全性**：34 个与安全相关的提交，以加固代码库

I'd like to thank all security folks for their hard work in helping us harden the project. We've released [machine-checkable security models](https://github.com/vignesh07/clawdbot-formal-models) this week and are continuing to work on additional security improvements. Remember that prompt injection is still an industry-wide unsolved problem, so it's important to use strong models and to study our [security best practices](https://docs.openclaw.ai/gateway/security).

我要感谢所有安全领域贡献者的辛勤工作，帮助我们加固项目。我们本周发布了[机器可验证的安全模型](https://github.com/vignesh07/clawdbot-formal-models)，并正在继续进行额外的安全改进。请记住，提示词注入仍然是整个行业尚未解决的问题，因此使用强大的模型并学习我们的[安全最佳实践](https://docs.openclaw.ai/gateway/security)非常重要。

## The Road Ahead

## 未来之路

What's next? Security remains our top priority. We're also focused on gateway reliability and adding polish plus support for more models and providers.

接下来呢？安全仍然是我们的首要任务。我们还将专注于网关可靠性、提升产品打磨度，以及支持更多模型和提供商。

This project has grown far beyond what I could maintain alone. Over the last few days I've worked on adding maintainers and we're slowly setting up processes so we can deal with the insane influx of PRs and Issues. I'm also figuring out how to pay maintainers properly—full-time if possible. If you wanna help, consider [contributing](https://github.com/openclaw/openclaw/blob/main/CONTRIBUTING.md) or [sponsoring the org](https://github.com/sponsors/openclaw).

这个项目已经远远超出了我一人能维护的范围。过去几天，我一直在努力增加维护者，并逐步建立流程，以应对疯狂涌入的 PR 和 Issue。我也在想办法为维护者提供合理的报酬——如果可能的话，全职报酬。如果你想帮忙，可以考虑[贡献代码](https://github.com/openclaw/openclaw/blob/main/CONTRIBUTING.md)或[赞助组织](https://github.com/sponsors/openclaw)。

## Thank You

## 致谢

To the Claw Crew—every clawtributor who's shipped code, filed issues, joined our Discord, or just tried the project: thank you. You are what makes OpenClaw special.

致 Claw Crew 的每一位成员——每一位提交过代码、提出过 issue、加入过 Discord 或只是尝试过这个项目的 clawtributor：谢谢你们。正是你们让 OpenClaw 与众不同。

The lobster has molted into its final form. Welcome to OpenClaw.

龙虾已经蜕壳成了它的最终形态。欢迎来到 OpenClaw。

---

*Get started: [openclaw.ai](https://openclaw.ai/)*

*开始使用：[openclaw.ai](https://openclaw.ai/)*

*Join the Claw Crew: [Discord](https://discord.gg/openclaw)*

*加入 Claw Crew：[Discord](https://discord.gg/openclaw)*

*Star on GitHub: [github.com/openclaw/openclaw](https://github.com/openclaw/openclaw)*

*在 GitHub 上标星：[github.com/openclaw/openclaw](https://github.com/openclaw/openclaw)*

— Peter

— Peter

P.S. Yes, the mascot is still a lobster. Some things are sacred.

P.S. 是的，吉祥物仍然是一只龙虾。有些东西是神圣的。
