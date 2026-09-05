# OpenDesign 安装说明（Windows）

截至 2026-09-05，版本 v0.21.1。以下步骤均实测通过。

## 1. 下载

从 [Releases 页面](https://github.com/nexu-io/open-design/releases) 获取 Windows 安装包：

```
open-design-0.21.1-win-x64-setup.exe   # 约 400MB
```

GitHub 直连不稳时走镜像前缀下载（镜像方案见仓库根目录 [github-proxy.md](../github-proxy.md)）：

```bash
curl -L -o open-design-0.21.1-win-x64-setup.exe \
  "https://gh-proxy.com/https://github.com/nexu-io/open-design/releases/download/open-design-v0.21.1/open-design-0.21.1-win-x64-setup.exe"
```

## 2. 校验 SHA256（强烈建议）

Releases 里每个安装包都附带同名 `.sha256` 文件，记录了官方哈希。下载后本地比对：

```powershell
$hash = (Get-FileHash "open-design-0.21.1-win-x64-setup.exe" -Algorithm SHA256).Hash.ToLower()
$expected = (Get-Content "open-design-0.21.1-win-x64-setup.exe.sha256").Split(' ')[0]
if ($hash -eq $expected) { 'PASS' } else { 'FAIL' }
```

v0.21.1 win-x64 的官方 SHA256：

```
cd203b1c931fe1f7621929945b5aa8d2387a5c6fa5273cf66325a0b443224e7e
```

## 3. 安装

双击安装包按向导安装；或静默安装：

```powershell
& ".\open-design-0.21.1-win-x64-setup.exe" /S
```

- 安装路径可在向导中选择（示例机器装在 `D:\tool\Open Design\`）
- 该版本无需登录（Community First）
- 安装完成应用自动启动

## 4. 验证

启动后（Electron 应用，进程名 `Open Design`）：

- 应用主界面可正常打开
- 资源自带各 agent 的运行时适配器，位于安装目录 `resources/open-design/agent-runtimes/`（含 `deepseek-harness`）

## 常见问题

- **headless 模式起不来**：用系统 Node 直接跑 `resources/app/prebundled/daemon/daemon-cli.mjs` 会报 `better-sqlite3` ABI 不匹配——daemon 要靠桌面应用自带的 Node 运行，命令行方式仅适合排查，日常直接用 GUI。
- **卸载**：控制面板或安装目录下的 `Uninstall Open Design.exe`。
