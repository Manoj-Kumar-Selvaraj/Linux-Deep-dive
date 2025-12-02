# 📘 **CHAPTER 04 — LOOPS (THE COMPLETE INTERNALS GUIDE)**

> “If you understand Bash loops deeply, you understand Unix philosophy.”

This chapter explains loops **from first principles**.

---

# 🌑 **4.1 — First Principles: How Bash Sees a Loop**

Before learning “for” or “while”, understand this core truth:

### ✔ **A Bash loop is not a control structure like in other languages.**

### ✔ **A Bash loop is a text-expansion engine.**

When you write:

```bash
for x in a b c; do echo $x; done
```

Bash does the following:

1. **Parse the loop** into tokens:
   `{for} {x} {in} {a} {b} {c} {; do} … {done}`

2. **Perform expansions** (variables, globs, command substitution)

3. **Produce a *final list of words*** that will be iterated over

4. For each word, Bash:

   * creates or overwrites the loop variable
   * runs the body
   * returns control to loop head

This is why loops break when:

* words contain spaces
* words contain wildcards
* globs expand into filenames
* command output splits incorrectly
* subshells lose variables

A “for loop” is simply:

> **Run a block of code for each word that appears after expansions.**

That’s it.

The rest of the chapter builds from this truth.

---

# 🔥 **4.2 — The FOR Loop: The Original Unix Iterator**

Not like Python/Javascript.

In Bash:

* FOR loops **do not operate on arrays inherently**
* They operate on **WORDS**
* “Words” are determined **after expansions and splitting**

Let’s go deeper.

---

# 🌟 **4.2.1 — Step-by-Step Internals of a FOR Loop**

Take:

```bash
for item in $LIST; do echo "$item"; done
```

Here’s what actually happens:

### **STEP 1 — Variable expansion**

Suppose:

```bash
LIST="file 1.txt file 2.txt"
```

After expansion:

```
for item in file 1.txt file 2.txt
```

### **STEP 2 — Word splitting**

Bash splits on IFS (default: spaces, tabs, newlines):

* `file`
* `1.txt`
* `file`
* `2.txt`

Your loop becomes:

```
file
1.txt
file
2.txt
```

THIS is why beginners break filenames with spaces.

In contrast, arrays do NOT split like this.

---

# 🌟 **4.2.2 — Why Globs Expand Before the Loop**

Example:

```bash
for f in *.log; do echo "$f"; done
```

**Globbing (`*.log`) expands BEFORE the loop begins.**

If your directory contains:

```
app.log
system.log
error.log
```

Your loop becomes:

```
for f in app.log system.log error.log; do
```

No wildcards remain inside the loop.

Huge difference compared to Python/Java.

---

# 🌟 **4.2.3 — The “No Match” Trap (Unknown to Beginners)**

If `*.log` matches nothing, behavior depends on shell:

* Bash (default): leaves literal `*.log`
* Bash (`shopt -s nullglob`): expands to nothing
* zsh: prints error unless configured

This matters in automation.

---

# 🌟 **4.2.4 — Why FOR loops fail on command output (The REAL reason)**

Example:

```bash
for user in $(cat users.txt); do echo "$user"; done
```

Why is this **wrong**?

### Reason 1 — **Word splitting**

`$(cat users.txt)` expands to:

```
user1 user2 user3   (spaces separate words)
```

Lines with spaces break into multiple words.

### Reason 2 — **Filename expansion**

If a line contains `*`, it expands into files.

### Reason 3 — **Newlines collapse into whitespace**

Unix processes streams, not structured data.

### Therefore:

> **NEVER use FOR for reading files.**

We use `while read` loops instead.

---

# 🔥 **4.3 — The WHILE Loop: The Streaming Loop**

While loops are fundamentally different:

```bash
while COMMAND; do
    body
done
```

The loop runs **as long as COMMAND returns exit code 0.**

### Example:

```bash
while ping -c1 google.com > /dev/null; do
    echo "online"
    sleep 3
done
```

Here:

* If ping succeeds → exit code 0 → loop continues
* If network goes down → exit code != 0 → loop stops

This style is used heavily in SRE scripts.

---

# 🌟 **4.3.1 — Why WHILE Is Used for File Reading**

Correct:

```bash
while IFS= read -r line; do
    echo "line: $line"
done < file.txt
```

Explanation:

* `IFS=` → disables whitespace trimming
* `-r` → prevents backslash escaping
* `< file.txt` → file is fed into stdin

Difference from FOR loop:

* While reads **exactly line-by-line**
* Preserves whitespace
* No splitting unless you request it
* Can safely process logs, CSVs, JSON lines

