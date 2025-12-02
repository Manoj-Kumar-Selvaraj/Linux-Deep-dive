# 🧠 **4.2 — Variables, Quotes & Expansions (The Real Brain of Bash)**

Bash is not like Python or Java.
It doesn’t interpret your script line-by-line in a classical programming language way.
Instead, bash **expands**, **transforms**, and **rewrites** your text before executing commands.

If you understand THIS chapter, everything in bash becomes easy.

---

# 📌 **4.2.1 — What Exactly Is a Variable in Bash?**

In bash, **variables are just text**.

There is no type system.

A variable is:

* not an integer
* not a string
* not a list
* not a boolean

Everything is a **string by default**.

Example:

```bash
x=10
y="Hello"
name=manoj
```

These are all **strings**.

If you want bash to treat numbers as integers, you explicitly say so (later in arithmetic expansion).

---

# 📌 **4.2.2 — How Bash Stores Variables**

When you write:

```bash
a=10
```

Bash does NOT allocate memory like C or Python.

It simply stores:

```
key = text
```

Variables live:

* in the shell’s memory
* only for the duration of the session
* unless exported

---

# 📌 **4.2.3 — Shell Variables vs Environment Variables**

### **Shell variable**

Lives only inside that shell:

```bash
MESSAGE="hello"
```

Running a child program like:

```bash
./script.sh
```

Will NOT see MESSAGE.

---

### **Environment variable**

Visible to child processes.

```bash
export MESSAGE="hello"
./script.sh
```

Now the script will see it.

This is why in systemd, Kubernetes, Docker, CI/CD:

* environment variables matter
* shell variables do NOT propagate

---

# 📌 **4.2.4 — How Bash **Expands** Variables**

This is critical.

When you type:

```bash
echo $USER
```

The shell performs **variable expansion**:

* Locates `$USER`
* Replaces it with the stored value
* Only then executes the command

Example:

```
echo $HOME
```

Becomes:

```
echo /home/manoj
```

Commands always get the **expanded version**, never `$HOME`.

---

# 📌 **4.2.5 — Why `${VAR}` Exists (VERY IMPORTANT)**

Consider:

```bash
filename="report"
echo "$filename.txt"
```

Expected:

```
report.txt
```

But bash sees:

```
$filename.txt
```

It tries to expand variable `filename.txt` (not found).

Correct version:

```bash
echo "${filename}.txt"
```

Always use braces `{}` in complex expressions.

---

# 🧨 **4.2.6 — Single Quotes ' ' (Literal Mode)**

Everything inside single quotes is taken **literally**.

```bash
name="Manoj"
echo '$name'
```

Output:

```
$name
```

No expansions.
No variables.
No escapes.

Single quotes are “write exactly what I type”.

---

# ⭐ **4.2.7 — Double Quotes " " (Expansion Mode)**

Everything inside double quotes is:

* variable-expanded
* command-substituted
* escaped characters processed

Example:

```bash
echo "$name"
```

Output:

```
Manoj
```

### Inside double quotes, bash recognizes:

* $VAR
* $(command)
* \n \t escapes
* Wildcards are preserved but not expanded

Use **double quotes 90% of the time**.

---

# 🧠 **4.2.8 — The 4 Expansion Rules of Bash (Critical)**

When you write a line, bash transforms it in this order:

### **1) Brace expansion**

```bash
{A,B,C}
{1..5}
```

### **2) Tilde expansion**

```bash
~
~/scripts
```

### **3) Parameter expansion (Variables)**

```bash
$name
${path}
```

### **4) Command substitution**

```bash
$(date)
```

### **5) Wildcard expansion (Globbing)**

```bash
*.log
file?.txt
```

### **6) Word splitting**

Breaks text into tokens.

### **7) Pathname expansion**

Final file matches.

Understanding this flow = mastery.

---

# 🎯 **4.2.9 — Command Substitution $( )**

Run a command and store the result:

```bash
today=$(date)
echo "Today is $today"
```

Example inside a loop:

```bash
count=$(wc -l < file.txt)
```

Avoid the old style:

```bash
`command`  # bad practice
```

Use `$( )`. More readable.

---

# 🧮 **4.2.10 — Arithmetic Expansion $(( ))**

Bash does NOT do math unless you use `$(( ))`.

Example:

```bash
x=$((10 + 5))
echo $x
```

Increment:

```bash
i=$((i+1))
```

Multiplication:

```bash
echo $((5*3))
```

### Internally:

Bash converts numbers → integer → does math → converts back to string.

---

# 🎨 **4.2.11 — Brace Expansion { } (Sequence/Pattern Generator)**

Useful for automation.

Examples:

### Generate multiple names:

```bash
echo file{1..5}.txt
```

Output:

