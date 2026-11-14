# OverTheWire Bandit: Level 26 → Level 27

## Challenge Description

> Good job getting a shell! Now hurry and grab the password for bandit27!

**Goal:** Using the shell gained as `bandit26` (via the `vi` escape from the
previous level), find and run the setuid binary that leads to `bandit27`.

**Connection:**

This level has no separate SSH login step it continues directly from the
shell you broke into at the end of Level 25 → 26.

## Concepts Involved

- **Setuid binaries**: a program that runs with the privileges of its owner.

- **Restricted home directories**: `bandit26`'s own home directory may have
  limited permissions, so looking elsewhere (like other users' home folders)
  is necessary.

- **`ls -l`**: reveals file ownership and the setuid bit (`s` in the permission string).

## Solution

### Step 1: Confirm who we are

Continuing from the `vi` escape in the previous level:

    $ whoami
    bandit26

### Step 2: Look around for a setuid binary

`bandit26`'s home directory is restricted, so check `bandit27`'s home
directory instead (bandit26's shell being `showtext` is a strong hint that the
useful file lives with bandit27):
```bash

    bandit26@bandit:~$ ls -l /home/bandit27/
    -rwsr-x--- 1 bandit27 bandit26 <size> <date> bandit27-do
```

The `s` in `-rwsr-x---` shows the setuid bit, and it is owned by `bandit27`,
readable/executable by the `bandit26` group.

### Step 3: Run the binary without arguments
```bash

    bandit26@bandit:~$ /home/bandit27/bandit27-do
    Run a command as another user.
      Example: ./bandit27-do id
```

This is the same pattern as `bandit20-do` from Level 19 → 20: it runs whatever
command you give it with the privileges of its owner (`bandit27`).

### Step 4: Read the password file

    bandit26@bandit:~$ /home/bandit27/bandit27-do cat /etc/bandit_pass/bandit27
    <bandit27-Password>

## Key Takeaways

- Once a restricted shell is escaped, standard reconnaissance (`ls -l` across
  readable directories) is the way to find the next foothold.
- This setuid pattern (a binary that runs an arbitrary command as its owner)
  repeats throughout Bandit, so recognizing it quickly saves time.
- Group permissions (`bandit26` group readable) can let one user reach
  another's setuid tool even without owning it.