# OverTheWire Bandit — Level 7 → Level 8

**Objective**

*The goal of this level is to find the password for Bandit Level 8.*

*The password is stored in a file called data.txt, next to the word millionth.*


# Step 1 — Connect to Bandit Level 7

Connect to the Bandit server:

```bash
ssh bandit7@bandit.labs.overthewire.org -p 2220
```

After entering the Level 7 password, we are logged in as bandit7.

# Step 2 — Check the Files

List the contents of the current directory:

```bash
ls
```

We can see:

```bash
data.txt
```

*The challenge tells us that the password is next to the word:*

`millionth`

# Step 3 — Search for the Word

Instead of manually reading the entire file, we can use grep:

```bash
grep "millionth" data.txt
```

The command searches data.txt for the word millionth.

The output will look similar to:

```bash
millionth    <Bandit Level 8 Password>
```

*The value after millionth is the password for Level 8.*

*Command Breakdown*

`grep`

Is used to search for specific text inside files.

`grep "millionth" data.txt`


- Here:

grep → search command

"millionth" → text we want to find

data.txt → file we want to search


**Conclusion**

This level demonstrates how grep can quickly locate specific information inside a file without manually reading the entire file.