```
file1.txt file2.txt file3.txt file4.txt file5.txt
```

### Generate directories:

```bash
mkdir -p dir/{logs,configs,data}
```

### Generate repeated patterns:

```bash
echo {A,B,C}_{1,2}
```

---

# 🎯 **4.2.12 — Wildcard Expansion (Globbing)**

Before executing:

```bash
ls *.log
```

Bash replaces `*.log` with **matching filenames**.

Example:

```
app.log
system.log
```

Output:

```
ls app.log system.log
```

If no match:

* Some shells return literal `*.log`
* Bash with “nullglob” option expands to empty list

---

# ⚠️ **Major Pitfall: Word Splitting**

Example:

```bash
files=$(ls)
```

If filenames contain spaces:

```
my file.txt
```

This breaks.

Correct safe method:

```bash
for f in *; do
    echo "$f"
done
```

This handles spaced filenames beautifully.

---

# 🔥 **4.2.13 — Parameter Expansion (Deep, Detailed, Easy-to-Understand Guide)**

Parameter expansion is how bash **manipulates variables BEFORE executing commands**.

Think of it like:

> “Take the variable → transform its content → give the modified version to the command.”

Parameter Expansion starts with:

```
${ ... }
```

Inside this `{ }`, you can apply advanced operations.

---

# 🎯 1. **Basic Expansion — `${var}`**

Simplest form.

```bash
name="Manoj"
echo "Hello ${name}"
```

This just returns the value.

Why use `{}` instead of `$name`?

Because of cases like:

```bash
file="data"
echo "$file.txt"        # WRONG
# It searches for variable file.txt

echo "${file}.txt"       # CORRECT
# data.txt
```

---

# 🎯 2. **Default Values (VERY IMPORTANT)**

These make scripts **bulletproof**.

There are **4 operators**:

---

## ✔ **2.1 — `${var:-default}` → Use default if var is EMPTY/UNSET**

Does NOT modify the variable.

```bash
name=""
echo "Hello ${name:-Guest}"
```

Output:

```
Hello Guest
```

If name has a value → use it.
If empty → use default.

Real use-case:

```bash
PORT=${PORT:-8080}
```

If PORT not set → run on 8080.

---

## ✔ **2.2 — `${var:=default}` → Assign default if var EMPTY/UNSET**

This **changes the variable**.

```bash
username=""
echo "${username:=anonymous}"
```

Output:

```
anonymous
```

Now:

```bash
echo "$username"
```

Output:

```
anonymous
```

Used in scripts to ensure required variables:

```bash
LOGDIR=${LOGDIR:=/var/log/app}
```

---

## ✔ **2.3 — `${var:+replacement}` → If var is set, use replacement**

Example:

```bash
name="Manoj"
echo ${name:+Hello}
```

Output:

```
Hello
```

If empty:

```
# name=""
echo ${name:+Hello}
```

Output:

```
# nothing
```

Used to conditionally enable flags.

---

## ✔ **2.4 — `${var:?message}` → Throw an error if var is undefined!**

Used for safety in production:

```bash
: ${DB_HOST:?DB_HOST is required}
```

If DB_HOST is missing → script immediately stops.

Output:

```
DB_HOST is required
```

This prevents dangerous defaults.

---

# 🎯 3. **Length of a variable — `${#var}`**

```bash
name="Manoj"
echo ${#name}
```

Output:

```
5
```

Used in:

* validating passwords
* checking if a variable is empty
* working with substrings

---

# 🎯 4. **Substring Extraction — `${var:position:length}`**

Example:

```bash
text="ABCDEFGHI"
echo ${text:2:4}
```

Output:

```
CDEF
```

Start at index 2 → take 4 chars.

### More examples:

Start from index 3 to end:

```bash
echo ${text:3}
```

Extract last 2 characters:

```bash
echo ${text: -2}
```

(NOTE the space before `-2` is required!)

---

# 🎯 5. **Remove Prefix/Suffix (Filepath magic)**

This is extremely useful for filenames and paths.

---

## ✔ **5.1 — Remove suffix pattern — `${var%pattern}`**

Removes the **shortest** match from the end.

```bash
file="backup.tar.gz"
echo ${file%.gz}
```

Output:

```
backup.tar
```

---

## ✔ **5.2 — Remove longest suffix — `${var%%pattern}`**

```bash
echo ${file%%.tar.gz}
```

If `file=backup.tar.gz`, it removes **the longest match**.

Example with directories:

```bash
path="/home/manoj/project"
echo ${path%/*}
```

Output:

```
/home/manoj
```

Remove only last segment.

---

## ✔ **5.3 — Remove prefix pattern — `${var#pattern}`**

```bash
path="/var/log/syslog"
echo ${path#/var/}
```

Output:

```
log/syslog
```

---

