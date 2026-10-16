# OverTheWire Bandit: Level 21 → Level 22

## Challenge Description

> A program is running automatically at regular intervals from **cron**, the
> time-based job scheduler. Look in `/etc/cron.d/` for the configuration and see
> what command is being executed.

**Goal:** Find the cron job, read the script it runs, and use what it does to
get the password for `bandit22`.

**Connection:**

    ssh bandit21@bandit.labs.overthewire.org -p 2220

## Concepts Involved

- **Cron**: a Linux service that runs commands automatically on a schedule.

- **`/etc/cron.d/`**: a directory holding cron job definitions. Each file says
  when to run, as which user, and which command.

- **Cron schedule syntax**: five time fields (`minute hour day month weekday`),
  followed by the user and the command. `* * * * *` means "every minute".

- **Shell redirection (`>`)**: sends a command's output into a file instead of the terminal.

- **File permissions (`chmod 644`)**: `644` means the owner can read and write,
  while everyone else can only read.

## Solution

### Step 1: Look at the cron configuration
```bash

    bandit21@bandit:~$ ls /etc/cron.d/
    cronjob_bandit15_root  cronjob_bandit17_root  cronjob_bandit22
    cronjob_bandit23       cronjob_bandit24       cronjob_bandit25_root ...
```

The file for the next level is `cronjob_bandit22`:
```bash

    bandit21@bandit:~$ cat /etc/cron.d/cronjob_bandit22
    @reboot bandit22 /usr/bin/cronjob_bandit22.sh &> /dev/null
    * * * * * bandit22 /usr/bin/cronjob_bandit22.sh &> /dev/null
```

How to read this:

- `@reboot`:    Also runs once when the system starts.
- `* * * * *`:  Runs **every minute**.
- `bandit22`:   The job runs as user `bandit22`, so it can read bandit22's password.
- `/usr/bin/cronjob_bandit22.sh`:  The script that gets executed.
- `&> /dev/null`:  Throws away any output, so nothing is shown.

### Step 2: Read the script
```bash

    bandit21@bandit:~$ cat /usr/bin/cronjob_bandit22.sh
    #!/bin/bash
    chmod 644 /tmp/<tmpfile>
    cat /etc/bandit_pass/bandit22 > /tmp/<tmpfile>
```

The script does two things:

1. Makes the file `/tmp/<tmpfile>` readable by everyone (`chmod 644`).
2. Writes the password for `bandit22` into that same file.

Since it runs as `bandit22` every minute, the temporary file always contains
bandit22's password, and it is world-readable.

### Step 3: Read the file the script writes
```bash

    bandit21@bandit:~$ cat /tmp/<tmpfile>
    <Bandit22_password>
```

## Key Takeaways

- Cron jobs run automatically as **another user**, so their scripts can access
  files you cannot read yourself.

- Always read both the cron entry and the script it runs. The script reveals
  what is being done and where output goes.
  
- A script that writes secrets into a world-readable file in `/tmp` is a
  security risk, since any user can read it.