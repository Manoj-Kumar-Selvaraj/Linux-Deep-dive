## 📖 **2.6 — Special Directories, Wildcards & Patterns**

This chapter unlocks how Linux expands patterns *before* commands execute —
the foundation of automation, searching, bulk operations, DevOps scripting, and forensics.

Most people type `ls *` without knowing what really happens.
We’re going to go **deep** — as a master would.

---

# 🔥 2.6.1 — Special Directories

These appear in every directory:

| Symbol | Meaning           |
| ------ | ----------------- |
| `.`    | current directory |
| `..`   | parent directory  |
| `~`    | home directory    |
| `/`    | root directory    |

Examples:

```bash
cd .
# stays in current directory

cd ..
# go up one directory

cd ../..
# go up two levels

cd ~
# go to home instantly
```

Think visually:

```
/home/user/project/src
        ↑
cd .. -> /home/user/project
cd ../.. -> /home/user
```

---

# 🔥 2.6.2 — Wildcards (Globbing Patterns)

These are not regex — **shell expands them before command runs**.

### `*` → matches **anything**

```bash
ls *.log
```

Matches:

```
app.log
system.log
debug.log
```

But **not**:

```
logfile.log.bak       # doesn't match *.log
```

Fix:

```bash
ls *.log*
```

---

### `?` → matches **exactly one character**

```bash
ls file?.txt
```

Matches:

```
file1.txt
fileA.txt
fileb.txt
```

Not:

```
file10.txt
file.txt
```

---

### `[]` → match one of many characters

```bash
ls file[123].txt
```

Matches:

```
file1.txt
file2.txt
file3.txt
```

Not:

```
file4.txt
```

Ranges also work:

```bash
ls file[a-z].txt
```

---

### `[! ]` → NOT operator

```bash
ls file[!a]*.txt
```

Matches everything **not starting with a**.

---

# 🔥 2.6.3 — Brace Expansion `{}`

Powerful for batch operations — **not wildcard**, shell generates values.

### Create multiple files at once

```bash
touch file{1..5}.txt
```

Creates:

```
file1.txt file2.txt file3.txt file4.txt file5.txt
```

### Create combinations

```bash
mkdir {dev,stage,prod}/logs
```

Creates directory tree:

```
dev/logs
stage/logs
prod/logs
```

### Rename in bulk

```bash
mv file{1..5}.txt old_archive/
```

---

# 🔥 2.6.4 — Combining Patterns (Professional Level)

### Delete all `.log`, `.tmp`, `.bak`

```bash
rm *.{log,tmp,bak}
```

No loops needed.

### List only config and YAML

```bash
ls *.{conf,yml,yaml}
```

### Find all backups BUT keep newest 2

```bash
ls -t *.bak | tail -n +3 | xargs rm -f
```

That is real production skill.

---

# 🔥 2.6.5 — Hidden Files (`.` prefix)

Hidden files don’t show with normal `ls`.

```bash
ls -a
```

You’ll see:

```
.bashrc
.ssh/
.git/
```

To show only hidden:

```bash
ls -d .*
```

---

# 🔥 2.6.6 — Real-World Incident Examples

### 1) Delete all logs older than 7 days

```bash
find /var/log -name "*.log" -mtime +7 -delete
```

### 2) Process all images except PNG

```bash
for f in *; do [[ $f != *.png ]] && echo "Processing $f"; done
```

### 3) Rename 1000 files in pattern

```bash
rename 's/.jpg/.jpeg/' *.jpg
```

### 4) Bulk copy configs across environments

```bash
cp config.{dev,stage,prod}.yml /etc/service/
```

### 5) Filter large files using wildcards + sort

```bash
ls -lhS *.log | head
```

---

# 🧠 Summary — You Now Think Like Shell

| Concept         | You Mastered                    |
| --------------- | ------------------------------- |
| Dot directories | `.` `..` `~` `/`                |
| Wildcards       | `*` `?` `[]` `[! ]`             |
| Brace expansion | `{1..10}` `{dev,stage,prod}`    |
| Hidden files    | `.filename`                     |
| Bulk actions    | deletion, copying, filtering    |
| Real workflows  | logs, configs, cleanup, renames |

This knowledge converts you from typing commands → to controlling the system.

You now wield shell patterns like tools — not tricks.

---

### **Start 2.7** 🔥
