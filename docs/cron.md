# cron 配置说明

这个文档专门讲：**怎么把 XServer 自动续期项目挂到 cron 上稳定运行。**

---

## 一、最简单可用版

如果你只想先跑起来，可以直接在 crontab 里写完整命令。

编辑 crontab：

```bash
crontab -e
```

加入示例：

```cron
# XServer 自动续期示例
0 8 * * * cd /opt/xserver-renew-onvps/xserver && TG_BOT_TOKEN='你的TG_BOT_TOKEN' TG_CHAT_ID='你的CHAT_ID' EMAIL='your@example.com' PASSWORD='your-password' ACCOUNT_LABEL='account-1' /usr/bin/python3 /opt/xserver-renew-onvps/run_xserver_notify.py >> /opt/xserver-renew-onvps/logs/xserver_account1.log 2>&1
```

---

## 二、这条 cron 每一部分是什么意思？

以这条命令为例：

```cron
0 8 * * * cd /opt/xserver-renew-onvps/xserver && TG_BOT_TOKEN='你的TG_BOT_TOKEN' TG_CHAT_ID='你的CHAT_ID' EMAIL='your@example.com' PASSWORD='your-password' ACCOUNT_LABEL='account-1' /usr/bin/python3 /opt/xserver-renew-onvps/run_xserver_notify.py >> /opt/xserver-renew-onvps/logs/xserver_account1.log 2>&1
```

拆开解释：

- `0 8 * * *`：每天早上 8 点执行
- `cd /opt/xserver-renew-onvps/xserver`：先切换到项目目录
- `TG_BOT_TOKEN=...`：注入 Telegram bot token
- `TG_CHAT_ID=...`：注入 Telegram chat id
- `EMAIL=...`：XServer 账号邮箱
- `PASSWORD=...`：XServer 账号密码
- `ACCOUNT_LABEL=...`：这个账号的备注名
- `/usr/bin/python3 /opt/xserver-renew-onvps/run_xserver_notify.py`：执行包装脚本
- `>> ...log 2>&1`：把标准输出和错误输出都写进日志

---

## 三、推荐版：不要把密码直接写在 crontab 里

虽然把变量直接写进 crontab 最快，但长期看不够优雅。

更推荐：**把变量放到单独的 env 文件里。**

### 第一步：创建 env 文件

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

### 第二步：改 cron 为 env 文件模式

```cron
0 8 * * * . /opt/xserver-renew-onvps/.env.runtime && cd /opt/xserver-renew-onvps/xserver && /usr/bin/python3 /opt/xserver-renew-onvps/run_xserver_notify.py >> /opt/xserver-renew-onvps/logs/xserver_account1.log 2>&1
```

这样以后你只需要维护一个文件，不用每次改 crontab。

---

## 四、多账号怎么写？

如果你有多个 XServer 账号，建议：

- 每个账号分开一条 cron
- 每个账号分开一个日志文件
- 每个账号用不同的 `ACCOUNT_LABEL`

示例：

```cron
# account 1
0 8 * * * . /opt/xserver-renew-onvps/.env.runtime.account1 && cd /opt/xserver-renew-onvps/xserver && /usr/bin/python3 /opt/xserver-renew-onvps/run_xserver_notify.py >> /opt/xserver-renew-onvps/logs/xserver_account1.log 2>&1

# account 2
30 8 * * * . /opt/xserver-renew-onvps/.env.runtime.account2 && cd /opt/xserver-renew-onvps/xserver && /usr/bin/python3 /opt/xserver-renew-onvps/run_xserver_notify.py >> /opt/xserver-renew-onvps/logs/xserver_account2.log 2>&1
```

示例 env 文件：

### `/opt/xserver-renew-onvps/.env.runtime.account1`

```env
TG_BOT_TOKEN=你的 Telegram Bot Token
TG_CHAT_ID=你的 Telegram Chat ID
EMAIL=account1@example.com
PASSWORD=account1-password
ACCOUNT_LABEL=account-1
```

### `/opt/xserver-renew-onvps/.env.runtime.account2`

```env
TG_BOT_TOKEN=你的 Telegram Bot Token
TG_CHAT_ID=你的 Telegram Chat ID
EMAIL=account2@example.com
PASSWORD=account2-password
ACCOUNT_LABEL=account-2
```

---

## 五、如何检查 cron 是否生效

### 1. 查看当前 cron

```bash
crontab -l
```

### 2. 查看项目日志

```bash
tail -f /opt/xserver-renew-onvps/logs/xserver_account1.log
```

### 3. 看 Telegram 是否收到消息

如果你启用了通知，cron 成功执行后通常会有推送。

---

## 六、推荐执行时间

你可以按自己的节奏安排，比如：

- 每天 1 次
- 每天 2 次
- 多账号错峰执行

建议：

- 不要多个账号同一秒一起跑
- 错开 10~30 分钟更稳妥

例如：

- 账号 1：08:00
- 账号 2：08:30
- 账号 3：09:00

---

## 七、常见问题

### 1）cron 里找不到命令
因为 cron 环境很干净，建议：

- 用完整路径，比如 `/usr/bin/python3`
- 用完整项目路径
- 不要依赖当前 shell 配置

### 2）手动运行正常，cron 不执行
检查：

- crontab 是否真的保存了
- 文件路径是否正确
- env 文件是否存在
- 日志目录是否存在
- 输出是否写进 log

### 3）日志里什么都没有
先确认日志目录是否存在：

```bash
mkdir -p /opt/xserver-renew-onvps/logs
```

---

## 八、一键加 cron 的思路

如果你习惯手动编辑，可以直接 `crontab -e`。

如果你喜欢命令式写法，也可以：

```bash
( crontab -l 2>/dev/null; echo "0 8 * * * . /opt/xserver-renew-onvps/.env.runtime && cd /opt/xserver-renew-onvps/xserver && /usr/bin/python3 /opt/xserver-renew-onvps/run_xserver_notify.py >> /opt/xserver-renew-onvps/logs/xserver_account1.log 2>&1" ) | crontab -
```

这条命令会：

- 保留原来的 cron
- 追加一条新的任务

---

## 九、安全建议

- 不要把真实密码提交到 GitHub
- 不要把 `.env.runtime` 提交到 GitHub
- 不要把 bot token 写死到源码里
- 如果变量曾经泄露，建议立刻更换
