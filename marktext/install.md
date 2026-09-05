# MarkText 安装说明（Windows）

截至 2026-09-06，版本 v0.19.1。以下步骤均实测通过。

## 1. 下载

从 [Releases 页面](https://github.com/marktext/marktext/releases) 获取 Windows 安装包：

```
marktext-win-x64-0.19.1-setup.exe   # 约 106MB
```

GitHub 直连不稳时走镜像前缀下载（镜像方案见仓库根目录 [github-proxy.md](../github-proxy.md)）：

```bash
curl -L -o marktext-win-x64-0.19.1-setup.exe \
  "https://gh-proxy.com/https://github.com/marktext/marktext/releases/download/v0.19.1/marktext-win-x64-0.19.1-setup.exe"
```

实测在 2~3MB/s 镜像速度下约 41 秒下完 105.6MB。

## 2. 校验 SHA256（强烈建议）

Releases 附带 `SHA256SUMS.txt` 汇总了所有安装包的官方哈希。下载后本地比对：

```bash
# 下载校验文件
curl -sL -o SHA256SUMS.txt \
  "https://gh-proxy.com/https://github.com/marktext/marktext/releases/download/v0.19.1/SHA256SUMS.txt"

# 比对（只验 setup.exe 那一行）
grep "marktext-win-x64-0.19.1-setup.exe$" SHA256SUMS.txt | sha256sum -c -
```

v0.19.1 win-x64 setup 的官方 SHA256：

```
0de6c0aa854728f3e5c21d74b4138f0a515d70cd80921eb3aed6b8e975838a2f
```

实测输出 `marktext-win-x64-0.19.1-setup.exe: OK` 即通过。

## 3. 安装

双击安装包按向导安装；或静默安装（electron-builder 的 NSIS 包）：

```powershell
& ".\marktext-win-x64-0.19.1-setup.exe" /S
```

- 安装时**勾选关联 `.md` / `.markdown` 等扩展名**，之后双击 md 文件直接用 MarkText 打开。
- 默认安装到 `%LOCALAPPDATA%\Programs\marktext\`（用户级，无需管理员）。

## 4. 切换中文界面

MarkText 内置简体中文语言包（仓库 `packages/desktop/static/locales/zh-CN.json`），无需汉化补丁：

1. 打开 MarkText
2. `File` → `Preferences`（或 `Ctrl+,`）
3. `General` → `Language` → 选择 **简体中文 (zh-CN)**
4. 重启应用生效

> 在中文版 Windows 上，首次启动大概率直接就是中文界面（跟随系统语言）。

## 5. 验证

启动后（Electron 应用，进程名 `MarkText`）：

- 能正常打开并实时预览 `.md` 文件
- `Ctrl+,` 偏好设置可打开，语言列表含 `zh-CN`

## 常见问题

- **下载被浏览器/卫士拦截**：Setup 是 electron-builder NSIS 包，未签名，部分安全软件会误报。走上面命令行下载 + SHA256 校验通过后可放心安装。
- **静默安装没反应**：NSIS `/S` 区分大小写，必须大写 S；也可直接双击走向导。
- **`.md` 关联被 VS Code 等抢走**：右键 `.md` → 打开方式 → 选 MarkText 并勾"始终"；或在 Windows 设置 → 应用 → 默认应用里按扩展名指定。
- **卸载**：安装目录下的 `Uninstall MarkText.exe`，或 Windows 设置 → 应用里卸载。
