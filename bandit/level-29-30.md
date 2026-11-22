# OverTheWire Bandit: Level 29 → Level 30

## Challenge Description

> There is a git repository at `ssh://bandit30-git@localhost/home/bandit30-git/repo`.
> The password for the user `bandit30-git` is the same as for the user
> `bandit30`. Clone the repository and find the password for the next level.

**Goal:** Clone the repository, and realize the password isn't on the default
branch, it's hiding in another branch.

**Connection:**

    ssh bandit29@bandit.labs.overthewire.org -p 2220

## Concepts Involved

- **Git over SSH**: same mechanism as previous Git levels, with a dedicated Git-only user.
- **Git branches**: a repository can have multiple branches, each holding its
  own version of the files. `git clone` checks out only the **default branch**
  (usually `master`) by default.
- **`git branch -a`**: lists all branches, including remote-tracking ones that
  aren't checked out locally.
- **`git checkout`**: switches the working directory to a different branch.

## Solution

### Step 1: Prepare a working directory and clone
```bash

    bandit29@bandit:~$ cd $(mktemp -d)
    bandit29@bandit:/tmp/tmp.XXXXXXXXXX$ git clone ssh://bandit30-git@localhost/home/bandit30-git/repo
    bandit30-git@localhost's password: <bandit29 password>
    Cloning into 'repo'...
```

### Step 2: Check the default branch's files
```bash

    bandit29@bandit:/tmp/tmp.XXXXXXXXXX$ cd repo
    bandit29@bandit:/tmp/tmp.XXXXXXXXXX/repo$ ls -la
    .git  README.md

    bandit29@bandit:/tmp/tmp.XXXXXXXXXX/repo$ cat README.md
    -no password here either, just some placeholder text-
```

Nothing useful on the branch Git checked out by default.

### Step 3: List all branches
```bash
    bandit29@bandit:/tmp/tmp.XXXXXXXXXX/repo$ git branch -a
    * master
      remotes/origin/HEAD -> origin/master
      remotes/origin/master
      remotes/origin/dev
```

There is a `dev` branch that isn't checked out locally yet.

### Step 4: Switch to the other branch
```bash

    bandit29@bandit:/tmp/tmp.XXXXXXXXXX/repo$ git checkout dev
    Branch 'dev' set up to track remote branch 'dev' from 'origin'.
    Switched to a new branch 'dev'
```

### Step 5: Read the file on that branch

    bandit29@bandit:/tmp/tmp.XXXXXXXXXX/repo$ cat README.md
    <bandit30-password>

## Troubleshooting

    |                  Problem              |                                       Cause / Fix                                                           |

    | Only `master` shows up                | Run `git branch -a` (not plain `git branch`) to also see remote branches.                                   |

    | `git checkout dev` fails              | Make sure the branch name matches exactly what `git branch -a` listed (minus the `remotes/origin/` prefix). |

    | README still looks empty on `dev`     | Also check `git log -p` on that branch in case the password was later removed, as in the previous level.    |
    
    | `error: pathspec 'dev' did not match` | Run `git fetch` first if the remote branch wasn't picked up during clone.                                   |

## Key Takeaways

- `git clone` only checks out the default branch; other branches exist on the
  remote and must be explicitly fetched/checked out.
- `git branch -a` is essential for discovering branches that aren't visible locally by default.
- Sensitive data can be isolated in a branch that looks unrelated or unused,
  so always enumerate all branches when auditing a repository.
