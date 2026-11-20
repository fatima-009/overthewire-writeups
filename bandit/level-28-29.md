# OverTheWire Bandit: Level 28 → Level 29

## Challenge Description

> There is a git repository at `ssh://bandit29-git@localhost/home/bandit29-git/repo`.
> The password for the user `bandit29-git` is the same as for the user
> `bandit29`. Clone the repository and find the password for the next level.

**Goal:** Clone the repository, notice the current files don't contain the
password, and dig through the commit history to find it.

**Connection:**

    ssh bandit28@bandit.labs.overthewire.org -p 2220

## Concepts Involved

- **Git over SSH**: same mechanism as Level 27 → 28, a dedicated Git-only user
  serves the repository.
- **Commit history**: Git keeps every past version of tracked files, even
  after they are edited or removed from the latest commit.
- **`git log`**: lists commits, optionally with their changes (`-p`).
- **`git show`**: displays the contents or diff of a specific commit.
- **Secrets removed from code are not actually gone**: once committed, old
  values remain recoverable from history unless the repository itself is rewritten.

## Solution

### Step 1: Prepare a working directory and clone
```bash

    bandit28@bandit:~$ cd $(mktemp -d)
    bandit28@bandit:/tmp/tmp.XXXXXXXXXX$ git clone ssh://bandit29-git@localhost/home/bandit29-git/repo
    bandit29-git@localhost's password: <bandit28 password>
    Cloning into 'repo'...
```

### Step 2: Check the current files
```bash

    bandit28@bandit:/tmp/tmp.XXXXXXXXXX$ cd repo
    bandit28@bandit:/tmp/tmp.XXXXXXXXXX/repo$ ls -la
    .git  README.md

    bandit28@bandit:/tmp/tmp.XXXXXXXXXX/repo$ cat README.md
    -password removed-
```

The current version of the file has had its password **deliberately removed**.

### Step 3: Look at the commit history
```bash

    bandit28@bandit:/tmp/tmp.XXXXXXXXXX/repo$ git log
    commit <hash3> (HEAD -> master)
    Author: ...
        fix info leak

    commit <hash2>
    Author: ...
        add new unpublished data

    commit <hash1>
    Author: ...
        initial commit
```

The commit message `fix info leak` is a strong hint that an earlier commit
exposed something that was later removed.

### Step 4: Compare an earlier commit to the latest one
```bash

    bandit28@bandit:/tmp/tmp.XXXXXXXXXX/repo$ git show <hash2>:README.md
```

Or view the diff that removed the password:
```bash

    bandit28@bandit:/tmp/tmp.XXXXXXXXXX/repo$ git log -p -- README.md
```

The diff for the `fix info leak` commit shows a line being deleted that
contains the password:

    -password: <******>

### Alternative: Walk through every commit automatically
```bash

    bandit28@bandit:/tmp/tmp.XXXXXXXXXX/repo$ git log --all -p | grep -i password
```

## Key Takeaways

- Deleting a secret from the latest version of a file does **not** remove it
  from a Git repository's history.
- `git log -p` (or `git log --all -p`) reveals exactly what changed in every
  commit, including removed lines.
- Suspicious commit messages are a useful lead when hunting for exposed secrets.
- This mirrors a very common real-world mistake: committing credentials and
  then "fixing" it by only editing the latest file, not the history.