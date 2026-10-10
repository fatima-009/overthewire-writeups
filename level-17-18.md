# OverTheWire Bandit: Level 17 → Level 18

## Challenge Description

> There are 2 files in the homedirectory: `passwords.old` and `passwords.new`.
> The password for the next level is in `passwords.new` and is the **only line
> that has been changed** between `passwords.old` and `passwords.new`.

> NOTE: If you have solved this level and see 'Byebye!' when trying to log into
  bandit18, this is related to the next level, bandit19.

**Goal:** Compare the two files and find the single line that differs.

**Connection:**

    ssh bandit17@bandit.labs.overthewire.org -p 2220

(Log in using the password found in the previous level, or with the private key
from Level 16 → 17.)

## Concepts Involved

- **`diff`**: compares two files line by line and prints the differences.
- **Reading `diff` output**: lines starting with `<` belong to the first file,
  lines starting with `>` belong to the second file.
- **`grep`**: filters lines matching a pattern (used in the alternative solution).

## Solution

### Step 1: Look at the files

    bandit17@bandit:~$ ls
    passwords.new  passwords.old

Each file contains 100 lines, so reading them by eye to find the difference
would be slow and error-prone.

### Step 2: Compare the files with `diff`

    bandit17@bandit:~$ diff passwords.old passwords.new
    42c42
    < <old line>
    ---
    > <bandit18_password>

How to read this output:

`42c42`: Line 42 in the first file was **c**hanged and became line 42 in the second file.
`< ...`: The line as it appears in `passwords.old`.
`> ...`: The line as it appears in `passwords.new`.

The task says the password is in `passwords.new`, so it is the line starting with `>`.

### Alternative: Print only the new line

    bandit17@bandit:~$ diff passwords.old passwords.new | grep '^>'
    > <bandit18_password>

Remove the leading `> ` (the `>` and one space) and what remains is the password.

## Troubleshooting

                   Problem                     |                                Cause / Fix                                

  Password rejected                            | You may have used the `<` line (from `passwords.old`). Use the `>` line.  
  Extra `> ` at the start                      | Do not copy the `>` and the space. Only copy the password itself.         
  `Byebye!` appears when logging into bandit18 | This is expected. It is related to the next level (Level 18 → 19), where `.bashrc` logs you out. 
