---
layout: post
title: "Claude Code 完全使用指南"
subtitle: "从安装、权限模式到记忆系统与 Hooks 的一站式上手教程"
date: 2026-08-19 12:00:00
author: "wyh"
header-style: text
catalog: true
tags:
  - AI
  - Claude Code
  - Agent
  - VS Code
  - 工具
---

Anthropic 官方的 AI 编程工具 Claude Code，早已不只是"终端里的聊天框"：它能读懂整个代码库、自主拆解任务、跑测试、提交 Git，还长出了权限模式、记忆系统、子代理、Hooks 等一整套工程化能力。功能多了，上手门槛也随之而来——安装时该选哪种方式？对话面板上 Rewind、Effort、Thinking 这些开关都是干什么的？权限模式里的 Manual、Auto 有什么区别？

这篇文章把这些问题一次讲清楚，覆盖安装登录、VS Code 插件、交互语法、权限模式、斜杠命令、记忆系统、自定义命令、子代理、Hooks 与 MCP 集成。内容以 Claude Code CLI v2.1.220 与 VS Code 扩展 v2.1.235（2026 年 8 月）的实测为准；这个工具迭代非常快，遇到与本文不一致的地方，请以本地 `/help`、`claude --help` 和官方文档 <https://code.claude.com/docs> 为准。

## 一、Claude Code 是什么

Claude Code 是一个 **agentic（智能体式）编程工具**。和传统的"问答式 AI 助手"不同，它可以直接：

- 阅读、搜索、编辑代码库中的文件
- 运行终端命令、执行测试、操作 Git
- 把一个复杂任务拆成多步，自主规划并执行到底

它有四种使用形态，底层引擎相同，可以组合使用：

| 形态 | 说明 |
| --- | --- |
| **终端 CLI** | `claude` 命令，功能最完整的形态 |
| **VS Code 插件** | 侧边栏面板 + 内联 diff，日常开发最顺手 |
| **JetBrains 插件** | IDEA / PyCharm / WebStorm 等 |
| **桌面 App / 网页版（claude.ai/code）** | 图形界面版本 |

## 二、安装与登录

### 安装 CLI

```bash
# 方式一：原生安装器（官方推荐，无需 Node.js）
# macOS / Linux：
curl -fsSL https://claude.ai/install.sh | bash
# Windows（PowerShell）：
irm https://claude.ai/install.ps1 | iex

# 方式二：npm（需要 Node.js 18+）
npm install -g @anthropic-ai/claude-code
```

安装完成后验证：

```bash
claude --version     # 查看版本
claude doctor        # 健康检查，排查环境问题
```

### 登录

直接运行 `claude`，首次启动会引导登录：

- **订阅账号**：Claude Pro / Max / Team / Enterprise，走浏览器 OAuth 授权
- **API Key**：设置环境变量 `ANTHROPIC_API_KEY`
- 会话内可以随时用 `/login`、`/logout` 切换账号

## 三、VS Code 插件

### 安装

**方式一：扩展市场（推荐）**——`Ctrl+Shift+X` 打开扩展面板，搜索 **"Claude Code"**，认准发布者为 **Anthropic**（带官方认证标识，注意甄别同名社区插件），点击 Install。

**方式二：命令行**——

```bash
code --install-extension anthropic.claude-code
```

**方式三：自动安装**——在 VS Code 集成终端里运行 `claude` 时，会自动检测并提示安装插件（可用环境变量 `CLAUDE_CODE_IDE_SKIP_AUTO_INSTALL=1` 禁用）。

要求：VS Code ≥ 1.98.0，且工作区需开启 **Workspace Trust**（受限模式下不可用）。

### 核心功能

| 功能 | 说明 |
| --- | --- |
| **侧边栏面板** | 点击 ✦ 图标打开对话面板，写代码时不用切终端 |
| **内联 Diff** | 代码修改直接在 VS Code diff 视图里展示，可逐处接受/拒绝 |
| **上下文共享** | 当前打开的文件、选中的代码（含行号）、IDE 诊断信息（lint 错误等）自动同步给 Claude |
| **状态栏** | 实时显示 Claude 状态（思考中 / 读取中 / 写入中） |

### 快捷键

| 操作 | Windows/Linux | Mac |
| --- | --- | --- |
| 编辑器 ↔ Claude 焦点切换 | `Ctrl+Esc` | `Cmd+Esc` |
| 新标签页中打开 Claude | `Ctrl+Shift+Esc` | `Cmd+Shift+Esc` |
| **插入文件引用（@-mention）** | **`Ctrl+Alt+K`** | **`Option+Cmd+K`** |
| 输入框换行（不发送） | `Shift+Enter` | `Shift+Enter` |
| 循环切换权限模式 | `Shift+Tab` | `Shift+Tab` |
| 切换扩展思考（Thinking） | `Alt+T` | `Option+T` |
| 回退修改（Rewind） | `Esc` ×2 | `Esc` ×2 |

