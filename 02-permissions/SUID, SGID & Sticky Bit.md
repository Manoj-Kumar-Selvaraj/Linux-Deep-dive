# 🔥 First — Permission Bits You Already Know

Basic permission block:

```
r = read
w = write
x = execute
```

But there exist **3 special permission flags**:

| Special Bit | Affects                                 |
| ----------- | --------------------------------------- |
| SUID        | User execution privilege                |
| SGID        | Group execution privilege & shared dirs |
| Sticky Bit  | Delete protection in shared directories |

These aren't visible normally — they appear as **s** and **t** in permission fields.

---

## 🔥 3.2.1 SUID — Execute as Owner, Not Current User

This is **Set User ID on execution**.

Normally, when you run a binary:

> You run it as *your* user permissions.

But if SUID is set:

> You run it with **file owner's permissions**, even if it's root.

### Example:

```
-rwsr-xr-x 1 root root /usr/bin/passwd
```

Notice `s` in place of `x`.
This means:

```
passwd runs as root even when a normal user executes it
```

Because to change passwords, the command must modify `/etc/shadow`, which only root can access.

### How to set SUID:

```bash
chmod u+s file
```

Or numerically:

```bash
chmod 4755 file
```

Breakdown:

```
4 = SUID
7 = owner rwx
5 = group r-x
5 = others r-x
```

---

## ⚠ SUID IS DANGEROUS IF MISUSED

If you set SUID on a shell or script:

```bash
chmod u+s /bin/bash
```

Then **any normal user becomes root** simply by running bash.

This is a *full privilege escalation* → never do this in production.

---

## 🧠 3.2.2 SGID — Execute as Group, not Current User

and **Shared folder group inheritance**

If SUID affects USER,
**SGID affects GROUP**.

### 1) On Executables

```
chmod g+s file
```

Or:

```
chmod 2755 file
```

Meaning anyone who runs the binary runs it with **group permissions**.

Useful for multi-user applications.

---

### 2) On Directories — The Secret Power

This is **very important for DevOps & shared repos**.

If SGID is set on a directory:

> Any new file created inside inherits the directory's group,
> not the creator's primary group.

Example:

```
chmod g+s /shared
```

Observe permissions:

```
drwxrwsr-x  shared/
```

`s` appears in the **group** position.

Now:

* userA creates file → group = shared
* userB creates file → group = shared

Everyone collaborating, no group mismatch.

This is how `/var/www`, `/project/repos`, `/shared-storage` should be configured.

---

## ⚠ SGID + write = can cause unintended write access

If multiple users can write inside directory → they may overwrite each other's files.

Use together with Sticky Bit to protect deletions.

---

## 🔥 3.2.3 Sticky Bit — Prevent Deletion in Shared Directory

Symbol = **t**

Set with:

```bash
chmod +t dir
```

Or numeric:

```bash
chmod 1777 dir
```

Example:

```
drwxrwxrwt   /tmp
```

Notice the **t**.

Meaning:

| Action                          | Allowed? |
| ------------------------------- | -------- |
| Users create files              | ✔ Yes    |
| Users delete own files          | ✔ Yes    |
| Users delete each other’s files | ❌ No     |

**THIS IS WHY /tmp uses Sticky Bit.**

Multiple apps and users write there
but shouldn't delete each other's files.

Use cases:

| Directory            | Why sticky bit?                 |
| -------------------- | ------------------------------- |
| `/tmp`               | multi-process temp files safely |
| `/shared/upload`     | prevent accidental deletion     |
| Kubernetes emptyDirs | sometimes recommended           |

---

# 🔥 Real Production Use Cases

### 1) Shared deployment folder without permission chaos

```bash
chown root:devteam /var/www
chmod 2775 /var/www        # SGID ensures group stays devteam
```

### 2) User upload directory where no one can delete others' files

```bash
chmod 1777 /uploads
```

### 3) Binary that requires elevated privilege (controlled SUID)

Example: custom monitoring binary needing root to read /proc/kcore:

```bash
chmod 4750 monitor-tool
```

Owners = root
Group = ops
Only Ops team can run with root capability.

---

# 🔻 Security Risk Insight (This is Master Level)

Misconfigured special bits = privilege escalation.

### Vulnerability Example:

```
find . -exec /bin/sh \;
```

If **find** binary had SUID root set -> boom.
User gets a root shell instantly.

Never SUID normal binaries like:

| Dangerous           | Why                         |
| ------------------- | --------------------------- |
| `/bin/bash`         | instant root shell          |
| `/usr/bin/python`   | can spawn privileged shells |
| `/usr/bin/find`     | can exec commands           |
| `/usr/bin/nano/vim` | edit system files as root   |

SUID should be used *rarely and intentionally*.

---

# 🧠 Summary Map

| Feature    | Symbol      | Numeric | Purpose                         |
| ---------- | ----------- | ------- | ------------------------------- |
| SUID       | s in user   | 4xxx    | run as owner                    |
| SGID       | s in group  | 2xxx    | run as group / shared ownership |
| Sticky Bit | t in others | 1xxx    | stop deletion by non-owners     |

You now understand **privilege inheritance at file-system level**,
not just chmod numbers.

This is where Linux security **begins**, not ends.

---
