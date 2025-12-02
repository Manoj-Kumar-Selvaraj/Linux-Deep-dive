# 📘 CHAPTER 07 — FILE HANDLING

*(Stdin, stdout, stderr, pipes, redirection, tee, heredocs — explained SUPER CLEAR and easy)*

This chapter unlocks true automation power.
Every DevOps/SRE script uses these concepts.

I will explain everything **step-by-step**, in **simple language**, with **clear visuals**.

---

# 🌟 **0. THE FOUNDATION: The 3 Data Channels (Very Important)**

Every program in Linux automatically gets **3 streams**:

| FD    | Name   | Meaning              |
| ----- | ------ | -------------------- |
| **0** | stdin  | Input to the program |
| **1** | stdout | Normal output        |
| **2** | stderr | Error output         |

So:

```bash
cat file.txt
```

means:

* read from stdin (file.txt redirected internally)
* write to stdout (your screen)

---

# 🌟 **1. OUTPUT REDIRECTION (>)**

### **Send output to a file instead of screen**

```bash
echo "hello" > out.txt
```

If file exists → overwritten
If file doesn’t exist → created

### Visual:

```
echo ---> file
```

---

# 🌟 **2. APPEND OUTPUT (>>)**

Add output to end of file:

```bash
echo "new line" >> out.txt
```

---

# 🌟 **3. REDIRECT INPUT (<)**

Give a file as input to a program:

```bash
sort < names.txt
```

Equivalent to:

```bash
cat names.txt | sort
```

But MORE efficient (no pipe needed).

---

# 🌟 **4. REDIRECTING STDERR (2>)**

### Send only errors to a file:

```bash
command 2> errors.log
```

Example:

```bash
ls /root 2> denied.log
```

---

# 🌟 **5. REDIRECT BOTH STDOUT & STDERR**

### Method 1:

```bash
command > out.txt 2>&1
```

Method 2 (new Bash syntax):

```bash
command &> out.txt
```

Method 3 (append):

```bash
command &>> out.txt
```

---

# 🌟 **6. PIPES (|) — Send output of one command to another**

This is the MOST used feature in Linux.

Example:

```bash
ps aux | grep nginx
```

Meaning:

```
stdout of ps aux → stdin of grep
```

### Pipes create a chain:

```bash
cat file | grep error | wc -l
```

---

# 🌟 **7. Using tee — Save AND show output**

```bash
command | tee output.txt
```

Output:

* shown on screen
* saved in file

Append:

```bash
command | tee -a output.txt
```

---

# 🌟 **8. HEREDOC (<< EOF) — Give multiple lines as input**

Useful for:

* config files
* SQL commands
* emails
* multi-line text

Example:

```bash
cat <<EOF
Hello
This is multi-line input
EOF
```

Another example:
Write config file:

```bash
cat <<EOF > config.conf
name=manoj
env=prod
version=1
EOF
```

---

# 🌟 **9. HERESTRINGS (<<<)**

Single-line input to a command:

```bash
grep root <<< "root:x:0:0:/root:/bin/bash"
```

Equivalent to:

```bash
echo "root..." | grep root
```

---

# 🌟 **10. PROCESS SUBSTITUTION (<() and >())**

Advanced but SUPER USEFUL.

Send command output **as a file**:

```bash
diff <(ls /etc) <(ls /bin)
```

Meaning:

* ls /etc → FILE A
* ls /bin → FILE B
* diff FILE A FILE B

No temporary files needed!

Another example:

```bash
paste <(cat f1.txt) <(cat f2.txt)
```

---

# 🌟 **11. READING FILES THE RIGHT WAY (IMPORTANT)**

The **correct** method:

```bash
while IFS= read -r line
do
    echo "$line"
done < file.txt
```

Why this is safe:

* preserves spaces
* doesn’t break on quotes
* doesn’t break on special chars

---

# 🌟 **12. Writing Files Safely**

Overwrite:

```bash
echo "hello" > file
```

Append:

```bash
echo "hello" >> file
```

Write multi-line:

```bash
cat <<EOF > file
line1
line2
EOF
```

---

# 🌟 **13. Error Logs + Output Logs (SRE Style)**

Send output → log1
Send error → log2

```bash
command > out.log 2> err.log
```

Send both:

```bash
command &> all.log
```

---

# 🌟 **14. DevOps Real-World Examples**

---

## 🟦 Example 1 — Combine output + errors into one log

```bash
deploy_app &> deploy.log
```

---

## 🟦 Example 2 — Check CPU info

```bash
cat /proc/cpuinfo | grep "model name"
```

---

## 🟦 Example 3 — Monitor logs in real time

```bash
tail -f app.log | grep ERROR
```

---

## 🟦 Example 4 — Backup logs safely

```bash
tar -czf backup.tar.gz /var/log 2> errors.log
```

---

## 🟦 Example 5 — Apply SQL to MySQL using heredoc

```bash
mysql <<EOF
USE accounts;
SELECT * FROM users;
EOF
```

---

## 🟦 Example 6 — Compare two directory trees

```bash
diff <(ls /opt/app1) <(ls /opt/app2)
```

---

# 🌟 **15. Summary (Very Clear)**

You learned:

### ✔ stdin, stdout, stderr

### ✔ >, >>, <

### ✔ 2> and redirecting errors

### ✔ &> (redirect all)

### ✔ Pipes (|)

### ✔ tee

### ✔ heredoc (<<EOF)

### ✔ herestring (<<<)

### ✔ process substitution <()

### ✔ safe file reading

### ✔ real-world SRE scenarios

This is EXACTLY what DevOps/SRE scripts use daily.

---