# 📘 **CHAPTER 09 — CRON JOBS & SCHEDULING**

*(FULL BOOK-LEVEL, DETAILED, SLOW, EASY + DEEP)*

This chapter builds you into a **Linux Scheduling Master**.
You will understand:

* cron vs systemd timers
* crontab formats
* special strings (@daily, @hourly…)
* environment handling
* logs
* debugging failed cron jobs
* real DevOps use cases
* production-grade scheduling

Let’s begin in an extremely clear way.

---

# ----------------------------------------------------

# 🟦 **9.1 What Is Cron? (Deep but very clear)**

# ----------------------------------------------------

Cron is one of the OLDEST and most RELIABLE parts of Unix/Linux.

It is a **time-based job scheduler**.

Think of cron as:

> “A robot that runs commands at specific times automatically.”

Cron is used for:

* backups
* cleaning logs
* rotating files
* syncing data
* restarting services
* sending alerts
* running scripts every minute/hour/day
* automation in servers without pipelines

Cron runs even when **you are logged out**.
It is part of the system.

---

# ----------------------------------------------------

# 🟦 **9.2 The Cron Service (cron vs crond)**

# ----------------------------------------------------

### ✔ Cron daemon

The actual background service is:

* `crond` (RedHat/CentOS/Amazon Linux)
* `cron` (Ubuntu/Debian)

To check status:

```bash
systemctl status cron
```

or

```bash
systemctl status crond
```

To start/enable:

```bash
sudo systemctl enable --now cron
```

This daemon checks **every minute** for upcoming jobs.

---

# ----------------------------------------------------

# 🟦 **9.3 Crontab Files (VERY IMPORTANT)**

# ----------------------------------------------------

There are **three types of cron files**:

### 1. **User crontab**

Runs as that user.

Edit:

```bash
crontab -e
```

List:

```bash
crontab -l
```

Delete:

```bash
crontab -r
```

---

### 2. **System-wide crontab**

File:

```
/etc/crontab
```

These require **specifying user**, because root controls them.

Format:

```
minute hour day month weekday user command
```

---

### 3. **Cron directories**

These run automatically if files are dropped inside:

```
/etc/cron.hourly/
/etc/cron.daily/
/etc/cron.weekly/
/etc/cron.monthly/
```

If you put a script in `/etc/cron.daily/`, it runs daily.

---

# ----------------------------------------------------

# 🟦 **9.4 Cron Schedule Format (The heart of cron)**

# ----------------------------------------------------

Cron uses **5 time fields**:

```
* * * * *
│ │ │ │ │
│ │ │ │ └── Day of week (0–7)   (0 or 7 = Sunday)
│ │ │ └──── Month (1–12)
│ │ └────── Day of month (1–31)
│ └──────── Hour (0–23)
└────────── Minute (0–59)
```

### Example:

```
30 2 * * *   /opt/backup.sh
↑  ↑
|  └─ Run at 2:30 AM
└──── Run at 30th minute
```

---

# ----------------------------------------------------

# 🟦 **9.5 Cron Symbol Meanings**

# ----------------------------------------------------

### ⭐ Asterisk (`*`)

Means “every possible value”.

`* * * * *` = run every minute

### ⭐ List

```
1,5,10 * * * *   → minute 1, 5, 10
```

### ⭐ Range

```
1-5 * * * *   → minutes 1 to 5
```

### ⭐ Step values

```
*/10 * * * *   → every 10 minutes
0  */2  * * *  → every 2 hours
```

### ⭐ Multiple ranges

```
1-5,10-15 * * * *
```

---

# ----------------------------------------------------

# 🟦 **9.6 Special Cron Keywords (Shortcut Format)**

# ----------------------------------------------------

Cron also allows special strings:

| Keyword     | Meaning            |
| ----------- | ------------------ |
| `@reboot`   | run on startup     |
| `@yearly`   | run once a year    |
| `@monthly`  | run monthly        |
| `@weekly`   | run weekly         |
| `@daily`    | run everyday       |
| `@hourly`   | run every hour     |
| `@every 5m` | systemd timer only |

Example:

```
@daily /opt/backup.sh
```

---

# ----------------------------------------------------

# 🟦 **9.7 Understanding Cron’s Environment (Critical)**

# ----------------------------------------------------

This is where **most people FAIL**.

### ✔ Cron uses a VERY LIMITED environment.

* No `$PATH` like your shell
* No aliases
* No user profile
* Minimal environment variables

### So commands like these FAIL in cron:

```
python
aws
kubectl
```

because they are not in cron's PATH.

### Solution?

