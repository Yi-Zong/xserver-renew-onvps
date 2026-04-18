# Cron setup

This document records the cron style used on the source VPS and shows how to deploy it safely without exposing secrets.

## Public-safe example crontab

Replace all placeholder values before use.

```cron
# XServer renewal account 1
0 8 * * * cd /opt/xserver-renew-onvps/xserver && TG_BOT_TOKEN='YOUR_TG_BOT_TOKEN' TG_CHAT_ID='YOUR_CHAT_ID' EMAIL='your1@example.com' PASSWORD='your-password-1' ACCOUNT_LABEL='account-1' /usr/bin/python3 /opt/xserver-renew-onvps/run_xserver_notify.py >> /opt/xserver-renew-onvps/logs/xserver_account1.log 2>&1

# XServer renewal account 2
0 9 * * * cd /opt/xserver-renew-onvps/xserver && TG_BOT_TOKEN='YOUR_TG_BOT_TOKEN' TG_CHAT_ID='YOUR_CHAT_ID' EMAIL='your2@example.com' PASSWORD='your-password-2' ACCOUNT_LABEL='account-2' /usr/bin/python3 /opt/xserver-renew-onvps/run_xserver_notify.py >> /opt/xserver-renew-onvps/logs/xserver_account2.log 2>&1
```

## How to install cron

Edit crontab:

```bash
crontab -e
```

List current cron jobs:

```bash
crontab -l
```

## Recommendation

For safer operations, avoid hardcoding secrets directly in crontab. Prefer one of these approaches:

1. Source an env file in the cron command
2. Use a wrapper shell script with restricted permissions
3. Use a secret manager if available

Example with env file:

```cron
0 8 * * * . /opt/xserver-renew-onvps/.env.runtime && cd /opt/xserver-renew-onvps/xserver && /usr/bin/python3 /opt/xserver-renew-onvps/run_xserver_notify.py >> /opt/xserver-renew-onvps/logs/xserver_account1.log 2>&1
```

Example `.env.runtime`:

```env
TG_BOT_TOKEN=your_telegram_bot_token
TG_CHAT_ID=your_telegram_chat_id
EMAIL=your_xserver_email
PASSWORD=your_xserver_password
ACCOUNT_LABEL=account-1
```
