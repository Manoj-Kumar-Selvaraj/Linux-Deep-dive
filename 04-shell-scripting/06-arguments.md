# 📘 **CHAPTER 06 — ARGUMENTS (SUPER CLEAR + DETAILED)**

This chapter explains:

* What `$1`, `$2` mean
* What `$#`, `$?`, `$@`, `$*` really mean
* What `shift` does
* How to write scripts that accept flags (`-f`, `-v`, `--file`)
* How to use `getopts` like real CLI tools

Let’s begin.

---

# 🌟 **1. What Are Arguments in Bash?**

When you run a script:

```bash
./deploy.sh production v1.2
```

These are **arguments**.

Bash automatically assigns them to special variables:

| Symbol | Meaning                          |
| ------ | -------------------------------- |
| `$0`   | script name                      |
| `$1`   | first argument                   |
| `$2`   | second argument                  |
| `$3`   | third argument                   |
| ...    | ...                              |
| `$#`   | number of arguments              |
| `$@`   | all arguments (as list)          |
| `$*`   | all arguments (as single string) |

---

# 🌟 **2. Simple Example**

Script: **greet.sh**

```bash
#!/bin/bash
echo "Hello $1"
echo "Your age is $2"
```

Run:

```bash
./greet.sh Manoj 26
```

Output:

```
Hello Manoj
Your age is 26
```

Very simple.

---

# 🌟 **3. `$0`, `$#`, `$@`, `$*` — Must Understand**

### ✔ `$0` — Script name

```bash
echo $0
```

If script is `/home/manoj/run.sh`, it prints that.

---

### ✔ `$#` — Number of arguments

```bash
echo "Count: $#"
```

Example:

```
./run.sh a b c
```

Output:

```
Count: 3
```

---

### ✔ `$@` — List of arguments (best version)

Use this to loop safely over arguments.

```bash
for arg in "$@"
do
    echo "Argument: $arg"
done
```

Double quotes are **required** to preserve spaces.

---

### ✔ `$*` — All arguments collapsed into one string

Rarely used.

Example:

```bash
echo "$*"
```

If args are:

```
one  two  "three four"
```

Output (bad):

```
one two three four
```

It BREAKS multi-word arguments.

Use `$@` always.

---

# 🌟 **4. How to Check Required Arguments**

Example: script needs 2 arguments.

```bash
if [[ $# -lt 2 ]]; then
    echo "Usage: $0 <name> <age>"
    exit 1
fi
```

---

# 🌟 **5. The SHIFT Command (VERY IMPORTANT)**

`shift` removes the **first** argument and shifts everything left.

### Example:

Args:

```
$1 = a
$2 = b
$3 = c
```

After:

```bash
shift
```

Now:

```
$1 = b
$2 = c
```

Useful when you process arguments one by one.

---

# 🌟 **6. Loop Through All Arguments**

```bash
for arg in "$@"
do
    echo "$arg"
done
```

If running:

```
./script.sh apple banana orange
```

Output:

```
apple
banana
orange
```

---

# 🌟 **7. Handling Flags (like -f, -v, --file)**

Let’s create a simple flag parser.

Example script:

```bash
while [[ $# -gt 0 ]]
do
    case "$1" in
        -f|--file)
            file=$2
            shift 2
            ;;
        -v|--verbose)
            verbose=true
            shift
            ;;
        *)
            echo "Unknown option: $1"
            exit 1
            ;;
    esac
done
```

Usage:

```bash
./script.sh -f data.txt -v
```

Now:

```
file="data.txt"
verbose=true
```

This is EXACTLY how real CLI tools work in Linux.

---

# 🌟 **8. REAL PRO METHOD — getopts (best way to parse flags)**

This is how `docker`, `kubectl`, `git`, etc parse flags.

Syntax:

```bash
while getopts "f:v" opt; do
    case "$opt" in
        f) file="$OPTARG" ;;
        v) verbose=true ;;
        ?) echo "Unknown option" ;;
    esac
done
```

Run:

```
./script.sh -f app.conf -v
```

Variables:

```
file="app.conf"
verbose=true
```

---

# 🌟 **9. Example: Professional Argument Parsing**

```bash
#!/bin/bash

usage() {
    echo "Usage: $0 -e <env> -v <version>"
    exit 1
}

while getopts "e:v:" opt; do
    case "$opt" in
        e) env="$OPTARG" ;;
        v) version="$OPTARG" ;;
        *) usage ;;
    esac
done

if [[ -z "$env" || -z "$version" ]]; then
    usage
fi

echo "Deploying version $version to $env"
```

---

# 🌟 **10. Real SRE Example — Script That Accepts Commands (start/stop/restart)**

```bash
case "$1" in
    start)
        systemctl start nginx
        ;;
    stop)
        systemctl stop nginx
        ;;
    restart)
        systemctl restart nginx
        ;;
    *)
        echo "Usage: $0 {start|stop|restart}"
        exit 1
        ;;
esac
```

Run:

```
./nginx.sh start
./nginx.sh restart
```

---

# 🌟 **11. Perfect Template for Argument Handling (Production-Ready)**

```bash
#!/bin/bash

set -e

while [[ $# -gt 0 ]]; do
    case "$1" in
        -e|--env)
            ENV=$2
            shift 2
            ;;
        -v|--version)
            VERSION=$2
            shift 2
            ;;
        -f|--force)
            FORCE=true
            shift
            ;;
        *)
            echo "Unknown arg: $1"
            exit 1
            ;;
    esac
done

echo "ENV=$ENV"
echo "VERSION=$VERSION"
echo "FORCE=$FORCE"
```

---

# 🌟 **12. Summary (Very Clear)**

### ✔ `$1`, `$2` → arguments

### ✔ `$@` → all arguments (best)

### ✔ `$*` → all arguments (avoid)

### ✔ `$#` → number of arguments

### ✔ `shift` → remove processed arguments

### ✔ `case` → handle commands & flags

### ✔ `getopts` → professional flag parsing

This chapter is enough to write:

* CLI tools
* automation scripts
* deployment utilities
* monitoring scripts
* wrapper tools around kubectl, aws cli, docker, etc.

---