## ✔ **5.4 — Remove longest prefix — `${var##pattern}`**

Example:

```bash
url="https://google.com/search"
echo ${url##*/}
```

Output:

```
search
```

This extracts the last part after `/`.

---

# 🎯 6. **Search & Replace — `${var/pattern/replacement}`**

---

## ✔ **6.1 — Replace FIRST occurrence**

```bash
text="bananas"
echo ${text/na/NA}
```

Output:

```
bANanas
```

---

## ✔ **6.2 — Replace ALL occurrences — `${var//old/new}`**

```bash
echo ${text//na/NA}
```

Output:

```
bANANAs
```

Used for cleaning logs, strings, and filenames.

---

# 🎯 7. **Remove Matching Prefix/Suffix dynamically**

Example: remove extension:

```bash
file="report.pdf"
echo ${file%.*}
```

Output:

```
report
```

Extract only extension:

```bash
echo ${file##*.}
```

Output:

```
pdf
```

This is used in:

* file parsing
* log name generation
* renaming tools

---

# 🎯 8. **Indirection — `${!var}` (Advanced but powerful)**

If:

```bash
name="USER"
```

And:

```bash
USER="manoj"
```

Then:

```bash
echo ${!name}
```

Output:

```
manoj
```

You accessed variable **whose name is stored inside another variable**.

Used in dynamic configs.

---

# 🎯 9. **Array Parameter Expansion**

If array:

```bash
arr=(a b c)
echo ${arr[@]}
```

Output:

```
a b c
```

Count elements:

```bash
echo ${#arr[@]}
```

---

# 💥 REAL-WORLD Examples That Make You a Pro

---

## 1️⃣ Extract domain from URL

```bash
url="https://example.com/home"
echo ${url#*//}
```

Output:

```
example.com/home
```

---

## 2️⃣ Extract filename from path

```bash
path="/var/log/app/error.log"
echo ${path##*/}
```

Output:

```
error.log
```

---

## 3️⃣ Get directory name from path

```bash
echo ${path%/*}
```

Output:

```
/var/log/app
```

---

## 4️⃣ Remove extension

```bash
fname="photo.png"
echo ${fname%.*}
```

---

## 5️⃣ Change extension

```bash
echo "${fname%.png}.jpg"
```

Output:

```
photo.jpg
```

---

## 6️⃣ Default database configuration

```bash
DB_HOST=${DB_HOST:-"127.0.0.1"}
DB_PORT=${DB_PORT:-3306}
```

---

## 7️⃣ Fail fast if required variable missing

```bash
: ${API_TOKEN:?API_TOKEN is required}
```

Script stops IMMEDIATELY if missing.

---

# ⭐ FINAL SUMMARY — You Now Fully Understand Parameter Expansion

Parameter expansion allows you to:

* validate variables
* set defaults
* extract parts of strings
* manipulate paths
* replace values
* safely construct filenames
* avoid bugs
* write production-grade bash scripts

This is **exactly** what senior Linux/DevOps/SRE engineers use every day.

---

# 📌 **4.2.14 — Escape Characters**

In double quotes:

* `\n` → newline
* `\t` → tab
* `\\` → backslash
* `\"` → literal quote

Example:

```bash
echo "Path is \"correct\""
```

---

# 🧨 4.2.15 — Real-World Mistakes Engineers Make

Let’s cover problems you will surely face in production.

---

### ❌ **Mistake 1: Unquoted variables**

```bash
rm -rf $dir/*
```

If `$dir` is empty →
Command becomes:

```
rm -rf /*
```

= DISASTER

Correct:

```bash
rm -rf "$dir"/*
```

---

### ❌ Mistake 2: Using ls for loops

```bash
for f in $(ls); do
```

Breaks on filenames with spaces.

Correct:

```bash
for f in *; do
```

---

### ❌ Mistake 3: Using backticks instead of $( )

Backticks don’t nest.

Correct:

```bash
result=$(command $(inner_command))
```

---

### ❌ Mistake 4: Not using braces {}

```bash
echo "$path_backup"
```

If variable is `$path`, wrong expansion happens.

Correct:

```bash
echo "${path}_backup"
```

---

# ⭐ **4.2 Summary (You Now Know the Soul of Bash)**

You deeply understand:

* variable creation
* environment variables
* quoting rules
* expansions
* arithmetic
* wildcard & brace expansion
* parameter expansion
* pitfalls & safe practices

This is the foundation of advanced bash scripting.

---

# 👉 Ready for the next chapter?

**4.3 — Input & Output (stdin, stdout, stderr, pipes, redirects)**

This chapter teaches:

* `>`
* `>>`
* `<`
* `2>`
* redirecting errors
* tee
* heredocs
* pipelines
* file descriptors

Say **“Start 4.3”** and we go deeper.
