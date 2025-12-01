# 📖 **Chapter 2.1 — The Filesystem Hierarchy**

When you log into Linux, you don’t see icons or folders visually.
You see **a tree of directories** hanging from the root `/`.

Most beginners think of Linux folders like Windows' `C:\`, `D:\`, etc.
But Linux has **no drive letters**, no multiple roots — everything begins at **one root**:

```
/
├── bin
├── etc
├── home
├── usr
└── var
```

This single slash `/` is **the top of the universe**.
Every file, every device, every disk lives under it — somehow.

To truly master Linux, you must know **what each top-level directory represents**.

We don't list; we *understand*.

---

### `/` — The Root of All Things

Think of `/` as the **universe origin point**.
A user can live their entire life below this one directory.

No matter how many disks you mount, how many partitions you create,
everything ultimately attaches under `/`.

There is *nothing* above it.

```
There is no "C drive". 
There is only `/`.
```

---

### `/bin` — **Essential User Commands**

If you boot into single-user recovery mode, `/bin` is one of the few places still mounted.

Programs found here are **lifesaving tools**:

* `ls` (list files)
* `cp` (copy)
* `mv` (move)
* `rm` (remove)
* `cat` (view file contents)
* `bash` (the shell itself!)

If `/bin` is broken, you don’t just lose commands —
you lose **your hands and mouth inside the system**.

---

### `/etc` — **The Brain (Configuration)**

Everything that defines how your system behaves lives here:

* User accounts → `/etc/passwd`
* Hostname → `/etc/hostname`
* Network config → `/etc/network/interfaces` or `/etc/sysconfig`
* Services → `/etc/systemd/system/`
* SSH daemon config → `/etc/ssh/sshd_config`

If `/bin` is your hands, `/etc` is your **memory and personality**.

One line inside `/etc` can change the entire system’s destiny.

---

### `/home` — **Where Humans Live**

Each user gets a personal space here:

```
/home/username/
```

Documents, scripts, downloads, SSH keys —
this is your bedroom in the Linux house.

When `/home` is on a separate disk, you can reinstall the OS **without losing user data**.
That’s why separating `/` and `/home` is a pro-level layout.

---

### `/usr` — **Userland Applications & Libraries**

This is where most software you install will go.

```
/usr/bin      → Applications
/usr/lib      → Libraries
/usr/share    → Shared data
/usr/local/   → Software *you* manually install
```

**Important distinction:**

| `/bin`                 | `/usr/bin`               |
| ---------------------- | ------------------------ |
| Must exist for booting | Optional during repair   |
| Core rescue commands   | Applications & utilities |
| Minimal survival kit   | Full toolbox             |

---

### `/var` — **Everything That Grows Over Time**

Logs expand. Databases grow. Caches fill.

Linux separates these into `/var` so your root filesystem doesn’t explode.

Examples:

```
/var/log      → System logs
/var/cache    → Service caches
/var/lib      → Databases, VM metadata
/var/spool    → Print/mail/job queues
```

If `/var` fills up → server dies.
If `/var/log` alone fills → services crash silently.

Knowing `/var` behavior is how SREs prevent outages.

---

### `/proc` & `/sys` — **Not Files. Portals to the Kernel.**

These are not real directories.

They are **views into the kernel**.

#### `/proc` shows running process & kernel runtime data

```bash
/proc/cpuinfo
/proc/meminfo
/proc/1   # systemd/journald PID tree
```

You can read them like text — because kernel exposes itself as files.

#### `/sys` shows hardware & device information

```bash
/sys/class/net/eth0
/sys/devices/system/cpu
```

You can **change** kernel behavior by *writing* into `/sys` files.

Yes — writing text can change how CPU throttles, how network behaves.

---

### `/dev` — **Hardware as Files**

Linux treats devices as files:

```
/dev/sda  → disk
/dev/tty  → terminal
/dev/null → a black hole
/dev/random → entropy generator
```

This is why tools like:

```bash
dd if=/dev/sda of=/backup.img
```

can clone entire disks like copying a file.

Windows cannot do this elegantly.

Linux can do it with one line.

---

### `/tmp` — **Chaos Playground**

Temporary scratch space.

* Anyone can write to it.
* It may be cleared at reboot.
* Never trust files here long-term.

But masters use it to test ideas without ruining `/home`.

---

## Summary (Not bullet — concept)

You now know the filesystem not as a list to memorize,
but as a **living organism**:

* `/` is root consciousness
* `/bin` is your hands
* `/etc` is memory & personality
* `/home` is where humans sleep
* `/usr` is the software universe
* `/var` is growth, logs, lifeblood
* `/proc` and `/sys` are live access to the kernel
* `/dev` bridges hardware into files

Linux is not folders —
Linux is design, purpose, and clarity.

---
