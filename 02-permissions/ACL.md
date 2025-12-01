### 📖 **3.3 — ACLs (Access Control Lists) — Beyond chmod, this is real precision control**

Traditional Linux permissions (rwx for **u/g/o**) are limited:

```
User       (u) → owner only
Group      (g) → only ONE group
Others     (o) → everyone else
```

Problem?

✔ What if you want *two groups* to access a file?
✔ What if you want one user to read, another to write?
✔ What if you want K8s service accounts to have access without changing ownership?
✔ chmod can’t do this — ACL can.

ACL = **fine-grained permission layer on top of chmod**.

It lets you do:

```
Give userA read, userB write, groupC execute
without touching actual ownership.
```

---

# 🔥 Check if ACL is supported

```bash
mount | grep acl
```

If not enabled (older systems):

```
defaults,acl
```

must be in `/etc/fstab`.

Most modern distros already support it.

---

# 🔥 3.3.1 — Viewing ACLs

```bash
getfacl filename
```

Example output:

```
# file: report.log
# owner: root
# group: admin
user::rw-
user:bob:r--
group::r--
group:devops:rwx
mask::rwx
other::---
```

This file has **extra permissions beyond chmod**:

* default owner = root
* devops group has full rwx
* bob has read-only

This is impossible with only chmod.

---

# 🔥 3.3.2 — Setting ACL Permissions

### Give user specific permission:

```bash
setfacl -m u:bob:rwx file.txt
```

### Give group specific permission:

```bash
setfacl -m g:devops:rw file.txt
```

### Give multiple permissions at once:

```bash
setfacl -m u:alice:rw,u:bob:r-- file.txt
```

### Remove ACL permission:

```bash
setfacl -x u:bob file.txt
```

### Remove ALL ACL:

```bash
setfacl -b file.txt
```

Equivalent to resetting to normal chmod only.

---

# 🔥 3.3.3 — ACL on directories (mass inheritance)

To make ACL auto apply to **new files created inside a directory**:

```bash
setfacl -R -m u:john:rwx /shared/data
setfacl -R -m g:devops:r-x /shared/data
```

But to enforce inheritance:

```bash
setfacl -d -m u:john:rwx /shared/data
setfacl -d -m g:devops:r-x /shared/data
```

`-d` = default permissions for new files created inside.

This is how enterprise shared workspaces are built.

---

# ⚡ Real World Use Cases (Serious Production Examples)

---

### 🔥 1) Allow Jenkins to write into `/var/www` without changing ownership

```bash
setfacl -m u:jenkins:rwx /var/www
```

---

### 🔥 2) Allow application (`appuser`) read-only on secrets

```bash
setfacl -m u:appuser:r-- /etc/secret/token.txt
```

No need to chown or make readable to world.

---

### 🔥 3) Multiple teams access same logs differently

```bash
setfacl -m g:devops:rw /logs
setfacl -m g:security:r-- /logs
```

Different access level → same resource.

---

### 🔥 4) Kubernetes PV mounted into pod but app runs as non-root

```bash
setfacl -R -m u:1000:rwx /data
```

UIDs matter more than usernames in containers.

---

### 🔥 5) Production migration trick

Remember your case: millions of files, chown too slow?

Instead:

```bash
setfacl -R -m u:newuser:rwx /data
```

Fast, non-recursive inode rewrite.
Old data remains — new user gains access immediately.

No metadata explosion like chown.

---

# ⚠ ACL Complexity Warning

ACL gives power — but power increases risk if not documented.

Problems that appear in real production:

| Issue                                    | Cause                |
| ---------------------------------------- | -------------------- |
| User has permission but chmod shows none | ACL overriding       |
| Permission mismatch debugging is harder  | More layers          |
| Backup/restore sometimes loses ACL       | Need ACL aware tools |

Check both when debugging:

```bash
ls -l
getfacl file
```

Never trust chmod alone after ACL enabled.

---

# 🧠 Summary — You Now Control More Than chmod

| Feature                 | chmod (UGO) | ACL      |
| ----------------------- | ----------- | -------- |
| one owner               | ✔           | ✔        |
| one group               | ✔           | ✔        |
| multiple users          | ❌           | ✔        |
| multiple groups         | ❌           | ✔        |
| fine-grained access     | ❌           | ✔        |
| inheritance on subfiles | ❌           | ✔ (`-d`) |

ACL is an enterprise-grade access model.
This is what real sysadmins use — not only chmod.

---

and we continue.