其中 `Ctrl+Alt+K` 使用频率最高：把当前文件或选中代码（带行号）作为 @ 引用插入输入框，让 Claude 精确定位你正在说的那段代码。

### 对话面板上的核心选项

打开插件面板，菜单里有一排选项，新手最容易困惑的就是它们。以下是扩展 v2.1.235 中的官方定义（括号内为界面英文原文）：

| 选项 | 官方说明与解释 |
| --- | --- |
| **Rewind（回退）** | "Restore code and conversation to an earlier point"——把代码和对话回退到之前的某个时间点。Claude 每轮修改前会自动打检查点（checkpoint），回退时可选择只回退对话、只回退代码、或两者一起。注意：只影响 Claude 改过的文件，手动编辑和 bash 命令造成的改动不受 Rewind 保护。等价于连按两次 `Esc` 或 `/rewind` |
| **Clear conversation（清空对话）** | "Start a new conversation"——开一个新会话、清空上下文。旧会话不会丢，之后可用 Resume conversation 或 `/resume` 恢复。切换任务时用它，避免旧上下文干扰 |
| **New conversation / Resume conversation** | 在新标签页开新会话 / 继续之前的会话 |
| **Switch model…（切换模型）** | "Change the AI model"，等价于 `/model` |
| **Thinking（扩展思考）** | "Toggle extended thinking mode"——开关 Claude 的"思考过程"。开启后回答前会做更长的内部推理，复杂问题质量更高，但速度变慢、token 消耗增加。快捷键 `Alt+T` |
| **Effort（推理强度）** | 调节模型投入的推理深度。**可选档位取决于当前模型支持的范围**：基础档为 `low` / `medium` / `high`，支持的模型会追加 `xhigh` 和 `max` 档。低档响应快、适合简单问答；高档适合复杂架构设计和疑难 bug，但更慢更贵。等价于 `/effort` |
| **Account & usage…（账号与用量）** | "View account info and usage limits"，查看登录账号、订阅限额和 token 消耗，等价于 `/usage` |
| **Toggle fast mode** | 切换快速模式（仅 Opus），响应更快，等价于 `/fast` |
| **Focus view** | 专注视图，只显示你的提问和 Claude 的回复 |
| **Attach file… / Mention file from this project…** | 上传文件 / 用 @ 引用项目文件加入对话 |
| **Manage plugins / MCP servers / General config…** | 插件管理、MCP 服务器配置、扩展设置入口 |

一句话总结：**Thinking 是"要不要深思"，Effort 是"深思到什么程度"**——前者是开关，后者是旋钮，且旋钮的档位随模型而变。

## 四、交互模式核心语法

在输入框里除了自然语言，还有四个特殊前缀：

### `@` —— 引用文件/目录

```text
请解释 @src/utils.ts 中的日期处理逻辑
对比 @src/old-api.ts 和 @src/new-api.ts 的差异
@src/components/ 这个目录的结构合理吗？
```

- 支持路径补全（输入 `@` 后自动提示）
- 支持行号范围：`@src/utils.ts:10-50`
- 路径含空格用引号：`@"my folder/file.ts"`

### `!` —— Bash 模式

直接执行 shell 命令，输出进入对话上下文：

```text
!git status
!npm test
```

### `#` —— 快速写入记忆

把一条内容快速保存到记忆文件（会提示选择项目级或用户级）：

```text
# 本项目使用 pnpm，不要用 npm install
```

### `/` —— 斜杠命令

所有内置命令、自定义命令、插件命令的入口，输入 `/` 后有自动补全列表，详见第六章。

### 其他常用交互操作

| 操作 | 说明 |
| --- | --- |
| `Esc` | 中断 Claude 当前输出/工具执行（可立刻追加新指令纠偏） |
| `Esc` ×2 | Rewind，回退到之前的对话状态 |
| `↑` / `↓` | 浏览历史输入 |
| `Tab` | 命令/路径补全 |
| 粘贴图片 | 直接把截图拖入或粘贴进输入框（终端中也可用 `@screenshot.png` 引用图片文件） |

## 五、权限模式：Manual / Edit automatically / Plan / Auto

Claude 每次要改文件、跑命令时"要不要先问你"，由权限模式决定。在插件面板或终端里按 `Shift+Tab` 循环切换。扩展 v2.1.235 的模式选择器中，每个选项的官方说明如下：

| 界面选项 | 官方说明（原文） | 内部名称 | 适用场景 |
| --- | --- | --- | --- |
| **Manual** | "Claude will ask for approval before making each edit" | `default` | 最保守，每次修改前逐条确认，适合陌生代码库或危险操作 |
| **Edit automatically** | "Claude will edit your selected text or the whole file" | `acceptEdits` | 日常编码，文件编辑不再打断你 |
| **Plan** | "Claude will explore the code and present a plan before editing" | `plan` | 复杂重构、不熟悉的项目，先审计划再动手 |
| **Auto** | "Claude will approve actions that pass a safety check and pause for anything risky" | `auto` | 想省心又不想裸奔时的选择 |

