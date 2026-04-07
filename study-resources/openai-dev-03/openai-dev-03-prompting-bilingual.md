# Prompting | OpenAI API

> **Source / 原文来源**: https://developers.openai.com/api/docs/guides/prompting/  
> **Compiled for study on / 整理日期**: 2026-04-07

**Prompting** is the process of providing input to a model. The quality of your output often depends on how well you’re able to prompt the model.

**提示（Prompting）**是向模型提供输入的过程。输出质量通常取决于你提示模型的能力。

Prompting is both an art and a science. OpenAI has some strategies and API design decisions to help you construct strong prompts and get consistently good results from a model. We encourage you to experiment.

提示既是一门艺术，也是一门科学。OpenAI 有一些策略和 API 设计决策，可以帮助你构建强大的提示并从模型中获得持续良好的结果。我们鼓励你进行实验。

### Prompts in the API

OpenAI provides a long-lived prompt object, with versioning and templating shared by all users in a project. This design lets you manage, test, and reuse prompts across your team, with one central definition across APIs, SDKs, and dashboard.

OpenAI 提供了一个长生命周期的 prompt 对象，具有版本控制和模板化功能，可供项目中的所有用户共享。这种设计让你可以在团队中管理、测试和重用提示，并在 API、SDK 和仪表板之间实现一个中心化的定义。

Universal prompt IDs give you flexibility to test and build. Variables and prompts share a base prompt, so when you create a new version, you can use that for [evals](/api/docs/guides/evals) and determine whether a prompt performs better or worse.

通用的 prompt ID 为你提供了测试和构建的灵活性。变量和提示共享一个基础提示，因此当你创建新版本时，可以将其用于 [evals](/api/docs/guides/evals)，以确定某个提示表现更好还是更差。

### Prompting tools and techniques

提示工具与技术：

* **[Prompt caching](/api/docs/guides/prompt-caching)**: Reduce latency by up to 80% and cost by up to 75%
  **[Prompt caching](/api/docs/guides/prompt-caching)**：将延迟降低多达 80%，并将成本降低多达 75%
* **[Prompt engineering](/api/docs/guides/prompt-engineering)**: Learn strategies, techniques, and tools to construct prompts
  **[Prompt engineering](/api/docs/guides/prompt-engineering)**：学习构建提示的策略、技术和工具

Log in and use the OpenAI [dashboard](https://platform.openai.com/chat) to create, save, version, and share your prompts.

登录并使用 OpenAI [dashboard](https://platform.openai.com/chat) 来创建、保存、版本化和共享你的提示。

1. **Start a prompt**

   开始一个提示

   In the [Playground](https://platform.openai.com/playground), fill out the fields to create your desired prompt.

   在 [Playground](https://platform.openai.com/playground) 中，填写字段以创建你想要的提示。
2. **Add prompt variables**

   添加提示变量

   Variables let you inject dynamic values without changing your prompt. Use them in any message role using `{{variable}}`. For example, when creating a local weather prompt, you might add a `city` variable with the value `San Francisco`.

   变量让你无需更改提示即可注入动态值。你可以在任何消息角色中使用 `{{variable}}`。例如，在创建本地天气提示时，你可以添加一个值为 `San Francisco` 的 `city` 变量。
3. **Use the prompt in your [Responses API](/api/docs/guides/text?api-mode=responses) call**

   在 [Responses API](/api/docs/guides/text?api-mode=responses) 调用中使用该提示

   Find your prompt ID and version number in the URL, and pass it as `prompt_id`:

   在 URL 中找到你的 prompt ID 和版本号，并将其作为 `prompt_id` 传递：

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
   curl -s -X POST "https://api.openai.com/v1/responses" \
   -H "Content-Type: application/json" \
   -H "Authorization: Bearer $OPENAI_API_KEY" \
   -d '{
       "prompt": {
       "prompt_id": "pmpt_123",
       "variables": {
           "city": "San Francisco"
       }
       }
   }'
   ```
4. **Create a new prompt version**

   创建一个新的提示版本

   Versions let you iterate on your prompts without overwriting existing details. You can use all versions in the API and evaluate their performance against each other. The prompt ID points to the latest published version unless you specify a version.

   版本让你可以在不覆盖现有细节的情况下迭代提示。你可以在 API 中使用所有版本，并评估它们彼此之间的表现。prompt ID 指向最新发布的版本，除非你指定了某个版本。

   To create a new version, edit the prompt and click **Update**. You’ll receive a new prompt ID to copy and use in your Responses API calls.

   要创建新版本，请编辑提示并点击 **Update**。你会收到一个新的 prompt ID，可以复制并在 Responses API 调用中使用。
5. **Roll back if needed**

   如果需要可回滚

   In the [prompts dashboard](https://platform.openai.com/chat), select the prompt you want to roll back. On the right, click **History**. Find the version you want to restore, and click **Restore**.

   在 [prompts dashboard](https://platform.openai.com/chat) 中，选择你想要回滚的提示。在右侧点击 **History**。找到你想要恢复的版本，然后点击 **Restore**。

* Put overall tone or role guidance in the system message; keep task-specific details and examples in user messages.
  将整体语气或角色指导放在 system message 中；将任务特定细节和示例保留在 user message 中。
* Combine few-shot examples into a concise YAML-style or bulleted block so they’re easy to scan and update.
  将 few-shot 示例组合成简洁的 YAML 风格或项目符号块，以便于浏览和更新。
* Mirror your project structure with clear folder names so teammates can locate prompts quickly.
  使用清晰的文件夹名称来映射你的项目结构，以便团队成员能够快速找到提示。
* Rerun your linked eval every time you publish—catching issues early is cheaper than fixing them in production.
  每次发布时都要重新运行关联的评估——尽早发现问题比在生产环境中修复更便宜。

When you feel confident in your prompts, you might want to check out the following guides and resources.

当你对提示充满信心时，你可能想查看以下指南和资源。

[Build a prompt in the Playground

Use the Playground to develop and iterate on prompts.](https://platform.openai.com/chat/edit)
[Text generation

Learn how to prompt a model to generate text.](/api/docs/guides/text)
[Engineer better prompts

Learn about OpenAI’s prompt engineering tools and techniques.](/api/docs/guides/prompt-engineering)
