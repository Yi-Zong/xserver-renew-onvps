# xserver-renew-onvps

一个用于 **XServer 免费 VPS 自动续期** 的公开仓库，包含主脚本、Telegram 通知包装脚本、部署文档和 cron 示例。

这个仓库已经做过公开发布所需的基础安全清理：**不包含真实账号、密码、Bot Token、Chat ID、日志、截图或本地环境文件**。

---

## 功能

- 自动登录 XServer 面板
- 检查当前账号是否进入可续期窗口
- 在允许续期时自动提交续期
- 保存运行输出与日志
- 通过 Telegram 推送执行结果
- 通过 cron 定时自动运行

---

## 仓库结构

```text
xserver-renew-onvps/
├── README.md
├── package.json
├── env.runtime.example
├── run_xserver_notify.py
├── docs/
│   ├── cron.md
│   └── deployment.md
└── xserver/
    ├── .env.example
    ├── main.py
    ├── requirements.txt
    ├── Dockerfile
    ├── entrypoint.sh
    └── run-docker.sh
```

---

## 仓库中不会包含的内容

出于安全原因，以下内容不会进入仓库：

- 真实 XServer 邮箱
- 真实 XServer 密码
- 真实 Telegram Bot Token
- 真实 Telegram Chat ID
- 本地 `.env`
- 运行日志
- 截图文件
- 虚拟环境目录
- 调试缓存

因此，仓库 clone 下来后，需要你自己补齐运行时变量。

---

## 配置说明

### 1. 配置 XServer 主脚本

先复制环境变量模板：

```bash
cp xserver/.env.example xserver/.env
```

然后填写：

```env
EMAIL=your-xserver-email@example.com
PASSWORD=your-xserver-password
PROXY_SERVER=
DEBUG=false
```

说明：

- `EMAIL`：XServer 登录邮箱
- `PASSWORD`：XServer 登录密码
- `PROXY_SERVER`：可选代理
- `DEBUG`：是否启用调试模式

### 2. 配置 Telegram 通知脚本

如果你想收到执行通知，还需要这些变量：

```env
TG_BOT_TOKEN=your-telegram-bot-token
TG_CHAT_ID=your-telegram-chat-id
EMAIL=your-xserver-email@example.com
PASSWORD=your-xserver-password
ACCOUNT_LABEL=account-1
```

可选图床变量：

```env
XSERVER_IMAGE_UPLOAD_URL=https://example.com/api/upload
XSERVER_IMAGE_UPLOAD_REFERER=https://example.com
XSERVER_IMAGE_STRATEGY_ID=
```

如果不需要图床回退，可以先不填这些可选项。

---

## 快速开始

### 安装基础依赖

适用于 Debian / Ubuntu：

```bash
apt update && apt upgrade -y
apt install -y git python3 python3-venv python3-pip xvfb curl unzip
```

### 克隆仓库

```bash
git clone https://github.com/Yi-Zong/xserver-renew-onvps.git
cd xserver-renew-onvps
```

### 初始化 Python 环境

```bash
cd xserver
python3 -m venv .venv
. .venv/bin/activate
pip install --upgrade pip
pip install -r requirements.txt
python -m playwright install
cd ..
```

### 创建日志目录

```bash
mkdir -p logs
```

---

## 使用方法

### 方式一：手动执行主脚本

```bash
cd xserver
. .venv/bin/activate
python main.py
```

### 方式二：执行带 Telegram 通知的包装脚本

```bash
EMAIL='your-email' \
PASSWORD='your-password' \
ACCOUNT_LABEL='account-1' \
TG_BOT_TOKEN='your-bot-token' \
TG_CHAT_ID='your-chat-id' \
python3 run_xserver_notify.py
```

---

## cron 示例

```cron
0 8 * * * cd /your/path/xserver-renew-onvps/xserver && TG_BOT_TOKEN='your-bot-token' TG_CHAT_ID='your-chat-id' EMAIL='your-email' PASSWORD='your-password' ACCOUNT_LABEL='account-1' /usr/bin/python3 /your/path/xserver-renew-onvps/run_xserver_notify.py >> /your/path/xserver-renew-onvps/logs/xserver_account1.log 2>&1
```

更多说明见：

- [docs/deployment.md](docs/deployment.md)
- [docs/cron.md](docs/cron.md)

---

## 常见问题

### 为什么手动运行正常，cron 却失败？

通常是因为 cron 环境更干净，容易缺：

- 完整路径
- 环境变量
- 虚拟环境激活
- 日志输出重定向

### 为什么脚本会突然失效？

因为这类自动化高度依赖网页结构。如果 XServer 修改了：

- 登录页结构
- 按钮文字
- CAPTCHA 流程
- Turnstile 验证逻辑

脚本就需要同步调整。

### 为什么不能把密码或 Token 直接放进仓库？

因为这是公开仓库。一旦提交到 GitHub，敏感信息就可能被索引、缓存或泄露。

---

## 建议

推荐做法是：

1. **公开仓库只放代码和文档**
2. **所有 secrets 都放在运行环境中**
3. 使用 `.env`、cron 变量或 CI secrets 管理敏感信息
4. 定期检查日志与截图，确认续期链路仍然可用