This is why DevOps engineers **ALWAYS** use while-read loops for files.

---

# 🔥 **4.4 — The Subshell Trap: A Concept EVERY Engineer Must Understand**

This is the most confusing behavior in Bash.

### What is a subshell?

When you use a pipe:

```bash
echo hello | while read line; do
    var="set"
done
```

The `while` loop runs in a **subshell**.

Any changes inside:

* variables
* arrays
* environment

do NOT affect the parent shell.

Thus:

```bash
echo "$var"
```

prints **empty**.

### Why does this happen?

Because Unix pipes create **child processes** to handle each side of the pipe.

Bash cannot magically synchronize variables between processes.

### Correct ways to avoid this:

Method 1: Process substitution

```bash
while read -r line; do
    var="set"
done < <(echo "hello")
```

Method 2: Redirect input

```bash
while read -r line; do
    var="set"
done <<< "hello"
```

Method 3: Avoid pipes entirely

```bash
while read -r line; do
    var="set"
done < file.txt
```

This is senior-level shell engineering.

---

# 🔥 **4.5 — UNTIL Loop: Inverted While**

Opposite logic:

```bash
until COMMAND; do
    # runs while COMMAND is failing
done
```

### Real SRE use case:

Wait for MySQL to come up:

```bash
until mysql -h db -e "select 1" >/dev/null 2>&1; do
    echo "Waiting for DB..."
    sleep 2
done
```

This loop continues until the command **SUCCEEDS**.

---

# 🔥 **4.6 — Looping Over Files (The Safe Way — not taught in tutorials)**

Beginners:

```bash
for f in $(ls *.txt); do
```

Wrong because:

* ls output splits
* filenames break
* newlines compress
* unsafe with spaces

Correct:

```bash
for f in *.txt; do
```

Even safer (nullglob):

```bash
shopt -s nullglob
for f in *.txt; do
```

Safest: handle special chars:

```bash
find . -type f -print0 |
while IFS= read -r -d '' file; do
    echo "$file"
done
```

This handles filenames with:

* spaces
* tabs
* newlines
* quotes
* unicode
* emojis

This is how production scripts work.

---

# 🔥 **4.7 — C-Style FOR Loops: The Fastest Loop in Bash**

```bash
for ((i=1; i<=100000; i++)); do
    echo $i
done
```

Why is this faster?

1. No expansion
2. No word splitting
3. No filenames
4. Pure integer increment built inside Bash
5. Zero external processes

This is the fastest loop type and used for:

* large numeric iterations
* timed loops
* performance testing

---

# 🔥 **4.8 — Internal Behavior of Bash Loops (Hardcore Section)**

When a Bash loop runs:

### ✔ Variable assignment happens **before** body

Bash writes to the variable name you chose.

### ✔ The loop body may run in a subshell if:

* piping into the loop
* using process substitution incorrectly
* using parentheses `()` inside the loop

### ✔ Loop bodies do NOT generate a new process

Unless:

* using pipelines
* using external commands
* using $(command substitution)

### ✔ Loops tightly integrate with stdin

While loops often attach to stdin, unlike for loops.

---

# 🔥 **4.9 — Performance Deep Dive**

These loop patterns:

```bash
for word in $(cat file)
```

* spawns 1 external `cat`
* performs word splitting
* loses structure

This pattern:

```bash
while read line; do ... done < file
```

* zero external processes
* extremely fast
* handles huge files

C-style loops:

* fastest
* ideal for counters

FOR loops over globs:

* fast
* expands once
* safe for filenames

FOR loops over command substitution:

* slow
* unsafe
* should be avoided

---

# 🔥 **4.10 — Real PRODUCTION SRE Examples (Deep)**

---

## 🟦 Example 1 — Monitor a port until it opens

```bash
until nc -z localhost 5432; do
    echo "Waiting for PostgreSQL..."
    sleep 1
done
```

This loop uses network exit codes — true Unix style.

---

## 🟦 Example 2 — Process logs line-by-line

```bash
while IFS= read -r line; do
    [[ "$line" =~ ERROR ]] && echo "ALERT: $line"
done < /var/log/app.log
```

This is log monitoring without `tail`.

---

## 🟦 Example 3 — Restart service if it dies

```bash
while true; do
    if ! pgrep nginx >/dev/null; then
        echo "Restarting nginx..."
        systemctl restart nginx
    fi
    sleep 2
done
```

This is the idea behind simple watchdogs.

---

## 🟦 Example 4 — Process substitutions for parallel reading

```bash
while read -r line1 && read -r line2; do
    echo "$line1 | $line2"
done < <(paste file1 file2)
```

Bash’s internal data plumbing is incredibly deep.

---

