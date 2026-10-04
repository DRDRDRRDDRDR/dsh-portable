# DeepSeek Harness 便携包 (dsh-portable)

把 DeepSeek Harness（桌面程序本体 + 全部插件 + 配置 + 记忆模块 python 模型与设置）打包成开箱即用的便携包。**不含记忆内容本体、会话记录与任何凭据。**

## 下载

进入 [Releases](../../releases) 下载最新 `dsh-portable-*.zip` 与对应的 `.SHA256.txt`（校验文件）。

## 内容

- `app\` — DeepSeek Harness 桌面程序（含 Electron 资源、app.asar 运行内核、内置 node/pnpm 运行时）
- `dsh-data\` — 用户数据 `~/.dsh`：插件全量 node_modules（开箱即用，无需联网装依赖）、配置、记忆模块 python 模型与全部设置

### 插件状态（打包时刻）
- dsh-codearts-auth 0.3.1004（npm 正式版，含 benefit not found 修复）
- auto-memory 3.2.9 / cost-meter 1.8.11 / free-search 0.7.4 / zh_pro 0.9.6 /
  client-auto-continue 0.12.2 / image-pathify 0.2.1 / mcp-connector 0.2.66 /
  dshmarket 1.66.8 / better-sidebar 0.24.1 / dream-skin 9.29.0 / vision-toolkit 0.1.45
- dsh-auto-continue-429 含本地修复：source 改为 `{ kind: "plugin:auto-continue-429" }`，
  修复 session format v4 报「requires a producer-owned source kind」

### 记忆模块（python 模型 + 设置，已包含）
- 设置：`dsh-auto-memory.json`（记忆模块全部设置项）
- python 环境：`python-engine\.venv`（约 440 MB）
- 语义模型两份：
  - `python-engine\models`（sentencepiece + model_int8.onnx，约 570 MB）
  - `models\js-semantic\multilingual-e5-small`（onnx 约 118 MB）

## 明确未打包
- 记忆内容本体：`~/.dsh/memory`、各工作区 `.dsh-memory`
- 会话记录：`~/.dsh/sessions`、`~/.dsh/storages`
- 凭据：`.credentials.yaml`、openai-gateway api-key、jet-hub 账号状态
- 日志与缓存：logs / cache / attachments

## 恢复步骤（新机器）

1. 解压 `app\` 到任意目录（如 `%LOCALAPPDATA%\Programs\DeepSeek Harness`）。
2. 把 `dsh-data\` 下所有内容复制为 `C:\Users\<用户名>\.dsh\`。
3. 启动 `DeepSeek Harness.exe`。

无需联网、无需 pnpm install（node_modules 已全量包含）。

## 校验

下载后核对：

```powershell
Get-FileHash .\dsh-portable-*.zip -Algorithm SHA256
```

与 `.SHA256.txt` 内容一致即为完整包。

## 说明

- 本仓库仅分发便携包，不含 DeepSeek Harness 源码（那是 Electron 打包产物）。
- 如果只想装某个插件/配置，直接取 `dsh-data\profiles\desktop` 里的对应部分即可。
