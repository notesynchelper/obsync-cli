# obsync-cli — 手机电脑同步 · 命令行客户端

「手机电脑同步」（obsync）的**无界面命令行客户端**：在**服务器 / NAS / 群晖 / cron / CI** 上同步 Obsidian 笔记库，而不必开一个带图形界面的 Obsidian。安装后命令是 `obsync`。

和手机 / 电脑插件用的是**同一套端到端加密同步引擎**——笔记在离开本机前就已用你本地掌管的密钥加密，服务端只存密文。

## 📖 完整使用教程

👉 **<https://shoujidiannao.bijitongbu.site/tutorials/cli>**

（安装、登录、接入库、常驻同步 / cron / systemd、命令面、退出码、数据安全等完整图文说明都在这里。）

更多教程见教程中心：<https://shoujidiannao.bijitongbu.site/tutorials/>

## 快速安装

前置：Node.js ≥ 20，以及一个已经存在的同步库（新建库请先用手机 / 电脑插件或网页版）。

```bash
# 用 npm 从下载地址直接安装（推荐）
npm install -g https://shoujidiannao.bijitongbu.site/downloads/obsync-cli-latest.tgz

# 或下载自包含单文件（除 Node ≥20 外零依赖）
curl -fL -o /usr/local/bin/obsync https://shoujidiannao.bijitongbu.site/downloads/obsync.cjs
chmod +x /usr/local/bin/obsync

obsync --version
```

## 快速上手

```bash
obsync login                                   # 微信 6 位码登录
obsync sync-list-remote                        # 看有哪些库
cd ~/my-vault
obsync sync-setup --recovery-key-file key.txt  # 用恢复密钥接入一个已有库
obsync sync                                     # 同步一轮
obsync sync --continuous                        # 常驻自动同步
```

每一步的详细说明、cron / systemd 常驻、配置项、退出码、数据安全说明 —— 都见上面的**完整使用教程**。

## 说明

- 当前为**闭源试用版**，仅发布可执行文件，不含源码。
- 端到端加密，密钥本地掌管，服务端只存密文。**请保管好你的恢复密钥。**
- 问题反馈：<https://github.com/notesynchelper/obsync-cli/issues>
