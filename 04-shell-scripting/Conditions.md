# 📘 **CHAPTER 03 — DECISION MAKING IN BASH (MASTER-LEVEL)**

*How the shell evaluates truth, tests conditions, branches logic, and controls flow — from fundamentals → internals → real-world engineering.*

---

# 🌑 **3.1 — What Does “Condition” Mean in Bash?**

Most modern languages (Python, Java) think this way:

> **A condition is a boolean expression.
> True/False.**

But Bash is not a modern language.

Bash is an **interface to Unix**, and Unix is older than boolean logic in programming languages.

So in Bash:

> **A condition is NOT “true/false”.
> A condition is “exit status is 0 or not 0”.**

Everything in Unix returns a number when it finishes.

This is the **exit code**:

* `0` → success → TRUE
* non-zero → failure → FALSE

Example:

```bash
grep "ERROR" logfile
echo $?   # exit code
```

If “ERROR” is found → exit 0
If not found → exit 1

So in Bash:

```bash
if grep "ERROR" logfile; then
```

The **condition is simply a command**, not a boolean.

This idea will shape every example you see from now.

It’s the **root of Bash logic**.

---

# ☑ **3.2 — Why `[ ]` Exists (Historical Background)**

`[ ]` is not syntax.
It is literally a **program**.

Try this:

```bash
which [
```

You will see:

```
/usr/bin/[
```

It’s actually a command that:

* takes arguments
* evaluates them
* returns exit code

The last character must be `]`, which is why syntax looks weird:

```bash
[ "$a" = "$b" ]
```

This is basically:

```
test "$a" = "$b"
```

So the real reason `[ ]` exists is:

> **Old Unix did not have structured syntax.
> Everything was done using commands.**

This explains all its limitations:

* strict quoting rules
* ugly syntax
* no regex
* no patterns
* unescaped spaces break it
* variables must be quoted

---

# 🔷 **3.3 — Why `[[ ]]` Was Created (Modern Bash)**

`[[ ]]` was introduced by Bash to fix all the problems of `[ ]`.

Unlike `[ ]`:

> **`[[ ]]` is part of the Bash language, not an external command.**

This gives MANY advantages:

* No need to quote variables
* Safer with empty values
* No globbing issues
* Built-in pattern matching
* Built-in regex
* Operators behave consistently
* Doesn’t do word-splitting or pathname expansion

For example:

### Using `[ ]` (old):

```bash
if [ $name = manoj ]; then
```

If `$name` is empty → syntax error.

### Using `[[ ]]` (modern):

```bash
if [[ $name = manoj ]]; then
```

This will NEVER break.

This is why:

### **In serious scripts → ALWAYS use `[[ ]]` unless POSIX compatibility is needed.**

---

# 🔥 **3.4 — Understanding String Comparisons Deeply**

When you compare strings:

### Inside `[ ]` (confusing, brittle):

```bash
[ "$a" = "$b" ]
```

### Inside `[[ ]]` (recommended):

```bash
[[ $a == $b ]]
```

Why does this matter?

Because string comparison includes:

* ASCII order
* whitespace sensitivity
* variable expansion
* wildcard expansion (in `[ ]`)
* word splitting

Let’s break that.

---

## 🔍 **3.4.1 — The Subtle Difference Between `=` and `==`**

In `[[ ]]` both mean the same.

But in `[ ]`:

* `=` is standard
* `==` is sometimes unsupported

Example:

```bash
[[ $name == m* ]]
```

matches patterns.

Inside `[ ]`, this becomes dangerous because wildcards expand to filenames.

---

## 🔍 **3.4.2 — Comparing Empty Strings**

The classic bug beginners hit:

```bash
if [ $name = "manoj" ]; then
```

If `$name` is empty:

```
[ = manoj ]
```

This is invalid syntax → error.

Correct:

```bash
if [[ $name = "manoj" ]]; then
```

OR universally correct:

```bash
if [ "$name" = "manoj" ]; then
```

---

## 🔍 **3.4.3 — Checking empty or non-empty**

```bash
[[ -z $var ]]   # empty
[[ -n $var ]]   # not empty
```

Bash does **not** trim whitespace.
So `" "` is not empty.

---

# 🔢 **3.5 — Numeric Comparison (Why You Can’t Use < and >)**

In Bash, the `<` symbol *redirects files*.

So:

```bash
if [ $x < 5 ]; then
```

