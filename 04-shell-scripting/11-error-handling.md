Great — now we enter **one of the most critical chapters** for becoming a *real* DevOps / SRE / Platform Engineer:

This chapter is NOT about “if exit code != 0”…
It's about building **resilient**, **self-healing**, **fault-tolerant** shell scripts used in production systems.

This is another **BOOK-STYLE**, **deep**, **clear**, **practical** chapter.

---

# 📘 **CHAPTER 11 — ERROR HANDLING (PRODUCTION-GRADE)**

This chapter includes:

* Understanding failure modes
* Graceful vs hard failures
* Retry logic
* Backoff algorithms
* Transactional operations
* Cleanup on failure
* Script reliability patterns
* Protecting critical operations
* Error handling for pipes, loops, commands, functions
* Building robust deployment scripts
* Real DevOps/SRE patterns

Let’s go step-by-step.

---

# -----------------------------------------------------------

# 🟦 **11.1 What Does “Error” Really Mean in Bash?**

# -----------------------------------------------------------

In Bash, **an error simply means exit code ≠ 0**.

Example:

```bash
ls /root
echo $?
```

Output:

```
2
```

But here is the **real truth**:

> ❗ Not all failures should stop your script.
> ❗ Not all errors are fatal.
> ❗ Some errors require retries.
> ❗ Some errors require fallback.
> ❗ Some errors MUST stop everything immediately.

This chapter teaches how to handle each case.

---

# -----------------------------------------------------------

# 🟦 **11.2 Strict Mode (Foundation of Error Handling)**

# -----------------------------------------------------------

Every production script must start with:

```bash
set -euo pipefail
```

Let’s break each:

---

## ✔ `set -e` — Exit on ANY failure

If any command fails → script stops immediately.

Example:

```bash
rm file1
rm file2        # if file2 missing → script stops here
rm file3
```

This prevents half-executed operations.

---

## ✔ `set -u` — Fail on undefined variables

Example:

```bash
echo $username   # username NOT defined
```

With `set -u`: script stops
Without `set -u`: script prints empty → BUG

---

## ✔ `set -o pipefail` — Catch pipe errors

Example:

```bash
cat file | grep keyword | awk '{print $2}'
```

If grep fails, awk still runs → BAD
With pipefail → pipe fails properly

---

# -----------------------------------------------------------

# 🟦 **11.3 Catching Failures Using `trap`**

# -----------------------------------------------------------

The `trap` command lets you catch runtime failures and react.

Example:

```bash
trap 'echo "Error at line $LINENO"; exit 1' ERR
```

Now if ANY command fails:

```
Error at line 52
```

You instantly know what broke.

---

## ⭐ Professional failure alert:

```bash
trap 'echo "[ERROR] Failed at $(date) in line $LINENO" | tee -a /var/log/app.log' ERR
```

---

## ⭐ Cleanup on failure (IMPORTANT)

Example:

```bash
tmpfile=$(mktemp)

trap "rm -f $tmpfile" EXIT
```

This guarantees cleanup even if:

* script fails
* user presses Ctrl+C
* system kills script

---

# -----------------------------------------------------------

# 🟦 **11.4 Graceful Error Handling Using IF Conditions**

# -----------------------------------------------------------

Sometimes failure is expected.

Example:

```bash
if ! systemctl is-active --quiet nginx; then
    echo "Nginx is down"
fi
```

This does NOT stop the script — you choose what to do.

---

# -----------------------------------------------------------

# 🟦 **11.5 Retry Logic (With or Without Backoff)**

# -----------------------------------------------------------

Real world needs retries:

* network is slow
* database not ready
* API timeout
* S3 upload failed
* k8s pod still starting

Let’s build retry mechanisms.

---

## ⭐ Basic Retry Loop

```bash
for i in {1..5}; do
    command && break
    echo "Attempt $i failed"
    sleep 2
done
```

---

## ⭐ Retry with exit code if ALL attempts fail

```bash
max=5

for i in $(seq 1 $max); do
    if command; then
        echo "Success on attempt $i"
        exit 0
    fi
    echo "Attempt $i failed"
    sleep 2
done

echo "All attempts failed"
exit 1
```

---

