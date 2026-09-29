# OverTheWire Bandit — Level 8 → Level 9

**Objective**

*The password is stored in data.txt and is the only line that occurs exactly once.*

# Step 1 — Connect

```bash
ssh bandit8@bandit.labs.overthewire.org -p 2220
```

# Step 2 — Check the File

```bash
ls
```

Output:

```bash
data.txt
```

# Step 3 — Find the Unique Line

Use:

```bash
sort data.txt | uniq -c | grep '^ *1 '
```

**Explanation**

`sort data.txt` → sorts the lines.

`uniq -c` → counts occurrences.

`grep '^ *1 '` → shows only lines occurring once.

Output:

1 <Bandit Level 9 Password>

*The value after 1 is the password for Bandit Level 9.*


**Understanding the Regex**

The pattern is:

`^ *1`

It can be understood as:

- ^ → start of the line

- * → zero or more spaces

- 1 → the count must be 1

The final space separates the count from the actual line

So:

```bash
grep '^ *1 '
```

means:

Find lines where the occurrence count is exactly 1.