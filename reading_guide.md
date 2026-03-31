# 📚 学习与阅读指南：AI 工具系统提示词与模型库 (system-prompts-and-models-of-ai-tools)

## 📌 1. 项目概览 (What is this project?)

`system-prompts-and-models-of-ai-tools` 是一个专注于收集市面上各主流 AI 工具（尤其是 AI 编程助手和生产力工具）的**系统提示词 (System Prompts)**、**模型配置**以及**工具调用架构**的开源仓库。

从项目目录中可以看到，这里收录了大量知名 AI 产品的提示词，例如：

- **AI 编程助手**：Cursor, Devin AI, Replit, Windsurf, Trae, Augment Code, CodeBuddy 等。
- **大模型及官方工具**：Anthropic, Google 等。
- **生产力工具**：NotionAi, v0 (Vercel), Replit 等。
- **开源工具及智能体**：Open Source prompts, Kiro, Lovable 等。

这些系统提示词不仅是构建强大 AI 应用的“灵魂”，更展现了顶级科技公司在设计 AI Agent（智能体）、约束 AI 行为、以及处理人机交互时的最佳实践。

---

## 🎯 2. 为什么要阅读这个项目？

通过研究这个仓库，你可以学到：

1. **Prompt Engineering (提示词工程)**：顶级公司是如何编写长达数千字的 System Prompt 的？如何设定清晰的 Persona（人设）、如何处理边界情况 (Edge cases) 以及如何进行格式化输出控制。
2. **Agent Architecture (智能体架构)**：这些工具是如何思考 (Thought Process)、规划 (Planning) 和执行 (Execution) 复杂任务的。
3. **Tool Use / Function Calling (工具调用)**：AI 是怎么通过提示词理解并调用终端命令行、读写文件、甚至运行代码进行自我验证的。
4. **防护与安全边界 (Guardrails & Security)**：各大厂商如何通过提示词防止模型输出有害信息、防止 Prompt Injection（提示词注入）或泄露底层配置。

---

## 🚀 3. 详细阅读与学习方案

### 阶段一：初窥门径 (寻找你最熟悉的工具)

不要一开始就去读所有的内容，**从你最熟悉或者最感兴趣的一个工具开始阅读。**

- **如果你是开发者**：强烈建议先阅读 `Cursor Prompts` 或 `Devin AI` 目录。看一看每天辅助你写代码的 AI，背后到底被灌输了什么样的底层规则。
- **如果你是产品经理/设计师**：可以阅读 `v0 Prompts and Tools` 或 `NotionAi`，了解 UI 生成或文本处理类 AI 是如何被约束和设计的。

**阅读重点**：

- 文件的开头通常是如何定义 AI 身份的（比如 "You are an expert software engineer..."）。
- 梳理他们通常会提供哪些上下文 (Context) 给大模型。

### 阶段二：庖丁解牛 (拆解 System Prompt 的结构)

挑选几篇经典的且比较长的提示词（如 Devin 或 Anthropic 的相关文档），重点拆解其中的结构模式。
你可以把优秀的 System Prompt 拆解为以下几个部分并进行学习记录：

1. **Role & Identity（角色定义）**：它们是怎么自我介绍的？
2. **Core Directives（核心指令）**：绝对不能违反的规则（通常用 `NEVER`, `ALWAYS`, `MUST` 标记）。
3. **Workflow / Standard Operating Procedure (SOP)**：AI 解决问题的标准工作流（例如：思考 -> 寻找文件 -> 阅读 -> 规划 -> 修改 -> 验证）。
4. **Formatting Rules（格式规范）**：如何要求大模型输出特定的 Markdown 格式、JSON 结构或代码块。
5. **Tool Descriptions（工具描述）**：如何向 AI 解释它可以使用的外部工具（比如 `bash`, `read_file`, `write_file`）及其参数规范。

### 阶段三：横向对比 (归纳最佳实践)

当你读完 3-5 个不同工具的提示词后，开始进行横向对比：

- **同类工具对比**：对比 `Cursor` 和 `Windsurf` 的提示词，看看两者在代码修改 (Code Editing) 和差异应用 (Diff Applying) 上的策略有什么不同。
- **演进路线分析**：观察这些提示词中如何处理“AI 幻觉”。很多提示词会要求大模型在写代码前，先使用搜索工具确认文件结构，这就是为了防止幻觉而设计的标准实践。

### 阶段四：学以致用 (实践与复刻)

纸上得来终觉浅，最重要的学习方式是**把它们用到你的项目中**。

1. **提取组件**：把你认为写得非常精妙的“约束条款”或“思考框架”（例如 `<thought>` 标签引导思考的模式）提取出来，保存为你的个人 Prompt 库。
2. **构建你自己的 Agent**：基于这个仓库里的工具调用架构，使用 LangChain、AutoGen 或纯 OpenAI API 写一个小型的代码助手脚本。
3. **压力测试**：了解了这些工具的底层提示词后，试着在你的日常开发中去测试它们的边界，你会更深刻地理解为什么要这样设计提示词。

---

## 💡 4. 推荐的阅读顺序

1. 🥇 **Cursor / Devin**：当前最顶级的 AI 编程工具，学习如何让 AI 精准控制代码修改。
2. 🥈 **v0 (Vercel)**：前端开发者的神器，学习如何让 AI 结合 UI 组件库（如 shadcn/ui）生成高质量前端代码。
3. 🥉 **Anthropic / Google**：官方给出的提示词或配置，学习基座模型厂商眼中最标准的提示词写法。
4. 🏅 **Open Source prompts**：学习开源社区的智慧，看看社区是如何集思广益优化提示词架构的。

---
*开始探索吧！在这个仓库里，你直接面对的是世界上最聪明的一批大脑和最前沿 AI 工具的“源代码”。*
