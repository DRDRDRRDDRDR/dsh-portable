# DeepSeek Harness 便携包（2026-10-05）

---

# ⭐ 先看这个：迁移到官方 DeepSeek Harness

**如果你要把这套配置迁到官方 DeepSeek Harness**，把下面整段（含代码块内容）复制给任意 AI 助手，或直接照着做。完整版另见包内 `迁移Prompt-可直接复制.md`。

```
我要把当前这套 DeepSeek Harness（便携包里的 dsh-data）迁移到官方 DeepSeek Harness，请按下面步骤帮我：
（先确认官方版本已安装，启动过一次并正常打开 GUI；关闭它再开始）

【第 0 步：确认路径】
- 官方程序目录：通常是 %LOCALAPPDATA%\Programs\DeepSeek Harness
- 便携包数据目录：本压缩包解压后的 dsh-data\（相当于 ~/.dsh）
- 用户数据目录：C:\Users\<我的用户名>\.dsh

【第 1 步：备份现有数据（必做）】
把现有 C:\Users\<用户名>\.dsh 整个复制一份到 C:\Users\<用户名>\.dsh-backup-<日期时间>\
若 .dsh 里已有 .credentials.yaml（登录凭据），备份时原样保留。

【第 2 步：迁移数据】
把便携包 dsh-data\ 下的内容复制到 C:\Users\<用户名>\.dsh\，规则：
- 保留我原有的 .credentials.yaml（包里不含凭据，不要覆盖）
- 保留我原有的 sessions\ 和 storages\（历史会话，包里不含）
- 覆盖/补齐：profiles\、skills\、python-engine\、models\、dsh-runtimes\、recovery\
- 复制这些配置：dsh-auto-memory.json、desktop-host-plugins.json、
  desktop-host-sources.patch.yml、auto-continue-429.json、dream-skin.json、
  auto-memory-archive-ledger.json、settings.yaml.imported
- 若官方版要求某配置用它自己的默认值，先保留官方默认，把我的版本另存为
  <原名>.from-portable 备用，并告诉我哪些文件被这样处理

【第 3 步：检查插件是否齐全】
打开 profiles\desktop\package.json，核对 dependencies 里的插件是否都在，
并确认 profiles\desktop\node_modules 已随包带来（本包已全量包含，无需联网装依赖）。
若某个插件目录缺失或损坏，在 profiles\desktop 下执行 pnpm install。
务必保留 profiles\desktop\pnpm-workspace.yaml 里的 minimumReleaseAge: 0
（否则 pnpm 会以「发布未满 32 小时」为由拒绝安装依赖）。
核对 dsh.profile.bundles 列表，没列进去的插件不会加载。

【第 4 步：记忆模块（python 语义检索）自检】
确认三份东西就位：
  1) python-engine\.venv\（专用 Python 环境，约 440MB）
  2) python-engine\models\（model_int8.onnx + sentencepiece，约 570MB）
  3) models\js-semantic\multilingual-e5-small\（onnx，约 118MB）
检查 dsh-auto-memory.json 里 pythonBackendWorkerPath 指向的路径实际存在，
semanticEngineMode 为 "python"、pythonBackendEnabled 为 true。
若日志报「python backend 不可用 / 找不到模型」，把报错原文发我。

【第 5 步：启动并验证】
启动官方 DeepSeek Harness，逐项确认：
- GUI 能正常打开
- 设置→插件页能看到插件且无报错
- 记忆模块能检索到历史记忆
- 发一条消息能正常收到回复
把任何报错原文（含完整堆栈）发给我，不要自行删改配置绕过报错。

【纪律】
- 每步先说明要动哪些文件，确认后再动
- 不要删除或覆盖 .credentials.yaml、sessions\、storages\
- 不要贴出任何 token / 密钥 / 凭据内容
- 迁移失败就从第 1 步的备份整体还原
```

**迁移前须知**：官方程序请自行从其官方渠道安装（本包只搬数据不搬程序）；凭据、历史会话、记忆内容均不随包，迁移后需重新登录。

---

## 这是什么

"DSH" 即 DeepSeek Harness。本包 = 程序本体 + 用户数据：

- `app\` — DeepSeek Harness 桌面程序（0.2.0-rc.2）：主程序 exe、Electron 资源、`resources\app.asar`（运行内核）、原生模块、内置 node/pnpm 运行时
- `dsh-data\` — 用户数据 `~/.dsh`：插件全量 node_modules（开箱即用）、配置、记忆模块 python 模型与全部设置

## 打包时插件状态（2026-10-05）

启用插件（13 个）：
- `dsh-codearts-auth` 0.3.1004（npm 正式版，含 benefit not found 修复）
- `@a9i5k4/dsh-auto-memory` 3.2.9（记忆模块）
- `dsh-cost-meter` 1.8.11 · `dsh-free-search` 0.7.4 · `dsh-image-pathify` 0.2.1
- `dsh-mcp-connector` 0.2.66 · `dshmarket` 1.66.8 · `dsh-dream-skin` 9.29.0
- `dsh-client-auto-continue` 0.12.2 · `dsh-my-guardian` 0.4.4 · `i-am-yuike` 1.0.7
- `@adwmc/helm-d`（GitHub release 直装）
- `dsh-infinite-gen-4`（git 源 jipika 分支，钉 commit e85918f）

## 明确未打包

- 记忆内容本体（`memory\`、各工作区 `.dsh-memory`）
- 会话记录（`sessions\`、`storages\`）
- 全部凭据（`.credentials.yaml`、openai-gateway key、jet-hub 状态）
- 日志与缓存（logs / cache / attachments / .plugin-manager / .generations）
- 本地备份与 `.bak-*` 文件

## 记忆模块（python 模型 + 设置，已包含）

- 设置：`dsh-data\dsh-auto-memory.json`
- python 环境：`dsh-data\python-engine\.venv\`（约 440 MB）
- 语义模型：`dsh-data\python-engine\models\`（约 570 MB）+ `dsh-data\models\js-semantic\multilingual-e5-small\`（约 118 MB）

## 恢复步骤（不想用官方版，直接用本包）

1. 解压 `app\` 到任意目录（如 `%LOCALAPPDATA%\Programs\DeepSeek Harness`）
2. 把 `dsh-data\` 下所有内容复制为 `C:\Users\<用户名>\.dsh\`
3. 启动 `DeepSeek Harness.exe`

无需联网、无需 pnpm install（node_modules 已全量包含）。

## 校验

```powershell
Get-FileHash .\dsh-portable-20261005.zip -Algorithm SHA256
```

与随附 `.SHA256.txt` 一致即为完整包。
