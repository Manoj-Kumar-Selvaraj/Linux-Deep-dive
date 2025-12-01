

# 📖 **2.5 — File Creation, Deletion & Linking**

In Linux, a *file is not just a name*.
It is **inode + data blocks**, and operations like `cp`, `mv`, `rm` behave differently depending on filesystem structure, inode allocation, and link counts.

This chapter covers:

1. Creating files
2. Modifying files
3. Deleting files (what actually happens)
4. Hard links & soft links (real-world use)
5. File movement vs copy (inode behavior)
6. Safe-delete methods
7. Common mistakes that destroy servers

---

# 🔥 2.5.1 — Creating Files

### Method 1 — using `touch` (empty file)

```bash
touch notes.txt
```

Creates inode entry, size = 0 bytes.
No data blocks allocated yet.

### Method 2 — using redirect operator

```bash
echo "Hello" > file.txt
```

This creates file **and writes data**.

Overwrites if exists.

Append instead:

```bash
echo "New line" >> file.txt
```

### Method 3 — using `cat`

```bash
cat > todo.txt
Write anything here...
CTRL+D to save
```

---

# 🔥 2.5.2 — Copying Files (`cp`)

```bash
cp file1 file2
```

Creates a new inode → duplicates data.

For directories:

```bash
cp -r source/ dest/
```

Verbose (see every file):

```bash
cp -rv source/ dest/
```

---

# 🔥 2.5.3 — Moving Files (`mv`)

Important:

| Operation                  | What happens                  |
| -------------------------- | ----------------------------- |
| mv inside same filesystem  | Instant → inode pointer moves |
| mv to different filesystem | Data copied block-by-block    |

Example:

```bash
mv /data/file /backup/file   # instant if same FS
```

---

# 🔥 2.5.4 — Deleting Files (`rm`)

```bash
rm file.txt
```

🚫 Does *not* delete file data
✔ Only removes filename → inode reference count drops

File is deleted only when:

```
Link count = 0  
AND no process has file open
```

Which is why you can recover deleted logs if app still has it open.

---

# 🔥 2.5.5 — Hard Links vs Soft Links

| Hard Link                               | Soft Link                       |
| --------------------------------------- | ------------------------------- |
| Points to inode directly                | Points to path (name)           |
| File continues even if original deleted | Breaks if original removed      |
| Must be same filesystem                 | Can cross filesystem boundaries |

### Create hard link

```bash
ln file1 file2
```

Both filenames → same inode.

Rename/delete one → other survives.

### Create soft link (symlink)

```bash
ln -s /path/original shortcut
```

Works like Windows shortcut.

---

# 🔥 2.5.6 — Real-World Link Usage

| Scenario                   | Use                                      |
| -------------------------- | ---------------------------------------- |
| Avoid duplicate storage    | Hard links for large datasets            |
| Config version control     | `/etc/nginx/sites-enabled` → symlinks    |
| Shared binaries            | `/bin/sh` is symlink to `bash` or `dash` |
| Repoint storage seamlessly | Change symlink → zero downtime switch    |

Example zero-downtime switch:

```bash
ln -s app_v2 app
mv -Tf app_v2 app    # replace atomically, no downtime
```

---

# 🔥 2.5.7 — Safe Delete Techniques (Very Important)

To delete safely:

```bash
rm -i file       # interactive confirmation
rm -rI dir       # ask before deep delete
trash file       # sends to recoverable trash (if enabled)
```

Worst command in Linux:

```
rm -rf /
rm -rf *     # in wrong directory = disaster
```

Always check:

```bash
pwd
ls
```

Before you pull the trigger.

---

## Summary

You now understand file operations deeply — not as commands, but as **inode-based transformations**.

✔ touch creates inode only
✔ echo writes without editor
✔ cp duplicates data, new inode
✔ mv may be instant if same FS
✔ rm removes name — inode may survive
✔ hard links = multiple names → one inode
✔ soft links = path shortcut
✔ safe-delete is a survival skill

You are not operating Linux —
You’re controlling it.

---
