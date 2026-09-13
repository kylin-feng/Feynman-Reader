# Feynman Reader — fork 推送测试

## 这是什么

这个仓库是 [`HachikoJ/Feynman-Reader`](https://github.com/HachikoJ/Feynman-Reader)（MIT） 的 fork，托管在 https://reader.deline.top/ 。

`FORK_TEST.md` 是 2026-09-13 的一次 fork + push 通道测试，目的是验证：

1. `gh repo fork` 能把官方仓库复制到 `kylin-feng/Feynman-Reader` ✅
2. 用 GitHub Contents API 直接 PUT，能对 fork 成功写入 ✅
3. push 权限链路（token `repo` scope + SSH keyring）可用 ✅

## 为什么没完整 clone 到本地

官方仓库体积约 46 MB（带完整 history + 大量截图/资源）。在这台机器上：

- 直连 `https://github.com/...git` 被网络拦截（Empty reply from server）
- 直连 `codeload.github.com` tarball 也超时（exit 28）
- `gh repo clone` 走 `gh auth` 通道成功但极慢（30+ 分钟没拉完）

因此改用 `gh api repos/{owner}/{repo}/contents/...` 直接对 fork 做提交来验证 push 通道。

## 恢复完整本地副本

网络恢复后可执行：

```bash
gh repo clone kylin-feng/Feynman-Reader
# 或
gh repo clone HachikoJ/Feynman-Reader
```

## 上游

- 官网：https://reader.deline.top/
- 源码：https://github.com/HachikoJ/Feynman-Reader
- License：MIT