Always use **full paths**:

Example:

```
/usr/bin/python3 /opt/myapp/app.py
/usr/bin/aws s3 sync /logs s3://bucket
/usr/bin/kubectl get pods
```

To find full path:

```bash
which python3
which aws
which kubectl
```

---

# ----------------------------------------------------

# 🟦 **9.8 Redirecting Output (Essential for Debugging)**

# ----------------------------------------------------

Cron does NOT show output on screen.

To debug cron scripts, redirect output:

```
* * * * *  /opt/script.sh >> /var/log/script.log 2>&1
```

### Meaning:

* stdout → log
* stderr → also log

This makes debugging easy.

---

# ----------------------------------------------------

# 🟦 **9.9 Where Cron Logs Are Stored**

# ----------------------------------------------------

On Ubuntu/Debian:

```
/var/log/syslog
```

Check:

```bash
grep CRON /var/log/syslog
```

On RHEL/CentOS/Amazon:

```
/var/log/cron
```

---

# ----------------------------------------------------

# 🟦 **9.10 Writing a PRODUCTION-READY Cron Script**

# ----------------------------------------------------

Example backup script:

**/opt/scripts/backup.sh**

```bash
#!/bin/bash
set -euo pipefail

SOURCE="/var/www"
DEST="/backup/www-$(date +%F).tar.gz"

/bin/tar -czf "$DEST" "$SOURCE"
echo "$(date) Backup successful: $DEST"
```

Make executable:

```bash
chmod +x /opt/scripts/backup.sh
```

Add to crontab:

```
0 2 * * * /opt/scripts/backup.sh >> /var/log/backup.log 2>&1
```

---

# ----------------------------------------------------

# 🟦 **9.11 Systemd Timers (Modern Replacement for Cron)**

# ----------------------------------------------------

Cron is old but great.
Systemd timers are newer and stronger.

Why big companies prefer **systemd timers**:

* easier to debug
* independent logs
* retry automatically
* can require network availability
* can depend on other services
* accurate time
* millisecond resolution

We’ll cover them next.

---

# ----------------------------------------------------

# 🟦 **9.12 Systemd Timer — Basic Example**

# ----------------------------------------------------

### Two files required:

### 1. Service file → what to run

`/etc/systemd/system/myjob.service`

```ini
[Unit]
Description=My scheduled job

[Service]
Type=oneshot
ExecStart=/opt/scripts/cleanup.sh
```

### 2. Timer file → when to run

`/etc/systemd/system/myjob.timer`

```ini
[Unit]
Description=Run cleanup job every day

[Timer]
OnCalendar=daily
Persistent=true

[Install]
WantedBy=timers.target
```

### Start timer:

```bash
systemctl enable --now myjob.timer
```

### View logs:

```bash
journalctl -u myjob.service
```

CRON cannot do this — systemd timers are far better.

---

# ----------------------------------------------------

# 🟦 **9.13 Systemd Timer Scheduling Options**

# ----------------------------------------------------

### `OnCalendar` supports:

* `daily`
* `hourly`
* `weekly`
* `monthly`
* `*:0/15` → every 15 minutes
* `Mon-Fri 09:00` → weekdays 9 AM
* `*-*-01 03:00` → first day of every month

Systemd timers are extremely flexible.

---

# ----------------------------------------------------

# 🟦 **9.14 Real DevOps / SRE Examples**

# ----------------------------------------------------

### ✔ Auto-restart service if down

```
*/5 * * * *  systemctl restart nginx
```

---

### ✔ Backup all logs nightly

```
0 1 * * *  /opt/backup_logs.sh
```

---

### ✔ Clean temp directory every hour

```
0 * * * *  find /tmp -type f -mtime +1 -delete
```

---

### ✔ Health check every minute

```
* * * * * curl -s http://localhost:8080/health || systemctl restart app
```

---

### ✔ Kubernetes CronJob (extra bonus)

YAML:

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: cleanup
spec:
  schedule: "*/5 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: cleaner
            image: busybox
            args: ["sh", "-c", "echo CLEANUP"]
          restartPolicy: OnFailure
```

---

# ⭐ **SUMMARY — You Now Fully Understand CRON at a True Production-Level**

You learned:

### ✔ Cron basics

### ✔ Cron schedule format

### ✔ Special strings

### ✔ Environment problems & fixes

### ✔ Logs & debugging

### ✔ System crontab vs user crontab

### ✔ Cron directories

### ✔ How to write production-ready scripts for cron

### ✔ How systemd timers work

### ✔ Real SRE/DevOps examples

This is **complete professional mastery**.

---