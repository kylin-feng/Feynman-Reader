# Fork 设置与本地运行说明

本文档记录 `kylin-feng/Feynman-Reader` 这个 fork 是怎么建起来的、当前在 `kylin-feng` 账号下的状态、以及如何在你本地把它跑起来。

## 1. 仓库关系

- **上游（官方）**：[`HachikoJ/Feynman-Reader`](https://github.com/HachikoJ/Feynman-Reader)
  - License: MIT
  - 官网: <https://reader.deline.top/>
  - 当前版本: v0.4.2
- **Fork（本仓库）**：[`kylin-feng/Feynman-Reader`](https://github.com/kylin-feng/Feynman-Reader)
  - 默认分支: `master`
  - 关系: `parent = HachikoJ/Feynman-Reader`，`fork = true`，`allow_forking = true`

## 2. 这个 fork 已经做了的事

### 2.1 推送通道验证（commit `6128d1b1`）

第一次提交通过 **GitHub Contents API** 完成，目的是验证 fork + push 权限链路：

```bash
gh repo fork HachikoJ/Feynman-Reader --clone=false
gh api \
  --method PUT \
  repos/kylin-feng/Feynman-Reader/contents/FORK_TEST.md \
  -f message="test: verify push channel via Contents API (kylin-feng fork)" \
  -f branch=master \
  -f content="<base64>"
```

提交后 `FORK_TEST.md` 落到 fork 的 `master` 分支，作者 `_kylin_`。

### 2.2 真实 `git push`（commit `2e155a2`）

在解决代理问题后，用真正的 git 协议完成了一次 push：在 `README.md` 末尾追加一行注脚（说明这次推送经过的代理），commit + push。

```bash
git config --global http.proxy socks5://127.0.0.1:7897
git config --global https.proxy socks5://127.0.0.1:7897
git clone --depth=1 https://github.com/kylin-feng/Feynman-Reader.git
# 编辑 README.md
git add README.md
git commit -m "docs: add proxy-fork timestamp line to README"
git push origin master   # 成功：6128d1b..2e155a2 master -> master
```

### 2.3 本地依赖与构建验证

在本机（macOS, Node v22.23.2, npm 10.9.8）成功执行：

```bash
npm ci              # 953 包，无错误
npm run build       # next 16.3.3 webpack，Compiled successfully in 36.5s
                    # TypeScript 通过，38/38 静态页生成
```

构建产物只用于验证编译，没有发布。

### 2.4 本地运行验证（截图）

在 `NEXT_PUBLIC_FEYNMAN_LOCAL_AUTH_BYPASS=true` 的本地旁路模式下，`npm run dev` 启动后访问 <http://127.0.0.1:8080/> 真实渲染出书架界面（书卡《追风筝的人》、复习提示、标签筛选全部可见）。

![本地旁路模式书架](bookshelf-local-bypass.png)

> 截图脚本：`/tmp/shoot.mjs`（Playwright + 系统 Chrome，`networkidle` + 2.5s 等待）。
> Console 日志确认 IndexedDB 创建/打开/初始化完成 → 数据走浏览器本地存储，不需要 Postgres。

## 3. 复现：把上游 fork 到你自己的账号

```bash
# 1. 登录 gh CLI（任选一种认证方式）
gh auth login

# 2. 一行 fork
gh repo fork HachikoJ/Feynman-Reader --clone=false

# 3. 拉到自己机器
gh repo clone kylin-feng/Feynman-Reader   # 换成你的用户名
cd Feynman-Reader

# 4. 装依赖
npm ci

# 5. 跑起来
cp .env.example .env.local
# 生成两个密钥：
node -e "console.log('FEYNMAN_AUTH_STATE_SECRET=' + require('crypto').randomBytes(32).toString('base64'))"
node -e "console.log('FEYNMAN_API_KEY_ENCRYPTION_KEY=' + require('crypto').randomBytes(32).toString('base64'))"
# 把输出填进 .env.local
npm run dev
# → http://localhost:8080
```

## 4. 网络代理（如果你在国内）

直连 GitHub 在一些网络下会被拦截（`Empty reply from server` / codeload 超时）。如果你本地跑了 Clash / Clash Verge / mihomo，把 git 走代理即可：

```bash
git config --global http.proxy  socks5://127.0.0.1:7897
git config --global https.proxy socks5://127.0.0.1:7897
# 端口换成你的代理端口（Clash Verge 默认 7897）
```

macOS 系统代理对浏览器生效；git / curl / npm 这些终端命令需要上面的全局代理才走代理。

## 5. 本地旁路模式 vs. 完整云端模式

`.env.local` 里关键的开关：

```env
# === 本地预览（不需要 Postgres / OAuth / 管理员绑定） ===
NEXT_PUBLIC_FEYNMAN_LOCAL_AUTH_BYPASS=true
FEYNMAN_WATCHA_OAUTH_ENABLED=false
FEYNMAN_TOKENDANCE_ENABLED=false
FEYNMAN_DEEPSEEK_OFFICIAL_ENABLED=true

# === 完整云端模式（生产部署，需要全部 .env.example 字段） ===
NEXT_PUBLIC_FEYNMAN_LOCAL_AUTH_BYPASS=false
FEYNMAN_WATCHA_OAUTH_ENABLED=true
FEYNMAN_TOKENDANCE_ENABLED=true
FEYNMAN_DEEPSEEK_OFFICIAL_ENABLED=false
DATABASE_URL=postgresql://...            # 必须可达
FEYNMAN_PUBLIC_ORIGIN=https://your.domain
TOKENDANCE_OAUTH_CLIENT_ID=...          # 观猹凭据
TOKENDANCE_OAUTH_CLIENT_SECRET=...
FEYNMAN_ADMIN_USER_ID=...                # 管理员绑定（可选）
FEYNMAN_ADMIN_PROVIDER_SUBJECT=...
```

**注意**：本地旁路模式下，所有书架 / 笔记 / 练习数据都只存在你浏览器的 IndexedDB 里，不会同步到任何云端。清浏览器数据会丢。

## 6. 这台机器（kylin-feng）上的实际配置

`/Users/shixianping/dev/feynman-reader-clone/Feynman-Reader/.env.local`（不提交，已在 `.gitignore`）：

```env
DATABASE_URL=postgresql://feynman_app:placeholder@127.0.0.1:5432/feynman_reader
FEYNMAN_AUTH_STATE_SECRET=<openssl rand -base64 32>
FEYNMAN_API_KEY_ENCRYPTION_KEY=<openssl rand -base64 32>
FEYNMAN_WATCHA_OAUTH_ENABLED=false
FEYNMAN_TOKENDANCE_ENABLED=false
FEYNMAN_DEEPSEEK_OFFICIAL_ENABLED=true
NEXT_PUBLIC_FEYNMAN_LOCAL_AUTH_BYPASS=true
FEYNMAN_PUBLIC_ORIGIN=http://127.0.0.1:8080
```

## 7. 待办 / 后续

- [ ] 网络环境允许时跑一次完整 `npm run dev` 看是否一切正常（仅本机验证用，不部署）
- [ ] 如果想给官方贡献 PR：从 `kylin-feng/Feynman-Reader:master` 拉分支 → 改代码 → `gh pr create --repo HachikoJ/Feynman-Reader`
- [ ] 上游有新版本时同步：`gh repo sync kylin-feng/Feynman-Reader --source HachikoJ/Feynman-Reader --branch master`
- [ ] 不需要这个 fork 时，到 <https://github.com/kylin-feng/Feynman-Reader/settings> → Danger Zone → Delete