is not a comparison.
It redirects the file “5” into the command and completely breaks.

The correct numeric operators:

| Operator | Meaning          |
| -------- | ---------------- |
| `-eq`    | equal            |
| `-ne`    | not equal        |
| `-lt`    | less than        |
| `-le`    | less or equal    |
| `-gt`    | greater          |
| `-ge`    | greater or equal |

Example:

```bash
if [[ $age -ge 18 ]]; then
```

### Bash does *silent* type coercion

If `age="hello"`:

```bash
[[ $age -ge 18 ]]
```

Bash treats `"hello"` as **0**.
No error.

This is why validation is important.

---

# 📁 **3.6 — File Conditions (THE SRE SECTION)**

These are the backbone of DevOps/Infra scripting.

When Bash evaluates:

```bash
[[ -f /etc/shadow ]]
```

Here is what really happens:

1. Bash checks the inode of `/etc/shadow`
2. Determines file type
3. Checks permission bits
4. Returns 0 (true) or 1 (false)

This is a direct interface into the **Unix filesystem API**.

### Most important operators:

| Flag      | Means                     |
| --------- | ------------------------- |
| `-e file` | exists                    |
| `-f file` | regular file              |
| `-d file` | directory                 |
| `-L file` | symlink                   |
| `-s file` | size > 0                  |
| `-r file` | readable                  |
| `-w file` | writable                  |
| `-x file` | executable                |
| `-O file` | owned by current UID      |
| `-G file` | group matches current GID |

Example: check directory emptiness:

```bash
if [[ -z $(ls -A "$dir") ]]; then
    echo "Directory empty"
fi
```

### SRE Real Example:

Check if log rotation created today’s file:

```bash
if [[ -f /var/log/app/log-$(date +%F).gz ]]; then
```

---

# 🔗 **3.7 — Logical Operators (Short-Circuit Logic)**

Bash evaluates conditions left → right.

### AND:

```bash
if [[ $age -ge 18 && $city == "Chennai" ]]; then
```

If left side is false → right side NOT evaluated.

### OR:

```bash
if [[ $role == root || $role == admin ]]; then
```

If left side is true → second condition skipped.

### NOT:

```bash
if [[ ! -d /opt/mysql ]]; then
```

---

# 🧬 **3.8 — Regex Matching (The Strongest Feature of `[[ ]]`)**

Pattern matching with regex:

```bash
[[ $email =~ ^[A-Za-z0-9._%+-]+@gmail\.com$ ]]
```

Key rules:

### 1. Do NOT quote regex.

```bash
[[ $x =~ "^[0-9]+$" ]]   # WRONG
```

### 2. Variables on left must be quoted.

```bash
[[ "$name" =~ ^m.*j$ ]]
```

### 3. `BASH_REMATCH[]` stores groups.

---

# 🌀 **3.9 — Case Statement (Shell’s Switch-Case)**

Used heavily in:

* CLI tools
* systemd scripts
* service controllers

Pattern matching:

```bash
case $action in
  start|restart)
      do_start
      ;;
  stop)
      do_stop
      ;;
  *)
      echo "Usage: $0 {start|stop}"
      ;;
esac
```

Patterns are NOT regex.
They are shell globs (`*`, `?`, []).

---

# 🛠 **3.10 — Commands as Conditions (Real DevOps Power)**

Because every command returns exit status, you can do:

```bash
if pgrep nginx; then
    echo "Nginx running"
fi
```

Check if website reachable:

```bash
if curl -Is google.com >/dev/null; then
```

Check if S3 bucket is reachable:

```bash
if aws s3 ls s3://mybucket >/dev/null 2>&1; then
```

Check if port is open:

```bash
if nc -zv localhost 5432; then
```

This is REAL SRE scripting.

---

# 🎯 **3.11 — Production-Grade Examples (Deep)**

---

## 🔥 Example: Safe Backup

```bash
if tar -czf backup.tar.gz /data; then
    echo "Backup OK"
else
    echo "Backup failed"
    exit 1
fi
```

---

## 🔥 Example: Detect Disk Full

```bash
if df / | awk 'NR==2 {exit ($5 > 85)}'; then
    echo "Disk OK"
else
    echo "Disk CRITICAL"
fi
```

---

## 🔥 Example: Validate Kubernetes Node Ready

```bash
if kubectl get nodes | grep -w Ready >/dev/null; then
    echo "Cluster healthy"
fi
```
---
