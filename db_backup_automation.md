# 🗄️ Automated Database Backup with Failure Alert System

> **Assignment:** Write a Python script that performs daily end-of-day database backups and automatically sends an alert to Outlook/Teams if the backup fails for **3 consecutive days**.

---

## 📋 Table of Contents

1. [Overview](#overview)
2. [Project Structure](#project-structure)
3. [Prerequisites & Dependencies](#prerequisites--dependencies)
4. [Configuration](#configuration)
5. [Core Script – `db_backup.py`](#core-script--db_backuppy)
6. [Alert Module – `alert_sender.py`](#alert-module--alert_senderpy)
7. [Scheduler Setup](#scheduler-setup)
8. [How It Works – Flow Diagram](#how-it-works--flow-diagram)
9. [How to Run](#how-to-run)
10. [Testing the Alert](#testing-the-alert)
11. [Troubleshooting Guide](#troubleshooting-guide)

---

## Overview

This solution consists of two Python modules:

| Module | Purpose |
|---|---|
| `db_backup.py` | Performs the DB backup and tracks consecutive failures |
| `alert_sender.py` | Sends alert to **Outlook (SMTP)** or **MS Teams (Webhook)** |

A `backup_state.json` file persists the failure count across days so the script remembers how many days in a row the backup has failed — even after a system restart.

---

## Project Structure

```
db_backup_project/
│
├── db_backup.py          # Main backup script
├── alert_sender.py       # Alert sending module (Outlook + Teams)
├── config.py             # All configuration (DB creds, emails, webhook)
├── backup_state.json     # Auto-generated: tracks failure count
├── logs/
│   └── backup.log        # Auto-generated: daily logs
└── backups/
    └── backup_YYYYMMDD.sql  # Auto-generated: actual backup files
```

---

## Prerequisites & Dependencies

### Install Required Libraries

```bash
pip install schedule pymysql python-dotenv requests
```

| Library | Purpose |
|---|---|
| `schedule` | Runs the backup job daily at a set time |
| `pymysql` | Connects to MySQL/MariaDB (swap for `psycopg2` for PostgreSQL) |
| `requests` | Sends HTTP POST to Microsoft Teams webhook |
| `smtplib` | Built-in Python – sends email via Outlook SMTP |
| `python-dotenv` | Loads secrets from a `.env` file safely |

---

## Configuration

### `config.py`

```python
# ─────────────────────────────────────────────
#  config.py  –  All settings in one place
# ─────────────────────────────────────────────

import os
from dotenv import load_dotenv

load_dotenv()  # Load from .env file

# ── Database ──────────────────────────────────
DB_HOST     = os.getenv("DB_HOST", "localhost")
DB_PORT     = int(os.getenv("DB_PORT", 3306))
DB_USER     = os.getenv("DB_USER", "root")
DB_PASSWORD = os.getenv("DB_PASSWORD", "")
DB_NAME     = os.getenv("DB_NAME", "my_database")

# ── Backup Settings ───────────────────────────
BACKUP_DIR          = "backups"
STATE_FILE          = "backup_state.json"
LOG_FILE            = "logs/backup.log"
BACKUP_TIME         = "23:55"          # Daily backup at 11:55 PM
CONSECUTIVE_FAIL_THRESHOLD = 3         # Alert after this many failures in a row

# ── Outlook (SMTP) Alert ──────────────────────
SMTP_SERVER   = "smtp.office365.com"
SMTP_PORT     = 587
SENDER_EMAIL  = os.getenv("SENDER_EMAIL")      # Your Outlook email
SENDER_PASS   = os.getenv("SENDER_PASS")       # Your Outlook password / App password
RECEIVER_EMAIL = os.getenv("RECEIVER_EMAIL")   # Who gets the alert

# ── Microsoft Teams Webhook Alert ─────────────
TEAMS_WEBHOOK_URL = os.getenv("TEAMS_WEBHOOK_URL")  # Incoming webhook URL from Teams channel
```

### `.env` File (keep this secret – never commit to Git!)

```env
DB_HOST=localhost
DB_PORT=3306
DB_USER=your_db_user
DB_PASSWORD=your_db_password
DB_NAME=your_database_name

SENDER_EMAIL=you@yourcompany.com
SENDER_PASS=your_outlook_app_password
RECEIVER_EMAIL=dba_team@yourcompany.com

TEAMS_WEBHOOK_URL=https://yourcompany.webhook.office.com/webhookb2/xxxx
```

> ⚠️ **Add `.env` to your `.gitignore`** so credentials are never exposed.

---

## Core Script – `db_backup.py`

```python
# ─────────────────────────────────────────────
#  db_backup.py  –  Main backup + failure tracker
# ─────────────────────────────────────────────

import os
import json
import logging
import subprocess
from datetime import datetime

import schedule
import time

from config import (
    DB_HOST, DB_PORT, DB_USER, DB_PASSWORD, DB_NAME,
    BACKUP_DIR, STATE_FILE, LOG_FILE,
    BACKUP_TIME, CONSECUTIVE_FAIL_THRESHOLD
)
from alert_sender import send_outlook_alert, send_teams_alert

# ── Logging Setup ─────────────────────────────
os.makedirs("logs", exist_ok=True)
os.makedirs(BACKUP_DIR, exist_ok=True)

logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s  [%(levelname)s]  %(message)s",
    handlers=[
        logging.FileHandler(LOG_FILE),
        logging.StreamHandler()           # Also print to console
    ]
)
logger = logging.getLogger(__name__)


# ── State Management (persist failure count) ──
def load_state() -> dict:
    """Load backup state from JSON file."""
    if os.path.exists(STATE_FILE):
        with open(STATE_FILE, "r") as f:
            return json.load(f)
    return {"consecutive_failures": 0, "last_backup_date": None}


def save_state(state: dict):
    """Save backup state to JSON file."""
    with open(STATE_FILE, "w") as f:
        json.dump(state, f, indent=4)


# ── Backup Function ───────────────────────────
def perform_backup() -> bool:
    """
    Run mysqldump to back up the database.
    Returns True if successful, False if failed.
    """
    timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
    backup_file = os.path.join(BACKUP_DIR, f"backup_{timestamp}.sql")

    command = [
        "mysqldump",
        f"--host={DB_HOST}",
        f"--port={DB_PORT}",
        f"--user={DB_USER}",
        f"--password={DB_PASSWORD}",
        DB_NAME
    ]

    try:
        logger.info(f"Starting backup for database: '{DB_NAME}'")
        with open(backup_file, "w") as f:
            result = subprocess.run(
                command,
                stdout=f,
                stderr=subprocess.PIPE,
                timeout=3600          # 1-hour timeout
            )

        if result.returncode != 0:
            error_msg = result.stderr.decode()
            logger.error(f"mysqldump failed: {error_msg}")
            # Remove partial/empty file
            if os.path.exists(backup_file):
                os.remove(backup_file)
            return False

        size_mb = os.path.getsize(backup_file) / (1024 * 1024)
        logger.info(f"Backup successful → {backup_file}  ({size_mb:.2f} MB)")
        return True

    except subprocess.TimeoutExpired:
        logger.error("Backup timed out after 1 hour.")
        return False
    except FileNotFoundError:
        logger.error("'mysqldump' not found. Is MySQL client installed?")
        return False
    except Exception as e:
        logger.exception(f"Unexpected error during backup: {e}")
        return False


# ── Alert Logic ───────────────────────────────
def send_failure_alert(consecutive_failures: int):
    """Send alert to Outlook AND Teams when threshold is reached."""
    subject = f"🚨 DB Backup FAILING – {consecutive_failures} Consecutive Days!"
    message = (
        f"⚠️  URGENT: The daily backup for database '{DB_NAME}' on host '{DB_HOST}' "
        f"has FAILED for {consecutive_failures} consecutive day(s).\n\n"
        f"📅  Date: {datetime.now().strftime('%Y-%m-%d %H:%M:%S')}\n"
        f"📁  Backup Directory: {os.path.abspath(BACKUP_DIR)}\n"
        f"📋  Log File: {os.path.abspath(LOG_FILE)}\n\n"
        f"🔧  Please troubleshoot ASAP:\n"
        f"   1. Check if the DB server is running\n"
        f"   2. Verify DB credentials in config\n"
        f"   3. Check disk space on backup directory\n"
        f"   4. Review the log file for detailed errors\n\n"
        f"— Automated Backup Monitor"
    )

    logger.warning(f"Sending failure alert – {consecutive_failures} consecutive failures.")
    send_outlook_alert(subject, message)
    send_teams_alert(subject, message, consecutive_failures)


# ── Main Job (runs daily) ─────────────────────
def run_backup_job():
    """Main job: run backup, update state, alert if needed."""
    logger.info("=" * 60)
    logger.info("Daily Backup Job Started")

    state = load_state()
    success = perform_backup()
    today = datetime.now().strftime("%Y-%m-%d")

    if success:
        logger.info("✅ Backup completed successfully. Resetting failure counter.")
        state["consecutive_failures"] = 0
        state["last_backup_date"] = today
    else:
        state["consecutive_failures"] += 1
        state["last_backup_date"] = today
        logger.warning(
            f"❌ Backup FAILED. Consecutive failures: {state['consecutive_failures']}"
        )

        if state["consecutive_failures"] >= CONSECUTIVE_FAIL_THRESHOLD:
            send_failure_alert(state["consecutive_failures"])

    save_state(state)
    logger.info("Daily Backup Job Finished")
    logger.info("=" * 60)


# ── Scheduler ─────────────────────────────────
if __name__ == "__main__":
    logger.info(f"Backup scheduler started. Will run daily at {BACKUP_TIME}.")
    schedule.every().day.at(BACKUP_TIME).do(run_backup_job)

    while True:
        schedule.run_pending()
        time.sleep(60)   # Check every minute
```

---

## Alert Module – `alert_sender.py`

```python
# ─────────────────────────────────────────────
#  alert_sender.py  –  Outlook + Teams alerts
# ─────────────────────────────────────────────

import smtplib
import logging
import requests
import json
from email.mime.text import MIMEText
from email.mime.multipart import MIMEMultipart

from config import (
    SMTP_SERVER, SMTP_PORT,
    SENDER_EMAIL, SENDER_PASS, RECEIVER_EMAIL,
    TEAMS_WEBHOOK_URL, DB_NAME, DB_HOST
)

logger = logging.getLogger(__name__)


# ── Outlook / SMTP Alert ──────────────────────
def send_outlook_alert(subject: str, message: str):
    """Send an email alert via Outlook SMTP."""
    try:
        msg = MIMEMultipart("alternative")
        msg["Subject"] = subject
        msg["From"]    = SENDER_EMAIL
        msg["To"]      = RECEIVER_EMAIL

        # Plain text version
        msg.attach(MIMEText(message, "plain"))

        with smtplib.SMTP(SMTP_SERVER, SMTP_PORT) as server:
            server.ehlo()
            server.starttls()                          # Encrypt the connection
            server.login(SENDER_EMAIL, SENDER_PASS)
            server.sendmail(SENDER_EMAIL, RECEIVER_EMAIL, msg.as_string())

        logger.info(f"✉️  Outlook alert sent to: {RECEIVER_EMAIL}")

    except smtplib.SMTPAuthenticationError:
        logger.error("Outlook SMTP auth failed. Check SENDER_EMAIL / SENDER_PASS.")
    except smtplib.SMTPException as e:
        logger.error(f"SMTP error when sending Outlook alert: {e}")
    except Exception as e:
        logger.exception(f"Unexpected error sending Outlook alert: {e}")


# ── Microsoft Teams Webhook Alert ─────────────
def send_teams_alert(subject: str, message: str, fail_count: int):
    """Send an adaptive card alert to a Microsoft Teams channel via webhook."""
    if not TEAMS_WEBHOOK_URL:
        logger.warning("TEAMS_WEBHOOK_URL not set. Skipping Teams alert.")
        return

    # Teams Adaptive Card payload
    payload = {
        "type": "message",
        "attachments": [
            {
                "contentType": "application/vnd.microsoft.card.adaptive",
                "content": {
                    "$schema": "http://adaptivecards.io/schemas/adaptive-card.json",
                    "type": "AdaptiveCard",
                    "version": "1.4",
                    "body": [
                        {
                            "type": "TextBlock",
                            "text": f"🚨 {subject}",
                            "weight": "Bolder",
                            "size": "Large",
                            "color": "Attention"
                        },
                        {
                            "type": "FactSet",
                            "facts": [
                                {"title": "Database",      "value": DB_NAME},
                                {"title": "Host",          "value": DB_HOST},
                                {"title": "Failure Count", "value": str(fail_count)},
                            ]
                        },
                        {
                            "type": "TextBlock",
                            "text": "**Action Required:** Please troubleshoot ASAP.\n\n"
                                    "1. Check if DB server is running\n"
                                    "2. Verify credentials in config\n"
                                    "3. Check disk space\n"
                                    "4. Review backup.log",
                            "wrap": True
                        }
                    ]
                }
            }
        ]
    }

    try:
        response = requests.post(
            TEAMS_WEBHOOK_URL,
            headers={"Content-Type": "application/json"},
            data=json.dumps(payload),
            timeout=10
        )
        if response.status_code == 200:
            logger.info("📣 Teams alert sent successfully.")
        else:
            logger.error(f"Teams webhook returned HTTP {response.status_code}: {response.text}")

    except requests.exceptions.ConnectionError:
        logger.error("Could not connect to Teams webhook. Check network/URL.")
    except requests.exceptions.Timeout:
        logger.error("Teams webhook request timed out.")
    except Exception as e:
        logger.exception(f"Unexpected error sending Teams alert: {e}")
```

---

## Scheduler Setup

### Option A – Run as a Python Process (simplest)

```bash
python db_backup.py
```

Keep it running in the background. Best used with `nohup` on Linux:

```bash
nohup python db_backup.py > /dev/null 2>&1 &
```

### Option B – Windows Task Scheduler

1. Open **Task Scheduler** → Create Basic Task
2. Set trigger: **Daily** → Time: `11:55 PM`
3. Action: **Start a program** → `python.exe`
4. Arguments: `C:\path\to\db_backup.py --run-now`

Add a `--run-now` flag check at the bottom of `db_backup.py`:

```python
import sys
if __name__ == "__main__":
    if "--run-now" in sys.argv:
        run_backup_job()      # Run once immediately (for Task Scheduler)
    else:
        # Use schedule loop (for always-on process)
        schedule.every().day.at(BACKUP_TIME).do(run_backup_job)
        while True:
            schedule.run_pending()
            time.sleep(60)
```

### Option C – Linux Cron Job

```bash
# Open crontab editor
crontab -e

# Add this line: run at 11:55 PM every day
55 23 * * * /usr/bin/python3 /path/to/db_backup.py --run-now
```

---

## How It Works – Flow Diagram

```
Every day at 11:55 PM
        │
        ▼
  ┌─────────────────┐
  │  Run mysqldump  │
  └────────┬────────┘
           │
    ┌──────┴──────┐
    │  Success?   │
    └──────┬──────┘
           │
     ┌─────┴──────┐
    YES           NO
     │             │
     ▼             ▼
Reset failure   failure_count += 1
counter = 0     Save to backup_state.json
     │             │
     │      ┌──────┴──────────┐
     │      │ failure_count   │
     │      │   >= 3 ?        │
     │      └──────┬──────────┘
     │           YES
     │             │
     │             ▼
     │    ┌─────────────────────┐
     │    │ Send Outlook email  │
     │    │ Send Teams alert    │
     │    └─────────────────────┘
     │
     ▼
 Log result → backup.log
 Sleep until next day
```

---

## How to Run

### Step 1 – Clone / Set Up the Project

```bash
mkdir db_backup_project && cd db_backup_project
# Place db_backup.py, alert_sender.py, config.py here
```

### Step 2 – Install Dependencies

```bash
pip install schedule pymysql python-dotenv requests
```

### Step 3 – Create `.env` File

```bash
cp .env.example .env
# Fill in your DB credentials and email/Teams settings
```

### Step 4 – Start the Scheduler

```bash
python db_backup.py
```

You should see:

```
2024-11-15 23:55:00  [INFO]  Backup scheduler started. Will run daily at 23:55.
2024-11-15 23:55:00  [INFO]  ============================================================
2024-11-15 23:55:00  [INFO]  Daily Backup Job Started
2024-11-15 23:55:01  [INFO]  Starting backup for database: 'my_database'
2024-11-15 23:55:04  [INFO]  Backup successful → backups/backup_20241115_235501.sql  (12.45 MB)
2024-11-15 23:55:04  [INFO]  ✅ Backup completed successfully. Resetting failure counter.
```

---

## Testing the Alert

To test the alert system without waiting for 3 real failures, temporarily lower the threshold:

```python
# In config.py – for testing only!
CONSECUTIVE_FAIL_THRESHOLD = 1   # Alert on first failure
```

Then simulate a failure by using wrong DB credentials, then run:

```bash
python -c "from db_backup import run_backup_job; run_backup_job()"
```

You should receive both an Outlook email and a Teams card message.

---

## Troubleshooting Guide

| Problem | Likely Cause | Fix |
|---|---|---|
| `mysqldump: command not found` | MySQL client not installed | Install MySQL client: `sudo apt install mysql-client` |
| `SMTP Auth Error` | Wrong email/password | Use an App Password if 2FA is on in Outlook |
| `Teams webhook 400 error` | Malformed webhook URL | Re-copy the webhook URL from Teams channel settings |
| `backup_state.json` not found | First run | Normal – it gets created automatically on first run |
| Backup file is 0 bytes | Wrong DB name or no tables | Check `DB_NAME` in config and confirm DB has data |
| Alert sent every day (not just at 3) | Threshold not set correctly | Check `CONSECUTIVE_FAIL_THRESHOLD` in config |
| Script exits on its own | Unhandled exception | Check `logs/backup.log` for the error stack trace |

---

## Key Concepts Summary

| Concept | Where Used |
|---|---|
| `subprocess.run()` | Runs `mysqldump` shell command from Python |
| `json.load/dump` | Persists failure count across days |
| `schedule` library | Triggers the job daily at a set time |
| `smtplib` + `SMTP.starttls()` | Sends encrypted email via Outlook |
| Teams Adaptive Card JSON | Sends a rich-formatted card to Teams channel |
| `logging` module | Writes timestamped logs to file + console |
| `.env` + `dotenv` | Keeps credentials out of source code |

---

*Built for SME Assignment – Python Automation & Database Management*
