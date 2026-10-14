# OverTheWire Bandit: Level 19 → Level 20

## Challenge Description

> To gain access to the next level, you should use the **setuid binary** in the
> homedirectory. Execute it without arguments to find out how to use it. The
> password for this level can be found in the usual place (`/etc/bandit_pass`),
> after you have used the setuid binary.

**Goal:** Use the setuid binary to run a command with the privileges of
`bandit20`, and read the password file that only `bandit20` can access.

**Connection:**

    ssh bandit19@bandit.labs.overthewire.org -p 2220

## Concepts Involved

- **Setuid (set user ID) permission**: when a file with the setuid bit is
  executed, it runs with the privileges of the file's **owner**, not the user
  who started it.

- **Effective user ID (EUID)**: the user ID that the system uses for permission
  checks. A setuid program changes the EUID to the file owner's ID.

- **File permissions**: `/etc/bandit_pass/bandit20` can only be read by `bandit20`.

- **`ls -l`**: shows permissions. An `s` in the owner's execute position
  (`-rwsr-x---`) means the setuid bit is set.

## Solution

### Step 1: Look at the home directory
```bash
    bandit19@bandit:~$ ls -l
    -rwsr-x--- 1 bandit20 bandit19 <size> <date> bandit20-do
```

The `s` in `-rwsr-x---` shows the setuid bit. The file is owned by `bandit20`,
so it will run as `bandit20`.

### Step 2: Run the binary without arguments
```bash

    bandit19@bandit:~$ ./bandit20-do
    Run a command as another user.
      Example: ./bandit20-do id
```
The program takes a command as an argument and runs it as the file's owner.

### Step 3: Verify the privilege change (optional)
```bash

    bandit19@bandit:~$ ./bandit20-do id
    uid=11019(bandit19) gid=11019(bandit19) euid=11020(bandit20) groups=11019(bandit19)
```

The real user ID (`uid`) is still `bandit19`, but the effective user ID
(`euid`) is now `bandit20`.

### Step 4: Read the password file
```bash

    bandit19@bandit:~$ ./bandit20-do cat /etc/bandit_pass/bandit20
    <bandit20_password>
```

The `cat` command runs with `bandit20`'s privileges, so it is allowed to read the file.


## Troubleshooting

                             Problem                                  |                             Cause / Fix 

`Permission denied` when reading `/etc/bandit_pass/bandit20` directly | Only `bandit20` can read it. Run the command through `./bandit20-do`. 
`./bandit20-do: No such file or directory`                            | Make sure you are in the home directory and use `./` before the name.
Command not found inside `bandit20-do`                                | Use the command's name or full path, such as `cat` or `/bin/cat`. 
