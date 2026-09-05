# OpenDesign 对接说明

OpenDesign 的 agent 对接架构：daemon 自动扫描 PATH 上已安装的 agent CLI，为每种 CLI 定义一个**纯数据适配器**（`RuntimeAgentDef`），引擎统一负责探测、启动、调用和流式解析。给用户的扩展点有三层：

```
本地 Profile 配置（零代码）  >  实现 ACP 协议（CLI 侧改造）  >  MCP（只暴露工具）
```

---

## 一、接入 DeepSeek Harness (dsh) —— 一等公民支持

OpenDesign 内置了 dsh 的 native runtime 适配器（安装目录 `resources/open-design/agent-runtimes/deepseek-harness/`），支持结构化思考、工具调用、模型发现、取消与 session resume。

**步骤：**

1. 安装 dsh CLI（全局 npm 包）：
   ```bash
   npm install -g @deepseek-ai/dsh
   ```
2. 在 dsh 侧配置模型：启动 `dsh web`（默认 `http://127.0.0.1:3080`）→ Settings → Models → 填 DeepSeek API key。
3. 打开 OpenDesign → 设置/Agents 页 → 运行时选择 **DeepSeek Harness**（daemon 扫描 PATH 自动发现）。
4. 官方 CLI 形式的接入命令：`od agent setup deepseek-harness`（需 OD daemon 运行中）。

**重要认知**：dsh 不是装进 OpenDesign 的"插件"，OpenDesign 也不是 dsh 的"插件"——前者是桌面应用，后者是被它当作引擎的 agent 运行时。GitHub topic `dsh-plugin` 里的项目**不都能互相安装**，先确认集成方向。

---

## 二、接入自定义 CLI —— 本地 Profile（零代码，推荐先试这个）

daemon 启动时自动读取用户配置文件 `~/.open-design/agents.local.json`（Windows 即 `C:\Users\<你>\.open-design\agents.local.json`；可用环境变量 `OD_AGENT_PROFILES_CONFIG` 指向别处），把其中每个条目作为一个 agent 注册。

### 配置文件格式

```json
{
  "agents": [
    {
      "id": "my-cli",
      "name": "我的 CLI 工具",
      "baseAgent": "claude",
      "bin": "my-cli",
      "args": ["--prefix-flag"],
      "env": { "MY_API_KEY": "xxx" },
      "models": [{ "id": "my-model", "label": "My Model" }],
      "defaultModel": "my-model",
      "versionArgs": ["--version"],
      "helpArgs": ["--help"]
    }
  ]
}
```

### 字段说明

| 字段 | 必填 | 说明 |
|------|------|------|
| `id` | ✅ | 唯一标识，`[A-Za-z0-9][A-Za-z0-9._-]{0,79}`，不能与内置 id 冲突 |
| `baseAgent` | ❌ | 继承哪个内置适配器（默认 `claude`），决定传 prompt 方式和输出流解析 |
| `name` | ❌ | 显示名，默认用 `id` |
| `bin` | ❌ | 可执行文件名（需在 PATH）或绝对路径，默认继承 base 的 |
| `args` | ❌ | 前置参数，拼接在 base 的 `buildArgs` 结果之前 |
| `env` | ❌ | 注入子进程的环境变量（键名须合法） |
| `models` | ❌ | 模型列表（字符串数组或 `{id,label}` 数组） |
| `defaultModel` | ❌ | 默认模型 id |
| `versionArgs` / `helpArgs` | ❌ | 探测/能力询问参数 |

**核心机制**：profile 继承 `baseAgent` 的全部行为（含 `streamFormat`），只覆盖声明的字段。所以选对 `baseAgent` = 你的 CLI 得兼容哪种"方言"：

| 你的 CLI 兼容 | baseAgent 填 |
|---|---|
| Claude Code 风格（`--output-format stream-json`，stdin 收 prompt） | `claude` |
| ACP 协议（JSON-RPC over stdio） | `devin` / `hermes` / `kimi` 等 ACP 系 |
| Codex 风格 | `codex` |
| OpenCode 风格 | `opencode` |

内置适配器 id 一览（`apps/daemon/src/runtimes/defs/`）：aider, amp, amr, antigravity, atomcode, byok-opencode, claude, **codebuddy**, codex, copilot, cursor-agent, **deepseek-harness**, deepseek, devin, grok-build, hermes, kilo, kimi, kiro, mimo, opencode, pi, qoder 等。

改完**重启 OpenDesign**，daemon 探测到 `bin` 后即出现在 agent 选择器。

---

## 三、实现 ACP over stdio（CLI 是全新协议时）

官方推荐的新运行时接入形态（源文档 `docs/new-agent-runtime-acp.md`）：

```
OpenDesign daemon ──spawn──> your-cli acp
   stdin  <- JSON-RPC 请求
   stdout -> JSON-RPC 响应/通知（必须可按行解析，严禁混入日志）
   stderr -> 仅日志与诊断
```

需要实现的 JSON-RPC 方法：

| 方向 | 方法 | 说明 |
|------|------|------|
| 收 | `initialize` | 首条，含客户端元数据与能力 |
| 收 | `session/new` / `session/load` | 建会话/续会话，含工作目录 |
| 收 | `session/set_model` / `set_config_option` | 可选，模型选择 |
| 收 | `session/prompt` | 用户 prompt（text + resource_link 图片块） |
| 收 | `session/cancel` | 可选，取消当前轮 |
| 发 | `initialize` 响应 | 能力声明 |
| 发 | `session/new` 响应 | 必须含可用 `sessionId` |
| 发 | `session/update` 通知 | `agent_thought_chunk` / `agent_message_chunk` / `tool_call` / `tool_call_update` |
| 发 | `session/prompt` 响应 | 标记本轮结束，尽量带 usage |

生命周期要点：避免交互式终端提示（权限用 `session/request_permission`）；prompt 完成后 stdin 关闭即干净退出，或容忍 SIGTERM。协议参考 [agentclientprotocol.com](https://agentclientprotocol.com/protocol/transports)。

实现后用路线二的 profile 挂进来（`baseAgent` 选 ACP 系适配器），或给官方提 PR：在 `apps/daemon/src/runtimes/defs/` 加一个 `<cli>.ts` 并注册进 `registry.ts`——官方称这是"单文件改动"。

---

## 四、MCP：只暴露工具给 agent 调用

如果目标不是"当 agent"，只是让各类 agent 能调用你的工具：

```bash
od mcp [--daemon-url <url>]   # 起 stdio MCP server，代理项目工具调用
od mcp install <agent>        # 写入对应 agent 的 MCP 配置（claude/codex/cursor 等）
```

---

## 参考源码

- 适配器契约：`apps/daemon/src/runtimes/types.ts`（`RuntimeAgentDef`）
- 本地 Profile 加载：`apps/daemon/src/runtimes/local-profiles.ts`
- 注册表：`apps/daemon/src/runtimes/registry.ts`
- ACP 会话实现：`apps/daemon/src/agent-protocol/acp/session.ts`
