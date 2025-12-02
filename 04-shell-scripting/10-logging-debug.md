Alright — now we enter one of the **most important professional chapters** in all of shell scripting.
This is the chapter that separates beginner-level scripts from **production-grade, reliable, debuggable DevOps/SRE scripts.**

This is a **deep, slow, book-style** explanation.

# 📘 **CHAPTER 10 — LOGGING & DEBUGGING IN SHELL SCRIPTS**

*(Professional, production-level, SRE-grade chapter)*

In Linux scripting, **your real enemy is NOT syntax… it's debugging.**
If you know how to debug properly, you can fix any broken script in minutes.

This chapter covers:

* Understanding exit codes
* Safe script modes (`set -e`, `set -u`, `set -o pipefail`)
* Debug mode (`set -x`)
* Logging correctly to files
* Logging levels (INFO, WARN, ERROR)
* `trap` for catching failures
* Timestamped logs
* Color output
* Silent vs verbose mode
* Debugging pipes, subshells, loops

Let’s go step-by-step.

---

# -------------------------------------------------------

# 🟦 **10.1 Exit Codes (The Foundation of Debugging)**

# -------------------------------------------------------

Every command in Linux returns an **exit code**, also called *return status*.

| Code      | Meaning          |
| --------- | ---------------- |
| **0**     | Success          |
| **1-255** | Something failed |

Check last exit code:

```bash
echo $?
```

Example:

```bash
ls /root
echo $?
```

If `ls /root` is denied → exit code = **2**.

---

### Why exit codes matter?

Because every feature of debugging depends on them:

* `if` conditions
* loops
* error traps
* retry logic
* cron debugging
* monitoring scripts

Example:

```bash
if command; then
    echo OK
else
    echo FAIL
fi
```

The `if` block checks the exit code.

---

# -------------------------------------------------------

# 🟦 **10.2 The Single Most Important Debugging Tool: set -x**

# -------------------------------------------------------

`set -x` turns on **debug mode**.

### What does it do?

It prints every command *before* executing it, with values expanded.

Example script:

```bash
#!/bin/bash
set -x

name="manoj"
echo "Hello $name"
ls /notfound
```

Output:

```
+ name=manoj
+ echo Hello manoj
Hello manoj
+ ls /notfound
ls: cannot access '/notfound': No such file or directory
```

This instantly tells you:

* what commands executed
* what variables expanded to
* which command failed

To disable:

```bash
set +x
```

---

# -------------------------------------------------------

# 🟦 **10.3 Safe Scripting — The Holy Trinity**

# -------------------------------------------------------

This is how real production scripts start:

```bash
#!/bin/bash
set -euo pipefail
```

Let’s break each:

---

### ✔ `set -e`

Stop script **immediately** if any command fails.

Without `set -e`:

```
command1 fails → script STILL continues
```

With `set -e`:

```
command1 fails → script STOPPED
```

Prevents disasters:

* deleting wrong file
* half-running deployments
* inconsistent states

---

### ✔ `set -u`

Stop script if you use an **undefined variable**.

Example:

```bash
echo $username   # Oops, misspelled!
```

With `set -u` → script stops with error.
This prevents 99% silent failures.

---

### ✔ `set -o pipefail`

Pipes fail if *ANY* command fails.

Example:

```bash
cat file | grep something | awk '{print $1}'
```

Without pipefail:

* grep may fail
* awk still runs
* script “looks” successful

With pipefail:

* ANY pipe failure stops the chain
* debugging becomes correct

---

# -------------------------------------------------------

# 🟦 **10.4 Logging in Shell Scripts (Deep and Clear)**

# -------------------------------------------------------

Cron jobs and backend scripts MUST log correctly.

Let’s build a professional logging system.

---

## 🌟 **10.4.1 Basic Logging Functions**

```bash
log() {
    echo "[$(date +'%F %T')] $*"
}
```

Usage:

```bash
log "Starting backup..."
```

Output:

```
[2025-12-02 15:36:10] Starting backup...
```

---

## 🌟 **10.4.2 Logging Levels**

```bash
info()  { echo "[INFO]  $(date +'%F %T') $*"; }
warn()  { echo "[WARN]  $(date +'%F %T') $*"; }
error() { echo "[ERROR] $(date +'%F %T') $*" >&2; }
```

Usage:

```bash
info  "Deployment started"
warn  "Retrying..."
error "Service is DOWN"
```

