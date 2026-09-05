# AGENTS.md

本仓库是一个**好用工具的收集库**：记录各类实用工具的安装方法、对接/集成方式、踩坑经验与加速技巧，供自己和他人快速复用。

## 目录结构

| 路径 | 内容 |
|------|------|
| `opendesign/` | OpenDesign（开源 Claude Design 替代品，桌面设计应用）的安装与对接说明 |
| `github-proxy.md` | GitHub 访问加速方案（镜像克隆、release 下载、回滚方法） |

## 约定

- 每个工具一个目录，目录内放 `README.md`（总览/索引）+ 主题文档（安装、对接、踩坑等）。
- 文档中记录的命令和配置**必须是实际验证过的**，不要照抄官方 README 而不实测。
- 涉及版本的内容注明"截至何日、何版本"，因为工具迭代很快。
- 网络相关操作默认走本仓库 [`github-proxy.md`](github-proxy.md) 中配置的镜像加速。

## 给 Agent 的提示

- 本机器的 git 已配置 `url.<mirror>.insteadOf` 重写：`https://github.com/` 的 fetch/clone 自动走 gh-proxy.com 镜像，push 走直连。克隆任何 GitHub 仓库直接用原始 URL 即可。
- 修改文档后直接提交（commit），commit message 用中文、一行主题即可。
