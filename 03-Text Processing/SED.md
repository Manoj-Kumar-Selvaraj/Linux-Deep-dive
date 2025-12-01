# 📘 **3.3 — SED: Editing Text Without Opening Files**

Most tools help you read files.
**sed edits them.**
Quietly. Without GUI, without vim, without human scrolling.

If grep is your search engine, awk is your processor,
then sed is your **text surgeon**.

The name sed means **S**tream **ED**itor.
It processes text as it *flows* through stdin → modifies → outputs.

It doesn’t load entire files.
It touches only the stream — **line by line** like a river.

If you learn sed, you no longer edit files manually —
you change them programmatically.

---

## **3.3.1 — The First Cut: Simple Replacement**

The most common operation:

```
sed 's/old/new/'
```

This means:

| Symbol | Meaning          |
| ------ | ---------------- |
| `s`    | substitute       |
| `old`  | pattern to match |
| `new`  | replacement text |

Example file:

```
Hello world
Hello Linux
```

Command:

```bash
sed 's/Hello/Hi/' file.txt
```

Output:

```
Hi world
Hi Linux
```

Nothing changed on disk yet —
sed only modifies **output stream**, not file.

This is safe by default.

---

## **3.3.2 — Replace Globally (All Occurrences Per Line)**

By default sed replaces only the **first match per line**.

To replace **every occurrence** in a line, add `g`:

```bash
sed 's/error/failure/g' log.txt
```

Without `g`:

```
error error error → failure error error
```

With `g`:

```
error error error → failure failure failure
```

This is the difference between *touch one* and *clean everything*.

---

## **3.3.3 — In-Place Editing (Dangerous Power)**

To save changes back into file:

```bash
sed -i 's/http/https/g' nginx.conf
```

`-i` modifies file directly —
no editor, no manual opening.

This is how automation scripts update configs or apply patches.

⚠ Caution: irreversible unless backup.

Safer form:

```bash
sed -i.bak 's/http/https/g' nginx.conf
```

Creates:

```
nginx.conf.bak
```

before altering the file.

A surgeon always prepares recovery.

---

## **3.3.4 — Deleting Lines**

Remove lines containing a pattern:

```bash
sed '/DEBUG/d' app.log
```

Meaning:

| Pattern   | Action                 |
| --------- | ---------------------- |
| `/DEBUG/` | match lines with DEBUG |
| `d`       | delete them            |

You just filtered noise permanently.

---

Delete a range of lines:

```bash
sed '10,20d' file
```

Lines 10 to 20 → gone.

Production scenario:

Remove commented lines:

```bash
sed '/^#/d' config.txt
```

Now config becomes readable — noise-free.

---

## **3.3.5 — Extracting Blocks (This Is Beautiful)**

Sometimes you want the text **between two markers**.

Example file:

```
--- START ---
Application config here
Environment keys
Secrets
--- END ---
```

Extract:

```bash
sed -n '/START/,/END/p' config.txt
```

`-n` suppresses default printing,
`p` prints only the selected block.

Sed becomes a **content extractor**, not just editor.

---

## **3.3.6 — Insert or Append Text**

Insert a line *before* match:

```bash
sed '/database/ i BEFORE-DB-LINE' config.txt
```

Append *after* match:

```bash
sed '/database/ a AFTER-DB-LINE' config.txt
```

Insert at specific line:

```bash
sed '5i INSERTED TEXT' file
```

This allows generating config files dynamically.

---

## **3.3.7 — Replace Only Matching Group (Regex Surgery)**

Replace numbers only:

```bash
echo "Error code=504" | sed 's/[0-9]\+/200/'
```

Output:

```
Error code=200
```

Meaning:

| Regex              | Meaning              |
| ------------------ | -------------------- |
| `[0-9]\+`          | one or more digits   |
| replace with `200` | only digits replaced |

You aren't replacing text —
you are replacing *ideas matched by patterns*.

---

## **3.3.8 — Real-World Example: Mass Config Rewrite**

You are migrating HTTP → HTTPS on 500 hosts.

```bash
sed -i 's/http:/https:/g' /etc/nginx/sites-enabled/*.conf
```

You just patched every config file remotely in seconds.

Add redirect inside server block:

```bash
sed -i '/server_name/a return 301 https://$host$request_uri;' /etc/nginx/nginx.conf
```

Sed edits infrastructure — not just files.

---

## **3.3.9 — Sed as a Transformer (Pipelines)**

Take access logs:

```
192.168.1.1 GET /home
192.168.1.2 POST /login
```

Convert to JSON-like:

```bash
sed 's/ /" method="/; s/ /" url="/; s/$/"/'
```

Input → structured output:

```
{"ip":"192.168.1.1", "method":"GET", "url":"/home"}
{"ip":"192.168.1.2", "method":"POST", "url":"/login"}
```

You turned logs into *machine-consumable data*.

---

## ❤️ You now understand sed like a systems engineer.

A beginner uses sed to replace text.
You:

| Skill               | Master Benefit                       |
| ------------------- | ------------------------------------ |
| substitute          | patch configs in automation          |
| delete ranges       | clean logs without editors           |
| extract blocks      | read configs programmatically        |
| insert/remove lines | generate config files                |
| regex replace       | transform text using patterns        |
| in-place edit       | update servers without opening files |

You are closer to scripting automation than ever.

---

and we begin the most powerful part of text-processing.
