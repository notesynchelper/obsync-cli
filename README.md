# obsync-cli — 手机电脑同步 · 命令行客户端

> 「手机电脑同步」（obsync）的 **无界面命令行客户端**。让你在 **服务器 / NAS / 群晖 /
> cron / CI** 上同步 Obsidian 笔记库，而不必开一个带图形界面的 Obsidian。
>
> 安装后命令是 `obsync`。当前为 **试用版（0.1.x）**，欢迎反馈。

---

## 这是什么

`obsync-cli` 复用「手机电脑同步」插件同一套端到端加密同步引擎，把它包成一个
Node 命令行工具。你的笔记在离开本机前就已用**你本地掌管的密钥**加密，服务端只存密文
——命令行版和手机 / 电脑插件是同一条链路、同一份数据。

适用场景：

- 一台 Linux / macOS 服务器上放着一个 vault，想让它跟手机 / 电脑上的笔记保持同步；
- 用 `cron` 定时把服务器上生成的笔记（脚本产出、日报、剪藏）同步进你的库；
- NAS / 群晖上常驻一个同步进程；
- CI / 自动化流程里读写同一个加密库。

## 前置条件

1. **Node.js ≥ 20**（`node -v` 确认）。
2. 一个「手机电脑同步」账号 —— 通过微信公众号 **「笔记同步助手」** 的 6 位验证码登录。
3. **一个已经存在的同步库**。命令行版 v1 只**接入**已有的库，不新建库；新建库请先用
   手机 / 电脑插件或网页版创建，再用命令行接入。

## 安装

**方式一：npm 全局安装（推荐）**

```bash
npm install -g obsync-cli
obsync --version
```

**方式二：从 GitHub Release 手动安装（网络装 npm 不便时）**

