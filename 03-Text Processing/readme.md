# 📘 **3 — Text Processing & Pattern Extraction (grep · awk · sed · regex)**

Linux is a text-driven operating system.
Log files, configs, metrics, API responses — everything is text.

This module teaches how to **search, transform, extract & rewrite text like a data engine**, using regex-powered tools.

> After this chapter, you won’t *read* logs —
> you will *slice them, filter them & automate insights*.

---

## 🔍 GREP — Search & Filter Like a Machine

✔ Find lines, patterns, errors
✔ Recursively scan directories
✔ Extract matching values

| Action                 | Command                      |
| ---------------------- | ---------------------------- |
| Search simple text     | `grep "ERROR" app.log`       |
| Case-insensitive       | `grep -i error file`         |
| Count matches          | `grep -c failed log`         |
| Recursive search       | `grep -R "timeout" /var/log` |
| Only file names        | `grep -Rl password .`        |
| Show only matched text | `grep -oE "ERROR.*"`         |

---

## 🧠 AWK — The Text CPU

Awk reads files **column-wise**, perfect for logs/CSV/system output.

| Task                | Command                               |
| ------------------- | ------------------------------------- |
| Print fields        | `awk '{print $1,$3}' file`            |
| Custom separator    | `awk -F: '{print $1}' /etc/passwd`    |
| Filter by condition | `awk '$3 > 80 {print}' stats.csv`     |
| Sum column values   | `awk '{sum+=$5} END{print sum}' file` |
| Calculate avg       | `awk '{x+=$2}END{print x/NR}'`        |

---

## ✍ SED — Stream Editor (Find & Replace Without Opening File)

| Action          | Command                         |
| --------------- | ------------------------------- |
| Replace text    | `sed 's/error/fail/g' file`     |
| Edit in-place   | `sed -i 's/http/https/' config` |
| Delete lines    | `sed -i '/DEBUG/d' log.txt`     |
| Extract section | `sed -n '/START/,/END/p' file`  |

Sed allows editing live streams — **no editor needed**.

---

## 🎯 Regular Expressions (Regex)

Regex is the language behind grep/awk/sed.

| Pattern      | Meaning               |
| ------------ | --------------------- |
| `^word`      | line starts with word |
| `end$`       | line ends with end    |
| `[0-9]{3}`   | 3-digit number        |
| `.*`         | anything              |
| `[A-Z]{2,5}` | 2–5 uppercase chars   |

Real-world extractions:

| Extract    | Regex                                                       |
| ---------- | ----------------------------------------------------------- |
| IP Address | `grep -Eo "([0-9]{1,3}\.){3}[0-9]{1,3}"`                    |
| Email      | `grep -Eo "[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}"` |

Regex turns raw logs into structured data.

---

## ⚡ Combined Power (Pipeline Example)

```
grep ERROR app.log \
| awk '{print $1,$2,$5}' \
| sed 's/ERROR/⚠/' \
| sort | uniq -c
```

→ Filters logs, extracts fields, transforms text, summarizes output.
This is **production-grade log forensics**.

---

## 🧪 Practice Tasks

| Lab      | Goal                                         |
| -------- | -------------------------------------------- |
| Lab03-01 | Extract top IPs from nginx logs              |
| Lab03-02 | Mask emails using sed (privacy)              |
| Lab03-03 | Convert CSV → table → stats using awk        |
| Lab03-04 | Build log alerting pipeline using grep → awk |

Master these and you control log analysis, data extraction and automation.

---

Just say **4** to continue.
