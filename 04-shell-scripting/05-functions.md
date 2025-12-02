# 📘 **CHAPTER 05 — FUNCTIONS (SUPER CLEAR + DETAILED)**

---

# 🌟 **1. What is a function?**

A function is simply:

> **A group of commands given a name.
> Instead of repeating code, you put it in a function and call it whenever needed.**

Example:

Without function:

```bash
echo "Starting service"
systemctl start nginx

echo "Starting service"
systemctl start nginx
```

With function:

```bash
start_service() {
    echo "Starting service"
    systemctl start nginx
}

start_service
start_service
```

---

# 🌟 **2. How to Define a Function**

There are two valid syntaxes:

### Style 1 (Recommended)

```bash
myfunc() {
    echo "Inside function"
}
```

### Style 2

```bash
function myfunc {
    echo "Inside function"
}
```

Both work.
Most scripts use **Style 1**.

---

# 🌟 **3. Calling a Function**

Just use the function name:

```bash
myfunc
myfunc
```

No brackets needed.
No parentheses (unlike Python).

---

# 🌟 **4. Function Scope (VERY IMPORTANT)**

By default, Bash functions do **not** have their own local scope like Python.

Variables inside a function are **global**, unless we explicitly make them local.

Example:

```bash
x=5

change() {
    x=10
}

change
echo $x   # 10 (changed!)
```

To avoid this, use `local`.

```bash
x=5

change() {
    local x=10
}

change
echo $x   # 5 (not changed)
```

This is VERY important for reliable scripts.

---

# 🌟 **5. Passing Arguments to Functions**

Just like scripts use `$1`, `$2`, etc.,
**functions also use them**.

Example:

```bash
greet() {
    echo "Hello $1"
}

greet Manoj
```

Output:

```
Hello Manoj
```

More arguments:

```bash
add() {
    echo "$(($1 + $2))"
}

add 10 20
```

---

# 🌟 **6. Returning Values (IMPORTANT)**

## ⚠️ Bash function CANNOT return strings using `return`.

Wrong:

```bash
return "hello"  # ERROR
```

### ✔ Correct ways to return a value:

### Method 1: Print the value

```bash
get_date() {
    date +%F
}

today=$(get_date)
echo $today
```

### Method 2: Modify global variable (not recommended)

```bash
x=0

set_value() {
    x=100
}

set_value
echo $x
```

### Method 3: Use `echo` + command substitution (recommended)

```bash
result=$(myfunc)
```

---

# 🌟 **7. Return Codes (Exit Status)**

Every command in bash returns an exit code.

In functions:

```bash
myfunc() {
    command
    return $?   # optional
}
```

We check:

```bash
if myfunc; then
    echo "Success"
else
    echo "Failed"
fi
```

---

# 🌟 **8. Using `local` Variables**

`local` prevents side-effects.

Example:

```bash
counter=0

incr() {
    local counter=10
    ((counter++))
    echo "Inside: $counter"
}

incr
echo "Outside: $counter"
```

Output:

```
Inside: 11
Outside: 0
```

---

# 🌟 **9. Function with Named Parameters (More readable)**

Instead of `$1`, `$2`:

```bash
deploy() {
    local env=$1
    local version=$2

    echo "Deploying version $version to $env"
}

deploy prod v1.2
```

This is the style used in real DevOps scripts.

---

# 🌟 **10. Functions Returning Boolean / Success / Failure**

This is VERY common in SRE scripts.

Example:

```bash
is_running() {
    pgrep nginx >/dev/null
}
```

Usage:

```bash
if is_running; then
    echo "Nginx is running"
else
    echo "Nginx is NOT running"
fi
```

Because `pgrep` returns **0** if found.

---

# 🌟 **11. Functions with Default Parameters**

```bash
greet() {
    local name=${1:-Guest}
    echo "Hello $name"
}

greet Manoj
greet
```

Output:

```
Hello Manoj
Hello Guest
```

---

# 🌟 **12. Functions Calling Other Functions**

```bash
log() {
    echo "[INFO] $1"
}

deploy() {
    log "Deploying app"
    # deployment code here
}

deploy
```

---

# 🌟 **13. Function Libraries (Re-usable Files)**

You can store functions in a file:

**helpers.sh**

```bash
log() {
    echo "[INFO] $1"
}
```

Then include it:

```bash
source helpers.sh
log "Starting app"
```

This is how big Bash projects are organised.

---

# 🌟 **14. Function Indirection (Advanced but simple explanation)**

This allows calling a function whose name is stored in another variable.

```bash
run() {
    echo "Running"
}

func="run"
$func    # calls run()
```

Used for CLI tools with subcommands.

---

# 🌟 **15. Real World DevOps / SRE Examples**

---

## 🟦 Example 1 — Restart service with reusable function

```bash
restart_service() {
    systemctl restart "$1"
    systemctl status "$1" --no-pager
}

restart_service nginx
restart_service mysql
```

---

## 🟦 Example 2 — Robust health check

```bash
check_url() {
    curl -s "$1" | grep -q "200 OK"
}

if check_url https://google.com; then
    echo "OK"
else
    echo "Down"
fi
```

---

## 🟦 Example 3 — Backup function

```bash
backup_dir() {
    local src=$1
    local dest="/backup/$(basename $src)-$(date +%F)"

    cp -r "$src" "$dest"
    echo "Backup created at $dest"
}

backup_dir /var/log
```

---

## 🟦 Example 4 — Function that retries automatically

```bash
retry() {
    local attempts=$1
    shift
    local cmd="$@"

    for ((i=1;i<=attempts;i++)); do
        $cmd && return 0
        echo "Attempt $i failed..."
        sleep 1
    done

    return 1
}

retry 5 curl -s https://google.com
```

---