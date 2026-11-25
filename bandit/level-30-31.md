# OverTheWire Bandit: Level 30 → Level 31

## Challenge Description

> There is a git repository at `ssh://bandit31-git@localhost/home/bandit31-git/repo`.
> The password for the user `bandit31-git` is the same as for the user
> `bandit31`. Clone the repository and find the password for the next level.

**Goal:** Clone the repository, and realize the password is stored in a **Git
tag**, not in a branch or the working files.

**Connection:**

    ssh bandit30@bandit.labs.overthewire.org -p 2220

## Concepts Involved

- **Git over SSH**: same mechanism as previous Git levels, with a dedicated Git-only user.
- **Git tags**: a tag is a fixed, named pointer to a specific commit, often
  used to mark releases. Unlike branches, tags don't move as new commits are added.
- **`git tag`**: lists all tags in the repository.
- **`git show <tag>`**: displays the commit (and its message/diff) that a tag points to.
- **Annotated tags**: can carry their own message, separate from the commit's message.

## Solution

### Step 1: Prepare a working directory and clone
```bash

    bandit30@bandit:~$ cd $(mktemp -d)
    bandit30@bandit:/tmp/tmp.XXXXXXXXXX$ git clone ssh://bandit31-git@localhost/home/bandit31-git/repo
    bandit31-git@localhost's password: <bandit30 password>
    Cloning into 'repo'...
```

### Step 2: Check the working files
```bash
    bandit30@bandit:/tmp/tmp.XXXXXXXXXX$ cd repo
    bandit30@bandit:/tmp/tmp.XXXXXXXXXX/repo$ ls -la
    .git  README.md

    bandit30@bandit:/tmp/tmp.XXXXXXXXXX/repo$ cat README.md
    -no useful password here-
```

### Step 3: List the tags
```bash
    bandit30@bandit:/tmp/tmp.XXXXXXXXXX/repo$ git tag
    secret
```

A tag named `secret` is a strong hint.

### Step 4: Show what the tag points to
```bash

    bandit30@bandit:/tmp/tmp.XXXXXXXXXX/repo$ git show secret
    tag secret
    Tagger: ...
    Date:   ...

    <redacted>

    commit <hash>
    Author: ...
    Date:   ...

        <commit message>
```
If `git show` only prints commit info without the password, check the tag's
own annotation message or the commit's diff:
```bash

    bandit30@bandit:/tmp/tmp.XXXXXXXXXX/repo$ git cat-file -p secret
```

## Troubleshooting

    |                      Problem                  |                               Cause / Fix                        |
    | `git tag` shows nothing                       | Try `git fetch --tags` in case tags weren't pulled during clone. |
    | `git show secret` doesn't reveal the password | Try `git cat-file -p secret` to see the raw tag object, or check the commit it points to with `git log -p  <hash>`. |
    | Tag name is different                         | Use whatever name `git tag` actually lists; `secret` is just this level's typical tag name. |
