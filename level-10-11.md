# OverTheWire Bandit — Level 10 → Level 11

**Objective**

*The password is stored in data.txt, but the contents are Base64 encoded.*

# Step 1 — Connect

```bash
ssh bandit10@bandit.labs.overthewire.org -p 2220
```

# Step 2 — Check the File

```bash
ls
```

Output:

```bash
data.txt
```

# Step 3 — Decode the Password

Use:

```bash
cat data.txt | base64 -d
``` 

**Explanation**

`cat data.txt` → reads the contents of data.txt.

`|` → sends the output to the next command.

`base64 -d` → decodes the Base64 encoded data.


*What is Base64?*

- Base64 is an encoding method that represents binary or text data using a limited set of characters. It is commonly used to safely represent data as text.

- It is encoding, not encryption, so it can easily be decoded when you have the encoded data.

- The `-d` option tells base64 to decode the input.


*The decoded output is the password for Bandit Level 11.*
