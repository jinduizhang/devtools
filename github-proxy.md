# GitHub 访问加速方案（gh-proxy.com 镜像）

国内访问 `https://github.com/` 时通时断。本方案不依赖任何代理/VPN，用公共镜像 `gh-proxy.com` 重写 git 地址，2026-09-05 实测有效。

## 一、git 全局配置（fetch 走镜像、push 走直连）

```bash
# fetch/clone 走镜像：https 与 ssh 形式地址都重写
git config --global url."https://gh-proxy.com/https://github.com/".insteadOf "https://github.com/"
git config --global --add url."https://gh-proxy.com/https://github.com/".insteadOf "ssh://git@github.com/"

# push 走直连（镜像不支持 push，且避免凭证经过第三方）
git config --global url."https://github.com/".pushInsteadOf "https://gh-proxy.com/https://github.com/"
```

配置后**所有** GitHub 仓库直接用原始 URL 操作即可，无感知加速：

```bash
git clone https://github.com/owner/repo.git   # 自动经镜像
git pull                                       # 自动经镜像
git push                                       # 自动直连
```

### 踩坑：push 报 `could not read Username for 'https://gh-proxy.com'`

全局 `insteadOf` 对 `push` 同样生效（pushInsteadOf 只在 remote URL 本身是镜像地址时才生效）。实测结论：

- ❌ remote 是原始地址 `https://github.com/owner/repo.git` → push 被重写到镜像 → 镜像要求认证，失败
- ❌ remote 是镜像地址 + 显式设置了 `remote.<name>.pushurl` → 显式 pushurl 不受 pushInsteadOf 重写 → push 仍走镜像，失败
- ✅ **remote 设为镜像地址，且不设置 pushurl** → pushInsteadOf 自动把 push 转回直连

所以克隆后要推送的仓库，把 remote 改成镜像形式（只改这一处）：

```bash
git remote set-url origin "https://gh-proxy.com/https://github.com/owner/repo.git"
# 不要执行 git remote set-url --push origin ...
```

`git remote -v` 应显示：fetch 是镜像地址、push 是 `https://github.com/...`（说明规则生效）。

验证重写是否生效（看 GET 路径是否带 gh-proxy 前缀）：

```bash
GIT_TRACE_CURL=1 git ls-remote https://github.com/octocat/Hello-World.git HEAD 2>&1 | grep GET
# 期望输出: GET /https://github.com/octocat/Hello-World.git/info/refs?...
```

## 二、回滚（镜像失效时）

```bash
git config --global --unset-all url.https://gh-proxy.com/https://github.com/.insteadof
git config --global --unset url.https://github.com/.pushinsteadof
```

## 三、非 git 场景：release / raw 文件下载

镜像同样加速文件下载，在原 URL 前加前缀即可：

```bash
# 原始地址
https://github.com/owner/repo/releases/download/v1.0.0/file.exe
# 镜像加速
https://gh-proxy.com/https://github.com/owner/repo/releases/download/v1.0.0/file.exe
```

GitHub API 也可走同一前缀（用于查 release 列表等）：

```bash
curl -s "https://gh-proxy.com/https://api.github.com/repos/owner/repo/releases/latest"
```

## 四、镜像选型实测（2026-09-05，`git ls-remote` 计时）

| 通道 | 耗时 | 结论 |
|------|------|------|
| **gh-proxy.com** | 6.8s | ✅ 最快，已选用 |
| 直连 github.com | 7.6s | 时通时断 |
| ghfast.top | 20.4s | 可用但慢 |
| gitclone.com | 504 错误 | 不可用 |

镜像服务质量随时变化，失效时先换上面列表里的备选，改 `insteadOf` 里的域名即可。

## 五、注意事项

- **私有仓库**：clone/fetch 走镜像时，认证 token 会经过第三方镜像服务。敏感仓库建议临时禁用重写直连操作。
- **push 永远直连**：`pushInsteadOf` 已保证，不会被镜像截获。
- 本方案不涉及任何代理/VPN 工具，仅使用公开镜像服务。