部分版本/设置下还会出现两个额外模式：

| 界面选项 | 官方说明（原文） | 说明 |
| --- | --- | --- |
| **Don't ask** | "Claude will deny actions that need approval instead of asking" | 与 Auto 相反：需要批准的操作**直接拒绝**，不弹窗打扰。适合只读审查、演示等场景 |
| **Bypass permissions** | "Claude will not ask for approval before running potentially dangerous commands" | ⚠️ 完全不询问、全部放行，**仅限 Docker 容器、沙箱等隔离环境**（CLI 中的 `--dangerously-skip-permissions`） |

几个容易误解的点，值得澄清：

1. **Edit automatically 只管"编辑"**。界面文案只承诺"自动编辑选中代码或整个文件"，它对应内部的 `acceptEdits` 模式——完整语义是**文件编辑自动通过，但运行 shell 命令、删除文件等其他工具调用仍会询问**。不要以为开了它就什么都不问了。
2. **Auto 是"智能判别"，不是"全部放行"**。它是新版本引入的模式：背后有一个专门的分类器模型实时评估每个待执行操作——能明确判定安全的静默执行，判定有风险的拦截，无法判定时退回人工确认（fail-closed）。所以它比 Manual 省时，又比 Bypass permissions 安全得多，在新版本中已成为默认模式。
3. **Plan 模式下 Claude 不会改任何东西**，只做只读分析并产出计划，确认后才进入执行。

### 精细权限规则

模式之外，还可以在 `/permissions` 或 `settings.json` 里配置 allow/deny 规则，做命令级的精细授权：

```json
{
  "permissions": {
    "defaultMode": "acceptEdits",
    "allow": [
      "Bash(npm run test:*)",
      "Bash(git status)",
      "Read(./src/**)"
    ],
    "deny": [
      "Bash(rm -rf *)",
      "Read(./.env)"
    ]
  }
}
```

规则语法为 `工具名(命令模式)`，如 `Bash(git diff:*)`。deny 优先级最高，适合用来封死危险命令和敏感文件。

## 六、内置斜杠命令大全

Claude Code 的命令分三类：**内置命令**（核心 CLI）、**技能命令**（捆绑的提示词流程，如 `/code-review`）、**工作流命令**（多子代理协作，如 `/deep-research`）。以下清单以 v2.1.220 为准整理。

### 会话管理

| 命令 | 功能 |
| --- | --- |
| `/clear` | 开启新会话、清空上下文；旧会话可用 `/resume` 恢复（别名 `/new`、`/reset`） |
| `/compact [说明]` | 压缩对话历史以释放上下文，可带说明指定保留重点 |
| `/autocompact` | 设置上下文用到多少比例时自动压缩 |
| `/context` | **可视化上下文窗口占用**（彩色网格：系统提示、MCP 工具、记忆文件、对话历史各占多少 token），长会话必备 |
| `/resume` | 恢复之前的会话（别名 `/continue`） |
| `/export [路径]` | 导出对话为文件或剪贴板 |
| `/copy [N]` | 复制最近一条回复到剪贴板 |
| `/insights` | 生成报告，分析你的使用习惯 |
| `/exit` | 退出（别名 `/quit`） |

关于 `/compact` 需要多说明两句：它不是简单地"砍掉旧消息"，而是让模型把整段对话**摘要压缩**后继续，并且**可以带参数指定保留重点**：

```text
/compact 保留数据库 schema 相关的讨论和未完成的迁移步骤
/compact 重点保留刚才确定的 API 设计决策，其余可以丢弃
```

- 不带参数：按默认策略压缩整个对话历史
- 带说明：压缩时优先保留你指定的核心信息，丢弃其余细节——长任务做到一半发现上下文将满时非常有用
- 与 `/clear` 的区别：`/clear` 是彻底重开（旧会话只能从 `/resume` 找回），`/compact` 是"带着摘要继续干当前这件事"
- 自动压缩阈值用 `/autocompact` 设置；接近上限时 Claude 也会主动提示

### 分支与后台会话（v2.x 新增）

| 命令 | 功能 |
| --- | --- |
| `/fork` | 把当前对话**复制**成一个新的后台会话，本会话继续工作 |
| `/branch` | 在当前位置创建对话分支 |
| `/btw <问题>` | 不打断主对话，插问一个快速小问题 |
| `/subtask` | 派子代理带着完整上下文去执行，结果返回本会话 |
| `/stop` | 停止后台会话（transcript 保留） |
| `/goal <目标>` | 设定目标，Claude 停止前会自检是否达成 |
| `/cd <路径>` | 把本会话迁移到新的工作目录 |
| `/teleport` | 恢复一个来自 claude.ai 网页版的会话 |

