# OpenDesign 工具集

[OpenDesign](https://github.com/nexu-io/open-design) 是开源的 Claude Design 替代品：本地优先的桌面应用，把你的编码 agent CLI（Claude Code / dsh / Codex / Cursor 等 20+ 种）当作"设计引擎"，生成原型、落地页、仪表盘、幻灯片、图像与视频，可导出 HTML / PDF / PPTX / MP4。

> 截至 2026-09-05，实测版本 v0.21.1（Community First: No Login Required，无需登录）。

## 文档索引

| 文档 | 内容 |
|------|------|
| [install.md](install.md) | Windows 安装说明（下载、SHA256 校验、静默安装、常见问题） |
| [integration.md](integration.md) | 对接说明：接入 DeepSeek Harness (dsh)、**接入自定义 CLI**（本地 Profile / ACP 协议 / MCP 三条路线） |

## 一句话总结

- **它是独立桌面应用**，不是任何 agent 的"插件"——方向是它把你的 CLI 当引擎，而不是把它装进某个 CLI。
- 装好后它会自动扫描 PATH 上已安装的 agent CLI，扫描到的会出现在 agent 选择器里。
- 对 DeepSeek Harness (dsh) 是一等公民支持（native runtime）。
