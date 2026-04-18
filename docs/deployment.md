# 详细部署教程

这份文档按“**尽量简单、尽量少踩坑**”来写，适合把项目部署到一台新的 Linux VPS 上。

默认假设：

- 系统是 **Debian / Ubuntu**
- 你有 root 权限
- 你要部署的是这个仓库：
  `https://github.com/Yi-Zong/xserver-renew-onvps`

---

## 一、先理解这套项目怎么跑

这套项目主要分成两部分：

### 1. `xserver/main.py`
核心自动续期脚本。

负责：
- 打开 XServer 页面
- 登录账号
- 进入续期页面
- 检查是否到了可续期时间
- 能续期时提交操作

### 2. `run_xserver_notify.py`
包装脚本。

负责：
- 调用主脚本执行
- 记录日志
- 读取执行结果
- 可选发 Telegram 通知

你平时真正定时跑的，通常是这个：

```bash
python3 run_xserver_notify.py
```

---

## 二、准备服务器

先更新系统：

```bash
apt update && apt upgrade -y
```

安装基础工具：

```bash
apt install -y git python3 python3-venv python3-pip xvfb curl unzip
```

说明：

- `git`：拉代码
- `python3`：运行 Python
- `python3-venv`：创建虚拟环境
- `python3-pip`：安装依赖
- `xvfb`：无头图形环境，适合 VPS 跑浏览器脚本
- `curl` / `unzip`：常见辅助工具

---

## 三、下载项目

建议放到固定目录，例如 `/opt`。

```bash
cd /opt
git clone https://github.com/Yi-Zong/xserver-renew-onvps.git
cd xserver-renew-onvps
```

如果你不想放 `/opt`，也可以换成自己的目录，比如 `/root`。

---

## 四、配置 XServer 登录信息

先复制模板：

```bash
cp xserver/.env.example xserver/.env
```

打开编辑：

```bash
nano xserver/.env
```

填入：

```env
EMAIL=你的 XServer 邮箱
PASSWORD=你的 XServer 密码
PROXY_SERVER=
DEBUG=false
```

### 字段解释

#### `EMAIL`
XServer 登录邮箱。

#### `PASSWORD`
XServer 登录密码。

#### `PROXY_SERVER`
代理地址。没有代理就留空。

例如：

```env
PROXY_SERVER=http://user:password@host:port
```

#### `DEBUG`
是否调试模式。

- `false`：正常运行
- `true`：更适合调试问题

一般部署时用：

```env
DEBUG=false
```

---

## 五、安装 Python 依赖

进入项目目录：

```bash
cd /opt/xserver-renew-onvps/xserver
```

创建虚拟环境：

```bash
python3 -m venv .venv
```

激活虚拟环境：

```bash
. .venv/bin/activate
```

升级 pip：

```bash
pip install --upgrade pip
```

安装依赖：

```bash
pip install -r requirements.txt
```

安装 Playwright 浏览器依赖：

```bash
python -m playwright install
```

如果项目后续提示缺少浏览器系统库，再按报错补装。

---

## 六、先手动跑一次主脚本

这一步很重要，先确认环境没问题。

```bash
cd /opt/xserver-renew-onvps/xserver
. .venv/bin/activate
python main.py
```

### 如果运行正常
说明：
- Python 依赖没问题
- 浏览器环境基本可用
- 配置文件生效了

### 如果运行失败
重点检查：

1. `xserver/.env` 是否写对
2. 依赖是否装完整
3. 浏览器相关依赖是否缺失
4. XServer 页面结构是否变化

---

## 七、准备日志目录

返回仓库根目录：

```bash
cd /opt/xserver-renew-onvps
mkdir -p logs
```

后面 cron 跑出来的日志就放这里。

---

## 八、测试通知脚本（可选，但推荐）

如果你想在每次执行后收到 Telegram 消息，就测试这个脚本。

示例：

```bash
cd /opt/xserver-renew-onvps
EMAIL='你的邮箱' \
PASSWORD='你的密码' \
ACCOUNT_LABEL='account-1' \
TG_BOT_TOKEN='你的 Telegram Bot Token' \
TG_CHAT_ID='你的 Telegram Chat ID' \
python3 run_xserver_notify.py
```

### 这些变量分别是什么

- `EMAIL`：XServer 账号邮箱
- `PASSWORD`：XServer 密码
- `ACCOUNT_LABEL`：你给这个账号起的备注，比如 `main-account`
- `TG_BOT_TOKEN`：Telegram 机器人 token
- `TG_CHAT_ID`：你要接收消息的 chat id