### 用量与状态

| 命令 | 功能 |
| --- | --- |
| `/usage` | 用量面板：费用、订阅限额、活动统计（`/cost`、`/stats` 是其别名） |
| `/status` | 查看会话状态（版本、模型、账号、连接等） |
| `/todos` | 查看/管理任务清单 |

### 模型与配置

| 命令 | 功能 |
| --- | --- |
| `/model [模型]` | 查看或切换模型 |
| `/effort` | 设置推理强度（基础档 low / medium / high，支持的模型追加 xhigh / max） |
| `/fast [on\|off]` | 切换快速模式（Opus 高速输出） |
| `/config` | 打开设置界面 |
| `/permissions` | 管理工具权限规则（别名 `/allowed-tools`） |
| `/keybindings` | 打开键盘快捷键配置 |
| `/alias` | 创建或查看命令别名 |
| `/terminal-setup` | 配置终端集成 |
| `/statusline` | 自定义状态栏显示 |
| `/voice` | 切换语音输入 |
| `/privacy-settings` | 隐私与数据收集设置 |
| `/theme` | 切换主题 |

### 项目与代码

| 命令 | 功能 |
| --- | --- |
| `/init` | **扫描代码库并生成 CLAUDE.md**（新项目第一件事） |
| `/add-dir <路径>` | 添加额外的工作目录 |
| `/code-review` | 审查**当前工作区改动** |
| `/review` | ⚠️ 语义已变化：现在用于**审查 GitHub PR**；审查本地改动用 `/code-review` |
| `/security-review` | 对当前分支改动做专项安全审查 |
| `/diff` | 查看未提交改动和逐轮 diff |
| `/memory` | **记忆管理总入口**：打开交互式窗口，查看/编辑 CLAUDE.md、Auto Memory 记忆文件、开关自动记忆（详见第八章） |
| `/hooks` | 查看已配置的钩子 |
| `/skills` | 列出可用技能（`/skill-doctor` 可检查哪些技能白占上下文） |

### 工具与集成

| 命令 | 功能 |
| --- | --- |
| `/mcp` | 管理 MCP 服务器（查看状态、OAuth 认证） |
| `/ide` | 查看/管理 IDE 集成状态 |
| `/install-github-app` | 安装 GitHub App（PR 自动审查等） |
| `/plugin` | 管理插件与市场（别名 `/plugins`、`/marketplace`） |
| `/reload-plugins` / `/reload-skills` | 会话中热加载插件/技能变更 |
| `/remote-control` | 从 claude.ai/code 网页端查看和控制当前会话 |
| `/import` | 从其他 AI 编程工具导入配置 |

### 帮助与反馈

| 命令 | 功能 |
| --- | --- |
| `/help` | 查看帮助 |
| `/doctor` | 诊断安装与运行环境（别名 `/checkup`） |
| `/bug` / `/feedback` | 提交 bug / 反馈 |
| `/release-notes` | 查看版本更新日志 |
| `/login` / `/logout` | 登录 / 登出 |
| `/upgrade` | 升级订阅 |

### 已移除的命令

| 命令 | 状态 |
| --- | --- |
| `/vim` | v2.1.92 移除 |
| `/pr-comments` | v2.1.91 移除 |
| `/agents` | v2.1.220 标记移除——子代理改为直接编辑 `.claude/agents/` 目录管理 |

最可靠的命令列表永远是在会话中输入 `/` 看自动补全，或运行 `/help`。

## 七、CLI 命令行标志

### 基本调用

```bash
claude                          # 在当前目录启动交互会话
claude "修复 login 的 bug"        # 带初始提示启动
claude -p "解释这个函数"           # 无头模式：执行后输出并退出（适合脚本/管道）
cat error.log | claude -p "分析这个错误"   # 管道输入
```

### 会话恢复

| 标志 | 说明 |
| --- | --- |
| `-c`, `--continue` | 继续当前目录最近一次会话 |
| `-r`, `--resume [id]` | 恢复指定会话（不带参数则弹出选择器） |
| `--fork-session` | 以副本分支方式恢复会话 |

可与 `-p` 组合：`claude -c -p "再跑一遍测试"`

### 常用标志

| 标志 | 说明 | 示例 |
| --- | --- | --- |
| `--model` | 指定模型 | `claude --model opus` |
| `--add-dir <路径>` | 添加额外工作目录（可重复） | `claude --add-dir ../shared` |
| `--allowedTools` | 免确认的工具白名单 | `--allowedTools "Read,Grep,Bash(git *)"` |
| `--disallowedTools` | 禁用的工具 | `--disallowedTools "Write"` |
| `--permission-mode` | 启动权限模式 | `--permission-mode plan` |
| `--max-turns <n>` | 限制 agent 循环轮数 | `--max-turns 5` |
| `--output-format` | 输出格式 `text` / `json` / `stream-json` | 配合 `-p` 用于脚本 |
| `--append-system-prompt` | 追加系统提示 | 自动化场景定制行为 |
| `--max-budget-usd` | API 花费上限 | CI 防爆预算 |
| `--dangerously-skip-permissions` | 跳过所有权限确认（⚠️ 仅限沙箱/容器） | |
| `--settings <文件>` | 加载指定 settings 文件 | |
| `--mcp-config <文件>` | 加载 MCP 配置 | |
| `-v`, `--verbose` / `--debug` | 详细输出 / 调试日志 | |

