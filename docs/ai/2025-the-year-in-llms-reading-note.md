# 2025: The year in LLMs

原文链接：[2025: The year in LLMs](https://simonwillison.net/2025/Dec/31/the-year-in-llms/)

一篇优秀的文章，回顾了 12 个月 LLM 领域发生的一切。

本人是 AI 小白，下面的文字是我在阅读文章后，对自己感兴趣的主题进行的记录与梳理。

## 推理之年

读过文章后，我认为智能体的发展是思维链和工具调用不断组合演进的过程。

思维链一直存在，只是从 OpenAI o1 开始，业界将思维链融入了模型的训练阶段，而不仅仅是提示词。

而且将工具调用融入模型的推理阶段，也显著提高了模型的效果，例如 [AI 辅助搜索](https://simonwillison.net/2025/Apr/21/ai-assisted-search/)。

在此之前，AI 辅助搜索的效果并不理想，因为互联网充斥着垃圾信息，但是将搜索融入推理阶段，AI 能够进行反思，有效过滤垃圾信息，并进一步优化关键词迭代搜索。

但是实际使用中，依旧存在一些问题。例如我要求 AI 检索并总结 2025 年 HN 最热门的 10 篇文章，如果互联网上不存在现有的总结文章，AI 就无法正确回答该问题。智能体或许能够做到这一点，自动阅读 HN 的 API 文档并编写脚本执行。

## 智能体之年

[较为明确的智能体定义](https://simonwillison.net/2025/Sep/18/agents/)：一种循环运行工具以实现目标的 LLM。文章还统计了[一些其他定义](https://gist.github.com/simonw/beaa5f90133b30724c5cc1c4008d0654)。

[目前已被证明有效的两种类型的智能体](https://simonwillison.net/2025/Jan/10/ai-predictions/#one-year-code-research-assistants)：编码、研究。

深度研究模式在 2025 上半年很流行，但现在推理模型可以在更短的时间内生成类似的结果。

## 编码智能体和 Claude Code 之年

编码智能体发展的时间线：

- 23 年 OpenAI 实现 ChatGPT Code Interpreter，能够在 k8s 沙箱中运行 Python 代码，当时主要是用于优化 AI 的回答。现在的 AI 基本都内置了该功能，比如询问 12345 * 67890 的结果，会自动编码执行。
- 25 年 2 月，Claude Code 是第一个真正成熟的本地编码智能体。实现了自动编写代码、执行代码、检查结果、以及进一步迭代。
- 25 年 5 月的 OpenAI Codex Web 和 10 月的 Claude Code for web 实现了成熟的云端编码智能体。链接 Github 仓库，只需发任务，智能体自动 fork 代码进行编程，结束后为仓库提 PR。更加方便，也解决了本地智能体的安全问题。

## 命令行 LLMs 之年

LLM 与 Unix 命令行访问模型相当契合。

这让我想起了那些热衷于用命令行和键盘解决所有问题的系统程序员。CLI 强大而又简约，主要基于文本，命令、输入和输出都很结构化，信噪比高，易于程序交互。

相反，GUI 噪声高，充斥着对 LLM 无用的信息，想必基于 GUI 的智能体的发展会慢不少。

## Gemini 之年

[Google 对 2025 年的成就回顾](https://blog.google/technology/ai/google-ai-news-recap-2025/)。Google 有着垂直整合软硬件的优势。

## MCP 之年（唯一一年？）

前面说过，通过这篇文章，我觉得智能体的发展是思维链和工具调用不断组合演进的过程。下面是基于此思想梳理的时间线：

- 22 年，CoT 和 ReAct 概念的提出，这是理论奠基。
- 23 年，OpenAI 推出 Function Calling，让 LLM 能够感知外部世界并做出决策。Web Search 和 Code Execution 功能本质是还是函数调用。
- 24 年末，Anthropic 发布 MCP 标准，统一了工具调用的 API 格式。
- 24 年末，推理模型发展，思维链在训练阶段实现，而不仅仅是提示词。
- 25 年 2 月，Claude Code 发布，模型直接调用 CLI 工具。
- 25 年末，Skills 发布，让模型根据 Skills 文件在 Shell 中使用 CLI 工具或编写脚本运行。

MCP 的作用：客户端侧没什么本质变化，从写一个 OpenAI Function Definition 给 LLM 变成了写 MCP Schema。主要是为服务端侧制定了标准，让资源提供方部署 MCP 服务器，提供 MCP 标准的 API，方便 LLM 调用。

MCP 的缺陷：MCP 仅仅是为服务端侧制定了新的 API 标准，除此之外没有解决任何问题。Claude Code 等编码智能体的发展，让人们意识到客户端测可以通过调用现有的 CLI 工具或 LLM 自行为 API 编写脚本的方式来解决，MCP 唯一的作用也显得又些鸡肋。

- [一篇不错的 MCP 教程](https://zhuanlan.zhihu.com/p/29001189476)
- [一篇指出 MCP 缺陷的问答](https://www.zhihu.com/question/1903481980374476130/answer/1903527338873946826)

## 后记

AI 是技术风口，技术发展日新月异。可惜自己显然没实力卷进去，只是在门口徘徊，了解一些浅显的概念，不至于被时代落下太多。
