# 🧠 **PART 1 — What *Exactly* Happens When You Run a Command?**

(EVERY DevOps/SRE must understand this.)

You type:

```
ls -l
```

Your shell (`bash`, `zsh`, `sh`) performs **8 steps**, not 1.

---

## **Step 1 — Shell Parses the Command**

It splits `ls -l` into:

```
program = ls
arg1 = -l
```

Shell does:

* variable expansion
* wildcard expansion
* alias expansion
* PATH search for ls

The shell does **NOT** run ls directly.
It prepares metadata internally.

---

## **Step 2 — Shell Calls fork()**

`fork()` is a system call.

It creates a **child process** which is *exact copy* of shell’s memory.

Child inherits:

* environment variables
* file descriptors (stdin, stdout, stderr)
* working directory
* resource limits
* user ID

After fork:

```
Parent → still running your shell
Child  → copy of your shell
```

---

## **Step 3 — Child Process Calls exec()**

The child process **replaces itself** with the `ls` program.

After exec():

* CHILD is no longer a shell
* CHILD *becomes* the LS program

This is a complete memory replacement.

---

## **Step 4 — LS Program Starts Executing**

The kernel loads:

* LS binary from `/usr/bin/ls`
* LS data segments
* LS dynamic libraries (`libc.so`)

Kernel sets:

* program counter
* stack
* heap
* registers

Now the LS process is alive with its own PID.

---

## **Step 5 — Kernel Schedules the LS Process**

The Linux scheduler decides which CPU core runs ls.

Linux uses **CFS (Completely Fair Scheduler)**.

Scheduler considerations:

* process priority (nice value)
* CPU usage history
* I/O waiting
* cgroup limits
* fairness scheduling

---

## **Step 6 — LS Writes Output to STDOUT**

`ls` doesn't write to your screen directly.
It writes to **stdout**, a file descriptor:

```
stdout → your terminal
```

Terminal prints it.

---

## **Step 7 — LS Exits**

`ls` returns exit code (0 for success).

The child process dies, but NOT completely.

---

## **Step 8 — Parent Shell Calls wait()**

The shell must call `wait()` to collect the exit code.

If shell **does not** call wait() → child becomes a **ZOMBIE**.

---

# 🧠 **PART 2 — Understanding PIDs, PPIDs & Process Tree**

Every process has:

### PID

Process ID.

### PPID

Parent Process ID.

Example:

```bash
ps -o pid,ppid,cmd
```

Output:

```
PID   PPID   CMD
1010  1000   bash
1055  1010   ls
```

ls → child
bash → parent

---

# 🧠 **PART 3 — How the Linux Scheduler Works Internally**

Linux uses **CFS**.

### Key ideas:

### ✔ Each process gets “virtual runtime”

CFS tries to ensure each process gets equal share of CPU.

### ✔ Lower nice value = higher priority

Nice range:

```
-20   highest priority
+19   lowest priority
```

If a process uses too much CPU, CFS penalizes it.

---

# 🧠 **PART 4 — Signals (True Internals)**

Signals are **software interrupts** sent from:

* kernel
* another process
* terminal
* system calls
* OOM killer

List:

```bash
kill -l
```

### Important signals:

| Signal       | Meaning                    |
| ------------ | -------------------------- |
| SIGTERM (15) | Polite stop                |
| SIGKILL (9)  | Force kill (non-catchable) |
| SIGHUP (1)   | Reload config / hang-up    |
| SIGINT (2)   | Ctrl+C                     |
| SIGSTOP (19) | Pause process              |
| SIGCONT (18) | Resume paused process      |
| SIGCHLD      | Child terminated           |
| SIGQUIT      | Crash + core dump          |

---

### 🟣 SIGTERM vs SIGKILL

Most critical difference:

### SIGTERM (safe)

* process can clean up
* close DB connections
* save files
* shutdown gracefully

### SIGKILL (unsafe)

* process dies instantly
* cannot clean up
* can corrupt files
* can break DB connections
* cannot be trapped

Use SIGKILL ONLY when forced.

---

# 🧠 **PART 5 — Job Control (The REAL Explanation)**

Job control is **not kernel-level**, it is **shell-level**.

When you run:

```
ping google.com
```

Press:

```
Ctrl + Z
```

The shell sends **SIGSTOP** to pause job.

To resume in background:

```
bg
```

To bring to foreground:

```
fg
```

Shell internally tracks jobs like this:

```
Job ID → Process Group ID → PID list
```

Job control is why:

* multiple commands can pause
* background jobs run
* terminal can control multiple processes

---

# 🧠 **PART 6 — Zombie Processes (TRUE DEEP)**

A zombie process is:

> “A child that finished but the parent didn’t read its exit code.”

Its entry stays in process table:

```
Z+  <defunct>
```

### Why zombies exist?

Because kernel MUST keep:

* PID
* exit code
* signals

until parent collects them.

### How to fix zombies properly?

1. Identify parent:

```
ps -o pid,ppid,stat,cmd | grep Z
```

2. Restart parent:

```
systemctl restart <service>
```

3. If parent is stuck → kill parent.
   The zombie will be adopted by `systemd` which will reap it.

### Killing the zombie directly doesn’t work.

WHY?

Because zombie is already **dead**.

---

# 🧠 **PART 7 — Orphan Processes**

If parent dies before child, example:

```
Parent: bash
Child: python script
```

You exit the terminal — bash exits → python becomes orphan.

Linux adopts it:

```
systemd → new parent
```

`systemd` automatically reaps it later.

---

# 🧠 **PART 8 — Context Switching (Advanced)**

Linux switches between processes millions of times per second.

Switching involves:

* saving CPU registers
* loading new registers
* updating memory mappings
* updating scheduler queues

Context switching is expensive when:

* too many processes
* too many threads
* lots of I/O waiting

DevOps/SRE must monitor:

```
vmstat 1
```

If **cs (context switch)** column is too high → system is overloaded.

---

# 🧠 **PART 9 — File Descriptors & Processes**

Every process gets:

| FD | Purpose |
| -- | ------- |
| 0  | stdin   |
| 1  | stdout  |
| 2  | stderr  |

Other descriptors open for:

* files
* sockets
* logs
* pipes

To inspect:

```bash
ls -l /proc/<PID>/fd
```

This shows what files process is using.

---

# 🧠 **PART 10 — Real SRE/DevOps OS-Level Internals**

### ✔ Find process holding a file open

```bash
lsof | grep /var/log/app.log
```

---

### ✔ Find process hogging CPU

```bash
ps -eo pid,cmd,%cpu --sort=-%cpu | head
```

---

### ✔ See how processes are connected

```bash
pstree -ap
```

---

### ✔ Debug why process is stuck

```bash
strace -p <PID>
```

---

### ✔ See all threads inside a process

```bash
ps -T -p <PID>
```

---

# 🧠 **PART 11 — Processes & Systemd**

Every systemd service is a process:

```
systemd
 ├─ sshd
 ├─ nginx
 ├─ docker
 └─ systemd-logind
```

To see cgroup scope:

```bash
systemd-cgls
```

---