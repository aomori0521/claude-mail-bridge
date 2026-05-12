# claude-mail-bridge 📬

给你的 AI 一个邮箱。

两种模式，按需选择：

| | 轻量版 `bridge.py` | MCP 版 `mcp_server.py` |
|---|---|---|
| **谁做决定** | 脚本自动收发 | AI 自己决定收发 |
| **AI 感知** | AI 不知道自己在用邮箱 | AI 知道自己有邮箱 |
| **适合场景** | 让 AI 能被别人找到 | 让 AI 拥有邮箱能力 |
| **依赖** | `requests` | `mcp[cli]`, `uvicorn` |
| **需要** | 任何 OpenAI 兼容 API | MCP 兼容客户端 |

## 快速开始

### 1. 准备一个邮箱

任何支持 IMAP/SMTP 的邮箱都行：

| 邮箱 | IMAP 地址 | SMTP 地址 | 备注 |
|------|-----------|-----------|------|
| QQ 邮箱 | imap.qq.com:993 | smtp.qq.com:587 | 设置→账户→开启IMAP，生成授权码 |
| 163 邮箱 | imap.163.com:993 | smtp.163.com:465 | 设置→POP3/SMTP/IMAP→开启 |
| iCloud | imap.mail.me.com:993 | smtp.mail.me.com:587 | 需 App 专用密码 |
| Outlook | outlook.office365.com:993 | smtp.office365.com:587 | 需 App 密码 |
| Gmail | imap.gmail.com:993 | smtp.gmail.com:587 | 需 App 专用密码 |

### 2. 配置

```bash
cp config.example.json config.json
```

编辑 `config.json`，填入你的邮箱和 API 信息。详见 `config.example.json`。

**注意**：`password` 填的是邮箱的**授权码 / App 专用密码**，不是登录密码。

---

## 轻量版 bridge.py

脚本自动轮询收件箱，有新邮件就调 API 拿回复，自动发回去。

```
有人发邮件 → 脚本检查收件箱 → 调你的 API → 把回复发回去
```

### 安装

```bash
pip install requests
```

### 运行

```bash
python bridge.py
```

后台运行：

```bash
# nohup
nohup python bridge.py &

# 或 PM2
pm2 start bridge.py --name mail-bridge --interpreter python3

# 或 screen
screen -S mail-bridge
python bridge.py
# Ctrl+A, D 退出
```

### 配置说明

`config.json` 中 `api` 部分：

- `base_url`：你的中转站 API 地址，兼容 OpenAI `/v1/chat/completions` 格式
- `api_key`：你的 API key
- `model`：模型名
- `system_prompt`：AI 的人设

`bridge` 部分：

- `poll_interval`：检查新邮件的间隔（秒）
- `allowed_senders`：白名单，填邮箱地址，留空 `[]` 表示不限
- `max_body_chars`：邮件正文最大字符数
- `daily_limit`：每天最多调用 API 几次（默认 50），防止被刷爆钱包
- `sender_hourly_limit`：同一个发件人每小时最多几封（默认 5），防骚扰
- `auto_send`：**默认 false**，AI 写好的回复会保存到草稿箱，你审核后手动发送。设为 `true` 则自动发送（请确保你信任 AI 的回复质量）
- `notify_webhook`：收到新邮件并生成草稿后，推送通知到你的手机。支持 ntfy / Bark 等 webhook 地址

---

## MCP 版 mcp_server.py

把邮箱变成 AI 的工具。AI 可以主动查收件箱、读邮件、搜索、发邮件。

### 安装

```bash
pip install "mcp[cli]" uvicorn pydantic
```

### 工具列表

| 工具 | 功能 |
|------|------|
| `mail_inbox` | 查看收件箱最近的邮件 |
| `mail_read` | 读取某封邮件的完整内容 |
| `mail_search` | 按条件搜索邮件 |
| `mail_send` | 发送邮件 |
| `mail_folders` | 列出所有文件夹 |

### 运行

```bash
# 直接运行（默认端口 8877）
python mcp_server.py

# 指定端口
python mcp_server.py 9000

# PM2 后台
pm2 start mcp_server.py --name mail-mcp --interpreter python3
```

### 配置方式

**方式一：config.json**（和轻量版共用同一个文件，只读 `email` 部分）

**方式二：环境变量**

