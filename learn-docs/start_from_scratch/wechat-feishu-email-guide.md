# nanobot 微信 + 飞书 + 邮箱 全流程配置指南

本指南面向新手，从下载 nanobot 开始，逐步配置微信、飞书、邮箱三个通道，最终实现多端 AI 助手。

---

## 目录

- [一、环境准备](#一环境准备)
- [二、安装 nanobot](#二安装-nanobot)
- [三、配置 AI 模型](#三配置-ai-模型)
- [四、构建 WebUI（可选但建议）](#四构建-webui可选但建议)
- [五、启动 Gateway](#五启动-gateway)
- [六、配置微信通道](#六配置微信通道)
- [七、配置飞书通道](#七配置飞书通道)
- [八、配置邮箱通道](#八配置邮箱通道)
- [九、多通道同时运行](#九多通道同时运行)
- [十、常见问题](#十常见问题)

---

## 一、环境准备

### 1.1 确认 Python 版本

nanobot 需要 Python >= 3.11。在终端执行：

```bash
python --version
```

如果不是 3.11+，请先升级 Python。

### 1.2 使用 Conda 环境（推荐）

```bash
# 创建专用环境（如果还没有）
conda create -n ai-dev python=3.12 -y

# 激活环境
conda activate ai-dev
```

> 后续所有操作都在 `ai-dev` 环境下进行。

> `ai-dev` 只是示例名称。你可以使用任意 Conda 环境名，但后续激活的环境、
> 安装依赖和启动 nanobot 的环境必须保持一致。

### 1.3 进入项目目录

```bash
cd <nanobot 项目根目录>
```

例如，在 Windows 的 Git Bash 中可使用：

```bash
cd /d/PyProj/ollama-dev/nanobot-learn
```

---

## 二、安装 nanobot

### 2.1 安装 nanobot 核心

```bash
pip install -e .
```

`-e` 表示可编辑模式安装，修改源码后无需重新安装，适合学习和调试。

### 2.2 验证安装

```bash
python -m nanobot --help
```

如果能打印帮助信息，说明安装成功。

---

## 三、配置 AI 模型

nanobot 支持多种 AI 提供商。本指南以 DeepSeek 为例。

### 3.1 准备 API Key

如已有 DeepSeek API Key，可直接使用。推荐将 Key 写入 `.env` 或系统环境变量，
避免把真实密钥直接写进教程、聊天记录或提交到 Git：

```
DEEPSEEK_API_KEY=sk-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

当配置使用 `$DEEPSEEK_API_KEY` 时，应在启动 nanobot 的同一终端设置环境变量。例如：

```powershell
$env:DEEPSEEK_API_KEY = "sk-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
```

```bash
export DEEPSEEK_API_KEY=sk-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

### 3.2 创建配置文件

运行初始化向导：

```bash
python -m nanobot onboard --wizard
```

按提示选择：
- 提供商：选择 **OpenAI-compatible**，或直接选择 **DeepSeek**
- 模型 ID：`deepseek-chat`（或 `deepseek-v3-0324` 等可用模型）
- 启用 WebUI：选择 **Yes**
- WebUI 密码：设置一个密码，用于浏览器登录

向导会自动生成配置文件 `~/.nanobot/config.json`。

> 下文所有 JSON 都是**需要合并**到现有 `config.json` 的片段，不要用某个片段
> 覆盖整个文件，否则可能丢失已配置的模型、WebUI 或其他通道。

### 3.3 手动调整配置（可选）

如需指定 API 地址，编辑 `~/.nanobot/config.json`，确保 `deepseek` 段含 `apiBase`：

```json
{
  "providers": {
    "deepseek": {
      "apiKey": "$DEEPSEEK_API_KEY",
      "apiBase": "https://api.deepseek.com"
    }
  }
}
```

### 3.4 测试 AI 对话

```bash
python -m nanobot agent -m "你好，请用中文回复"
```

如果返回正常的 AI 回复，说明模型配置成功。

---

## 四、构建 WebUI（按安装方式选择）

WebUI 是浏览器对话界面。通过 pip 安装的已发布版本通常已经包含 WebUI，优先执行：

```bash
python -m nanobot webui
```

该命令会引导启用本地 WebUI、启动 Gateway 并打开浏览器。

本教程使用的是源码/可编辑安装方式；若访问 `http://127.0.0.1:8765/` 返回
`Not Found`，再按以下步骤手动构建前端资源。

### 4.1 安装依赖并构建

```bash
cd webui
npm install
npm run build
cd ..
```

如果 `npm install` 报 `E400`，且错误地址是 `registry.npmmirror.com`，可只对本次
安装改用官方 npm 源：

```bash
npm install --registry=https://registry.npmjs.org/
```

`npm audit` 的漏洞提示不等于构建失败。不要直接执行 `npm audit fix --force`，
它可能升级依赖并带来兼容性问题。

### 4.2 验证构建产物

```powershell
# PowerShell
Test-Path nanobot\web\dist\index.html
```

```bash
# Git Bash / macOS / Linux
ls nanobot/web/dist/index.html
```

PowerShell 输出 `True` 或 `ls` 能列出文件，说明构建成功。若 Gateway 已在构建前
启动，构建完成后还需要重启 Gateway。

---

## 五、启动 Gateway

Gateway 是 nanobot 的核心服务进程，所有通道的消息都由它接收和分发。

> 如果上一节已经用 `python -m nanobot webui` 成功启动服务，请不要再启动第二个
> Gateway。需要管理多个通道或手动排查时，再使用本节命令。

### 5.1 启动

```bash
python -m nanobot gateway
```

看到如下日志表示启动成功：

```
WebSocket server listening on ws://127.0.0.1:8765/
Health endpoint: http://127.0.0.1:18790/health
```

### 5.2 访问 WebUI

浏览器打开 `http://127.0.0.1:8765`，输入之前设置的密码即可。

### 5.3 查看通道状态

```bash
python -m nanobot channels status
```

此时 `websocket` 应显示为 `running` 或已启用状态。

> **提示**：Gateway 需要一直运行。后续配置各通道时，建议**另开一个终端窗口**执行命令，保持 Gateway 窗口不关。

---

## 六、配置微信通道

微信通道让你的 nanobot 成为一个"微信机器人"，可以在手机上通过微信与 AI 对话。

### 6.1 安装微信通道依赖

```bash
python -m nanobot plugins enable weixin
```

### 6.2 启用通道

编辑 `~/.nanobot/config.json`，在 `channels` 下添加：

```json
{
  "channels": {
    "weixin": {
      "enabled": true
    }
  }
}
```

> 不使用 `allowFrom` 表示启用配对模式：新用户需审批后才能使用。

### 6.3 扫码登录

```bash
python -m nanobot channels login weixin
```

终端会打印二维码或登录链接，用微信扫码即可完成登录。

> 如果已登录过，想换号或重新登录，加 `--force` 参数：
> ```bash
> python -m nanobot channels login weixin --force
> ```

### 6.4 配对审批（首次必做）

1. 重启 Gateway（确保微信通道生效）
2. 用手机微信给机器人**发一条私聊消息**
3. 机器人首次会回复一个配对码，格式如 `ABCD-EFGH`
4. 在电脑终端执行审批命令：

```bash
python -m nanobot agent -m "/pairing approve ABCD-EFGH"
```

5. 审批成功后，再发一条消息，机器人就会正常用 AI 回复了

### 6.5 群聊使用

默认支持。在微信群里 @机器人 即可触发。如需更精细控制，可添加 `allowFrom` 白名单限制。

### 6.6 WebUI 里自动审批

也可以直接在 WebUI 输入框输入 `/pairing approve ABCD-EFGH` 来审批，效果相同。

---

## 七、配置飞书通道

飞书通道使用 WebSocket 长连接，不需要公网 webhook URL。

### 7.1 安装飞书通道依赖

```bash
python -m nanobot plugins enable feishu
```

如果依赖下载超时，先重试插件安装；必要时使用受信任的镜像。

> 不要手动强制安装特定 `websockets` 版本，也不要忽略依赖解析冲突；请保留完整
> 报错信息后再排查，避免破坏当前 nanobot 环境。

### 7.2 扫码登录（推荐方式）

```bash
python -m nanobot channels login feishu
```

打开打印的 URL 跳转飞书开放平台完成授权。nanobot 会自动将 `appId`、`appSecret` 等写入配置。

### 7.3 手动配置（备选方式）

如果无法扫码，在飞书开放平台手动创建机器人应用，然后将配置写入 `~/.nanobot/config.json`：

```json
{
  "channels": {
    "feishu": {
      "enabled": true,
      "appId": "cli_xxxxxxxxxxxx",
      "appSecret": "$FEISHU_APP_SECRET",
      "groupPolicy": "mention",
      "streaming": true,
      "domain": "feishu"
    }
  }
}
```

> 手动创建时需在飞书开放平台开启：Bot 能力、接收消息事件、长连接模式。如果无法获取 `cardkit:card:write` 权限，设置 `"streaming": false`。

### 7.4 配对审批

和微信类似：
1. 上述配置之后，重启 Gateway
2. 在飞书给机器人**发一条私聊消息**
3. 机器人回复配对码
4. 在终端执行：

```bash
python -m nanobot agent -m "/pairing approve 配对码"
```

### 7.5 群聊使用

群聊中 @机器人 即可触发。默认只响应被 @ 的消息（`groupPolicy: "mention"`），防止在热闹群里被误触发。

### 7.6 流式卡片

飞书支持流式卡片输出，机器人回复会像打字一样逐步展开（需 `cardkit:card:write` 权限）。如果卡片显示异常，设置 `"streaming": false` 关闭。

---

## 八、配置邮箱通道

邮箱通道通过 IMAP 收件、SMTP 发件，将 nanobot 变成邮件 AI 助手。

### 8.1 准备邮箱

**强烈建议使用专用邮箱**，不要用主邮箱。推荐新建一个 Gmail 或 QQ 邮箱。

### 8.2 获取邮箱凭据

**Gmail 用户（推荐）：**
1. 确保 Gmail 已开启 IMAP：
   - Gmail 设置 → 查看所有设置 → 转发和 POP/IMAP → 启用 IMAP
2. 生成应用专用密码：
   - 需先开启两步验证，然后在 [Google 账户 → 安全性 → 应用专用密码](https://myaccount.google.com/apppasswords) 生成

**QQ 邮箱用户：**
1. 邮箱设置 → 账户 → 开启 IMAP/SMTP 服务
2. 生成授权码作为密码

### 8.3 配置通道

编辑 `~/.nanobot/config.json`，在 `channels` 下添加：

**Gmail 示例：**

```json
{
  "channels": {
    "email": {
      "enabled": true,
      "consentGranted": true,
      "imapHost": "imap.gmail.com",
      "imapPort": 993,
      "imapUsername": "my-nanobot@gmail.com",
      "imapPassword": "$GMAIL_APP_PASSWORD",
      "smtpHost": "smtp.gmail.com",
      "smtpPort": 587,
      "smtpUsername": "my-nanobot@gmail.com",
      "smtpPassword": "$GMAIL_APP_PASSWORD",
      "fromAddress": "my-nanobot@gmail.com",
      "allowFrom": ["your-real-email@gmail.com"],
      "autoReplyEnabled": true
    }
  }
}
```

**QQ 邮箱示例（IMAP 端口 993，SMTP 端口 587）：**

```json
{
  "channels": {
    "email": {
      "enabled": true,
      "consentGranted": true,
      "imapHost": "imap.qq.com",
      "imapPort": 993,
      "imapUsername": "your-nanobot@qq.com",
      "imapPassword": "$QQ_EMAIL_AUTH_CODE",
      "smtpHost": "smtp.qq.com",
      "smtpPort": 587,
      "smtpUsername": "your-nanobot@qq.com",
      "smtpPassword": "$QQ_EMAIL_AUTH_CODE",
      "fromAddress": "your-nanobot@qq.com",
      "allowFrom": ["your-real-email@qq.com"],
      "autoReplyEnabled": true
    }
  }
}
```

### 8.4 字段说明

| 字段 | 说明 |
|------|------|
| `consentGranted` | 设为 `true` 启用邮箱访问，`false` 完全禁用 |
| `imapHost` | IMAP 服务器地址 |
| `imapPort` | IMAP 端口，通常 993（SSL） |
| `imapUsername` | 邮箱账号 |
| `imapPassword` | 邮箱密码或授权码（推荐用环境变量） |
| `smtpHost` | SMTP 服务器地址 |
| `smtpPort` | SMTP 端口，Gmail 用 587，QQ 用 587 |
| `smtpUsername` | 发件认证账号 |
| `smtpPassword` | 发件认证密码或授权码 |
| `fromAddress` | 机器人发件地址 |
| `allowFrom` | **允许使用机器人的发件人列表**，设为 `["*"]` 表示任何人都可发邮件触发 |
| `autoReplyEnabled` | 是否自动回复收到的邮件 |
| `pollIntervalSeconds` | 收件轮询间隔，默认 30 秒 |

### 8.5 测试

1. 重启 Gateway
2. 用 `allowFrom` 列表中的邮箱，向机器人邮箱发一封邮件
3. 等待 Gateway 轮询到新邮件（默认约 30 秒，实际取决于邮箱服务），机器人会自动回复

### 8.6 安全提醒

- 邮箱通道**不使用配对码**，直接依赖 `allowFrom` 白名单控制访问
- 不要设为 `["*"]`，除非你完全清楚风险
- 邮箱密码强烈建议用环境变量替代硬编码，格式：`"imapPassword": "$IMAP_PASSWORD"`
- 不要提交 `.env`、`config.json` 或含真实凭据的截图；必要时先打码

---

## 九、多通道同时运行

nanobot 支持**同时启用多个通道**，所有消息统一由 AI 处理，各通道独立收发。

### 9.1 配置示例

三个通道同时启用的 `config.json` 示例：

```json
{
  "providers": {
    "deepseek": {
      "apiKey": "$DEEPSEEK_API_KEY",
      "apiBase": "https://api.deepseek.com"
    }
  },
  "channels": {
    "weixin": {
      "enabled": true
    },
    "feishu": {
      "enabled": true,
      "appId": "cli_xxxxxxxxxxxx",
      "appSecret": "$FEISHU_APP_SECRET",
      "groupPolicy": "mention",
      "domain": "feishu"
    },
    "email": {
      "enabled": true,
      "consentGranted": true,
      "imapHost": "imap.gmail.com",
      "imapPort": 993,
      "imapUsername": "my-nanobot@gmail.com",
      "imapPassword": "$GMAIL_APP_PASSWORD",
      "smtpHost": "smtp.gmail.com",
      "smtpPort": 587,
      "smtpUsername": "my-nanobot@gmail.com",
      "smtpPassword": "$GMAIL_APP_PASSWORD",
      "fromAddress": "my-nanobot@gmail.com",
      "allowFrom": ["your-real-email@gmail.com"],
      "autoReplyEnabled": true
    }
  }
}
```

### 9.2 启动方式

```bash
# 查看所有通道状态
python -m nanobot channels status

# 启动 Gateway（所有 enabled 通道同时启动）
python -m nanobot gateway
```

输出示例：

```
Channel     Status
websocket   running
weixin      running
feishu      running
email       running
```

### 9.3 消息流转

```
手机微信 ──→ ┐
飞书工作台 ─→ ├──→ Gateway ──→ AI 模型 ──→ 分别回复给各通道
邮箱来信 ──→ ┘
```

每个通道都可以独立对话，但回复只会返回到消息来源通道。默认情况下，会话按
通道和聊天对象隔离；例如微信中的上下文不会自动延续到 WebUI 或飞书。若需延续
上下文，请在同一通道继续对话，或在新通道提供必要的前文摘要。

### 9.4 常用命令速查

```bash
# 查看所有通道状态
python -m nanobot channels status

# 查看已安装的通道列表
python -m nanobot channels list

# 启用/禁用插件
python -m nanobot plugins enable <通道名>
python -m nanobot plugins disable <通道名>

# 登录通道（扫码或输入凭据）
python -m nanobot channels login <通道名>

# CLI 快速测试（不通过通道，直接命令行发消息）
python -m nanobot agent -m "你的问题"

# 配对审批
python -m nanobot agent -m "/pairing approve 配对码"
```

---

## 十、常见问题

### Q1: `nanobot: command not found`

用 `python -m nanobot` 代替，效果相同。例如：

```bash
python -m nanobot gateway
python -m nanobot agent -m "Hello"
```

### Q2: pip 安装超时

使用国内镜像：

```bash
pip install xxx -i https://mirrors.aliyun.com/pypi/simple/
```

### Q3: WebUI 打开后显示 "not found"

WebUI 前端没有构建。执行：

```bash
cd webui
npm install
npm run build
cd ..
```

然后重启 Gateway。

### Q3.1: `npm install` 报 `E400 Bad Request` 或镜像下载失败

通常是 npm 镜像配置失效。先只对本次安装改用官方 npm 源：

```bash
cd webui
npm install --registry=https://registry.npmjs.org/
npm run build
cd ..
```

不要为了消除 `npm audit` 提示直接运行 `npm audit fix --force`。

### Q4: LLM 返回 connection error

可能原因：
- `apiBase` 没有显式设置，在 `config.json` 中加上
- 网络代理干扰，尝试关闭代理或设置 `NO_PROXY`
- DeepSeek API 服务端偶发波动，稍后重试

### Q5: 微信扫码后没反应

- 确认 Gateway 在运行中
- 检查 `~/.nanobot/config.json` 中 `weixin.enabled` 为 `true`
- 尝试加 `--force` 重新登录：`python -m nanobot channels login weixin --force`

### Q6: 飞书扫码授权后失败

- 检查飞书开放平台是否开启了"长连接模式"
- 确认 Bot 权限包含"接收消息"事件
- 查看 Gateway 日志：`python -m nanobot gateway --verbose`

### Q7: 邮箱不发邮件

- 确认 `autoReplyEnabled` 为 `true`
- 检查 SMTP 端口和密码
- Gmail 必须用应用专用密码，不能用账号密码
- 发件人地址必须在 `allowFrom` 列表中

### Q8: 重启 Gateway 后微信/飞书需要重新登录吗？

不需要。登录状态持久化保存，重启自动恢复。只有用 `--force` 重新登录才需要再次扫码。

### Q9: 微信通道安全吗？

nanobot 使用个人微信 iLink API 的长轮询方式，不需要本地微信桌面客户端。仍应自行
评估平台规则和账号风险：
- 建议使用专用微信号，而非主号
- 使用配对模式（默认）控制谁可以使用机器人
- 有被封号的风险，请自行评估

### Q10: 配对审批是一次性的吗？

是的。每个用户只需审批一次，持久化保存，重启不丢失。只有清除配置或新增用户时才需要再次审批。
