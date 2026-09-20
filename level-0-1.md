# OverTheWire Bandit — Level 0 → 1

## Objective

The goal of this level is to find the password for **Bandit Level 1**.

After logging into Bandit Level 0, the password for the next level is stored in a file named `readme`, located in the home directory.

## Step 1: Check the Files

After logging into Bandit Level 0 with:

```bash
ssh bandit0@bandit.labs.overthewire.org -p 2220
```

run:

```bash
ls
```

The `ls` command lists the files and directories in the current directory.

You should see:

```text
readme
```

This means there is a file named `readme` in the home directory.

## Step 2: Read the File

Run:

```bash
cat readme
```

The `cat` command displays the contents of a file directly in the terminal.

## Step 3: Get the Password

The output of `cat readme` will show a long string of letters and numbers. That string is the password for **Bandit Level 1**.

## Step 4: Log Out and Log Into Level 1

Disconnect from Level 0:

```bash
exit
```

Then connect to Level 1 using the new password:

```bash
ssh bandit1@bandit.labs.overthewire.org -p 2220
```

When prompted for the password, enter the password you copied from the `readme` file.

## Important Commands

### List files

```bash
ls
```

Used to see files and directories in the current location.

### Read a file

```bash
cat filename
```

Used to display the contents of a file.

## What We Learned

In this level, we learned:

1. How to list files using `ls`.
2. How to read a file's contents using `cat`.
3. That passwords for the next level are often stored in plain readable files.
4. How to log out of one level and log into the next using a new password.