### 无头模式（CI/脚本）示例

```bash
claude -p \
  --model sonnet \
  --allowedTools "Read,Grep,Glob" \
  --max-turns 5 \
  --output-format json \
  "审查这个 PR 的安全问题" > review.json
```

生产/CI 环境中优先用 `--allowedTools` 做精细授权，而不是 `--dangerously-skip-permissions`。

## 八、记忆系统：CLAUDE.md 与 Auto Memory

每次会话开始时 Claude 都是"失忆"的，有两套机制把知识带到下一个会话（官方文档 <https://code.claude.com/docs/en/memory>）：

| | CLAUDE.md | Auto Memory |
| --- | --- | --- |
| **谁写的** | 你 | Claude 自己 |
| **内容** | 指令与规则 | Claude 总结的经验与模式 |
| **作用范围** | 项目 / 用户 / 组织 | 按仓库隔离（同一 repo 的所有 worktree、子目录共享） |
| **加载方式** | 每次会话全量加载 | 每次会话加载索引前 200 行或 25KB |
| **适合放** | 编码规范、工作流、项目架构 | 构建命令、调试心得、Claude 发现的偏好 |

### CLAUDE.md 的四个级别

CLAUDE.md 是你手写的长期规则文件，按作用范围从大到小分四级（加载顺序也是这个顺序，越具体越靠后）：

| 级别 | 路径 | 用途 | 是否提交 Git |
| --- | --- | --- | --- |
| 组织级（Managed policy） | macOS `/Library/Application Support/ClaudeCode/CLAUDE.md`；Linux/WSL `/etc/claude-code/CLAUDE.md`；Windows `C:\Program Files\ClaudeCode\CLAUDE.md` | 企业 IT/DevOps 统一下发的规范，不可被个人设置排除 | — |
| 用户级 | `~/.claude/CLAUDE.md` | 跨所有项目的个人偏好（代码风格、常用工具） | ❌ |
| 项目级 | `./CLAUDE.md` 或 `./.claude/CLAUDE.md` | 团队共享的项目约定 | ✅ 提交 |
| 项目本地级 | `./CLAUDE.local.md` | 个人的项目内偏好（沙箱地址、测试数据等），记得加进 `.gitignore` | ❌ |

### 加载机制：父目录全量加载，子目录按需加载

这是最容易被忽略、也最值得理解的一点：

- **启动时**：Claude 从当前工作目录**向上逐级查找**，把路径上每一级的 `CLAUDE.md` 和 `CLAUDE.local.md` **全部拼进上下文**（不是互相覆盖）。顺序是从文件系统根部到你的工作目录——离你启动位置越近的规则越靠后（越"新鲜"）；同一目录内 `CLAUDE.local.md` 排在 `CLAUDE.md` 之后
- **子目录**：当前目录**之下**的各子目录里也可以单独建立 `CLAUDE.md`，但它们**不会在启动时加载**，而是当 Claude **读取到那个子目录里的文件时才按需注入**。这对 monorepo 非常有用——每个 package 可以有自己的约定，只有 Claude 真的改到那个包时才占用上下文
- 想确认本次会话到底加载了哪些记忆文件，运行 `/context` 查看 **Memory files** 列表
- monorepo 里如果不想要其他团队的 CLAUDE.md，可在 settings 里用 `claudeMdExcludes` 按路径/glob 排除（组织级文件不可排除）
- 补充：`/compact` 之后，项目根的 CLAUDE.md 会从磁盘**重新注入**；但子目录的 CLAUDE.md 和路径规则不会自动恢复，要等 Claude 下次读到对应文件才重新加载

### `@` 导入其他文件

CLAUDE.md 里可以用 `@路径` 导入其他文件，导入的内容**在启动时随 CLAUDE.md 一起全量进入上下文**：

```text
详见 @README 和 @docs/git-workflow.md
```

- 相对路径**相对于包含 import 的那个文件**解析，不是工作目录；也支持绝对路径
- 可以递归导入，**最多 4 层嵌套**
- 代码块和行内代码中的 `@路径` 不会被解析——想提到某个文件但不导入，用反引号包起来：`` `@README` ``
- 项目级文件 import 了工作目录之外的文件（如 `@~/.claude/my-notes.md`）时，首次会弹**确认对话框**（防止别人提交到共享项目的恶意导入）
- 已有 `AGENTS.md` 的仓库，直接在 CLAUDE.md 里写 `@AGENTS.md` 即可共用一份说明，不用复制
- 注意：import 只是组织方式，**不能减少上下文占用**——导入的文件同样在启动时加载