## ⭐ Exponential Backoff (Used in AWS, K8s, Google, Netflix)

```bash
delay=1
max=5

for ((i=1; i<=max; i++)); do
    if command; then
        echo "Success"
        exit 0
    fi
    echo "Fail $i... retrying in $delay seconds"
    sleep $delay
    delay=$((delay*2))
done

exit 1
```

Backoff = 1, 2, 4, 8, 16 seconds.

---

# -----------------------------------------------------------

# 🟦 **11.6 Validation Before Critical Operations**

# -----------------------------------------------------------

Never run dangerous commands without validation.

Example: deleting a directory

```bash
if [[ -d "$target" && "$target" != "/" && "$target" != "" ]]; then
    rm -rf "$target"
else
    echo "Invalid target directory, aborting"
    exit 1
fi
```

Mistakes prevented:

* deleting `/`
* deleting empty variable
* deleting parent folder by accident

---

---

# -----------------------------------------------------------

# 🟦 **11.7 Transactional Operations (Do X only if Y succeeded)**

# -----------------------------------------------------------

Example:

```bash
backup && deploy && restart
```

Equivalent to:

* if backup succeeds → deploy
* if deploy succeeds → restart
* if restart fails → STOP

If ANY fails → entire chain stops.

---

# -----------------------------------------------------------

# 🟦 **11.8 Error-Proof File and Directory Handling**

# -----------------------------------------------------------

### Check file exists:

```bash
[[ -f "$file" ]] || { echo "File missing"; exit 1; }
```

### Check directory exists:

```bash
[[ -d "$dir" ]] || mkdir -p "$dir"
```

### Check disk space:

```bash
avail=$(df / | awk 'NR==2 {print $4}')

if (( avail < 100000 )); then
    echo "Low disk space"
    exit 1
fi
```

---

# -----------------------------------------------------------

# 🟦 **11.9 Function Error Handling**

# -----------------------------------------------------------

Inside functions:

```bash
download() {
    curl -f -O "$1" || return 1
}
```

In script:

```bash
if ! download "file.txt"; then
    echo "Download failed"
    exit 1
fi
```

---

# -----------------------------------------------------------

# 🟦 **11.10 Fail-Fast Production Template**

# -----------------------------------------------------------

Every professional script starts like this:

```bash
#!/bin/bash
set -euo pipefail
trap 'echo "Error at line $LINENO"; exit 1' ERR
```

This provides:

* safety
* debugging
* no silent failures

---

# -----------------------------------------------------------

# 🟦 **11.11 Real DevOps / SRE Examples**

# -----------------------------------------------------------

---

## ✔ Restart service if failed

```bash
if ! systemctl restart nginx; then
    echo "Restart failed, rolling back..."
    cp /backup/nginx.conf /etc/nginx/nginx.conf
    systemctl restart nginx
fi
```

---

## ✔ Retry S3 upload

```bash
for i in {1..5}; do
    aws s3 cp file.txt s3://bucket && break
    sleep 2
done
```

---

## ✔ Health check with backoff

```bash
attempt=0
max=10

until curl -fs http://localhost:8080/health; do
    ((attempt++))
    [[ $attempt -eq $max ]] && exit 1
    sleep 2
done
```

---

## ✔ Cleanup temp files on script exit

```bash
tmp=$(mktemp)
trap 'rm -f $tmp' EXIT
```

---

## ✔ Kubernetes rollout failure detection

```bash
kubectl rollout status deployment/app || {
    echo "Deployment failed, rolling back"
    kubectl rollout undo deployment/app
    exit 1
}
```

---

# ⭐ **SUMMARY OF CHAPTER 11 (FULL MASTERY)**

You learned:

### ✔ Strict mode: `set -euo pipefail`

### ✔ Using traps to catch errors

### ✔ Writing retry logic

### ✔ Exponential backoff

### ✔ Validation before dangerous actions

### ✔ Cleanup on EXIT

### ✔ Error handling in functions

### ✔ Transactional operations

### ✔ Real DevOps/SRE production examples

You now write scripts that are:

* safer
* resilient
* fault tolerant
* debuggable
* production ready

This is the difference between a junior engineer and an experienced DevOps/SRE.

---