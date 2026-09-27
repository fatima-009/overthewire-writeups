# OverTheWire Bandit — Level 6 → Level 7

**Objective**

*The goal of this level is to find the password for Bandit Level 7.*

- The password is stored somewhere on the server and has the following properties:

1. Owned by user bandit7
2. Owned by group bandit6
3. Exactly 33 bytes in size


# Step 1 — Connect to Bandit Level 6

Connect to the Bandit server using SSH:

```bash
ssh bandit6@bandit.labs.overthewire.org -p 2220
```

After entering the Level 6 password, we are logged in as bandit6.

# Step 2 — Search the Entire Server

The challenge tells us that the file can be located anywhere on the server, so we need to search from the root directory /.

We can use:

```bash
find / -type f -user bandit7 -group bandit6 -size 33c 2>/dev/null
```

*Command Breakdown*

`find /`

Starts searching from the root directory /, meaning the entire filesystem.

`-type f`

Searches only for regular files.

`-user bandit7`

Finds files owned by the user bandit7.

`-group bandit6`

Finds files owned by the group bandit6.

`-size 33c`

Finds files that are exactly 33 bytes.

The c represents bytes.

`2>/dev/null`

Some directories will return Permission denied errors because the bandit6 user does not have permission to access them.

- This part:

`2>/dev/null`

redirects error messages to /dev/null, so the permission errors are hidden.


# Step 3 — Find the Password File

The command should return a path similar to:

```bash
/var/lib/dpkg/info/... 
```

The exact path may vary depending on the environment.

- Once the matching file is found, use cat to read it:

```bash
cat <matching-file>
```

*The output will be the password for Bandit Level 7.*


*What does 2> mean?*

Linux uses different file descriptors:

File Descriptor	Meaning
- 0	 Standard input
- 1	 Standard output
- 2	 Standard error

Therefore:

2>/dev/null

means:

Redirect standard error to /dev/null.


**Conclusion**

This level introduces more advanced usage of the find command. Instead of searching only by filename, we can search based on ownership, group, and file size. We also learned how to redirect error messages using 2>/dev/null.