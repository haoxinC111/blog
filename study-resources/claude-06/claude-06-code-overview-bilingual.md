# Claude Code Overview | Claude Code Docs

**Source URL:** https://code.claude.com/docs/en/overview  
**整理日期:** 2026-04-07

---

Claude Code is an AI-powered coding assistant that helps you build features, fix bugs, and automate development tasks. It understands your entire codebase and can work across multiple files and tools to get things done.

Claude Code 是一款 AI 驱动的编程助手，可帮助你构建功能、修复错误并自动执行开发任务。它能理解你的整个代码库，并可以跨多个文件和工具协同工作来完成任务。

## Get started / 开始使用


Choose your environment to get started. Most surfaces require a [Claude subscription](https://claude.com/pricing?utm_source=claude_code&utm_medium=docs&utm_content=overview_pricing) or [Anthropic Console](https://console.anthropic.com/) account. The Terminal CLI and VS Code also support [third-party providers](/docs/en/third-party-integrations).

选择你的使用环境以开始使用。大多数平台需要 [Claude 订阅](https://claude.com/pricing?utm_source=claude_code&utm_medium=docs&utm_content=overview_pricing) 或 [Anthropic 控制台](https://console.anthropic.com/) 账户。终端 CLI 和 VS Code 还支持[第三方提供商](/docs/en/third-party-integrations)。

* Terminal
* VS Code
* Desktop app
* Web
* JetBrains

The full-featured CLI for working with Claude Code directly in your terminal. Edit files, run commands, and manage your entire project from the command line.

这是一款功能齐全的 CLI，可直接在终端中使用 Claude Code。你可以在命令行中编辑文件、运行命令并管理整个项目。

To install Claude Code, use one of the following methods:

安装 Claude Code，请使用以下方法之一：

* Native Install (Recommended)
* Homebrew
* WinGet

**macOS, Linux, WSL:**

```
curl -fsSL https://claude.ai/install.sh | bash
```

**Windows PowerShell:**

```
irm https://claude.ai/install.ps1 | iex
```

**Windows CMD:**

```
curl -fsSL https://claude.ai/install.cmd -o install.cmd && install.cmd && del install.cmd
```

If you see `The token '&&' is not a valid statement separator`, you're in PowerShell, not CMD. Use the PowerShell command above instead. Your prompt shows `PS C:\` when you're in PowerShell.

如果你看到 `The token '&&' is not a valid statement separator` 的提示，说明你当前在 PowerShell 中，而不是 CMD。请改用上面的 PowerShell 命令。当你在 PowerShell 中时，提示符会显示 `PS C:\`。

**Windows requires [Git for Windows](https://git-scm.com/downloads/win).** Install it first if you don't have it.

**Windows 需要安装 [Git for Windows](https://git-scm.com/downloads/win)。** 如果你还没有安装，请先安装它。

```
brew install --cask claude-code
```

```
winget install Anthropic.ClaudeCode
```

Then start Claude Code in any project:

然后在任意项目中启动 Claude Code：

```
cd your-project
claude
```

You'll be prompted to log in on first use. That's it! [Continue with the Quickstart →](/docs/en/quickstart)

首次使用时会提示你登录。就这么简单！[继续阅读快速入门 →](/docs/en/quickstart)

The VS Code extension provides inline diffs, @-mentions, plan review, and conversation history directly in your editor.

VS Code 扩展直接在编辑器中提供行内差异对比、@提及、计划审阅和对话历史等功能。

* [Install for VS Code](vscode:extension/anthropic.claude-code)
* [Install for Cursor](cursor:extension/anthropic.claude-code)

Or search for "Claude Code" in the Extensions view (`Cmd+Shift+X` on Mac, `Ctrl+Shift+X` on Windows/Linux). After installing, open the Command Palette (`Cmd+Shift+P` / `Ctrl+Shift+P`), type "Claude Code", and select **Open in New Tab**.

或者在扩展视图中搜索 "Claude Code"（在 Mac 上按 `Cmd+Shift+X`，在 Windows/Linux 上按 `Ctrl+Shift+X`）。安装后，打开命令面板（`Cmd+Shift+P` / `Ctrl+Shift+P`），输入 "Claude Code"，然后选择 **在新标签页中打开**。

[Get started with VS Code →](about:/docs/en/vs-code#get-started)

[开始使用 VS Code →](about:/docs/en/vs-code#get-started)

A standalone app for running Claude Code outside your IDE or terminal. Review diffs visually, run multiple sessions side by side, schedule recurring tasks, and kick off cloud sessions.

这是一款独立应用，可在 IDE 或终端之外运行 Claude Code。支持可视化审阅差异、并排运行多个会话、安排定时任务以及启动云端会话。

Download and install:

下载并安装：

* [macOS](https://claude.ai/api/desktop/darwin/universal/dmg/latest/redirect?utm_source=claude_code&utm_medium=docs) (Intel and Apple Silicon)
* [Windows](https://claude.ai/api/desktop/win32/x64/setup/latest/redirect?utm_source=claude_code&utm_medium=docs) (x64)
* [Windows ARM64](https://claude.ai/api/desktop/win32/arm64/setup/latest/redirect?utm_source=claude_code&utm_medium=docs) (remote sessions only)

After installing, launch Claude, sign in, and click the **Code** tab to start coding. A [paid subscription](https://claude.com/pricing?utm_source=claude_code&utm_medium=docs&utm_content=overview_desktop_pricing) is required.

安装后，启动 Claude，登录并点击 **Code** 标签页即可开始编程。需要[付费订阅](https://claude.com/pricing?utm_source=claude_code&utm_medium=docs&utm_content=overview_desktop_pricing)。

[Learn more about the desktop app →](/docs/en/desktop-quickstart)

[了解更多关于桌面应用的信息 →](/docs/en/desktop-quickstart)

Run Claude Code in your browser with no local setup. Kick off long-running tasks and check back when they're done, work on repos you don't have locally, or run multiple tasks in parallel. Available on desktop browsers and the Claude iOS app.

无需本地设置，直接在浏览器中运行 Claude Code。可以启动长时间运行的任务并在完成后查看，处理你本地没有的代码仓库，或并行运行多个任务。支持桌面浏览器和 Claude iOS 应用。

Start coding at [claude.ai/code](https://claude.ai/code).

在 [claude.ai/code](https://claude.ai/code) 开始编程。

[Get started on the web →](about:/docs/en/claude-code-on-the-web#getting-started)

[在网页版上开始使用 →](about:/docs/en/claude-code-on-the-web#getting-started)

A plugin for IntelliJ IDEA, PyCharm, WebStorm, and other JetBrains IDEs with interactive diff viewing and selection context sharing.

适用于 IntelliJ IDEA、PyCharm、WebStorm 和其他 JetBrains IDE 的插件，支持交互式差异查看和选区上下文共享。

Install the [Claude Code plugin](https://plugins.jetbrains.com/plugin/27310-claude-code-beta-) from the JetBrains Marketplace and restart your IDE.

从 JetBrains 插件市场安装 [Claude Code 插件](https://plugins.jetbrains.com/plugin/27310-claude-code-beta-)，然后重启 IDE。

[Get started with JetBrains →](/docs/en/jetbrains)

[开始使用 JetBrains →](/docs/en/jetbrains)

## What you can do / 你能做什么


Here are some of the ways you can use Claude Code:

以下是你可以使用 Claude Code 的一些方式：

## Use Claude Code everywhere / 随处使用 Claude Code


Each surface connects to the same underlying Claude Code engine, so your CLAUDE.md files, settings, and MCP servers work across all of them.

每个平台都连接到同一个底层 Claude Code 引擎，因此你的 CLAUDE.md 文件、设置和 MCP 服务器可以在所有平台上通用。

Beyond the [Terminal](/docs/en/quickstart), [VS Code](/docs/en/vs-code), [JetBrains](/docs/en/jetbrains), [Desktop](/docs/en/desktop), and [Web](/docs/en/claude-code-on-the-web) environments above, Claude Code integrates with CI/CD, chat, and browser workflows:

除了上述[终端](/docs/en/quickstart)、[VS Code](/docs/en/vs-code)、[JetBrains](/docs/en/jetbrains)、[桌面](/docs/en/desktop) 和 [网页](/docs/en/claude-code-on-the-web) 环境外，Claude Code 还集成了 CI/CD、聊天和浏览器工作流：

| I want to… | Best option |
| --- | --- |
| Continue a local session from my phone or another device | [Remote Control](/docs/en/remote-control) |
| Push events from Telegram, Discord, iMessage, or my own webhooks into a session | [Channels](/docs/en/channels) |
| Start a task locally, continue on mobile | [Web](/docs/en/claude-code-on-the-web) or [Claude iOS app](https://apps.apple.com/app/claude-by-anthropic/id6473753684) |
| Run Claude on a recurring schedule | [Cloud scheduled tasks](/docs/en/web-scheduled-tasks) or [Desktop scheduled tasks](/docs/en/desktop-scheduled-tasks) |
| Automate PR reviews and issue triage | [GitHub Actions](/docs/en/github-actions) or [GitLab CI/CD](/docs/en/gitlab-ci-cd) |
| Get automatic code review on every PR | [GitHub Code Review](/docs/en/code-review) |
| Route bug reports from Slack to pull requests | [Slack](/docs/en/slack) |
| Debug live web applications | [Chrome](/docs/en/chrome) |
| Build custom agents for your own workflows | [Agent SDK](https://platform.claude.com/docs/en/agent-sdk/overview) |

| 我想… | 最佳选择 |
| --- | --- |
| 从手机或其他设备继续本地会话 | [远程控制](/docs/en/remote-control) |
| 将 Telegram、Discord、iMessage 或自定义 webhook 的事件推送到会话中 | [Channels](/docs/en/channels) |
| 在本地开始任务，在移动设备上继续 | [网页版](/docs/en/claude-code-on-the-web) 或 [Claude iOS 应用](https://apps.apple.com/app/claude-by-anthropic/id6473753684) |
| 按计划循环运行 Claude | [云端定时任务](/docs/en/web-scheduled-tasks) 或 [桌面定时任务](/docs/en/desktop-scheduled-tasks) |
| 自动进行 PR 审阅和问题分类 | [GitHub Actions](/docs/en/github-actions) 或 [GitLab CI/CD](/docs/en/gitlab-ci-cd) |
| 在每个 PR 上获得自动代码审阅 | [GitHub Code Review](/docs/en/code-review) |
| 将 Slack 中的错误报告路由为 Pull Request | [Slack](/docs/en/slack) |
| 调试实时 Web 应用 | [Chrome](/docs/en/chrome) |
| 为自定义工作流构建专属 Agent | [Agent SDK](https://platform.claude.com/docs/en/agent-sdk/overview) |

## Next steps / 下一步


Once you've installed Claude Code, these guides help you go deeper.

安装 Claude Code 后，以下指南可帮助你深入了解：

* [Quickstart](/docs/en/quickstart): walk through your first real task, from exploring a codebase to committing a fix
* [Store instructions and memories](/docs/en/memory): give Claude persistent instructions with CLAUDE.md files and auto memory
* [Common workflows](/docs/en/common-workflows) and [best practices](/docs/en/best-practices): patterns for getting the most out of Claude Code
* [Settings](/docs/en/settings): customize Claude Code for your workflow
* [Troubleshooting](/docs/en/troubleshooting): solutions for common issues
* [code.claude.com](https://code.claude.com/): demos, pricing, and product details

* [快速入门](/docs/en/quickstart)：带你完成第一个真实任务，从探索代码库到提交修复
* [存储指令与记忆](/docs/en/memory)：通过 CLAUDE.md 文件和自动记忆为 Claude 提供持久化指令
* [常见工作流](/docs/en/common-workflows) 和 [最佳实践](/docs/en/best-practices)：充分利用 Claude Code 的使用模式
* [设置](/docs/en/settings)：根据你的工作流自定义 Claude Code
* [故障排除](/docs/en/troubleshooting)：常见问题的解决方案
* [code.claude.com](https://code.claude.com/)：演示、定价和产品详情
