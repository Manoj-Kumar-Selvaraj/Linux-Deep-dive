# 📖 **2.3 — Paths: Absolute vs Relative**

When you work inside Linux, you’re not just typing commands —
you are *navigating a universe of directories*.
To move correctly, you must understand **paths**,
because paths are the coordinates of this universe.

Most people think:

> `/home/user` is a folder.

A professional understands:

> `/home/user` is a **path location inside a hierarchical tree**,
> resolved relative to *current working directory or root*.

Let’s break this down deeper.

---

## 🔥 What is a Path?

A path is a string that tells the filesystem *where* something is.

There are two types:

### 1) **Absolute Path**

Starts from root `/`
Always leads to the same location.

Example:

```
/etc/ssh/sshd_config
/usr/local/bin/python3
/root/scripts/cleanup.sh
```

No matter where you are, these paths are valid.

Think of absolute path like GPS longitude/latitude — precise, global.

---

### 2) **Relative Path**

Depends on where you currently are → the **context matters**.

Example:

```
cd logs       # works only if you're in the right directory
../config     # go up one level and enter config
../../bin     # go up two levels and into bin
```

If your current directory changes, relative path resolves differently.

Relative path is like saying:

> “Turn right at the next street”
> It only makes sense if you know where you're standing.

---

## 🧭 Understanding CWD — Current Working Directory

Your shell always has a *current directory* context.

Check it anytime:

```bash
pwd
```

This is your anchor.
Every *relative* command is calculated based on this.

Example:

```
# If you are inside: /home/user/project
ls src      → means /home/user/project/src
cd ../      → goes to /home/user
cd ../../   → goes to /home
```

---

## 📌 `.` and `..` — The Most Important Navigation Symbols

| Symbol | Meaning           |
| ------ | ----------------- |
| `.`    | current directory |
| `..`   | parent directory  |

These are not shortcuts — they are filesystem entries that exist *inside every directory*.

Example structure:

```
/home/user/project/
    ├── src
    ├── config
    ├── data
```

Now try this mentally:

```
cd config
pwd                 → /home/user/project/config
cd ..               → /home/user/project
cd ../..            → /home/user
cd ../../..         → /
```

This is how senior engineers move without thinking —
their brain *visualizes* directory topography.

---

## 🏎 Speed Navigation Like a Pro (Real Examples)

Instead of:

```
cd /home/user/project/src
```

You can use:

```
cd ~/project/src
```

`~` always means **current user home directory**.

Or even:

```bash
cd !$    # repeats last argument (advanced shell trick)
```

Or jump back instantly:

```bash
cd -     # return to previous directory
```

This is how advanced users navigate in seconds, not minutes.

---

## 🧠 Real-World Scenario

You're debugging logs in `/var/log/app`, but config is in `/etc/app`.

Normal admin:

```
cd /
cd etc/app
# wrong
```

Experienced engineer:

```
cd /etc/app     # absolute path — always correct
```

Or from `/var/log/app`:

```
cd ../../etc/app   # relative — fast if mentally mapped
```

Knowledge of paths **eliminates guesswork**.

---

## 🔥 Summary — What you now understand deeply

| Concept       | You now know                                |
| ------------- | ------------------------------------------- |
| Absolute path | Always starts at `/` — globally reliable    |
| Relative path | Based on current working directory          |
| `pwd`         | shows where you currently exist in the tree |
| `.` `..`      | special entries for navigation              |
| `~`           | home expansion shortcut                     |

You don’t type paths anymore —
you navigate them like a map.

---


