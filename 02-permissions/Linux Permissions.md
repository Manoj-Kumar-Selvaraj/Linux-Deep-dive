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

That is a **real production problem**, and your pain is 100% valid.

### ❗ `chown -R user:group /data` is the worst option

Because recursive `chown` walks through *every inode* and updates metadata one-by-one → extremely slow on millions of files (could take hours/days depending on IOPS).

To do this **instantly**, we don’t touch each file.
Instead, we change ownership at the **filesystem level**, or remap ownership using **ID shifting** or **bind mounts with ACL mapping**.

Here are **the FASTEST real-world ways to do this**.

---

# 🥇 **Method 1 — UID/GID Mapping (Instant, No File Rewrites)**

Linux doesn’t store NAME in metadata — it stores **UID & GID numbers**.

Meaning:

| Username | UID  |
| -------- | ---- |
| olduser  | 1001 |
| newuser  | 2005 |

Changing the owner name instantly requires only remapping UID/GID to match.

### Step-by-step

### 1. Find old owner UID/GID

```bash
ls -nd /data/folder
```

Example output:

```
drwxr-xr-x 1001 1001 ...
```

Means:

```
UID = 1001
GID = 1001
```

### 2. modify `/etc/passwd` and `/etc/group` to assign Username to same UID/GID

```bash
usermod -u 1001 newuser
groupmod -g 1001 newgroup
```

Now all files that belonged to old user auto-belong to newuser instantly.

📌 **200GB, millions of files = done in seconds.**

No `chown`, no scanning, no recursion.

---

# 🥇🥇 Method 2 — Bind Mount with Ownership Mapping (Zero copy)

If you cannot modify system UID/GID, use mount option:

```bash
mount --bind /data /data
mount -o remount,uid=<newUID>,gid=<newGID> /data
```

Now files appear owned by new user/group — **instant applied.**

This is EXTREMELY useful for:

| Scenario                   | Works?               |
| -------------------------- | -------------------- |
| Large data migrations      | ✔ Fastest            |
| NFS shares                 | ✔ Supported          |
| Docker/K8s volume mapping  | ✔ Very common        |
| Prod apps with no downtime | ✔ Beautiful solution |

No inodes touched.

---

# 🥇🥇🥇 Method 3 — ACL Rewrite Instead of Ownership Rewrite

Instead of altering file ownership,
give full permissions using ACL (fast metadata write, parallelizable).

```bash
setfacl -R -m u:newuser:rwx /data
setfacl -R -m g:newgroup:rwx /data
```

Benefits:

| ACL                           | Advantage               |
| ----------------------------- | ----------------------- |
| Does not rewrite owner fields | Much faster             |
| Allows gradual transition     | Both old+new can access |
| Works well in 24x7 systems    | No outage required      |

---

# 🔥 Benchmark Comparison

| Method                     | Speed for 200GB / millions files |
| -------------------------- | -------------------------------- |
| `chown -R`                 | ❌ slow (hours/days)              |
| UID/GID remap              | 🥇 instantaneous                 |
| Bind mount ownership remap | 🥇 instantaneous                 |
| ACL propagation            | ⚡ near-instant + parallel        |
| parallel `chown`           | ⚡ if forced, but risky           |

---

## When does UID/GID remapping win?

| Case                                | Use?   |
| ----------------------------------- | ------ |
| Want instant ownership change       | ✔ Best |
| rename user w/o touching filesystem | ✔ Best |
| huge directory                      | ✔ Best |
| low IOPS system                     | ✔ Best |

---

# ✔ Practical Production Example

You migrated `/home/data` to new server, user changed from `appuser` → `apprun`.

Instead of:

```bash
chown -R apprun:apprun /home/data   # takes forever
```

Do:

```bash
# Find UID
ls -nd /home/data
# assume old UID=1001

# Map new username to old UID
usermod -u 1001 apprun
groupmod -g 1001 apprun
```

Done. Instantly.
All files now belong to `apprun`.

---