### `.claude/rules/`：按路径按需加载的规则

比子目录 CLAUDE.md 更精细的机制：把规则拆成 `.claude/rules/` 下的多个 Markdown 文件（如 `testing.md`、`api-design.md`），并通过 YAML frontmatter 的 `paths` 字段把规则**绑定到特定文件**：

````markdown
---
paths:
  - "src/api/**/*.ts"
---

# API 开发规则
- 所有接口必须做输入校验
- 使用统一的错误响应格式
````

不带 `paths` 的规则在启动时全量加载；带 `paths` 的规则**只在 Claude 读取匹配文件时才注入上下文**——既不占平时的 token，又保证改到对应文件时规则在场。用户级规则放在 `~/.claude/rules/`，对所有项目生效（优先级低于项目规则）。

### Auto Memory：自动记忆系统

Auto Memory 是 Claude **自己记笔记**的系统，**当前版本默认开启**。它保存的是 Claude 在工作过程中发现的、以后可能还有价值的经验，例如"这个项目的 integration test 必须先启动 Redis"、某条调试结论、你的工作习惯。Claude 不是每次会话都写——它自己判断哪些信息值得记。当你在界面看到 "Saved 2 memories" / "Recalled 2 memories" 这类提示，就是它在写/读记忆。

**存储位置与加载机制：**

- 每个项目的记忆保存在 `~/.claude/projects/<project>/memory/` 下。`<project>` 由 **git 仓库**推导，因此同一 repo 的**所有 worktree 和子目录共享同一份**自动记忆；不在 git 仓库里则以项目根目录为准。记忆是**本机存储**的，不跨机器同步
- 目录结构：`MEMORY.md` 是**索引入口** + 若干主题文件（如 `debugging.md`、`api-conventions.md`）
- 会话启动时**只加载 `MEMORY.md` 的前 200 行或前 25KB（先到为准）**，超出的部分直接丢弃；所以索引要保持一行一条，细节挪进主题文件。主题文件**启动时不加载**，Claude 需要时用文件工具按需读取
- 索引超限时写入仍会成功，但 Claude Code 会返回错误提示 Claude 重写精简索引
- 主会话的 Auto Memory **不会加载进子代理**（`/fork` 分叉出的会话除外）；子代理可以通过 `memory` 字段拥有自己的独立记忆目录
- 这些记忆文件**不受会话记录自动清理影响**，会一直保留直到你或 Claude 删改

**开关与自定义：**

- 在 `/memory` 窗口里切换开关（写入用户设置的 `autoMemoryEnabled`）
- 单项目关闭：项目 settings 里设 `"autoMemoryEnabled": false`
- 环境变量 `CLAUDE_CODE_DISABLE_AUTO_MEMORY=1` 全局禁用
- 想换存储位置：settings 里设 `autoMemoryDirectory`（绝对路径或 `~/` 开头）

### `/memory`：记忆管理总入口

`/memory` **不是简单命令，而是吊起一个交互式窗口**。在窗口里你可以：

- **浏览并选择要编辑的记忆文件**：列出用户级、项目级所有 CLAUDE.md / CLAUDE.local.md 位置，**包括还不存在的文件**——选中不存在的会先创建再打开
- **开关 Auto Memory**
- **打开 Auto Memory 目录**，直接翻看 Claude 都记了些什么（纯 Markdown，可改可删）
- 选中的文件会在你的编辑器里打开；GUI 编辑器（如 VS Code）在新窗口打开、不阻塞会话，终端编辑器（如 Vim）则会占用终端直到退出

配合 `/context` 使用：一个管"有哪些记忆文件可编辑"，一个管"本次会话实际加载了哪些"。

### 写入记忆的几种方式

- `/init`：自动扫描代码库，生成初始 CLAUDE.md（**新项目第一步**）
- `#内容`：会话中快速追加一条记忆（会提示选择项目级或用户级）
- **自然语言**：说"记住：这个项目用 pnpm"会写入 Auto Memory；说"把这条加到 CLAUDE.md"则写入规则文件；说"忘掉关于 X 的记忆"可删除
- **Auto Memory 自动写入**：开启后 Claude 自己判断、自己存

### CLAUDE.md 内容示例与最佳实践

````markdown
# 项目约定

## 技术栈
- TypeScript + React 18，状态管理用 Zustand
- 包管理器用 pnpm（不要用 npm install）

## 常用命令
- 构建：pnpm build
- 测试：pnpm test:unit（改代码后必须运行）

## 代码风格
- 组件用函数式 + Hooks，禁止 class 组件
- 提交信息遵循 Conventional Commits

