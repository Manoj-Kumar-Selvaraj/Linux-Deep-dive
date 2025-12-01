
---

# 🔥 2.7.1 — What Does Mounting Mean in Linux?

A **mount** connects a filesystem (disk, EBS volume, USB, NFS, etc) to a directory.

Before mounting, a block device (disk) is just a **raw binary blob.**

Example disk:

```
/dev/sdb
```

Mount it:

```bash
sudo mount /dev/sdb1 /mnt/data
```

Now content is accessible at:

```
/mnt/data
```

📌 **The directory `/mnt/data` is just a gateway — the real data is inside the device.**

Unmounting:

```bash
sudo umount /mnt/data
```

Directory remains — but now it’s empty.
Because filesystem isn't attached anymore.

---

# 🔥 2.7.2 — `/dev` → Devices as Files

Linux treats hardware as files.

| File           | Represents                   |
| -------------- | ---------------------------- |
| `/dev/sda`     | entire disk                  |
| `/dev/sda1`    | disk partition               |
| `/dev/nvme0n1` | NVMe SSD                     |
| `/dev/ttyX`    | terminal devices             |
| `/dev/zero`    | infinite stream of zeros     |
| `/dev/null`    | a black hole (discard input) |

Example "black hole" delete:

```bash
echo "hello" > /dev/null
```

Data vanishes. No storage used.

Write zeros to disk:

```bash
dd if=/dev/zero of=/dev/sda bs=1M   # ⚠ DANGEROUS
```

This wipes your entire disk in seconds.

Understanding /dev means understanding raw storage power.

---

# 🔥 2.7.3 — `/proc` → The Kernel’s Real-Time Memory as Files

`/proc` is not a disk — it's a **virtual filesystem** exposing kernel internals.

Examples:

| File            | Shows                   |
| --------------- | ----------------------- |
| `/proc/cpuinfo` | CPU model/core threads  |
| `/proc/meminfo` | Memory usage            |
| `/proc/<pid>`   | Per-process info        |
| `/proc/uptime`  | How long system running |

Try:

```bash
cat /proc/cpuinfo
cat /proc/meminfo
```

These files don’t exist on disk.
Reading them asks the kernel for live values **in real time**.

---

# 🔥 2.7.4 — `/sys` → Hardware & Kernel Tuning

`/sys` exposes kernel tunable parameters such as CPU scaling, block device settings, power, network.

Example: View CPU states

```bash
ls /sys/devices/system/cpu/
```

Change CPU frequency governor:

```bash
echo performance | sudo tee /sys/devices/system/cpu/cpu0/cpufreq/scaling_governor
```

You just reconfigured CPU behavior by writing to a **file.**

That is Linux power.

---

# 🔥 2.7.5 — tmpfs — Filesystem Stored in RAM

tmpfs = temporary filesystem stored **in memory (RAM)**.

Mounted as:

```
/dev/shm
/run
/tmp  (sometimes)
```

Fastest I/O on the system.

Example mount:

```bash
sudo mount -t tmpfs -o size=2G tmpfs /mnt/ramdisk
```

You now have a **2GB RAM drive** — ideal for caching, builds, AI memory pipes.

Data disappears on reboot.

---

# 🔥 2.7.6 — Bind Mounts (Very Useful & Underused)

Bind mounts map one directory to another.

```bash
sudo mount --bind /data/logs /var/log/app
```

Now `/var/log/app` shows content from `/data/logs` without copying anything.

Used for:

* redirecting logs
* Kubernetes hostPath mounting
* filesystem mirror mapping
* migration without downtime

---

# 🔥 2.7.7 — Mounting in Docker / Kubernetes

Inside containers, mounts come from **volumes or hostPath or PVCs**.

K8s PVC → PV → backend storage (EBS, EFS, FSx...)

Example PVC → Pod:

```yaml
volumeMounts:
- name: data-storage
  mountPath: /app/data
volumes:
- name: data-storage
  persistentVolumeClaim:
    claimName: app-pvc
```

Inside the pod:

```
/app/data  ===  filesystem mounted from PVC
```

Understanding mounting is crucial to troubleshooting PV issues.

---

# Summary Table

| Directory     | Purpose                                 |
| ------------- | --------------------------------------- |
| `/dev`        | Devices as files (disks, terminals)     |
| `/proc`       | Live kernel information                 |
| `/sys`        | Hardware + kernel tuning interface      |
| tmpfs         | RAM-backed filesystem (fast, temporary) |
| bind mount    | Map one dir → another                   |
| mount/unmount | Attach/detach filesystems               |

You moved from filesystem navigation → to **storage architecture mastery.**

This completes **Volume II.**

---

Totally okay that it’s not clicking yet — this stuff *feels* like magic until you see it very slowly, step by step.

Let’s **forget all 9 magics for now** and go deep into just **3** of them, in a very simple way:

1. RAM disk (tmpfs)
2. Bind mount (move a folder without copying)
3. Mounting an EBS (or any disk) to see files

If these 3 become clear, the rest will be much easier.

---

## 1️⃣ RAM Disk (tmpfs) – “Folder that actually lives in RAM”

### ✦ Idea in simple words

