# claude-mail-bridge 📬

给你的 AI 一个邮箱。

最轻量的方案——一个 Python 脚本，让你通过中转站 API 接入的 AI 能收发邮件。不需要 MCP，不需要框架，不需要服务器架构知识。

## 工作原理

```
有人发邮件 → 脚本检查收件箱 → 调你的 API → 把回复发回去
```

就这么简单。

## 快速开始

### 1. 准备一个邮箱

任何支持 IMAP/SMTP 的邮箱都行：

| 邮箱 | IMAP 地址 | SMTP 地址 | 备注 |
|------|-----------|-----------|------|
| QQ 邮箱 | imap.qq.com:993 | smtp.qq.com:587 | 设置→账户→开启IMAP，生成授权码 |
| 163 邮箱 | imap.163.com:993 | smtp.163.com:465 | 设置→POP3/SMTP/IMAP→开启IMAP |
| iCloud | imap.mail.me.com:993 | smtp.mail.me.com:587 | 需要 App 专用密码 |
| Outlook | outlook.office365.com:993 | smtp.office365.com:587 | 账户安全→App 密码 |
| Gmail | imap.gmail.com:993 | smtp.gmail.com:587 | 需要 App 专用密码 |

### 2. 安装依赖

```bash
pip install requests
```

Python 3.8+ 即可，其余都是标准库。

### 3. 配置

```bash
cp config.example.json config.json
```

编辑 `config.json`：

```json
{
  "email": {
    "address": "你的邮箱地址",
    "password": "你的授权码（不是登录密码）",
    "imap_host": "imap.qq.com",
    "imap_port": 993,
    "smtp_host": "smtp.qq.com",
    "smtp_port": 587,
    "display_name": "我的小克"
  },
  "api": {
    "base_url": "https://你的中转站地址/v1",
    "api_key": "sk-你的key",
    "model": "claude-sonnet-4-20250514",
    "system_prompt": "你是一个温柔的AI助手，名字叫小克。有人通过邮件联系你，请友善地回复。"
  },
  "bridge": {
    "poll_interval": 10,
    "max_body_chars": 3000,
    "allowed_senders": [],
    "log_file": "bridge.log"
  }
}
```

**配置说明：**

- `password`：是邮箱的**授权码 / App 专用密码**，不是你的登录密码
- `base_url`：你的中转站 API 地址，兼容 OpenAI 格式就行
- `allowed_senders`：白名单，填邮箱地址。留空 `[]` 表示所有人都能发
- `poll_interval`：检查新邮件的间隔（秒）

### 4. 运行

```bash
python bridge.py
```

后台运行（推荐）：

```bash
# 用 nohup
nohup python bridge.py &

# 或用 PM2（如果装了的话）
pm2 start bridge.py --name mail-bridge --interpreter python3

# 或用 screen
screen -S mail-bridge
python bridge.py
# Ctrl+A, D 退出 screen
```

## 常见邮箱配置

### QQ 邮箱

1. 打开 QQ 邮箱网页版
2. 设置 → 账户 → POP3/IMAP/SMTP/Exchange/CardDAV/CalDAV 服务
3. 开启 IMAP/SMTP 服务
4. 按提示发送短信验证
5. 获得授权码（16 位字母），填到 config.json 的 password 里

### 163 邮箱

1. 打开 163 邮箱网页版
2. 设置 → POP3/SMTP/IMAP
3. 开启 IMAP/SMTP 服务
4. 设置客户端授权密码
5. 填到 config.json

### iCloud 邮箱

1. 打开 appleid.apple.com
2. 登录 → App 专用密码 → 生成
3. 填到 config.json

## 安全建议

- **务必设置白名单**：`allowed_senders` 填上你和朋友的邮箱，不然谁都能让你的 AI 回复
- **不要把 config.json 提交到 git**：里面有密码和 API key
- **用授权码不用登录密码**：万一泄露了可以单独撤销

## 进阶用法

### 让 AI 主动发邮件

在 system_prompt 里加上指令，比如：

```
如果用户说"帮我给 xxx@qq.com 发一封信"，请回复格式：
[SEND_TO: xxx@qq.com]
[SUBJECT: 邮件主题]
邮件正文...
```

然后在 bridge.py 里加一段解析逻辑，检测到这个格式就调 send_reply。

### 多轮对话

当前版本是单轮的（每封邮件独立回复）。如果想要多轮对话记忆，可以：

1. 用 SQLite 存每个发件人的对话历史
2. 调 API 时把历史消息一起传过去

这个功能留给你自己扩展 :)

## License

MIT