到 [Releases](https://github.com/notesynchelper/obsync-cli/releases) 下载
`obsync-cli-<版本>.tgz`，然后：

```bash
npm install -g ./obsync-cli-0.1.0.tgz
```

或者直接下载 `obsync.cjs` 单文件（自带 `#!/usr/bin/env node`），`chmod +x` 后放进 PATH：

```bash
curl -fL -o /usr/local/bin/obsync \
  https://github.com/notesynchelper/obsync-cli/releases/latest/download/obsync.cjs
chmod +x /usr/local/bin/obsync
```

> 该单文件是自包含 bundle，除 Node.js 本身外无需安装任何依赖。

## 快速开始（5 步）

```bash
# 1) 登录（打印一个 6 位码，用微信关注公众号「笔记同步助手」并把码发过去）
obsync login

# 2) 看看账号下有哪些库
obsync sync-list-remote

# 3) 进入你要同步的目录，接入一个已有的库
cd ~/my-vault
#   —— 方式 A：在另一台已登录设备（插件）上「生成配对码」，然后：
obsync sync-setup --pairing-code 123456
#   —— 方式 B：用恢复密钥（从插件 / 网页导出的 32 字节密钥文件）：
obsync sync-setup --recovery-key-file ~/vault-key.txt

# 4) 同步一轮（下载 + 上传，直到收敛后退出）
obsync sync

# 5) 想让它一直守着自动同步：
obsync sync --continuous
```

- **配对码方式**会要求你在终端里核对一串 SAS 校验字（形如 `042-815 苹果 火车`），
  必须和另一台设备屏幕上显示的**完全一致**才输 `y` —— 这是防中间人的唯一人工闸，别跳过。
- 接入成功后会在库目录下建一个 `.obsync/` 状态目录（它**永远不会被上传**）。

## 命令一览

| 命令 | 作用 |
|---|---|
| `obsync login` | 微信 6 位码登录；`--token <jwt>` 可直注 token 跳过交互（自动化用） |
| `obsync logout` | 撤销服务端 token + 清本地凭据；`--local-only` 只清本地 |
| `obsync sync-list-remote` | 列出账号下可同步的库 |
| `obsync sync-setup` | 接入一个已有库：`--pairing-code <6位>` 或 `--recovery-key[-file]` |
| `obsync sync` | 同步一轮到收敛后退出；`--continuous` 常驻监听文件变化持续同步 |
| `obsync sync-config` | 读写本库配置：`get` / `set <key> <value>` |
| `obsync sync-status` | 只读查看同步状态（六态，绝不误报） |
| `obsync sync-unlink` | 本地解绑（删 `.obsync/`，**不动任何笔记、不删服务端数据**） |

每个命令都支持 `--json`（机器可读输出到 stdout，人类文案走 stderr）。

## 全局参数

| 参数 | 说明 |
|---|---|
| `--json` | 输出单个 JSON 对象到 stdout |
| `--path <dir>` | 指定库目录（默认当前目录） |
| `--timeout <sec>` | 单轮 `sync` 的收敛预算（默认 600 秒） |
| `--yes` | 跳过交互确认（危险操作用） |
| `--help` / `--version` | 帮助 / 版本 |

配置目录默认 `~/.config/obsync`（尊重 `XDG_CONFIG_HOME`，可用 `OBSYNC_CONFIG_DIR` 覆盖）。
登录凭据存 `~/.config/obsync/credentials.json`（权限 0600）。

## 退出码（自动化可依赖）

| code | 含义 |
|---|---|
| 0 | 成功 / 已收敛 |
| 1 | 未预期错误 |
| 2 | 用法错误（参数不合法） |
| 3 | 未登录 / 登录失效 → 先 `obsync login` |
| 4 | 未接入 / 库不存在 / 密钥不匹配 |
| 5 | **未收敛（瞬时，可重试）** —— 超时仍有待办，或扫描不健康。**不是失败**，下次重试即可 |
| 6 | 存在永久失败（确有文件无法同步，会给出清单） |
| 7 | 已有实例在跑（同一库同一时刻只允许一个进程） |
| 8 | 批量删除保护暂停 —— 需人工 `--confirm-mass-delete` 放行 |

> `cron` 里判断：`0` = 完事；`5` = 没跑完，下次周期会自然续上，**不要当成报警**。

## 配置项（`obsync sync-config`）

```bash
obsync sync-config get                       # 打印当前全部配置
obsync sync-config set conflictAction merge  # 冲突处理策略
obsync sync-config set excludedFolders '["templates",".trash"]'
obsync sync-config set perFileMax 52428800   # 单文件大小上限（字节）
obsync sync-config set deviceName my-nas      # 本设备名（显示用）
```

可配置键：`conflictAction` / `allowTypes.*` / `allowSpecialFiles` /
`excludedFolders` / `perFileMax` / `deviceName`。非法值会 exit 2 且**不写盘**。

## 数据安全

本工具遵循「**宁可不同步，绝不丢数据**」的第一铁律：

- `.obsync/` 状态目录被硬排除，**永远不上传**；
- `sync-unlink` 只删本地状态，**绝不触碰你的任何笔记文件、绝不删服务端数据**；
- 读到 0 字节 / 读失败的文件不会被当成正常版本推上去覆盖别的设备；
- 瞬时错误（断网、超时、5xx）会退避重试，**不会**谎报「同步失败」。

端到端加密：笔记正文与文件名在离开本机前即用你本地的密钥加密，服务端只见密文。
**请妥善保管你的恢复密钥** —— 它丢了，服务端也无法帮你解开任何数据。

## 常见问题

- **`obsync sync` 退出码是 5？** 正常 —— 表示这一轮没在预算内跑完（弱网 / 大库首次
  下载），再跑一次会从已落盘的进度续上。不是错误。
- **`sync-status` 显示 `disconnected`？** 只是当前没有活动连接，不代表坏了；跑一次
  `obsync sync` 即可。
- **提示未登录（exit 3）？** token 失效，重新 `obsync login`。
- **提示已有实例在跑（exit 7）？** 同一个库目录同一时刻只允许一个 `obsync` 进程；
  确认没有别的进程 / `--continuous` 常驻在跑同一个库。

## 版本与反馈

- 当前为 **试用版**，闭源分发（只发布可执行文件，不含源码）。
- 问题反馈：<https://github.com/notesynchelper/obsync-cli/issues>