详细 API 说明见 @docs/api-guide.md
````

**要点（官方建议）：**

1. **每个 CLAUDE.md 控制在 200 行以内**——全量加载进上下文，越长越占 token、遵守度越差；内容太多就拆到 `.claude/rules/` 按路径加载
2. **写 Claude 推断不出来的东西**：构建命令、测试流程、架构决策、已知坑（目录结构、依赖列表这类 Claude 自己能看出来的不必写）
3. **具体可验证**：写"提交前运行 `pnpm test:unit`"而不是"写好测试"；写"用 2 空格缩进"而不是"格式化好代码"
4. **避免规则互相矛盾**——两条规则冲突时 Claude 会任意选一条；定期审查各级 CLAUDE.md、子目录 CLAUDE.md 和 `.claude/rules/`
5. **不要存密钥**——纯文本且项目级通常会被提交到 Git
6. **必须在固定时机执行的"铁律"不要用记忆**，用 Hooks（记忆只是上下文提示，不保证严格执行）
7. **分工**：稳定、需要团队共享的约定写 CLAUDE.md；Claude 自己踩坑总结的经验交给 Auto Memory
8. CLAUDE.md 太大时可以跑 `/doctor`，它会建议删掉 Claude 能自己推导的内容（v2.1.206+）

## 九、自定义斜杠命令

把 Markdown 文件放到固定目录，即可创建自己的斜杠命令：

| 级别 | 路径 | 范围 |
| --- | --- | --- |
| 项目级 | `.claude/commands/<名称>.md` | 团队共享（提交 Git） |
| 用户级 | `~/.claude/commands/<名称>.md` | 个人所有项目 |

示例 `.claude/commands/review-frontend.md`：

````markdown
---
description: 前端代码专项审查
allowed-tools: Read, Grep, Glob
---

请审查以下前端代码，重点关注：React Hooks 依赖、可访问性、性能（不必要的重渲染）。
审查范围：$ARGUMENTS
````

调用方式：`/review-frontend src/components/`。其中 `$ARGUMENTS` 会被替换为命令后输入的全部文本；子目录会映射为命名空间，如 `.claude/commands/frontend/lint.md` → `/frontend:lint`。

## 十、子代理 Subagents

子代理是**有独立上下文窗口、专门职责和工具权限**的 AI 助手，Claude 可以根据任务自动委派给它们。

### 定义位置

| 级别 | 路径 |
| --- | --- |
| 项目级 | `.claude/agents/<名称>.md` |
| 用户级 | `~/.claude/agents/<名称>.md` |

### 示例 `.claude/agents/code-reviewer.md`

````markdown
---
name: code-reviewer
description: 资深代码审查专家，修改代码后应主动调用
tools: Read, Grep, Glob, Bash
model: sonnet
---

你是资深代码审查专家。收到代码后：
1. 检查正确性、边界条件与潜在 bug
2. 检查安全问题（注入、敏感信息泄露）
3. 按严重程度分级输出：严重 / 警告 / 建议
````

### 使用方式

- **自动委派**：Claude 根据 `description` 判断何时调用
- **显式调用**：`> 使用 code-reviewer 子代理检查我刚才的改动`
- **管理**：直接编辑 `.claude/agents/` 目录下的 Markdown 文件（`/agents` 管理命令已在 v2.1.220 移除）

**好处**：独立上下文不污染主对话；可限定工具（如只读）；可指定不同模型。

## 十一、Hooks 钩子

Hooks 是在 Claude Code 生命周期事件中**确定性执行**的 shell 命令（不依赖模型判断），适合做强制校验、自动格式化等"铁律"类操作。

### 常用事件

| 事件 | 触发时机 |
| --- | --- |
| `SessionStart` | 会话启动 |
| `UserPromptSubmit` | 用户提交输入时 |
| `PreToolUse` | 工具调用前（可阻止） |
| `PermissionRequest` | 请求权限时 |
| `PostToolUse` | 工具调用后（如自动跑 prettier） |
| `Notification` | 发送通知时 |
| `Stop` | 主代理结束时 |
| `SubagentStop` | 子代理结束时 |
| `PreCompact` | 压缩上下文前 |
| `SessionEnd` | 会话结束 |

### 配置示例（settings.json）

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "prettier --write \"$CLAUDE_FILE_PATH\""
          }
        ]
      }
    ]
  }
}
```

要点：Hook 从 **stdin 接收 JSON** 事件数据，通过退出码控制行为——`0` 放行、`2` 阻止；`PreToolUse` 返回阻止结果可拦截工具执行；用 `/hooks` 查看当前已配置的钩子。

## 十二、MCP 服务器集成

MCP（Model Context Protocol）是一套开放协议，让 Claude Code 接入外部工具和数据源（数据库、浏览器、内部 API 等）。

### CLI 管理

```bash
# 添加 stdio 服务器
claude mcp add filesystem -- npx -y @modelcontextprotocol/server-filesystem /path/to/dir

