Perfect — **3 → Volume III — Permissions & Ownership**
Now we enter one of the *most important master-level Linux topics.*
This is the part that separates *users* from *admins* and *admins* from *SRE/DevOps engineers.*

We go deep like a real book —
not shortcuts, not cheat cards.

---

# 📘 **Volume III — Permissions, Ownership & Security Model**

### Chapters:

```
3.1 chmod, chown, chgrp (Core Permission Mechanics)
3.2 SUID, SGID & Sticky Bit (Privilege Inheritance)
3.3 ACLs (Fine-Grained Access Control)
3.4 umask (Default File Permissions)
3.5 SELinux & AppArmor (Security Enforcement Layer)
3.6 Process Ownership & Privilege Model
3.7 Real-World Security Incidents & Exploits
```

We start now with **3.1 — chmod, chown, chgrp**, slowly and deeply.

---

# 📖 **3.1 — Linux Permissions: Understanding the Anatomy of a File**

Run:

```
ls -l
```

Example output:

```
-rwxr-xr-- 1 root admin  3542 Jan  4 14:10 script.sh
```

Let’s break this line down **character by character**.

```
- rwx r-x r--  1  root admin   3542  Jan 4 14:10  script.sh
|  |   |   |
|  |   |   └── Other (world)
|  |   └────── Group
|  └────────── Owner (user)
└───────────── Type of file
```

### 🔹 Field by field:

| Field       | Meaning                                                               |
| ----------- | --------------------------------------------------------------------- |
| `-`         | regular file (other options: `d` dir, `l` link, `c` char device, etc) |
| `rwx`       | Owner permissions                                                     |
| `r-x`       | Group permissions                                                     |
| `r--`       | World (others) permissions                                            |
| `root`      | File owner user                                                       |
| `admin`     | File owner group                                                      |
| `3542`      | File size in bytes                                                    |
| `script.sh` | Name                                                                  |

---

## 🧠 Permission Bits Explained

| Permission | Letter | Meaning                                      |
| ---------- | ------ | -------------------------------------------- |
| read       | `r`    | view contents of file / list directory       |
| write      | `w`    | modify file / create/delete inside directory |
| execute    | `x`    | run file / enter directory                   |

So:

| rwx   | Permission        |
| ----- | ----------------- |
| `rwx` | full access       |
| `rw-` | read + write only |
| `r--` | read only         |
| `---` | no access         |

---

# 🔥 Changing Permissions — `chmod`

Two ways:

## A) Symbolic mode

```bash
chmod u+x file     # give execute to user
chmod g-w file     # remove write from group
chmod o=r file     # give read only to others
chmod a+r file     # give read to everyone
```

Here:

| u | user (owner) |
| g | group |
| o | others |
| a | all (ugo) |

Operators:

| `+` add | `-` remove | `=` set exactly |

---

## B) Numeric (absolute) mode — **professional way**

Each permission is a number:

| Permission  | Value |
| ----------- | ----- |
| read (r)    | 4     |
| write (w)   | 2     |
| execute (x) | 1     |

So permissions become:

| rwx | 4+2+1 = 7 |
| rw- | 4+2 = 6 |
| r-x | 4+1 = 5 |
| r-- | 4 = 4 |

### Example:

```
chmod 755 script.sh
```

Breakdown:

| Owner   | Group   | Others  |
| ------- | ------- | ------- |
| 7 = rwx | 5 = r-x | 5 = r-x |

Meaning:

✔ owner can do everything
✔ group can read+execute
✔ others can read+execute

This is default for scripts.

Another example:

```
chmod 600 secrets.txt
```

Owner = **rw-**
Group = **---**
Others = **---**

Only the owner can read it — good for passwords.

---

# 🔥 Changing Ownership — `chown`

Give file to another user:

```bash
chown ubuntu file.txt
```

Change user + group:

```bash
chown ubuntu:devops file.txt
```

---

# 🔥 Change Only Group — `chgrp`

```bash
chgrp admin logs/
```

---

## 🔥 Real-World Case: Fix Permission Denied on a Directory

Problem:

```
bash: ./run.sh: Permission denied
```

Fix execute bit:

```bash
chmod +x run.sh
```

---

## 🔥 Real-World Case: Website accessible by nginx, not user

```
chmod 755 /var/www/html
```

or better:

```
chown -R nginx:nginx /var/www/html
```

---

## 🧠 You Now Understand:

✔ Read/Write/Execute meaning
✔ How ls -l encodes permissions
✔ chmod numeric & symbolic
✔ Ownership model (user vs group)
✔ Real debugging scenarios

This was **only the foundation** —
Now you're ready for the *hidden power layer*:

---
