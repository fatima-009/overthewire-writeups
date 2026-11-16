# OverTheWire Bandit: Level 27 → Level 28

## Challenge Description

> There is a git repository at `ssh://bandit27-git@localhost/home/bandit27-git/repo`.
> The password for the user `bandit27-git` is the same as for the user `bandit27`.
> Clone the repository and find the password for the next level.

**Goal:** Clone the Git repository over SSH and look through its contents for
the password.

**Connection:**

    ssh bandit27@bandit.labs.overthewire.org -p 2220

## Concepts Involved

- **Git over SSH**: a Git remote can be accessed via an `ssh://` URL, using a
  dedicated user (here `bandit27-git`) that only allows Git operations.
- **`git clone`**: downloads a full copy of a repository, including its commit
  history, into a new local directory.
- **Repository contents vs. working directory**: the password may be in a
  tracked file, so simply listing the cloned folder is often enough.
- **Working in `/tmp`**: the home directory is read-only, so cloning must
  happen in a writable location such as one created with `mktemp -d`.

## Solution

### Step 1: Prepare a working directory

    bandit27@bandit:~$ cd $(mktemp -d)

### Step 2: Clone the repository
```bash
    bandit27@bandit:/tmp/tmp.XXXXXXXXXX$ git clone ssh://bandit27-git@localhost/home/bandit27-git/repo
```
When prompted, enter `bandit27`'s own password (the task states it is shared
with `bandit27-git`):
```bash

    The authenticity of host '[localhost]:22 ...' can't be established.
    Are you sure you want to continue connecting (yes/no)? yes
    bandit27-git@localhost's password: <bandit27 password>
    Cloning into 'repo'...
```
### Step 3: Look at the cloned repository
```bash

    bandit27@bandit:/tmp/tmp.XXXXXXXXXX$ cd repo
    bandit27@bandit:/tmp/tmp.XXXXXXXXXX/repo$ ls -la
    .git  README.md
```

### Step 4: Read the file
```bash

    bandit27@bandit:/tmp/tmp.XXXXXXXXXX/repo$ cat README.md
    <bandit28-password>
```

The password for `bandit28` is right there in the repository's tracked file.

### Note: if the password isn't in the working files

If the current files don't show it, check the commit history, since a
password could have been added and later removed:
```bash

    bandit27@bandit:/tmp/tmp.XXXXXXXXXX/repo$ git log --all -p
```

## Result

    Password for bandit28: <*******>

## Key Takeaways

- Git repositories can be served over SSH using a dedicated, restricted user.
- Cloning gives access to the full commit history, not just the latest state
  of the files, which matters if secrets were ever added and later removed.
- Always inspect both the working tree (`ls`, `cat`) and the history (`git log`)
  when auditing a repository for sensitive data.