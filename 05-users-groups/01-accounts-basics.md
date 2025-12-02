# ---------------------------------------------------------

# 🟦 **5.1 — What Is a User in Linux?**

# ---------------------------------------------------------

A “user” in Linux is NOT a human —
it is simply:

> **An identity the OS can attach permissions to.**

There are two categories:

### ✔ Human users

* You
* Developers
* Admins

### ✔ System users

* nginx
* mysql
* nobody
* daemon

These users **cannot log in** and only exist for process ownership and security isolation.

---

# ---------------------------------------------------------

# 🟦 **5.2 — UID: User IDs (VERY IMPORTANT)**

# ---------------------------------------------------------

Linux does not work with names internally.
It works only with **numbers**.

Every user has:

* a username (text)
* a UID (number)

Example:

```
root:x:0:0:root:/root:/bin/bash
```

The UID is the **3rd field** → `0`.

### UID ranges:

| Range | Purpose          |
| ----- | ---------------- |
| 0     | root (superuser) |
| 1–999 | system users     |
| 1000+ | human users      |

So:

* user with UID 0 == **SUPERUSER**
* user with UID 1001 == normal user

---

# ---------------------------------------------------------

# 🟦 **5.3 — What Is a Group?**

# ---------------------------------------------------------

A group is a **collection of users**.
Permissions in Linux apply to:

* owner
* group
* others

Example:

```bash
chown manoj:devteam file.txt
```

Means:

* Owner = manoj
* Group = devteam

Anyone in `devteam` group gets group permissions.

---

# ---------------------------------------------------------

# 🟦 **5.4 — Primary Group vs Supplementary Groups**

# ---------------------------------------------------------

Every user has:

### 1. Primary group

The group associated with newly created files.

### 2. Supplementary groups

Extra groups for permissions.

Example:

```bash
id manoj
```

Output:

```
uid=1001(manoj) gid=1001(manoj) groups=1001(manoj),27(sudo),1002(devops)
```

* Primary → manoj
* Supplementary → sudo, devops

---

# ---------------------------------------------------------

# 🟦 **5.5 — /etc/passwd (Deep Explanation)**

# ---------------------------------------------------------

This file stores **basic user account info**.

Example line:

```
manoj:x:1000:1000:Manoj Kumar:/home/manoj:/bin/bash
```

Fields:

```
username : password : UID : GID : GECOS : home_dir : shell
```

### Detailed breakdown:

| Field       | Meaning                        |
| ----------- | ------------------------------ |
| manoj       | username                       |
| x           | password stored in /etc/shadow |
| 1000        | UID                            |
| 1000        | primary GID                    |
| Manoj Kumar | description                    |
| /home/manoj | home directory                 |
| /bin/bash   | login shell                    |

---

# ---------------------------------------------------------

# 🟦 **5.6 — /etc/shadow (Where passwords REALLY live)**

# ---------------------------------------------------------

Only root can read this file.

Example:

```
manoj:$6$ssdf892...:19649:0:99999:7:::
```

Fields:

```
username : hashedpassword : last_change : min_days : max_days : warn_days : inactive : expire
```

The password hash is stored here.

---

# ---------------------------------------------------------

# 🟦 **5.7 — /etc/group**

# ---------------------------------------------------------

Defines groups and their members:

Example:

```
devops:x:1002:manoj,siva
```

Fields:

```
groupname : placeholder : GID : group_members
```

---

# ---------------------------------------------------------

# 🟦 **5.8 — Creating & Managing Users**

# ---------------------------------------------------------

### ✔ Create a user

```bash
sudo useradd manoj
sudo passwd manoj
```

### ✔ Create user with home directory:

```bash
sudo useradd -m manoj
```

### ✔ Create user with shell:

```bash
sudo useradd -s /bin/bash manoj
```

### ✔ Delete user (keep home dir)

```bash
sudo userdel manoj
```

### ✔ Delete user + home dir

```bash
sudo userdel -r manoj
```

---

# ---------------------------------------------------------

# 🟦 **5.9 — Managing Groups**

# ---------------------------------------------------------

### Create group:

```bash
sudo groupadd devops
```

### Add user to group:

```bash
sudo usermod -aG devops manoj
```

### Remove user from group:

```bash
sudo gpasswd -d manoj devops
```

### Change primary group:

```bash
usermod -g devops manoj
```

---

# ---------------------------------------------------------

# 🟦 **5.10 — Password Aging (Security)**

*(In /etc/shadow)*

Example:

```bash
sudo chage -l manoj
```

Output:

```
Last password change : Nov 29, 2025
Password expires     : never
Minimum days         : 0
Maximum days         : 99999
Warning days         : 7
```

### Modify:

```bash
sudo chage -M 90 manoj    # max 90 days
sudo chage -W 7 manoj     # warn 7 days before
```

This is required for compliance.

---

# ---------------------------------------------------------

# 🟦 **5.11 — Understanding the Login Shell**

# ---------------------------------------------------------

Shell determines what environment user gets after login:

Examples:

* `/bin/bash`
* `/bin/sh`
* `/usr/sbin/nologin`  ← used for system users
* `/bin/false`         ← disable login

Check:

```bash
getent passwd manoj
```

Change:

```bash
sudo chsh -s /bin/bash manoj
```

---

# ---------------------------------------------------------

# 🟦 **5.12 — Sudo Access (Very Important for DevOps/SRE)**

# ---------------------------------------------------------

To give sudo access:

```bash
sudo usermod -aG sudo manoj          # Ubuntu
sudo usermod -aG wheel manoj         # RHEL/Amazon
```

Sudo config file:

```
/etc/sudoers
/etc/sudoers.d/<file>
```

Edit using ONLY this:

```bash
sudo visudo
```

### Example rule:

```
manoj ALL=(ALL) NOPASSWD: /usr/bin/systemctl restart nginx
```

User can restart nginx without password — common in automation.

---

# ---------------------------------------------------------

# 🟦 **5.13 — Real-World DevOps/SRE Scenarios**

# ---------------------------------------------------------

### ✔ Give Jenkins only permissions needed

```bash
usermod -s /usr/sbin/nologin jenkins
```

---

### ✔ Restrict access to a folder for a team

```bash
groupadd appteam
usermod -aG appteam manoj
chown -R root:appteam /opt/app
chmod -R 770 /opt/app
```

---

### ✔ Shared folder for developers

```bash
mkdir /data/share
groupadd developers
chgrp developers /data/share
chmod 2770 /data/share
```

The `2` (setgid) ensures new files inherit the group.

---

### ✔ Create a service account for automation

```bash
sudo useradd -r -s /usr/sbin/nologin deploy
sudo passwd -l deploy
```

---

### ✔ Disable a user safely

```bash
usermod -L manoj
```

OR expire:

```bash
chage -E 0 manoj
```

---