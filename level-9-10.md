# OverTheWire Bandit — Level 9 → Level 10

**Objective**

*The password is stored in data.txt as one of the few human-readable strings, preceded by several = characters.*

# Step 1 — Connect

```bash
ssh bandit9@bandit.labs.overthewire.org -p 2220
```

# Step 2 — Check the File

```bash
ls
```

Output:

```bash
data.txt
```

# Step 3 — Find the Password

```bash
strings data.txt | grep '^='
```

**Explanation**

`strings data.txt` → extracts human-readable strings from data.txt.

`|` → sends the output of strings to grep.

`grep '^='` → searches for lines that start with =.

*What does ^= mean?*

`^`

means the beginning of the line.

`=`

means the line must contain an equals sign at that position.

So:  `^=`

means:

Find lines that start with =.

This helps us identify the strings that match the conditions of the challenge.


*The relevant output contains the password for Bandit Level 10.*
