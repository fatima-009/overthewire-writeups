# OverTheWire Bandit — Level 5 → Level 6

**Objective**

*The goal of this level is to find the password for Bandit Level 6.*

- The password is stored in a file inside the inhere directory. According to the challenge, the file:

1. Is human-readable
2. Is exactly 1033 bytes in size
3. Is not executable


# Step 1 — Connect to Bandit Level 5

Connect to the Bandit server using SSH:

```bash
ssh bandit5@bandit.labs.overthewire.org -p 2220
```

After entering the Level 5 password, we are logged in as bandit5.

# Step 2 — Check the Current Directory

- First, check the current working directory:

```bash
pwd
```
Output:

```bash
/home/bandit5
```

- List the contents:

```bash
ls
```

We can see a directory called:

```bash
inhere
```

- Move into it:

```bash
cd inhere
``` 

# Step 3 — Find the Correct File

There are multiple files and directories inside inhere, so instead of checking them manually, we can use the find command.

Use:

```bash
find . -type f -size 1033c -not -executable -exec file {} + | grep ASCII
```

*Command Breakdown*

`find .`

Search recursively from the current directory.

`-type f`

Search only for regular files.

`-size 1033c`

Find files that are exactly 1033 bytes.

The `c` represents bytes.

`-not -executable`

Exclude executable files.

`-not -executable` and `! -executable` are equivalent in this case.

`-exec file {} +`

Run the `file` command on the matching files.

The `file` command determines the type/content of a file.

For example, it may return:

```bash
./maybehere07/.file2: ASCII text
```

The `{}` represents the files found by `find`.

The `+` allows multiple matching files to be passed to file together.

`| grep ASCII`

The pipe `|` sends the output of `file` to `grep`.

`grep ASCII` filters the output and displays only lines containing ASCII.

This helps us identify the human-readable ASCII text file.


# Step 4 — Read the Password

The command should return the path of the matching file, for example:

```bash
./maybehere07/.file2: ASCII text
```

Now read the file using:

```bash
cat ./maybehere07/.file2
```

*The output is the password for Bandit Level 6.*


# Conclusion

In this level, we learned how to combine multiple Linux commands to efficiently locate a specific file. The combination of find, file, and grep allows us to narrow down the results and identify the required human-readable file.