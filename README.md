# xserver-renew-onvps

这是一个 **XServer 免费 VPS 自动续期项目** 的公开备份仓库，已经做过公开发布安全清理。

我把原来散落在 VPS 上的核心项目、通知脚本、cron 思路和部署方法整理成了一个更适合长期保存和迁移的版本。

---

## 这个仓库能做什么？

这个项目主要用于：

- 自动登录 XServer 面板
- 检查免费 VPS 是否到了可续期时间
- 在可以续期时自动提交续期
- 运行后输出结果
- 可选通过 Telegram 推送执行结果
- 可配合 cron 定时自动执行

---

## 仓库内容结构说明

```text
xserver-renew-onvps/
├── README.md                     # 项目总说明（你现在看的这个）
├── package.json                  # Node 相关依赖说明（辅助脚本用）
├── run_xserver_notify.py         # 续期执行 + Telegram 通知包装脚本
├── docs/
│   ├── deployment.md             # 详细部署教程
│   └── cron.md                   # cron 配置说明
└── xserver/
    ├── .env.example              # 环境变量模板（示例）
    ├── main.py                   # XServer 自动续期主脚本
    ├── requirements.txt          # Python 依赖
    ├── Dockerfile                # Docker 部署文件
    ├── entrypoint.sh             # Docker 入口脚本
    └── run-docker.sh             # Docker 启动辅助脚本
```

---

## 仓库里故意没有放什么？

为了能安全公开，这个仓库 **不会包含**：

- 真实 XServer 邮箱
- 真实 XServer 密码
- 真实 Telegram Bot Token
- 真实 Telegram Chat ID
- 真实 Cookie
- 本机 `.env`
- 运行日志
- 截图文件
- 本地虚拟环境
- 调试缓存

也就是说：

**你 clone 下来后，需要自己填写变量，才能真正运行。**

---

## 你需要自己填写的变量

### 1）XServer 主脚本变量

先复制模板文件：

```bash
cp xserver/.env.example xserver/.env
```

然后编辑：

```env
EMAIL=你的 XServer 账号邮箱
PASSWORD=你的 XServer 密码
PROXY_SERVER=
DEBUG=false
```

说明：

- `EMAIL`：你的 XServer 登录邮箱
- `PASSWORD`：你的 XServer 登录密码
- `PROXY_SERVER`：可选，不需要可以留空
- `DEBUG`：调试模式，默认 `false`

---

### 2）Telegram 通知脚本变量

如果你想在执行后收到 Telegram 通知，还需要准备：

```env
TG_BOT_TOKEN=你的 Telegram Bot Token
TG_CHAT_ID=你的 Telegram Chat ID
EMAIL=你的 XServer 账号邮箱
PASSWORD=你的 XServer 密码
ACCOUNT_LABEL=你给这个账号起的备注名
```

可选变量：

```env
XSERVER_IMAGE_UPLOAD_URL=你的图床上传接口
XSERVER_IMAGE_UPLOAD_REFERER=你的图床站点地址
XSERVER_IMAGE_STRATEGY_ID=
```

如果你没有图床，这几个可选变量可以先不填。

---

## 最简单的使用方式

### 方式一：先手动测试

先安装依赖，然后手动跑一次，确认脚本在你机器上能工作。

详细步骤见：

- [docs/deployment.md](docs/deployment.md)

---

### 方式二：配合 cron 自动执行

适合放在 VPS 上定时运行。

详细说明见：

- [docs/cron.md](docs/cron.md)

---

## 一键部署（适合新机器直接上）

如果你的服务器是 Debian / Ubuntu，可以按下面这一套直接走。

### 1. 安装基础依赖

```bash
apt update && apt upgrade -y
apt install -y git python3 python3-venv python3-pip xvfb curl unzip
```

### 2. 拉取仓库

```bash
git clone https://github.com/Yi-Zong/xserver-renew-onvps.git
cd xserver-renew-onvps
```

### 3. 创建配置文件

```bash
cp xserver/.env.example xserver/.env
nano xserver/.env
```

填入你自己的：

```env
EMAIL=你的邮箱
PASSWORD=你的密码
PROXY_SERVER=
DEBUG=false
```

### 4. 安装 Python 依赖

```bash
cd xserver
python3 -m venv .venv
. .venv/bin/activate
pip install --upgrade pip
pip install -r requirements.txt
python -m playwright install
cd ..
```

### 5. 建立日志目录

```bash
mkdir -p logs
```

### 6. 手动测试一次

```bash
cd xserver
. .venv/bin/activate
python main.py
```

### 7. 如果要启用 Telegram 通知，手动测试包装脚本

```bash
cd ..
EMAIL='你的邮箱' PASSWORD='你的密码' ACCOUNT_LABEL='account-1' TG_BOT_TOKEN='你的bot token' TG_CHAT_ID='你的chat id' python3 run_xserver_notify.py
```

### 8. 设置 cron 定时任务

```bash
crontab -e
```

加入类似下面的内容：

```cron
0 8 * * * cd /你的路径/xserver-renew-onvps/xserver && TG_BOT_TOKEN='你的TG_BOT_TOKEN' TG_CHAT_ID='你的CHAT_ID' EMAIL='你的邮箱' PASSWORD='你的密码' ACCOUNT_LABEL='account-1' /usr/bin/python3 /你的路径/xserver-renew-onvps/run_xserver_notify.py >> /你的路径/xserver-renew-onvps/logs/xserver_account1.log 2>&1
```

---

## 推荐部署方式

我个人更推荐你这样做：

1. **仓库公开，但 secrets 永远不进仓库**
2. 本地只保留：
   - `.env.example`
   - 文档
   - 代码
3. 真正运行时：
   - 用 `.env`
   - 或者单独 `.env.runtime`
   - 或者 cron 环境变量

这样迁移最方便，也最安全。

---

## 常见问题

### 1）为什么仓库里不能直接放密码？

因为这是公开仓库，一旦提交到 GitHub，密码/token 就可能被搜索、缓存、泄露。

### 2）为什么我手动运行可以，cron 不行？

因为 cron 的环境变量更少，路径也更严格。通常要：

- 写完整路径
- 显式传变量
- 检查日志输出

### 3）为什么脚本突然失效？

因为这类自动化依赖网页结构，如果 XServer 改了：

- 登录页结构
- 按钮文字
- CAPTCHA
- Turnstile 验证逻辑

脚本就可能需要更新。

---

## 文档导航

- 详细部署教程：[`docs/deployment.md`](docs/deployment.md)
- cron 配置说明：[`docs/cron.md`](docs/cron.md)

---

## 安全提醒

在你自己的机器上部署前，请确认：

- 不要把 `.env` 提交到 Git
- 不要把 bot token 写死到源码里
- 不要把真实密码写进公开 README
- 推送前检查 `git diff`

如果你是从旧机器迁移过来的，建议顺手更换一次：

- GitHub Token
- Telegram Bot Token
- XServer 密码（如果曾在脚本里明文出现过）

---

## 项目状态

这是一个 **可备份、可迁移、可二次整理** 的版本。

如果后面还要继续完善，可以继续加：

- `env.runtime.example`
- `install.sh`
- `LICENSE`
- `SECURITY.md`
- GitHub Actions 自检
- 更细的多账号部署说明
