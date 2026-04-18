# Deployment tutorial

This guide explains how to deploy the XServer auto-renew project from scratch on a Linux VPS.

## 1. Prepare the server

Update packages:

```bash
apt update && apt upgrade -y
```

Install basic tools:

```bash
apt install -y python3 python3-venv python3-pip xvfb curl unzip
```

Depending on the browser stack you use, you may also need additional system libraries required by Playwright / Camoufox.

## 2. Get the project

Clone or download the repository:

```bash
git clone https://github.com/Yi-Zong/xserver-renew-onvps.git
cd xserver-renew-onvps
```

## 3. Configure secrets

Copy the env template:

```bash
cp xserver/.env.example xserver/.env
```

Edit it:

```bash
nano xserver/.env
```

Fill in:

```env
EMAIL=your_xserver_email
PASSWORD=your_xserver_password
PROXY_SERVER=
DEBUG=false
```

If you use Telegram notifications, also prepare runtime variables for `run_xserver_notify.py`:

```env
TG_BOT_TOKEN=your_telegram_bot_token
TG_CHAT_ID=your_telegram_chat_id
ACCOUNT_LABEL=your_account_name
```

## 4. Install Python dependencies

```bash
cd xserver
python3 -m venv .venv
. .venv/bin/activate
pip install --upgrade pip
pip install -r requirements.txt
playwright install
```

If `playwright` command is missing, try:

```bash
python -m playwright install
```

## 5. Manual test

Run the core script first:

```bash
cd xserver
. .venv/bin/activate
python main.py
```

If you want to test the notification wrapper:

```bash
cd /opt/xserver-renew-onvps
EMAIL='your@email' PASSWORD='yourpass' ACCOUNT_LABEL='main-account' TG_BOT_TOKEN='xxx' TG_CHAT_ID='xxx' python3 run_xserver_notify.py
```

## 6. Create log directory

```bash
mkdir -p /opt/xserver-renew-onvps/logs
```

## 7. Install cron job

Open crontab:

```bash
crontab -e
```

Add a job similar to:

```cron
0 8 * * * cd /opt/xserver-renew-onvps/xserver && TG_BOT_TOKEN='YOUR_TG_BOT_TOKEN' TG_CHAT_ID='YOUR_CHAT_ID' EMAIL='your@example.com' PASSWORD='your-password' ACCOUNT_LABEL='account-1' /usr/bin/python3 /opt/xserver-renew-onvps/run_xserver_notify.py >> /opt/xserver-renew-onvps/logs/xserver_account1.log 2>&1
```

## 8. Verify cron

```bash
crontab -l
```

Check logs after execution:

```bash
tail -f /opt/xserver-renew-onvps/logs/xserver_account1.log
```

## 9. Security checklist

Before publishing or sharing:

- Remove all real tokens
- Remove all real passwords
- Remove `.env`
- Remove screenshots and logs
- Remove cookies
- Review helper scripts for hardcoded credentials

## 10. Common problems

### CAPTCHA or Turnstile changed
The automation may fail if XServer changes their login or verification flow.

### Browser dependencies missing
Install the missing libraries requested by Playwright/Camoufox.

### Cron works manually but not automatically
Cron has a smaller environment. Always use full paths and explicitly set env vars.

### Telegram notification fails
Double-check `TG_BOT_TOKEN`, `TG_CHAT_ID`, network access, and firewall rules.