Errors go to stderr because of `>&2`.

---

## 🌟 **10.4.3 Logging to file**

Append both stdout + stderr:

```bash
/opt/myscript.sh >> /var/log/myscript.log 2>&1
```

---

## 🌟 **10.4.4 Real Production Logging Function**

```bash
logfile="/var/log/myscript.log"

log() {
    printf "[%s] %s\n" "$(date +'%F %T')" "$*" | tee -a "$logfile"
}
```

Runs:

* prints to console
* logs to file

---

# -------------------------------------------------------

# 🟦 **10.5 Debugging Why Cron Jobs Fail (Very Important)**

# -------------------------------------------------------

Cron has ZERO output unless you capture logs.

Use:

```
* * * * * /opt/script.sh >> /var/log/script.log 2>&1
```

Debug cron environment:

```
env > /tmp/env.txt
```

Common cron issues:

* PATH is different
* script not executable
* wrong shebang
* missing variables
* using relative paths instead of absolute paths
* script expecting interactive shell

---

# -------------------------------------------------------

# 🟦 **10.6 The Powerful `trap` Command (Catching Errors)**

# -------------------------------------------------------

`trap` lets you catch:

* EXIT
* ERR
* INT (Ctrl+C)
* TERM (kill)

### Example: Run function when script exits

```bash
cleanup() {
    echo "Cleaning temporary files..."
}

trap cleanup EXIT
```

---

### Example: Catch errors anywhere in script

```bash
trap 'echo "ERROR at line $LINENO"' ERR
```

If any command fails → prints line number.

---

### Example: Cleanup temp files when script fails

```bash
tmpfile=$(mktemp)

trap "rm -f $tmpfile" EXIT
```

This ensures cleanup always happens.

---

# -------------------------------------------------------

# 🟦 **10.7 Timing & Performance Debugging**

# -------------------------------------------------------

Measure command time:

```bash
start=$(date +%s)

# commands here

end=$(date +%s)
echo "Time taken: $((end-start)) seconds"
```

Or:

```bash
time command
```

---

# -------------------------------------------------------

# 🟦 **10.8 Verbose Mode**

# -------------------------------------------------------

Enable with flag:

```bash
#!/bin/bash

verbose=false

while getopts "v" opt; do
    [[ $opt == "v" ]] && verbose=true
done

log() {
    $verbose && echo "$*"
}
```

Run with:

```
script.sh -v
```

---

# -------------------------------------------------------

# 🟦 **10.9 Debugging Pipes & Subshells**

# -------------------------------------------------------

### Problem:

```bash
cat file | while read line; do
    var="x"
done

echo $var   # empty!
```

Because pipe creates **subshell**.

### Solution:

```bash
while read line; do
    var="x"
done < file
```

---

# -------------------------------------------------------

# 🟦 **10.10 Debugging Signals & Script Crashes**

# -------------------------------------------------------

Catch TERM/KILL:

```bash
trap 'echo "Script killed at $(date)" >> /var/log/script.log' TERM
```

Catch Ctrl+C:

```bash
trap 'echo "Interrupted"; exit' INT
```

---

# -------------------------------------------------------

# 🟦 **10.11 Production-Level Debug Template**

# -------------------------------------------------------

This is how REAL SRE scripts look:

```bash
#!/bin/bash
set -euo pipefail

logfile="/var/log/myscript.log"

log() {
    printf "[%s] %s\n" "$(date +'%F %T')" "$*" | tee -a "$logfile"
}

trap 'log "ERROR at line $LINENO"; exit 1' ERR
trap 'log "Script interrupted"; exit 2' INT

log "Starting script"

# your code

log "Finished successfully"
```

This script:

* cannot silently fail
* logs timestamps
* catches errors
* shows line numbers
* logs interruptions

This is **enterprise-grade**.

---

# ⭐ SUMMARY (ABSOLUTE MASTERY OF DEBUGGING)

You learned:

### ✔ Exit codes

### ✔ Debug mode (`set -x`)

### ✔ Strict mode (`set -euo pipefail`)

### ✔ Logging levels

### ✔ Logging to files

### ✔ Debugging cron

### ✔ Using traps

### ✔ Timing commands

### ✔ Handling subshell issues

### ✔ Production debugging templates

This chapter makes you **10× better** as a DevOps or SRE engineer.

---