# Claude Code 快速接入多种大模型 API


### 核心权衡：直连 vs 代理

在使用 Claude Code 时，想要更换底层的 AI 模型，通常面临两种选择：

1.  **原生支持 Anthropic API 的模型**：可以直接配置使用。
2.  **不支持 Anthropic API 的模型**（如 OpenAI、DeepSeek 等）：通常需要引入 `claude-code-router` 或 `claude-code-proxy` 等第三方中间件进行协议转换。

虽然第三方代理能提供更强大的路由和分流功能，但也引入了**额外的环境配置成本**。重新审视我接入多模型的核心诉求，其实非常简单：**便宜的模型处理简单任务，昂贵的模型处理复杂逻辑**。

我并不需要复杂的自动调度，对于任务的难易程度，我有足够的主观判断力。我真正痛点在于如何**毫秒级地在不同 AI 之间切换**。

**因此，本文旨在探索一种极简方案：适用于模型本身兼容 Anthropic 协议（或通过简单转发），且追求零依赖、快速切换体验的开发者。**

### 技术背景：生成式 AI 接口协议对比

当前主流的 LLM 接口主要分为 OpenAI 派系和 Anthropic 派系。了解它们的差异有助于我们进行配置：

| **维度**     | **OpenAI API**                                          | **Anthropic API**                                            |
| :----------- | :------------------------------------------------------ | :----------------------------------------------------------- |
| **标准端点** | `https://api.openai.com/v1/chat/completions`            | `https://api.anthropic.com/v1/messages`                      |
| **鉴权方式** | Header: `Authorization: Bearer <sk-key>`                | Header: `x-api-key: <sk-key>` 及 `anthropic-version`         |
| **消息结构** | `messages` 列表中包含 system/user/assistant             | `messages` 仅含 user/assistant，`system` 作为顶层参数独立传递 |
| **计费粒度** | 计费接口成熟，支持按 Project/Key 细分，易于对接企业监控 | 提供基础用量 API，但在精细化成本核算上（如优先级服务）稍微滞后 |

*(图表参考：[OpenAI API 和 Anthropic API的区别及对比](https://www.cnblogs.com/philry/p/19359139))*

### 实战配置：通过 Alias 实现“一键切换”

Claude Code 的灵活性在于它允许通过环境变量覆盖默认配置。

首先，建议移除 `~/.claude/config.json` 中关于模型的硬编码配置（如果有），这样我们就可以完全通过命令行参数来动态控制。同时，Claude Code 会在当前项目的 `.claude` 目录中存储上下文，这意味着我们可以通过 `cc-ds -c` 这样的命令在不同模型间共享对话历史。

以下以接入 **DeepSeek** 为例（DeepSeek目前已经支持Anthropic协议访问）：

我使用的是 `zsh`，在 `~/.zshrc` 中添加如下 `alias`：

```bash
# DeepSeek 快速启动别名
alias cc-ds='env \
  ANTHROPIC_BASE_URL="https://api.deepseek.com/anthropic" \
  ANTHROPIC_AUTH_TOKEN="你的api_key" \
  API_TIMEOUT_MS=600000 \
  ANTHROPIC_MODEL="deepseek-chat" \
  ANTHROPIC_SMALL_FAST_MODEL="deepseek-chat" \
  ANTHROPIC_DEFAULT_SONNET_MODEL="deepseek-chat" \
  ANTHROPIC_DEFAULT_HAIKU_MODEL="deepseek-chat" \
  CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC=1 \
  claude'

