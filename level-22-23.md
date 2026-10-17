# OverTheWire Bandit: Level 22 → Level 23

## Challenge Description

> A program is running automatically at regular intervals from **cron**, the
> time-based job scheduler. Look in `/etc/cron.d/` for the configuration and
> see what command is being executed.

> NOTE: Looking at shell scripts written by other people is a very useful skill.
> The script for this level is intentionally made easy to read. If you are
> having problems understanding what it does, try executing it to see the debug
> information it provides.

**Goal:** Understand how the cron script builds its output file name, then work
out the file name used for `bandit23` and read it.

**Connection:**

    ssh bandit22@bandit.labs.overthewire.org -p 2220

## Concepts Involved

- **Cron jobs**: scheduled tasks that run automatically as a specific user.
- **`whoami`**: prints the name of the current user. Inside the cron job it
  returns the user the job runs as.
- **`md5sum`**: computes an MD5 hash of its input. The same input always gives the same hash.
- **`cut`**: extracts selected parts of each line (here, the hash without the trailing `-`).
- **Command substitution `$(...)`**: runs a command and inserts its output into another command.

## Solution

### Step 1: Find the cron job
```bash

    bandit22@bandit:~$ cat /etc/cron.d/cronjob_bandit23
    @reboot bandit23 /usr/bin/cronjob_bandit23.sh &> /dev/null
    * * * * * bandit23 /usr/bin/cronjob_bandit23.sh &> /dev/null
```

The script runs **every minute as `bandit23`**.

### Step 2: Read the script
```bash

    bandit22@bandit:~$ cat /usr/bin/cronjob_bandit23.sh
    #!/bin/bash

    myname=$(whoami)
    mytarget=$(echo I am user $myname | md5sum | cut -d ' ' -f 1)

    echo "Copying passwordfile /etc/bandit_pass/$myname to /tmp/$mytarget"

    cat /etc/bandit_pass/$myname > /tmp/$mytarget
```

What the script does, line by line:

- `myname=$(whoami)` : Stores the current username. When cron runs the job, this is `bandit23`. 

- `mytarget=$(echo I am user $myname \| md5sum \| cut -d ' ' -f 1)` : Builds the text `I am user bandit23`, hashes it with MD5, and keeps only 
                                                                      the hash.

- `cat /etc/bandit_pass/$myname > /tmp/$mytarget` : Copies the password file for that user into `/tmp/<hash>`. 

If we run the script ourselves as `bandit22`, `whoami` returns `bandit22`, so
we would get bandit22's file name instead. We need the name used when
`myname` is `bandit23`.

### Step 3: Calculate the file name for bandit23

Reproduce the same command, replacing the username manually:
```bash

    bandit22@bandit:~$ echo I am user bandit23 | md5sum | cut -d ' ' -f 1
    <hash>
```

Because MD5 is deterministic, this is exactly the file name the cron job uses
when it runs as `bandit23`.

### Step 4: Read the file
```bash

    bandit22@bandit:~$ cat /tmp/<hash>
    <bandit23-password>
```

## Key Takeaways

- Reading and understanding other people's shell scripts is an essential skill.
- Deterministic functions like `md5sum` can be reproduced: the same input
  always gives the same file name, so the "hidden" name is not secret.
- Security through obscurity (an unpredictable-looking file name) is not
  real protection, especially when the file is world-readable.
- Run scripts with substituted values to test your understanding of their logic.