如果你不需要 Telegram 通知，也可以只跑主脚本，不用这个包装脚本。

---

## 九、设置 cron 自动执行

打开 crontab：

```bash
crontab -e
```

加入一条示例任务：

```cron
0 8 * * * cd /opt/xserver-renew-onvps/xserver && TG_BOT_TOKEN='你的TG_BOT_TOKEN' TG_CHAT_ID='你的CHAT_ID' EMAIL='你的邮箱' PASSWORD='你的密码' ACCOUNT_LABEL='account-1' /usr/bin/python3 /opt/xserver-renew-onvps/run_xserver_notify.py >> /opt/xserver-renew-onvps/logs/xserver_account1.log 2>&1
```

### 这条 cron 的意思

- 每天早上 `8:00` 执行
- 进入 `xserver` 目录
- 带上环境变量
- 执行 `run_xserver_notify.py`
- 把输出追加到日志文件

---

## 十、如何查看是否真的生效

查看当前 cron：

```bash
crontab -l
```

查看日志：

```bash
tail -f /opt/xserver-renew-onvps/logs/xserver_account1.log
```

如果你开了 Telegram 通知，也可以直接看机器人消息是否收到。

---

## 十一、更推荐的做法：把 secrets 放到单独 env 文件

直接把密码写进 crontab 虽然能用，但不够优雅。

更推荐这样：

### 1. 创建运行时变量文件

```bash
nano /opt/xserver-renew-onvps/.env.runtime
```

写入：

```env
TG_BOT_TOKEN=你的 Telegram Bot Token
TG_CHAT_ID=你的 Telegram Chat ID
EMAIL=你的 XServer 邮箱
PASSWORD=你的 XServer 密码
ACCOUNT_LABEL=account-1
```

### 2. 改成这样跑

```cron
0 8 * * * . /opt/xserver-renew-onvps/.env.runtime && cd /opt/xserver-renew-onvps/xserver && /usr/bin/python3 /opt/xserver-renew-onvps/run_xserver_notify.py >> /opt/xserver-renew-onvps/logs/xserver_account1.log 2>&1
```

这样以后改变量更方便，也更清晰。

---

## 十二、一键部署方式

如果你想尽量少手动敲命令，可以直接按下面这一套执行。

> 注意：里面仍然需要你自己手动填写 `.env` 和运行时变量。

```bash
apt update && apt upgrade -y && \
apt install -y git python3 python3-venv python3-pip xvfb curl unzip && \
cd /opt && \
git clone https://github.com/Yi-Zong/xserver-renew-onvps.git && \
cd xserver-renew-onvps && \
cp xserver/.env.example xserver/.env && \
mkdir -p logs && \
cd xserver && \
python3 -m venv .venv && \
. .venv/bin/activate && \
pip install --upgrade pip && \
pip install -r requirements.txt && \
python -m playwright install
```

跑完后你还需要做两件事：

1. 编辑 `xserver/.env`
2. 配置 cron

---

## 十三、排错建议

### 问题 1：手动能跑，cron 不行
通常原因：

- cron 环境变量不完整
- 路径没写全
- 虚拟环境没激活
- 当前目录不对

优先检查日志。

---

### 问题 2：浏览器依赖报错
通常是系统缺少图形/浏览器依赖库。

先执行：

```bash
python -m playwright install
```

如果还有报错，就按提示补系统包。

---

### 问题 3：XServer 页面改版导致失效
这类自动化依赖网页结构。

如果 XServer 改了：

- 登录表单
- 按钮文字
- 验证流程
- Turnstile / CAPTCHA

脚本就可能需要重新适配。

---

### 问题 4：Telegram 没收到通知
检查：

- `TG_BOT_TOKEN` 对不对
- `TG_CHAT_ID` 对不对
- VPS 网络能不能访问 Telegram API
- Bot 是否能给目标会话发消息

---

## 十四、部署完成后的建议

部署完成后，我建议你做这几件事：

1. 手动执行一次主脚本
2. 手动执行一次通知脚本
3. 检查 `logs/` 输出
4. 检查 cron 是否写对
5. 不要把 `.env` 提交回 GitHub

---

## 十五、最后的安全提醒

如果你曾经在任何地方明文暴露过这些内容，建议及时更换：

- GitHub Token
- Telegram Bot Token
- XServer 密码

因为这类信息一旦出现在聊天记录、脚本历史或公开仓库里，就不再适合长期继续使用。