```bash
export MAIL_ADDRESS="your@email.com"
export MAIL_PASSWORD="your_app_password"
export IMAP_HOST="imap.qq.com"
export SMTP_HOST="smtp.qq.com"
export DISPLAY_NAME="我的小克"
python mcp_server.py
```

### 接入 Claude Desktop / Claude Code

在 MCP 设置里添加：

```json
{
  "mail": {
    "url": "http://127.0.0.1:8877/mcp"
  }
}
```

如果服务器在远程，用你的域名替换 `127.0.0.1`，建议套 Cloudflare Tunnel。

---

## 各邮箱授权码获取方法

### QQ 邮箱

1. 打开 [QQ 邮箱网页版](https://mail.qq.com)
2. 设置 → 账户 → 往下翻到「POP3/IMAP/SMTP/Exchange/CardDAV/CalDAV 服务」
3. 开启 IMAP/SMTP 服务
4. 按提示用密保手机发短信验证
5. 获得 16 位授权码，填到 config.json 的 `password` 里

### 163 邮箱

1. 打开 [163 邮箱网页版](https://mail.163.com)
2. 设置 → POP3/SMTP/IMAP
3. 开启 IMAP/SMTP 服务
4. 设置客户端授权密码
5. 填到 config.json

### iCloud 邮箱

1. 打开 [appleid.apple.com](https://appleid.apple.com)
2. 登录 → 登录和安全 → App 专用密码
3. 点加号生成，随便起个名字
4. 拿到 xxxx-xxxx-xxxx-xxxx 格式的密码，填到 config.json

### Gmail

1. 打开 [Google 账号](https://myaccount.google.com)
2. 安全性 → 两步验证（必须先开启）
3. 两步验证页面最下方 → App 专用密码
4. 选择「邮件」，生成 16 位密码，填到 config.json

### Outlook / Hotmail

1. 打开 [Microsoft 账户安全](https://account.microsoft.com/security)
2. 高级安全选项 → App 密码
3. 创建新的 App 密码，填到 config.json

> ⚠️ 以上填的都是**授权码 / App 专用密码**，不是你的登录密码。授权码可以随时撤销，不影响正常登录。

## 云部署（Zeabur / Railway / Render）

如果你没有自己的服务器，可以部署到云平台。MCP 版特别适合这种方式。

### Zeabur 部署

1. Fork 本项目到你的 GitHub
2. Zeabur 新建项目 → 从 GitHub 导入
3. **重命名入口文件**：Zeabur 默认找 `main.py`，把 `mcp_server.py` 重命名为 `main.py`，或在 Zeabur 设置里指定启动命令：`python mcp_server.py`
4. 在 Variables 面板添加环境变量：
   - `MAIL_ADDRESS`：你的邮箱地址
   - `MAIL_PASSWORD`：授权码
   - `IMAP_HOST`：如 `imap.qq.com`
   - `IMAP_PORT`：`993`
   - `SMTP_HOST`：如 `smtp.qq.com`
   - `SMTP_PORT`：`587`
   - `DISPLAY_NAME`：AI 的显示名
   - `PORT`：`8080`（Zeabur 默认）
5. 部署完成后访问 `https://你的域名/sse`，看到 `event: endpoint` 就说明通了
6. 去 claude.ai → 设置 → MCP → 添加连接，填 `https://你的域名/sse`

### Railway / Render

同样的流程，设置环境变量 + 启动命令 `python mcp_server.py`。

### 常见问题

**421 Misdirected Request**：旧版代码 `host` 默认是 `127.0.0.1`，云平台反向代理过来的请求会被 MCP SDK 的 DNS 保护拦截。最新代码已修复（`host="0.0.0.0"`）。

**Not Found on /mcp**：最新代码默认使用 SSE 模式，端点在 `/sse`。如需 streamable-http 模式（端点 `/mcp`），设置环境变量 `MCP_TRANSPORT=streamable-http`。

**自定义域名**：在云平台 Networking 里加 Custom Domain，DNS 添加对应记录（A 或 CNAME，看平台指示）。

## 安全建议

- **设置白名单**：`allowed_senders` 填上信任的邮箱地址
- **不要提交 config.json**：里面有密码和 API key（已在 .gitignore 中排除）
- **用授权码**：万一泄露可以单独撤销，不影响邮箱登录

## License

MIT
