### 📖 **2.4 — Navigation (cd, ls, pwd, tree… commands in depth)**

Most beginners learn navigation as:

> *cd, ls, pwd = move, list, print.*

But professionals treat navigation as an **art**,
because filesystem traversal speed directly controls how fast they debug, read logs, modify configurations, and solve outages.

This chapter teaches you **navigation like a Linux master**, not like a user.

---

# 🔥 2.4.1 — `pwd` → Where Am I *Really*?

`pwd` prints the **current working directory** → your position inside the filesystem.

```bash
pwd
```

Example output:

```
/home/ubuntu/app/logs
```

But there is more:

| Command  | Meaning                                      |
| -------- | -------------------------------------------- |
| `pwd`    | prints logical path                          |
| `pwd -P` | prints **physical** path (resolves symlinks) |

Example:

```
/var/www/html -> /mnt/data/site
```

If you run:

```bash
cd /var/www/html
pwd       → /var/www/html      (logical)
pwd -P    → /mnt/data/site     (physical!)
```

Knowing this difference is important for debugging **symlink-mounted PVs** in Kubernetes and EKS.

---

# 🔥 2.4.2 — `cd` — The Most Underestimated Command

Basic:

```bash
cd /path/to/dir
```

But **masters use cd as a weapon**, not as a movement tool.

### Super-short navigation patterns

| Command    | Jumps to                         |
| ---------- | -------------------------------- |
| `cd`       | home (~)                         |
| `cd ~user` | another user’s directory         |
| `cd -`     | previous directory (toggle back) |
| `cd ..`    | parent directory                 |
| `cd ../..` | two levels up                    |

```bash
# Example toggle
cd /var/log
cd -
# returns to last location instantly
```

This is extremely useful during rapid log/config switching.

---

### Relative path navigation = fastest navigation humanly possible

Scenario:

```
/home/user/app/src/logs
```

To go to `config`, instead of absolute path:

```
cd ../../../config       # Fast mental movement
```

Speed = seniority.

---

### `pushd` and `popd` → directory stack navigation

This is navigation like **tabs in a browser**.

```bash
pushd /etc/nginx
pushd /var/log/nginx
pushd /usr/share/nginx/html
dirs                     # shows stack of locations
popd                     # returns one step back
```

You move like teleporting between folders.

Even many Linux professionals don’t know this trick.

---

# 🔥 2.4.3 — `ls` — This Is Where Skill Gaps Appear

Everyone uses `ls`, but few *master* it.

### Basic:

```bash
ls
ls -l        # long format
ls -a        # show hidden files
```

### Pro-Level Listing

| Command    | Meaning                |
| ---------- | ---------------------- |
| `ls -lh`   | human readable sizes   |
| `ls -ltr`  | reverse sorted by time |
| `ls -S`    | sorted by size         |
| `ls -d */` | list directories only  |
| `ls -i`    | show inode numbers     |
| `ls -R`    | recursive list         |
| `ls -1`    | one entry per line     |

Example:

```bash
ls -lhS /var/log        # find largest logs immediately
```

Used during storage issues **daily**.

---

# 🔥 2.4.4 — `tree` — See Filesystem as a Visual Map

Install:

```bash
sudo apt install tree   # Debian/Ubuntu
sudo yum install tree   # RHEL/Amazon
```

Use:

```bash
tree
tree -L 2        # limit depth
tree -a          # include hidden
tree -d          # show directories only
```

Example:

```
project/
├── src
│   ├── api
│   ├── utils
│   └── core
├── config
└── logs
```

This is powerful for:

✔ exploring microservice repos
✔ navigating Kubernetes config directories
✔ understanding unfamiliar codebases in seconds

---

# 🚀 2.4.5 — Combine Navigation Commands (Real Situations)

### Example 1 — Find and open config instantly

```bash
cd /etc && ls -lh | grep ssh && cd ssh
```

### Example 2 — Jump between work folders instantly

```bash
cd /var/log && cd - && cd /etc/nginx && cd -
```

### Example 3 — View tree, list, search all at once

```bash
tree -L 2; echo "\nLargest files:"; ls -lhS | head
```

SRE speed comes from combining movement + visibility.

You are building that.

---

# 🔥 Summary

| You Know Now   | Why It Matters                           |
| -------------- | ---------------------------------------- |
| pwd vs pwd -P  | Debug symlinks & mounted volumes         |
| cd shortcuts   | Move between directories at insane speed |
| pushd/popd     | Stack-based navigation like browser tabs |
| ls power flags | Sort, search, filter files efficiently   |
| tree           | Map directory structures visually        |
| combined flows | You move like a senior engineer          |

Navigation is not typing
Navigation is *thinking in file topology*.

You no longer walk — you *teleport*.

---

# 🧠 Think of `cd` vs `pushd` like this:

### `cd` = You walk somewhere.

### `pushd` = You walk somewhere AND leave a breadcrumb to come back easily.

### `popd` = Follow the breadcrumb back.

---

Let's pretend your home directory is your **house**.
You want to go visit **3 places**:

1. Grocery store
2. Library
3. Coffee shop

If you use **cd**, the journey looks like this:

```
cd grocery-store
cd library
cd coffee-shop
```

But now you want to go back home.
You have to walk the whole path again manually — because you didn't save your route.

---

### Now enter pushd/popd.

When you use `pushd`, Linux says:

> "I will go there AND remember this place."

Let’s see clearly.

---

# 🏠 Start at Home

```
$ pwd
/home/user
```

### Go to grocery store using `pushd`

```
pushd /grocery
```

Linux remembers this path.

Route memory now:

```
/grocery  /home/user
```

---

### Now go to library

```
pushd /library
```

Route memory:

```
/library  /grocery  /home/user
```

---

### Now go to coffee shop

```
pushd /coffee
```

Route stack:

```
/coffee  /library  /grocery  /home/user
```

---

# 🔥 Now the magic

### Instead of manually typing paths like:

```
cd /library
cd /grocery
cd /home/user
```

You simply use **popd**:

```
popd   → takes you to /library
popd   → takes you to /grocery
popd   → takes you home
```

One command = travel back in order.

---

# Visual Summary

```
pushd /grocery        📌 stores route
pushd /library        📌 stores route
pushd /coffee         📌 stores route

popd  → coffee → library
popd  → library → grocery
popd  → grocery → home
```

---

# 🔥 Quick Mental Trick That Will Make You Get It

### `pushd` = bookmark location & go there

### `popd`  = jump back to last bookmark

### `dirs`  = see bookmarks

Try this EXACTLY in terminal:

```
cd ~
pushd /etc
pushd /var/log
pushd /usr/bin
dirs
popd
popd
popd
```

You will watch yourself jump back like **undo navigation.**

---
