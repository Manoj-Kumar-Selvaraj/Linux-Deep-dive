# 📘 **3.4 — The Art of Text Pipelines**

(*grep → awk → sed → sort → uniq → wc → xargs*)

This is where Linux behaves less like a shell and more like a **factory assembly line**.

Data enters at one side →
each tool shapes it →
final output emerges clean, meaningful, quantified.

Like steel becoming a sword.

---

## **3.4.1 — The Philosophy of Unix Pipelines**

A pipe (`|`) means:

```
Take output of previous command
→ feed it as input to next
```

Example:

```bash
command1 | command2 | command3
```

Think of this as **thinking in verbs:**

> grep — find
> awk — interpret
> sed — transform
> sort — order
> uniq — collapse
> wc — quantify
> xargs — act on it

We don’t *run commands*,
we **build thought processes.**

---

## **3.4.2 — First Pipeline (Start Simple)**

Imagine you have a log:

```
192.168.1.3 GET /api/order 500
192.168.1.7 GET /login      200
192.168.1.3 GET /api/order 500
192.168.1.4 GET /register   404
```

Goal:

> Count how many times a 500 error occurred.

Pipeline:

```bash
grep " 500" access.log | wc -l
```

Breakdown:

| Stage         | Purpose                    |
| ------------- | -------------------------- |
| `grep " 500"` | find lines with status 500 |
| `wc -l`       | count matching lines       |

You didn’t read logs — you measured failure.

---

## **3.4.3 — Extract → Aggregate → Summarize**

Now you want **top 10 endpoints causing errors**.

Pipeline:

```bash
grep " 500" access.log | awk '{print $3}' | sort | uniq -c | sort -nr | head
```

Let’s watch this flow like a river:

| Command            | What it contributes          |
| ------------------ | ---------------------------- |
| `grep " 500"`      | only error logs survive      |
| `awk '{print $3}'` | extract **URL field only**   |
| `sort`             | group same URLs together     |
| `uniq -c`          | count occurrences            |
| `sort -nr`         | sort by count, highest first |
| `head`             | show top offenders           |

This is **production troubleshooting** in one breath.

---

## **3.4.4 — Extracting and Cleaning Data with sed**

A messy log:

```
[ERROR] user=john id=2391
[ERROR] user=mike id=4021 
[ERROR] user=john id=2490
```

Goal → count errors per user.

Pipeline:

```bash
grep "ERROR" app.log \
| sed 's/.*user=//' | sed 's/ id=.*//' \
| sort | uniq -c | sort -nr
```

Flow explanation:

| Tool     | Role                          |
| -------- | ----------------------------- |
| grep     | extract only ERROR lines      |
| sed1     | remove everything before user |
| sed2     | remove trailing id text       |
| sort     | group names                   |
| uniq -c  | count per user                |
| sort -nr | show highest failures first   |

Result:

```
user=john 2
user=mike 1
```

This is forensics.
You reduced chaos → to insight.

---

## **3.4.5 — xargs — The Final Execution Stage**

Everything above gathers information.
*xargs acts on it.*

Example: delete files older than 7 days containing "core":

```bash
grep -Rl "core" . | xargs rm -f
```

Pipeline thinking:

```
find  → produce file list
grep  → filter list
xargs → delete targeted files
```

Awk + xargs → terrifyingly powerful:

Kill processes consuming >90% CPU:

```bash
ps aux | awk '$3 > 90 {print $2}' | xargs kill -9
```

You just automated **decision → action.**

This is production operator-level power.

---

## **3.4.6 — When Pipelines Become Dashboards**

Live error rate monitoring:

```bash
tail -f access.log \
| grep " 500" \
| awk '{count++; print "500 errors:", count}'
```

Memory usage watcher:

```bash
free -m | awk '/Mem/{print "Used:",$3 "MB / " $2 "MB"}'
```

Pipelines don’t just process —
they visualize.

Your terminal becomes a dashboard.

---

# ❤️ Chapter Reflection

Awk, sed, grep alone are **tools.**
Connected by pipelines —
they become **systems.**

You now think in streams:

| Tool      | You use it to…        |
| --------- | --------------------- |
| grep      | find signal           |
| sed       | surgically rewrite    |
| awk       | interpret & compute   |
| sort/uniq | compress information  |
| wc        | count truth           |
| xargs     | execute based on data |

This is automation thinking.
This is production readiness.

---