Normally, files are stored on **disk** (slow but permanent).
With `tmpfs`, we create a **folder whose contents are stored in RAM** (very fast, but lost on reboot).

Think of it like:

> “Make `/mnt/fast` behave like a pendrive made out of RAM.”

### ✦ Step-by-step

#### 1. Create an empty folder

```bash
sudo mkdir -p /mnt/ramdisk
```

Right now, it’s just a normal folder on your disk.

#### 2. Mount a tmpfs there (RAM-backed filesystem)

```bash
sudo mount -t tmpfs -o size=1G tmpfs /mnt/ramdisk
```

Meaning:

* `-t tmpfs` → type is tmpfs (RAM-based FS)
* `size=1G` → allow up to 1 GB of RAM for files
* `/mnt/ramdisk` → where it is visible in the filesystem

Now, `/mnt/ramdisk` is like a **1GB ram-drive**.

#### 3. Check it

```bash
df -h /mnt/ramdisk
```

You’ll see something like:

```
tmpfs   1.0G   0   1.0G   /mnt/ramdisk
```

#### 4. Use it

```bash
cd /mnt/ramdisk
echo "hello" > test.txt
ls
cat test.txt
```

This file is stored in **RAM**, not disk. Super fast, but temporary.

#### 5. Unmount / remove it

```bash
cd /
sudo umount /mnt/ramdisk
```

Now `/mnt/ramdisk` is back to being a normal, empty folder.
Anything that was inside (while tmpfs was mounted) is gone.

> That’s why we use it for **temporary** work: builds, caches, etc.

---

## 2️⃣ Bind Mount – “Show same folder in two places, without copying”

This is the one that helps with things like:

* “My `/var/log` is on a small disk. I want to write logs somewhere else **without changing app config**.”

### ✦ Idea in simple words

You have:

* Real data in `/data/logs`
* App expects logs in `/var/log/app`

Instead of copying files, you can say:

> “Make `/var/log/app` actually show content from `/data/logs`.”

That’s a **bind mount**.

### ✦ Step-by-step

#### 1. Create a real folder with data

```bash
sudo mkdir -p /data/logs
echo "Real log" | sudo tee /data/logs/app.log
```

#### 2. Create target mount point

```bash
sudo mkdir -p /var/log/app
```

Right now:

* `/data/logs/app.log` exists
* `/var/log/app` is empty

Check:

```bash
ls -l /data/logs
ls -l /var/log/app
```

#### 3. Bind mount

```bash
sudo mount --bind /data/logs /var/log/app
```

Now, if you do:

```bash
ls -l /var/log/app
```

You’ll see:

```bash
app.log
```

But that file physically lives in `/data/logs`.
`/var/log/app` is just another **view** into that same directory.

Your app can keep writing to `/var/log/app/app.log` and data really goes into `/data/logs/app.log`.

#### 4. Verify it’s the same

```bash
echo "New line" | sudo tee -a /var/log/app/app.log
cat /data/logs/app.log
```

You’ll see both lines.

#### 5. Unmount bind

```bash
sudo umount /var/log/app
```

Now `/var/log/app` becomes an empty folder again,
but `/data/logs/app.log` still has all your content.

> So: **bind mount = two doors to the same room.**

---

## 3️⃣ Mount EBS/Disk to Read Data (like PV recovery)

This is close to your EKS/PV world.

When you have an EBS volume (or any disk like `/dev/nvme1n1`), Linux doesn’t show its files until you **mount** it somewhere.

### ✦ Idea

> “Take disk `/dev/nvme1n1p1` and attach its filesystem into a folder `/mnt/recovery` so I can see its files.”

### ✦ Step-by-step

> (I’ll assume EBS is already attached to EC2 and appears as `/dev/nvme1n1p1`)

#### 1. Check what block devices exist

```bash
lsblk
```

You’ll see something like:

```text
nvme0n1    100G  disk
└─nvme0n1p1 100G part  /
nvme1n1     20G  disk
└─nvme1n1p1 20G  part
```

Here `/dev/nvme1n1p1` is your extra disk (maybe PV data).

#### 2. Create a mount point

```bash
sudo mkdir -p /mnt/recovery
```

#### 3. Mount it

```bash
sudo mount /dev/nvme1n1p1 /mnt/recovery
```

Now, list:

```bash
ls -l /mnt/recovery
```

You’ll see whatever was inside that disk (maybe `/var/lib/mysql`, `/data`, etc).

#### 4. Copy what you want

```bash
cp -r /mnt/recovery /home/ubuntu/backup-from-pv
```

You just **recovered PV data** onto a safe path.

#### 5. Unmount when done

```bash
sudo umount /mnt/recovery
```

> So: **mount** = “plug this disk into that folder so I can read it”.

---

## Tiny recap in one table

| Concept      | Command example                        | What it really does                               |
| ------------ | -------------------------------------- | ------------------------------------------------- |
| tmpfs        | `mount -t tmpfs tmpfs /mnt/ramdisk`    | Creates a **RAM-based folder** (fast, temporary)  |
| bind mount   | `mount --bind /data/logs /var/log/app` | Makes **two paths show same files**               |
| normal mount | `mount /dev/nvme1n1p1 /mnt/recovery`   | Makes a **disk’s filesystem visible** at a folder |

