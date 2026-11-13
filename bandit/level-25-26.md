# OverTheWire Bandit: Level 25 → Level 26

## Challenge Description

> Logging into bandit26 from bandit25 should be **doable without a password**.
> The shell for user `bandit26` is not `/bin/bash`, but something else. Find out
> what it is, how it works and how to break out of it.

**Goal:** Log in as `bandit26` using the SSH key from the previous level, then
escape the restricted shell it drops you into.

**Connection:**

    ssh bandit25@bandit.labs.overthewire.org -p 2220

(This level starts with a private key, `bandit26.sshkey`, already present in
`bandit25`'s home directory, it logs in as `bandit26` without a password.)

## Concepts Involved

- **`/etc/passwd`**: lists each user's shell. A non-standard shell can restrict
  what a user is able to do after login.

- **Restricted/custom shells**: here, `bandit26`'s shell is a small program
  (`more`, or a script wrapping it) that runs once and exits, instead of
  giving a normal Bash prompt.

- **`more` pager controls**: `more` can open an editor on the current file with
  the `v` key, using the `$VISUAL` or `$EDITOR` environment variable (default `vi`).

- **Terminal resizing**: `more` only shows one screenful at a time; shrinking
  the terminal window forces it to paginate instead of showing everything at once.

- **`vi` command mode**: once inside an editor, shell commands can be run with `:!<command>`.

## Solution

### Step 1: Check bandit26's shell
```bash

    bandit25@bandit:~$ cat /etc/passwd | grep bandit26
    bandit26:x:11026:11026:bandit level 26:/home/bandit26:/usr/bin/showtext
```

`bandit26`'s shell is `/usr/bin/showtext`, not `/bin/bash`.

### Step 2: Read the custom shell script
```bash

    bandit25@bandit:~$ cat /usr/bin/showtext
    #!/bin/sh
    export TERM=linux
    more ~/text.txt
    exit 0
```

So logging in as `bandit26` just runs `more` on a text file and then exits,
instead of leaving us at a prompt.

### Step 3: Shrink the terminal window

`more` only pages through content it cannot fit on one screen. If the terminal
window is tall enough to show the whole file, `more` finishes immediately and
exits before we can act. **Resize the terminal to be very short** (just a few
lines), for example by dragging its edge, so that `more` has to paginate.

### Step 4: Log in as bandit26 using the provided key
```bash

    bandit25@bandit:~$ ssh -i bandit26.sshkey bandit26@localhost -p 2220
```

With the terminal short, `more` shows only the first part of `text.txt` and
waits for a key press instead of exiting right away.

### Step 5: Open the editor from inside `more`

While `more` is paused (showing `--More--` at the bottom), press:

    v

This opens the current file in the default editor, which here is `vi`.

### Step 6: Escape to a shell from vi

Inside `vi`, enter command mode and run a shell:

    :set shell=/bin/bash
    :shell

Or directly spawn a shell with:

    :!/bin/bash

This replaces or escapes the restricted environment with an interactive Bash shell.

### Step 7: Read the password
```bash

    bandit26@bandit:~$ cat /etc/bandit_pass/bandit26
    <bandit26-Password>
```

## Key Takeaways

- A custom login shell can restrict what a user can do, but pagers and editors
  invoked from within it often have their own escape hatches.
- `more`'s `v` command opens the file in an editor, which is a classic way to
  break out of restricted environments.
- `vi`'s `:!<command>` (or `:shell`) runs shell commands from inside the editor.
- Resizing the terminal can control whether a pager needs to paginate at all,
  which can be the deciding factor in whether an escape is even possible.