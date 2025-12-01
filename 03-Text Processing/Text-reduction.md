# 📘 **3.5 — The Four Pillars of Data Reduction**

*wc · sort · uniq · xargs*
(The muscle of log analytics)

Grep, sed, awk **extract and shape data** —
these four tools **compress, count, order and execute decisions**

You must treat them as:

| Tool      | Role in Pipeline                            |
| --------- | ------------------------------------------- |
| **wc**    | Count anything (lines, words, bytes)        |
| **sort**  | Order data in ascending/descending form     |
| **uniq**  | Collapse duplicates, reveal frequency       |
| **xargs** | Convert text into shell arguments & execute |

Understanding them deeply turns one-liners into **log intelligence systems**.

---

## 🔷 **1) wc — The Counter**

`wc` means **word count**, but it's more than words —
it counts **lines, words, characters, bytes**.

Example file:

```
ERROR
WARN
ERROR
INFO
```

Count lines:

```bash
wc -l file.log
```

Output:

```
4 file.log
```

Count words:

```bash
wc -w file.log
```

Count characters:

```bash
wc -m file.log
```

`wc` is your measurement tool.

When logs are your battlefield, you don’t guess —
you **count casualties**.

---

## 🔷 **2) sort — Bring Order To Chaos**

Raw log output is random.
Sorting transforms randomness into **pattern visibility.**

Example list of IP hits:

```
192.168.1.3
192.168.1.3
192.168.1.7
192.168.1.4
```

Sort numerically:

```bash
sort file.txt
```

Now IPs cluster together.
Patterns become visible.

Sort numerically (`-n`):

```bash
sort -n numbers.txt
```

Sort reverse (`-r`):

```bash
sort -nr numbers.txt
```

Sort by a column (`-k`):

```bash
sort -k3 access.log
```

Sort is like shuffling a deck into order —
suddenly **answers appear.**

---

## 🔷 **3) uniq — Collapsing Repetition Into Truth**

`uniq` doesn't remove duplicates globally —
it removes **consecutive duplicates only**.

So sort *before* uniq:

```bash
sort iplist.txt | uniq
```

Count occurrences:

```bash
sort iplist.txt | uniq -c
```

Example output:

```
  12 192.168.1.10
   3 192.168.1.4
   1 192.168.1.7
```

This is **frequency distribution** —
gold for security & traffic analysis.

Most attacked endpoint?
Most failing request?
Most active IP?

`uniq -c` tells you in one breath.

---

## 🔷 **4) xargs — Turning Thought Into Action**

Everything above gives knowledge.
xargs **executes on that knowledge.**

Convert output → arguments.

Delete files listed in error log:

```bash
grep -Rl "core" . | xargs rm -f
```

Kill top CPU process:

```bash
ps aux | sort -nrk3 | head -n1 | awk '{print $2}' | xargs kill -9
```

Dangerous. Powerful. Necessary.

---

# 📘 Now See All Four Combined

Real log:

```
500 GET /auth
404 GET /home
500 GET /auth
500 GET /cart
404 GET /home
```

Goal → Top failing endpoints:

```bash
grep "500" access.log \
| awk '{print $3}' \
| sort | uniq -c | sort -nr | head
```

Tools at work:

| Step         | Tool     | Purpose        |
| ------------ | -------- | -------------- |
| extract      | grep     | find 500s      |
| select field | awk      | pick URL       |
| order        | sort     | group together |
| collapse     | uniq -c  | count each URL |
| rank         | sort -nr | highest first  |
| view top     | head     | show summary   |

You now think in pipelines —
not commands.

---

## ❤️ You Now Own the COMPLETE Text-Processing Toolkit

| Category          | Tools                |
| ----------------- | -------------------- |
| Search            | **grep + regex**     |
| Extract & Compute | **awk**              |
| Modify Streams    | **sed**              |
| Reduce & Quantify | **wc / sort / uniq** |
| Execute Decisions | **xargs**            |

From here, you are ready for **War Room level log analysis**.

---

### Next Chapter (The Fun One):

# ⚔️ **3.6 — War Room & Real Production Log Challenges**

We will analyze **attacks, outages, slow systems, auth failures**, and solve them using:

```
grep → awk → sed → sort → uniq → wc → xargs
```
