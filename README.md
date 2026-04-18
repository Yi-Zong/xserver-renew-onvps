# xserver-renew-onvps

A public backup of an XServer free VPS auto-renew project, cleaned for safe publishing.

## What this repository contains

This repo preserves the main project files used to automate XServer free VPS renewal on a VPS, plus deployment notes and cron examples.

Included:
- `xserver/` Python-based renewal project
- `run_xserver_notify.py` notification wrapper
- `package.json` Node dependencies used by related helper scripts
- `docs/cron.md` example crontab and setup notes
- `docs/deployment.md` full deployment tutorial

Excluded on purpose:
- Real passwords
- Real Telegram bot token / chat id
- Real cookies
- Real `.env`
- Runtime logs
- Screenshots and debug artifacts
- Local virtual environments

## Required environment variables

You must fill your own secrets before running.

### XServer renewal (`xserver/.env`)
Create a file based on `.env.example`:

```env
EMAIL=your_xserver_email
PASSWORD=your_xserver_password
PROXY_SERVER=
DEBUG=false
```

### Telegram notifier (`run_xserver_notify.py` cron environment)
Set these in the shell, crontab, or a separate env file:

```env
TG_BOT_TOKEN=your_telegram_bot_token
TG_CHAT_ID=your_telegram_chat_id
EMAIL=your_xserver_email
PASSWORD=your_xserver_password
ACCOUNT_LABEL=your_account_label
```

Optional:

```env
XSERVER_IMAGE_UPLOAD_URL=https://your-upload-endpoint.example/api/upload
XSERVER_IMAGE_UPLOAD_REFERER=https://your-upload-site.example
XSERVER_IMAGE_STRATEGY_ID=
```

## Quick start

1. Install Python 3 and system packages needed by the project.
2. Copy `xserver/.env.example` to `xserver/.env` and fill your own values.
3. Install Python dependencies:
   ```bash
   cd xserver
   python3 -m venv .venv
   . .venv/bin/activate
   pip install -r requirements.txt
   playwright install
   ```
4. Test manually:
   ```bash
   cd xserver
   . .venv/bin/activate
   python main.py
   ```
5. If using the wrapper notifier:
   ```bash
   cd /path/to/repo
   EMAIL=your@email PASSWORD=yourpass ACCOUNT_LABEL=main-account TG_BOT_TOKEN=xxx TG_CHAT_ID=xxx python3 run_xserver_notify.py
   ```
6. Add cron jobs as needed. See `docs/cron.md`.

## Important notes

- This public repository does **not** include any real secrets.
- If you cloned from a live machine, re-check all files before pushing.
- Do not commit `.env`, logs, screenshots, or bot tokens.
- The automation may stop working if XServer changes UI, CAPTCHA, or anti-bot logic.

## Deployment guide

See:
- `docs/deployment.md`
- `docs/cron.md`