# 添加 HTTP 远程服务器
claude mcp add --transport http my-server https://example.com/mcp

# 用 JSON 添加
claude mcp add-json my-server '{"command":"npx","args":["-y","some-server"]}'

# 管理
claude mcp list
claude mcp get my-server
claude mcp remove my-server
```

作用域：`-s local`（默认，仅自己本项目）/ `-s project`（团队共享，写入 `.mcp.json`）/ `-s user`（跨项目）。

### 会话内

```text
/mcp        # 查看服务器状态、工具列表、OAuth 认证
```

MCP 工具以 `mcp__<服务器名>__<工具名>` 的形式出现在权限规则中，如 `mcp__github__create_issue`。

## 十三、配置文件 settings.json

### 层级（后者覆盖前者）

```text
内置默认
 → 用户级   ~/.claude/settings.json
 → 项目级   ./.claude/settings.json        （团队共享，可提交 Git）
 → 项目本地 ./.claude/settings.local.json  （个人，加入 .gitignore）
 → CLI 标志
```

### 常用配置示例

```json
{
  "model": "sonnet",
  "permissions": {
    "defaultMode": "acceptEdits",
    "allow": ["Bash(npm run test:*)", "Bash(git status)"],
    "deny": ["Read(./.env)"]
  },
  "env": {
    "EDITOR": "code --wait"
  },
  "hooks": {},
  "statusLine": {
    "type": "command",
    "command": "echo my-project"
  }
}
```

## 十四、常用工作流与最佳实践

### 黄金流程：探索 → 计划 → 编码 → 提交

```text
1. 探索  > 阅读这个代码库的结构，说明技术栈和入口文件
2. 计划  （Shift+Tab 切到 Plan）> 设计实现方案，先不要改代码
3. 编码  （Shift+Tab 切到 Edit automatically）> 按计划实现
4. 审查  > /code-review
5. 提交  > 总结改动并生成 Conventional Commits 提交信息
```

### 高效技巧

| 技巧 | 说明 |
| --- | --- |
| **先 `/init`** | 新项目第一件事，生成 CLAUDE.md 让 Claude 理解项目 |
| **及时 `/clear`** | 切换任务时清空上下文，避免旧上下文干扰 |
| **长会话看 `/context`** | 可视化上下文各部分占用，发现 MCP 工具或 CLAUDE.md 吃 token 太多就裁剪 |
| **及时 `/compact`** | 上下文将满时压缩，可带参数指定保留重点，如 `/compact 保留 API 设计决策` |
| **多用 `@` 引用** | 精确指向文件，比"那个处理登录的文件"高效得多 |
| **给验证手段** | 告诉 Claude 怎么验证（跑哪个测试、访问哪个 URL），它会自我修正 |
| **截图输入** | UI 问题直接贴截图；也可贴设计稿让 Claude 实现 |
| **任务拆解** | 大任务用 Plan 模式拆步骤，分步确认 |
| **Esc 纠偏** | 发现方向不对立刻 Esc 中断补充指令，比重开对话省 token |
| **TDD** | "先写失败的测试，再实现代码让测试通过"效果极佳 |
| **会话续接** | 下班前不用记进度，第二天 `claude -c` 继续 |

### Git 集成常用提示

```text
总结当前未提交的改动
把改动按功能拆成多个 commit
帮我解决 rebase 冲突
解释这个 commit 为什么导致测试失败：!git show abc123
```

### 常见误区

- ❌ 一次塞超大任务不拆分 → 先 Plan 后执行
- ❌ 上下文爆了硬撑 → `/compact` 或 `/clear` 重来
- ❌ 全程 Manual 逐条点确认 → 日常编码用 Auto 或 Edit automatically + allow 规则
- ❌ 把密钥写进 CLAUDE.md → 会被提交到 Git

## 十五、参考资料

- 官方文档：<https://code.claude.com/docs>
- [Claude Code Commands: Complete Slash Command Reference (2026)](https://explainx.ai/blog/claude-code-commands-complete-reference-guide-2026)
- [Claude Code in VS Code: Extension Setup, Shortcuts & Workflow Guide (2026)](https://www.morphllm.com/claude-code-vscode)
- [Claude Code Hooks: Complete Guide to All 30 Lifecycle Events](https://claudefa.st/blog/tools/hooks/hooks-guide)
- [Claude Code VS Code extension: a complete guide (2026)](https://www.eesel.ai/blog/claude-code-vs-code-extension)
- [What is the /context Command in Claude Code](https://www.claudelog.com/faqs/what-is-context-command-in-claude-code/)

> 最后提醒：Claude Code 更新频繁，命令、模式名称和可用档位都可能随版本变化。遇到与本文不一致的地方，请优先参考本地 `/help`、`claude --help` 的输出和官方文档。
