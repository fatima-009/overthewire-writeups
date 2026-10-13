# OverTheWire Bandit: Level 18 → Level 19

## Challenge Description

> The password for the next level is stored in a file `readme` in the
> homedirectory. Unfortunately, someone has modified `.bashrc` to log you out
> when you log in with SSH.

**Goal:** Read the `readme` file even though a normal SSH login immediately
logs you out.

**Connection:**

    ssh bandit18@bandit.labs.overthewire.org -p 2220

## Concepts Involved

- **`.bashrc`**: a startup script that Bash runs when a shell starts. Here it
  has been modified to print a message and end the session.

- **Running a remote command over SSH**: `ssh user@host command` runs only that
  command on the server and then closes the connection, without waiting for an
  interactive session.

- **Shell choice and options**: starting a different shell (such as `/bin/sh`)
  or starting Bash with `--norc` avoids loading `.bashrc`.

## Solution

### Step 1: Observe the problem
```bash

    $ ssh bandit18@bandit.labs.overthewire.org -p 2220
    bandit18@bandit.labs.overthewire.org's password:
    ...
    Byebye !
    Connection to bandit.labs.overthewire.org closed.
```    

We are logged in and kicked out immediately, so we never get a prompt to run `cat readme`.

### Step 2: Run the command directly through SSH

Instead of opening a shell, ask SSH to run a single command:

```bash

    $ ssh bandit18@bandit.labs.overthewire.org -p 2220 cat readme
    bandit18@bandit.labs.overthewire.org's password:
    <bandit19_password>
```
The password for `bandit19` is printed as the output of `cat readme`.

### Alternative: Use a different shell

`/bin/sh` does not read Bash's `.bashrc`, so the logout trick does not run:

```bash

    $ ssh bandit18@bandit.labs.overthewire.org -p 2220 /bin/sh
```

You now have a shell without the forced logout. Then run:

    ls
    cat readme

## Key Takeaways

- `ssh user@host <command>` is a quick way to run one command and bypass an
  interactive login restriction.
- Startup files like `.bashrc` can change a shell's behavior, but a different
  shell or `--norc` can avoid them.
- Restrictions placed in login scripts are weak protection, because the user
  can often choose how the shell starts.